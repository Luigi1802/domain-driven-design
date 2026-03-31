# Exemples d'API REST

---

## Introduction

Les endpoints REST ci-dessous sont des **maquettes conceptuelles** — elles ne sont pas implémentées, mais documentent comment les cas d'usage métier seraient exposés via une API HTTP. Ces endpoints respectent l'Ubiquitous Language et traduisent les commandes et requêtes métier en ressources REST. Chaque endpoint est porté par un REST Adapter qui traduit les requêtes HTTP en commandes métier, invoque les services applicatifs correspondants, puis retourne les résultats métier sous forme de JSON.

Les exemples ci-dessous couvrent deux cas d'usage principaux : la **création d'une Réservation** (ContexteRéservation) et la **consultation d'un Évènement** (ContexteRéférencement).

---

## Endpoint 1 : Créer une Réservation

### Méthode + URL

```
POST /reservations
```

### Description

Cet endpoint permet à un Spectateur authentifié de créer une Réservation pour un Évènement en sélectionnant une Place disponible. L'opération déclenche le cas d'usage métier `CréerRéservation` orchestré par la couche Application. Le service applicatif vérifie que l'Évènement est ouvert à la réservation (la date d'OuvertureRéservations est atteinte), que la Place demandée est disponible (non verrouillée, non réservée), puis crée une instance de l'agrégat `Réservation` en appliquant les invariants métier.

La Réservation créée est mise dans l'état `EN_ATTENTE_PAIEMENT` avec un VerrouillagePlace actif de 15 minutes. La Place devient immédiatement indisponible pour tout autre Spectateur. Le Spectateur doit procéder au Paiement dans le délai imparti pour confirmer définitivement sa Réservation. Si le délai expire sans Paiement, la Réservation passe automatiquement au statut `EXPIRÉE` et la Place est libérée.

**Bounded Context** : ContexteRéservation  
**Service Applicatif invoqué** : `CréerRéservation(spectateurId, evenementId, placeId)`  
**Agrégat métier sollicité** : `Réservation`  
**Invariants appliqués** : Unicité de Place par Évènement, Réservation conditionnée à l'OuvertureRéservations, Délai de verrouillage du Panier

### Exemple de requête (bloc JSON)

```json
{
  "spectateurId": "S-0012",
  "evenementId": "EV-88",
  "placeId": "P-1042"
}
```

**Champs de la requête** :
- `spectateurId` (string, obligatoire) : Identifiant métier unique du Spectateur effectuant la Réservation. Correspond à l'entité `Spectateur` dans l'Ubiquitous Language.
- `evenementId` (string, obligatoire) : Identifiant métier unique de l'Évènement ciblé. Correspond à l'entité racine `Évènement` de l'agrégat Évènement.
- `placeId` (string, obligatoire) : Identifiant métier unique de la Place sélectionnée. Correspond à l'objet valeur `Place` dans une CatégoriePlace donnée de la Salle.

**Note sur l'authentification** : Le `spectateurId` dans la requête doit correspondre au Spectateur authentifié. Le REST Adapter vérifie cette cohérence avant d'invoquer le service applicatif. En pratique, le `spectateurId` pourrait être extrait automatiquement du token JWT d'authentification, mais dans cet exemple conceptuel, il est explicitement fourni.

### Exemple de réponse (bloc JSON)

#### Cas de succès (HTTP 201 Created)

```json
{
  "reservationId": "RSV-9041",
  "statut": "EN_ATTENTE_PAIEMENT",
  "spectateurId": "S-0012",
  "evenementId": "EV-88",
  "placeId": "P-1042",
  "verrouillageExpireA": "2026-06-27T10:20:00Z",
  "dateReservation": "2026-06-27T10:05:12Z"
}
```

