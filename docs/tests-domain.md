# Scénarios de test de domaine

---

## Agrégat 1 : Évènement

---

### Invariant 1 — Unicité de l'Évènement

#### Scénario 1 — Happy path

```
Given un Programmateur authentifié responsable de la Salle "Le Zénith de Paris"
  And aucun Évènement n'existe pour l'Artiste "Indochine", la Salle "Le Zénith de Paris" et la Date "15 novembre 2026 à 20h00"
When le Programmateur soumet le référencement d'un Évènement avec :
  - Artiste   : "Indochine"
  - Salle     : "Le Zénith de Paris"
  - Date      : 15 novembre 2026 à 20h00
Then l'Évènement est créé et publié dans l'AffichageRéférentiel avec un EvenementId unique
```

#### Scénario 2 — Sad path

```
Given un Programmateur authentifié responsable de la Salle "Le Zénith de Paris"
  And un Évènement existe déjà pour l'Artiste "Indochine", la Salle "Le Zénith de Paris" et la Date "15 novembre 2026 à 20h00"
When le Programmateur tente de référencer un nouvel Évènement avec les mêmes valeurs :
  - Artiste   : "Indochine"
  - Salle     : "Le Zénith de Paris"
  - Date      : 15 novembre 2026 à 20h00
Then le système rejette la création avec l'erreur "Un Évènement identique existe déjà pour cet Artiste, cette Salle et cette Date"
  And aucun Évènement supplémentaire n'est créé dans le référencement
```

---

### Invariant 2 — Places limitées par la Salle

#### Scénario 1 — Happy path

```
Given un Évènement en cours de référencement pour la Salle "L'Olympia" dont la capacité maximale est de 2 000 Places
When le Programmateur définit les CatégoriePlaces suivantes :
  - Catégorie Fosse    : 800 Places à 45 €
  - Catégorie Tribune  : 700 Places à 60 €
  - Catégorie Balcon   : 500 Places à 35 €
  Total                : 2 000 Places
Then l'Évènement est créé avec 2 000 Places disponibles
  And le stock de Places correspond exactement à la capacité de la Salle
```

#### Scénario 2 — Sad path

```
Given un Évènement en cours de référencement pour la Salle "L'Olympia" dont la capacité maximale est de 2 000 Places
When le Programmateur définit les CatégoriePlaces suivantes :
  - Catégorie Fosse    : 1 000 Places à 45 €
  - Catégorie Tribune  : 800 Places à 60 €
  - Catégorie Balcon   : 600 Places à 35 €
  Total                : 2 400 Places (dépassement de 400)
Then le système rejette la création avec l'erreur "Le nombre total de Places (2 400) dépasse la capacité maximale de la Salle (2 000)"
  And aucun Évènement n'est publié
```

---

### Invariant 3 — Réservation conditionnée à l'OuvertureRéservations

#### Scénario 1 — Happy path

```
Given un Évènement "Concert Indochine – Zénith de Paris" dont la date d'OuvertureRéservations est le "27 juin 2026 à 10h00"
  And la date courante du système est le "27 juin 2026 à 10h05"
  And un Spectateur authentifié consulte cet Évènement
When le Spectateur sélectionne une Place en Catégorie Tribune et l'ajoute à son Panier
Then la Place est verrouillée dans le Panier du Spectateur
  And le Spectateur peut procéder au Paiement pour finaliser sa Réservation
```

#### Scénario 2 — Sad path

```
Given un Évènement "Concert Indochine – Zénith de Paris" dont la date d'OuvertureRéservations est le "27 juin 2026 à 10h00"
  And la date courante du système est le "20 juin 2026 à 14h30" (avant l'ouverture)
  And un Spectateur authentifié consulte cet Évènement dans l'AffichageRéférentiel
When le Spectateur tente d'ajouter une Place à son Panier
Then le système rejette l'action avec le message "La réservation pour cet Évènement n'est pas encore ouverte — ouverture le 27 juin 2026 à 10h00"
  And aucune Place n'est verrouillée
  And l'Évènement reste visible en consultation seule
```

---

### Invariant 4 — Cohérence des informations de l'Évènement

#### Scénario 1 — Happy path

```
Given un Programmateur en cours de saisie d'un Évènement
When le Programmateur renseigne l'ensemble des champs obligatoires :
  - Artiste                  : "Stromae"
  - Salle                    : "LDLC Arena, Lyon"
  - Date                     : 18 octobre 2026 à 20h30
  - OuvertureRéservations    : 1er septembre 2026 à 09h00
  - Description              : "Retour sur scène de Stromae pour sa tournée 2026"
  - Au moins une CatégoriePlace avec un tarif (ex. Catégorie A : 75 €, 500 Places)
  And valide la publication
Then l'Évènement est publié dans l'AffichageRéférentiel
  And il est consultable par les Spectateurs dès sa publication
```

#### Scénario 2 — Sad path

```
Given un Programmateur en cours de saisie d'un Évènement
When le Programmateur tente de publier l'Évènement sans avoir renseigné la date d'OuvertureRéservations
  And sans avoir défini aucune CatégoriePlace ni tarif
Then le système bloque la publication avec l'erreur "Les champs suivants sont obligatoires : Date d'OuvertureRéservations, au moins une CatégoriePlace avec un tarif"
  And l'Évènement reste en brouillon, non visible dans l'AffichageRéférentiel
```

---

## Agrégat 2 : Réservation

---

### Invariant 5 — Unicité de Place par Évènement

#### Scénario 1 — Happy path

