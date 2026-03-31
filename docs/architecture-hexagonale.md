# Architecture hexagonale

---

## Description des couches

### 1. Couche Domaine (Core Domain)

La couche Domaine constitue le cœur métier du système de billetterie événementielle. Elle contient l'ensemble des concepts métier identifiés dans l'Ubiquitous Language : les Entités (Évènement, Réservation, Spectateur), les Objets Valeur (Adresse, Horaires, VerrouillagePlace, Billet), les Agrégats (Agrégat Évènement avec ses invariants d'unicité et de capacité de Salle, Agrégat Réservation avec ses règles de verrouillage et de paiement), ainsi que les invariants métier qui garantissent la cohérence du domaine. Cette couche est **pure** : elle ne contient aucune dépendance technique (pas de base de données, pas de framework, pas de protocole HTTP). Elle est **stable** et évolue uniquement au rythme des changements métier validés par les experts du domaine (Programmateurs, Spectateurs, DirecteurCommercial, AgentBilleterie). Les règles métier comme l'unicité d'une Place par Évènement, le délai de verrouillage du Panier, ou l'obligation de Paiement pour valider une Réservation sont toutes encapsulées dans cette couche.

### 2. Couche Application

La couche Application orchestre les cas d'usage métier en coordonnant les actions sur les agrégats du domaine. Elle expose des services applicatifs qui structurent les scénarios métier de bout en bout, par exemple : `RéférencerÉvènement`, `CréerRéservation`, `ValiderPaiement`, `LibérerPlacesExpirées`, ou `CommuniquerBillet`. Ces services applicatifs ne contiennent pas de logique métier propre — celle-ci reste dans le domaine — mais définissent l'enchaînement des opérations et la transaction globale. La couche Application définit également des **ports** (contrats abstraits) pour interagir avec l'extérieur : par exemple, un `RepositoryÉvènement` pour persister les Évènements, un `RepositoryRéservation` pour gérer les Réservations, un `ServicePaiement` pour communiquer avec le ContextePaiement, ou un `ServiceNotification` pour déclencher l'envoi de Billets via le ContexteNotification. Ces ports sont des intentions métier, non des implémentations techniques. La couche Application applique aussi des règles transverses comme la gestion des transactions ou la propagation d'événements métier entre Bounded Contexts.

### 3. Couche Adapters (Infrastructure)

La couche Adapters fournit les implémentations concrètes des ports définis par la couche Application. Elle contient tous les détails techniques et les choix d'infrastructure : 
- **Adapters REST** : exposent les API HTTP permettant aux clients externes (interface web, application mobile) d'invoquer les cas d'usage. Par exemple, un `POST /evenements` pour référencer un Évènement ou un `POST /reservations` pour créer une Réservation.
- **Adapters de Persistence** : implémentent les repositories via des technologies concrètes (PostgreSQL, MongoDB, ou autre base de données choisie). Un `RepositoryÉvènementPostgreSQL` traduit les opérations métier en requêtes SQL tout en respectant les invariants définis dans le domaine.
- **Adapters de Messaging** : permettent la communication asynchrone avec d'autres Bounded Contexts via un message broker (RabbitMQ, Kafka, etc.). Par exemple, le ContexteRéservation peut envoyer un message `RéservationConfirmée` au ContexteNotification pour déclencher l'envoi du Billet.
- **Adapters externes** : interagissent avec des APIs tierces, comme les systèmes bancaires (3D Secure, AIS) pour le tunnel de Paiement.

La couche Adapters est **volatile** : on peut changer de base de données, de protocole, ou de format sans impacter le domaine ou l'application, grâce à l'inversion de dépendance. Les adapters dépendent des ports, et non l'inverse.

---

## Schéma de l'architecture hexagonale

```
                    [ Interface Externe : Web UI / Mobile App ]
                                      |
                                      | HTTP REST
                                      v
        +------------------------------------------------------------------+
        |                          ADAPTERS                                |
        |                                                                  |
        |  +------------------+     +------------------+                   |
        |  | REST Adapter     |     | Messaging        |                   |
        |  | (API HTTP)       |     | Adapter          |                   |
        |  +------------------+     +------------------+                   |
        |           |                         |                            |
        |           v                         v                            |
        |  +-------------------------------------------------------+       |
        |  |              COUCHE APPLICATION                       |       |
        |  |                                                       |       |
        |  |  Services Applicatifs :                              |       |
        |  |  - RéférencerÉvènement                               |       |
        |  |  - CréerRéservation                                  |       |
        |  |  - ValiderPaiement                                   |       |
        |  |  - CommuniquerBillet                                 |       |
        |  |                                                       |       |
        |  |  Ports (contrats abstraits) :                        |       |
        |  |  - RepositoryÉvènement                               |       |
        |  |  - RepositoryRéservation                             |       |
        |  |  - ServicePaiement                                   |       |
        |  |  - ServiceNotification                               |       |
        |  +-------------------------------------------------------+       |
        |           |                         |                            |
        |           v                         v                            |
        |  +-------------------------------------------------------+       |
        |  |               COUCHE DOMAINE (Core)                  |       |
        |  |                                                       |       |
        |  |  Agrégats :                                          |       |
        |  |  - Évènement (racine)                                |       |
        |  |  - Réservation (racine)                              |       |
        |  |                                                       |       |
        |  |  Entités : Évènement, Réservation, Spectateur        |       |
        |  |                                                       |       |
        |  |  Objets Valeur : Artiste, Salle, CatégoriePlace,     |       |
        |  |  Place, Horaires, OuvertureRéservations, Panier,     |       |
        |  |  VerrouillagePlace, Paiement, Billet                 |       |
        |  |                                                       |       |
        |  |  Invariants métier garantis par les agrégats         |       |
        |  +-------------------------------------------------------+       |
        |           |                         |                            |
        |           v                         v                            |
        |  +------------------+     +------------------+                   |
        |  | DB Adapter       |     | External API     |                   |
        |  | (PostgreSQL)     |     | Adapter (Banque) |                   |
        |  +------------------+     +------------------+                   |
        +------------------------------------------------------------------+
                    |                         |
                    v                         v
        [ Base de données ]         [ API Bancaire 3D Secure ]
```

---

## Exemple de flux : Création d'une Réservation (commande → réponse)

**Contexte** : Un Spectateur authentifié souhaite réserver une Place pour un Évènement dont les réservations sont ouvertes.

**Flux détaillé** :

1. **Interface externe** : Le Spectateur clique sur "Réserver" dans l'interface web pour l'Évènement "Concert Indochine – Zénith de Paris, 15 novembre 2026". L'interface envoie une requête HTTP `POST /reservations` avec les informations suivantes en JSON : `spectateurId`, `evenementId`, `placeId`.

2. **REST Adapter** : L'adapter REST reçoit la requête HTTP, valide la structure du JSON (présence des champs obligatoires), et traduit cette requête technique en une commande métier : `CréerRéservation(spectateurId: "S-0012", evenementId: "EV-88", placeId: "P-1042")`. Il invoque ensuite le service applicatif correspondant.

3. **Service Applicatif (Application)** : Le service `CréerRéservation` orchestre le cas d'usage. Il récupère l'Évènement via le port `RepositoryÉvènement`, vérifie que la date d'OuvertureRéservations est atteinte (invariant métier), puis récupère le Spectateur via le port `RepositorySpectateur`. Il crée ensuite une nouvelle instance de l'agrégat `Réservation` en appelant la méthode métier de l'agrégat : `Réservation.créer(spectateur, evenement, place)`.

4. **Agrégat Réservation (Domaine)** : L'agrégat `Réservation` applique les invariants métier. Il vérifie que la Place n'est pas déjà verrouillée ou réservée (invariant "Unicité de Place par Évènement"), crée un `VerrouillagePlace` avec un délai de 15 minutes, et change le statut de la Réservation à `EN_ATTENTE_PAIEMENT`. Si un invariant est violé (par exemple, la Place est déjà prise), l'agrégat lève une exception métier.

5. **Persistence (Adapter)** : Le service applicatif demande au port `RepositoryRéservation` de persister la nouvelle Réservation. L'adapter PostgreSQL traduit cette demande en une série de requêtes SQL INSERT pour enregistrer la Réservation et le VerrouillagePlace dans la base de données, tout en respectant l'intégrité transactionnelle.

6. **Réponse métier** : Le service applicatif retourne un résultat métier au REST Adapter : `RéservationCréée(reservationId: "RSV-9041", statut: "EN_ATTENTE_PAIEMENT", délaiExpiration: "2026-06-27T10:20:00")`.

7. **REST Adapter → Interface** : L'adapter REST traduit ce résultat métier en réponse HTTP 201 Created avec un JSON contenant `reservationId`, `statut`, et `délaiExpiration`. Le Spectateur voit dans son interface que sa Place est verrouillée pour 15 minutes et qu'il doit procéder au Paiement.

**Points clés du flux** :
- La logique métier (vérification des invariants, création du VerrouillagePlace) reste dans le domaine.
- L'orchestration (récupération des données, appel des agrégats, persistance) est gérée par l'application.
- Les détails techniques (HTTP, SQL) sont isolés dans les adapters.
- Chaque couche peut évoluer indépendamment sans casser les autres.

---

## Narration complète d'un flux métier traversant les couches

**Cas d'usage fil rouge** : Référencement d'un Évènement par un Programmateur, suivi de la consultation par un Spectateur.

### Étape 1 : Référencement de l'Évènement (ContexteRéférencement)

Un Programmateur responsable de la salle "Le Zénith de Paris" souhaite référencer un nouveau concert. Il se connecte à l'interface de gestion, saisit les informations de l'Évènement (Artiste : "Indochine", Salle : "Le Zénith de Paris", Date : "15 novembre 2026 à 20h00", OuvertureRéservations : "27 juin 2026 à 10h00", CatégoriePlaces : Fosse 800 places à 45€, Tribune 700 places à 60€, Balcon 500 places à 35€), puis clique sur "Publier".

L'interface envoie une requête `POST /evenements` au REST Adapter. L'adapter traduit la requête en commande métier `RéférencerÉvènement` et invoque le service applicatif correspondant. Le service applicatif crée une instance de l'agrégat `Évènement` en appelant `Évènement.référencer(artiste, salle, date, ouvertureRéservations, catégoriesPlaces)`. L'agrégat Évènement vérifie l'invariant "Unicité de l'Évènement" en s'assurant qu'aucun Évènement identique (même Artiste, même Salle, même Date) n'existe déjà. Il vérifie aussi l'invariant "Places limitées par la Salle" en s'assurant que le total des Places (2000) ne dépasse pas la capacité de la Salle. Si tous les invariants sont respectés, l'Évènement est créé avec le statut `PUBLIÉ`. Le service applicatif persiste l'Évènement via le port `RepositoryÉvènement`, implémenté par un adapter PostgreSQL qui exécute les requêtes SQL nécessaires. Le REST Adapter retourne une réponse HTTP 201 Created avec l'`evenementId` généré. L'Évènement est désormais visible dans l'AffichageRéférentiel.

### Étape 2 : Consultation de l'Évènement par un Spectateur (ContexteInterface)

Un Spectateur parcourt la plateforme et clique sur l'Évènement "Concert Indochine – Zénith de Paris" dans l'AffichageRéférentiel. L'interface envoie une requête `GET /evenements/{evenementId}` au REST Adapter du ContexteRéférencement. L'adapter invoque le service applicatif `ConsulterÉvènement`, qui récupère l'Évènement via le port `RepositoryÉvènement`. L'adapter PostgreSQL exécute une requête SQL SELECT pour récupérer les données de l'Évènement. Le service applicatif vérifie que l'Évènement est dans l'état `PUBLIÉ` et qu'il est consultable. Il retourne les informations métier : Artiste, Salle, Date, Horaires, Description, OuvertureRéservations, CatégoriePlaces disponibles. Le REST Adapter traduit ces informations en JSON et les retourne au Spectateur. Le Spectateur voit les détails de l'Évènement et constate que les réservations ouvriront le 27 juin 2026 à 10h00. Il peut s'inscrire à une alerte ou revenir plus tard.

**Ce flux illustre** :
- La séparation claire entre les Bounded Contexts (ContexteRéférencement pour la création, ContexteInterface pour la consultation).
- Le respect de l'architecture hexagonale : le domaine (agrégat Évènement) applique les invariants, l'application orchestre, et les adapters gèrent les détails techniques.
- L'utilisation de l'Ubiquitous Language à tous les niveaux : les termes métier (RéférencerÉvènement, ConsulterÉvènement, AffichageRéférentiel) sont cohérents du domaine jusqu'à l'API.