**Champs de la réponse** :
- `reservationId` (string) : Identifiant métier unique de la Réservation créée. Correspond à l'entité racine `Réservation` de l'agrégat Réservation.
- `statut` (string) : Statut métier de la Réservation. Ici `EN_ATTENTE_PAIEMENT`, signifiant que la Place est verrouillée et que le Spectateur doit procéder au Paiement.
- `spectateurId`, `evenementId`, `placeId` : Confirmation des identifiants fournis dans la requête.
- `verrouillageExpireA` (timestamp ISO 8601) : Date et heure d'expiration du VerrouillagePlace. Après cette date, si aucun Paiement n'a été effectué, la Réservation passera automatiquement au statut `EXPIRÉE`.
- `dateReservation` (timestamp ISO 8601) : Date et heure de création de la Réservation (correspond à l'objet valeur `Date d'une Réservation` dans l'Ubiquitous Language).

#### Cas d'erreur métier (HTTP 409 Conflict)

```json
{
  "erreur": "PLACE_DEJA_RESERVEE",
  "message": "La Place P-1042 est déjà réservée ou verrouillée pour cet Évènement. Veuillez sélectionner une autre Place.",
  "details": {
    "placeId": "P-1042",
    "evenementId": "EV-88"
  }
}
```

**Explication** : Ce cas d'erreur se produit lorsque l'invariant "Unicité de Place par Évènement" est violé. Un autre Spectateur a déjà verrouillé ou réservé cette Place entre le moment où le Spectateur actuel a affiché le plan de la Salle et le moment où il a tenté de la réserver. Le REST Adapter retourne un code HTTP 409 Conflict pour signaler cette situation métier.

#### Cas d'erreur métier (HTTP 403 Forbidden)

```json
{
  "erreur": "RESERVATIONS_NON_OUVERTES",
  "message": "Les réservations pour cet Évènement ne sont pas encore ouvertes. Ouverture prévue le 2026-06-27 à 10:00.",
  "details": {
    "evenementId": "EV-88",
    "ouvertureReservations": "2026-06-27T10:00:00Z"
  }
}
```

**Explication** : Ce cas d'erreur se produit lorsque l'invariant "Réservation conditionnée à l'OuvertureRéservations" est violé. Le Spectateur tente de réserver avant la date d'OuvertureRéservations définie par le Programmateur. Le REST Adapter retourne un code HTTP 403 Forbidden pour signaler que cette action n'est pas autorisée dans le contexte métier actuel.

### Relation avec l'Ubiquitous Language

- **Réservation** : Terme métier central du ContexteRéservation, correspond à l'entité racine de l'agrégat.
- **Spectateur** : Rôle utilisateur métier, correspond à l'entité Spectateur.
- **Évènement** : Terme métier central du ContexteRéférencement, correspond à l'entité racine de l'agrégat Évènement.
- **Place** : Objet valeur représentant une unité de réservation dans une CatégoriePlace.
- **VerrouillagePlace** : Objet valeur représentant le statut temporaire d'une Place ajoutée au Panier.
- **EN_ATTENTE_PAIEMENT** : Statut métier de la Réservation correspondant à la phase où le Spectateur a sélectionné une Place mais n'a pas encore validé son Paiement.

### Flux métier correspondant

1. Le Spectateur consulte l'Évènement "Concert Indochine – Zénith de Paris, 15 novembre 2026" et visualise le plan de la Salle avec les Places disponibles.
2. Il sélectionne la Place "Tribune B – Rangée 3 – Siège 12" (placeId: "P-1042") et clique sur "Ajouter au Panier".
3. L'interface envoie une requête `POST /reservations` avec les identifiants du Spectateur, de l'Évènement, et de la Place.
4. Le REST Adapter traduit cette requête en commande métier `CréerRéservation` et invoque le service applicatif.
5. Le service applicatif vérifie que l'Évènement est ouvert à la réservation, que la Place est disponible, puis crée l'agrégat Réservation en appliquant les invariants métier.
6. La Réservation est persistée via le RepositoryRéservation avec un VerrouillagePlace actif de 15 minutes.
7. Le REST Adapter retourne une réponse HTTP 201 Created avec les détails de la Réservation.
8. Le Spectateur voit dans son interface que sa Place est verrouillée pour 15 minutes et qu'il doit procéder au Paiement avant expiration.

---

## Endpoint 2 : Consulter un Évènement

### Méthode + URL

```
GET /evenements/{evenementId}
```

### Description

Cet endpoint permet à un Spectateur (ou tout utilisateur, authentifié ou non) de consulter les détails d'un Évènement publié dans l'AffichageRéférentiel. L'opération déclenche le cas d'usage métier `ConsulterÉvènement` orchestré par la couche Application. Le service applicatif récupère l'agrégat `Évènement` via le RepositoryÉvènement, vérifie que l'Évènement est dans l'état `PUBLIÉ` (les Évènements en brouillon ou non publiés ne sont pas accessibles), puis retourne toutes les informations métier de l'Évènement : Artiste, Salle, Date, Horaires, Description, OuvertureRéservations, CatégoriePlaces avec leurs tarifs et Places disponibles.

Cette consultation est une opération en lecture seule (query) qui n'affecte pas l'état du domaine. Elle permet au Spectateur de prendre connaissance de l'offre d'Évènements et de décider s'il souhaite effectuer une Réservation (si les réservations sont ouvertes).

**Bounded Context** : ContexteRéférencement  
**Service Applicatif invoqué** : `ConsulterÉvènement(evenementId)`  
**Agrégat métier sollicité** : `Évènement`  
**Invariants appliqués** : Cohérence des informations de l'Évènement (seuls les Évènements publiés avec toutes les informations obligatoires sont retournés)

### Exemple de requête

```
GET /evenements/EV-88
```

**Paramètres de la requête** :
- `evenementId` (string, dans l'URL) : Identifiant métier unique de l'Évènement à consulter.

**Note** : Aucun corps de requête (body) n'est nécessaire pour une opération GET.

### Exemple de réponse (bloc JSON)

#### Cas de succès (HTTP 200 OK)

```json
{
  "evenementId": "EV-88",
  "artiste": {
    "nom": "Indochine",
    "type": "Groupe"
  },
  "salle": {
    "nom": "Le Zénith de Paris",
    "adresse": {
      "rue": "211 Avenue Jean Jaurès",
      "ville": "Paris",
      "codePostal": "75019",
      "pays": "France"
    },
    "capaciteMaximale": 6293
  },
  "date": "2026-11-15T20:00:00Z",
  "horaires": {
    "heureDebut": "20:00",
    "heureFin": "23:30"
  },
  "ouvertureReservations": "2026-06-27T10:00:00Z",
  "description": "Concert exceptionnel du groupe Indochine dans le cadre de leur tournée 2026. Retrouvez tous leurs plus grands succès ainsi que des titres inédits de leur nouvel album.",
  "categoriesPlaces": [
    {
      "nom": "Fosse",
      "tarif": 45.00,
      "devise": "EUR",
      "placesDisponibles": 512,
      "placesTotales": 800
    },
    {
      "nom": "Tribune",
      "tarif": 60.00,
      "devise": "EUR",
      "placesDisponibles": 423,
      "placesTotales": 700
    },
    {
      "nom": "Balcon",
      "tarif": 35.00,
      "devise": "EUR",
      "placesDisponibles": 498,
      "placesTotales": 500
    }
  ],
  "statut": "PUBLIE",
  "reservationsOuvertes": true
}
```

**Champs de la réponse** :
- `evenementId` (string) : Identifiant métier unique de l'Évènement.
- `artiste` (objet) : Objet valeur `Artiste` de l'Ubiquitous Language, contenant le nom et le type (solo, groupe, compagnie).
- `salle` (objet) : Objet valeur `Salle` contenant le nom, l'adresse (objet valeur `Adresse d'une salle`), et la capacité maximale.
- `date` (timestamp ISO 8601) : Date et heure de l'Évènement (objet valeur `Date`).
- `horaires` (objet) : Objet valeur `Horaires d'un Évènement` contenant l'heure de début et l'heure de fin.
- `ouvertureReservations` (timestamp ISO 8601) : Objet valeur `OuvertureRéservations`, date à partir de laquelle les réservations sont possibles.
- `description` (string) : Description textuelle de l'Évènement fournie par le Programmateur.
- `categoriesPlaces` (array) : Liste des objets valeur `CatégoriePlace` avec, pour chaque catégorie :
  - `nom` : Nom de la catégorie (Fosse, Tribune, Balcon, etc.).
  - `tarif` : Prix unitaire d'une Place dans cette catégorie.
  - `devise` : Devise du tarif (EUR, USD, etc.).
  - `placesDisponibles` : Nombre de Places actuellement disponibles à la réservation dans cette catégorie (stock en temps réel).
  - `placesTotales` : Nombre total de Places dans cette catégorie (capacité initiale définie lors du référencement).
- `statut` (string) : Statut métier de l'Évènement. Ici `PUBLIE`, signifiant que l'Évènement est visible dans l'AffichageRéférentiel.
- `reservationsOuvertes` (boolean) : Indique si les réservations sont actuellement possibles (true si la date courante est >= `ouvertureReservations`, false sinon).

#### Cas d'erreur (HTTP 404 Not Found)

```json
{
  "erreur": "EVENEMENT_NON_TROUVE",
  "message": "Aucun Évènement n'a été trouvé avec l'identifiant EV-999.",
  "details": {
    "evenementId": "EV-999"
  }
}
```

**Explication** : Ce cas d'erreur se produit lorsque l'`evenementId` fourni ne correspond à aucun Évènement publié dans le référencement. Le REST Adapter retourne un code HTTP 404 Not Found.

### Relation avec l'Ubiquitous Language

- **Évènement** : Terme métier central du ContexteRéférencement, correspond à l'entité racine de l'agrégat Évènement.
- **Artiste** : Objet valeur représentant l'entité se produisant lors de l'Évènement.
- **Salle** : Objet valeur représentant le lieu géographique de l'Évènement.
- **Adresse d'une salle** : Objet valeur décrivant l'emplacement précis de la Salle.
- **Date** : Objet valeur représentant le repère temporel complet de l'Évènement.
- **Horaires d'un Évènement** : Objet valeur encadrant la durée de l'Évènement.
- **OuvertureRéservations** : Objet valeur correspondant à la phase du cycle de vie d'un Évènement où celui-ci devient disponible à la Réservation.
- **CatégoriePlace** : Objet valeur regroupant des Places au sein d'une Salle, représentant une tranche tarifaire et une zone géographique.
- **AffichageRéférentiel** : Concept métier désignant la liste des Évènements référencés et consultables par les Spectateurs.

### Flux métier correspondant

1. Le Spectateur parcourt la plateforme de billetterie et voit dans l'AffichageRéférentiel ou l'AffichagePromotionnel un Évènement qui l'intéresse : "Concert Indochine – Zénith de Paris, 15 novembre 2026".
2. Il clique sur l'Évènement pour en consulter les détails.
3. L'interface envoie une requête `GET /evenements/EV-88` au REST Adapter.
4. Le REST Adapter traduit cette requête en query métier `ConsulterÉvènement(evenementId: "EV-88")` et invoque le service applicatif.
5. Le service applicatif récupère l'agrégat Évènement via le RepositoryÉvènement, vérifie que l'Évènement est publié, et retourne toutes les informations métier.
6. Le REST Adapter traduit ces informations en JSON et les retourne au Spectateur.
7. Le Spectateur voit les détails de l'Évènement : date, lieu, description, catégories de Places avec leurs tarifs et disponibilités. Il constate que les réservations sont ouvertes (`reservationsOuvertes: true`) et peut procéder à la sélection d'une Place pour effectuer une Réservation via l'endpoint `POST /reservations`.

---

## Notes sur l'architecture REST et l'Ubiquitous Language

Les endpoints REST ci-dessus illustrent les principes suivants :

1. **Séparation des préoccupations** : Les endpoints REST sont des adapters techniques qui traduisent les requêtes HTTP en commandes/queries métier. La logique métier (invariants, règles) reste encapsulée dans les agrégats du domaine. Les services applicatifs orchestrent, et les adapters traduisent.

2. **Respect de l'Ubiquitous Language** : Les noms des ressources REST (`/reservations`, `/evenements`), les champs JSON (`spectateurId`, `evenementId`, `placeId`, `ouvertureReservations`, `categoriesPlaces`), et les statuts métier (`EN_ATTENTE_PAIEMENT`, `PUBLIE`) utilisent le vocabulaire de l'Ubiquitous Language défini avec les experts métier.

3. **Codes HTTP sémantiques** : Les codes de retour HTTP reflètent les résultats métier : `201 Created` pour une création réussie, `200 OK` pour une lecture réussie, `409 Conflict` pour une violation d'invariant d'unicité, `403 Forbidden` pour une action non autorisée dans le contexte métier, `404 Not Found` pour une ressource inexistante.

4. **Versioning (conceptuel)** : Dans une implémentation réelle, les endpoints seraient versionnés (ex : `/v1/reservations`) pour permettre l'évolution de l'API sans casser les clients existants. Ce versioning est transparent pour le domaine.

5. **Sécurité et authentification (conceptuel)** : Les endpoints nécessitant une authentification (comme `POST /reservations`) seraient protégés par un mécanisme d'authentification (JWT, OAuth2, etc.). Le REST Adapter vérifie l'identité du Spectateur avant d'invoquer le service applicatif. Cette préoccupation technique est isolée de la logique métier.