```
Given un Évènement "Concert Stromae – LDLC Arena" dont les réservations sont ouvertes
  And la Place "Tribune B – Rangée 3 – Siège 12" est disponible (non verrouillée, non réservée)
  And un Spectateur A est authentifié
When le Spectateur A sélectionne la Place "Tribune B – Rangée 3 – Siège 12" et l'ajoute à son Panier
Then la Place est verrouillée (VerrouillagePlace actif) et associée exclusivement au Panier du Spectateur A
  And la Place n'est plus sélectionnable par aucun autre Spectateur
```

#### Scénario 2 — Sad path

```
Given un Évènement "Concert Stromae – LDLC Arena" dont les réservations sont ouvertes
  And la Place "Tribune B – Rangée 3 – Siège 12" est déjà verrouillée dans le Panier du Spectateur A
  And un Spectateur B est authentifié et parcourt le plan de la Salle
When le Spectateur B tente de sélectionner la Place "Tribune B – Rangée 3 – Siège 12"
Then le système rejette la sélection avec le message "Cette Place est temporairement indisponible"
  And la Place reste verrouillée pour le Spectateur A
  And le Spectateur B est invité à choisir une autre Place disponible
```

---

### Invariant 6 — Délai de verrouillage du Panier

#### Scénario 1 — Happy path

```
Given un Spectateur A ayant verrouillé la Place "Fosse – Zone C – N°42" dans son Panier pour l'Évènement "Concert Indochine"
  And le délai de verrouillage autorisé est de 15 minutes
  And 8 minutes se sont écoulées depuis le verrouillage
When le Spectateur A valide son Panier et procède au Paiement dans ce délai
Then la Réservation est confirmée
  And la Place est définitivement attribuée au Spectateur A
  And le VerrouillagePlace se transforme en Réservation confirmée
```

#### Scénario 2 — Sad path

```
Given un Spectateur A ayant verrouillé la Place "Fosse – Zone C – N°42" dans son Panier pour l'Évènement "Concert Indochine"
  And le délai de verrouillage autorisé est de 15 minutes
  And 15 minutes se sont écoulées sans que le Spectateur A ait validé son Paiement
When le délai de verrouillage expire
Then le système libère automatiquement la Place "Fosse – Zone C – N°42"
  And la Place redevient disponible à la sélection pour tous les Spectateurs
  And le Panier du Spectateur A est vidé avec le message "Votre réservation a expiré, les Places ont été libérées"
```

---

### Invariant 7 — Paiement obligatoire pour valider la Réservation

#### Scénario 1 — Happy path

```
Given un Spectateur ayant un Panier contenant la Place "Balcon – Rangée 1 – Siège 5" pour l'Évènement "Pièce de Molière – Comédie-Française"
  And le montant à payer est de 35 €
When le Spectateur saisit ses informations bancaires et valide le Paiement
  And le système bancaire autorise la transaction (réponse 3D Secure confirmée)
Then la Réservation est créée avec le statut "Confirmée"
  And un Billet est généré et transmis au Spectateur par email via le ContexteNotification
  And la Place est définitivement retirée du stock disponible
```

#### Scénario 2 — Sad path

```
Given un Spectateur ayant un Panier contenant la Place "Balcon – Rangée 1 – Siège 5" pour l'Évènement "Pièce de Molière – Comédie-Française"
  And le montant à payer est de 35 €
When le Spectateur saisit ses informations bancaires et valide le Paiement
  And le système bancaire refuse la transaction (fonds insuffisants)
Then aucun Billet n'est émis
  And la Réservation reste au statut "En attente de paiement"
  And le Spectateur est notifié de l'échec avec le message "Votre paiement a été refusé, veuillez réessayer ou utiliser un autre moyen de paiement"
  And la Place reste verrouillée le temps que le Spectateur relance un Paiement ou abandonne
```

---

### Invariant 8 — Conformité du Billet avec la Réservation

#### Scénario 1 — Happy path

```
Given une Réservation confirmée (ReservationId : R-00421) pour :
  - Spectateur  : "Pierre Martin" (SpectateurId : S-0012)
  - Évènement   : "Concert Indochine – Zénith de Paris, 15 novembre 2026 à 20h00"
  - Place       : "Tribune B – Rangée 3 – Siège 12" (CatégoriePlace : Tribune, tarif : 60 €)
  And le Paiement a été confirmé
When le système génère le Billet et l'envoie par email
Then le Billet contient exactement :
  - Nom du Spectateur : "Pierre Martin"
  - Évènement         : "Concert Indochine – Zénith de Paris"
  - Date              : 15 novembre 2026 à 20h00
  - Place             : "Tribune B – Rangée 3 – Siège 12"
  - Catégorie         : Tribune
  - Montant payé      : 60 €
  And le Billet est signé avec un identifiant unique non falsifiable lié au ReservationId R-00421
```

#### Scénario 2 — Sad path

```
Given une Réservation confirmée (ReservationId : R-00421) pour :
  - Spectateur  : "Pierre Martin" (SpectateurId : S-0012)
  - Place       : "Tribune B – Rangée 3 – Siège 12" (tarif : 60 €)
When un Agent de Billeterie tente de modifier manuellement les données du Billet émis
  en changeant la Place associée en "Fosse – Zone A – N°01" (tarif : 45 €)
Then le système rejette la modification avec l'erreur "Un Billet émis ne peut pas être modifié — toute correction nécessite l'annulation de la Réservation et la création d'une nouvelle Réservation"
  And le Billet original reste inchangé et valide avec les données de la Réservation R-00421
```