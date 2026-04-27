# Contrats d'échange entre contexts

---

## Contrat 1 — ContexteRéférencement → ContexteRéservation

Le ContexteRéservation interroge le ContexteRéférencement pour valider qu'un événement est ouvert à la réservation et connaître le stock de places disponibles avant de créer une réservation.

| Propriété | Valeur |
|-----------|--------|
| **Context source** | ContexteRéférencement |
| **Context cible** | ContexteRéservation |
| **Type d'échange** | REST — GET |

### Schéma du message

```yaml
openapi: "3.0.3"
info:
  title: API Référencement → Réservation
  version: "1.0.0"

paths:
  /evenements/{evenementId}/disponibilite:
    get:
      summary: Consulter la disponibilité d'un événement
      description: >
        Vérifie qu'un événement est ouvert à la réservation et retourne
        le nombre de places encore disponibles.
      parameters:
        - name: evenementId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        "200":
          description: Disponibilité de l'événement
          content:
            application/json:
              schema:
                type: object
                required:
                  - evenementId
                  - statut
                  - placesDisponibles
                  - dateOuvertureReservation
                properties:
                  evenementId:
                    type: string
                    format: uuid
                  statut:
                    type: string
                    enum: [EN_ATTENTE, OUVERT, COMPLET, ANNULE]
                    description: >
                      EN_ATTENTE : date d'ouverture non encore atteinte.
                      OUVERT : réservations acceptées.
                      COMPLET : plus aucune place disponible.
                      ANNULE : événement annulé.
                  placesDisponibles:
                    type: integer
                    minimum: 0
                  dateOuvertureReservation:
                    type: string
                    format: date-time
        "404":
          description: Événement introuvable
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Erreur"

components:
  schemas:
    Erreur:
      type: object
      required: [code, message]
      properties:
        code:
          type: string
        message:
          type: string
```

---

## Contrat 2 — ContexteRéservation → ContextePaiement

Lorsque le spectateur valide son panier, le ContexteRéservation soumet une demande de paiement au ContextePaiement. Le résultat (succès ou échec) est renvoyé via un événement `PaiementResultat`.

| Propriété | Valeur |
|-----------|--------|
| **Context source** | ContexteRéservation |
| **Context cible** | ContextePaiement |
| **Type d'échange** | REST — POST |

### Schéma du message — Requête de paiement (POST)

```yaml
openapi: "3.0.3"
info:
  title: API Réservation → Paiement
  version: "1.0.0"

paths:
  /paiements:
    post:
      summary: Initier un paiement pour une réservation
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - reservationId
                - montantTotal
                - devise
                - spectateur
              properties:
                reservationId:
                  type: string
                  format: uuid
                montantTotal:
                  type: number
                  format: float
                devise:
                  type: string
                spectateur:
                  type: object
                  required: [nom, email]
                  properties:
                    nom:
                      type: string
                    email:
                      type: string
                      format: email
      responses:
        "202":
          description: Demande de paiement acceptée, traitement en cours
          content:
            application/json:
              schema:
                type: object
                properties:
                  paiementId:
                    type: string
                    format: uuid
                  redirectUrl:
                    type: string
                    format: uri
                    description: URL vers le tunnel de paiement (3DS, etc.)
        "422":
          description: Données de paiement invalides
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Erreur"

components:
  schemas:
    Erreur:
      type: object
      required: [code, message]
      properties:
        code:
          type: string
        message:
          type: string
```

### Schéma de l'événement retour — `PaiementResultat`

```yaml

event: PaiementResultat
schema:
  type: object
  required:
    - paiementId
    - reservationId
    - statut
    - horodatage
  properties:
    paiementId:
      type: string
      format: uuid
    reservationId:
      type: string
      format: uuid
    statut:
      type: string
      enum: [SUCCES, ECHEC, ANNULE]
    motifEchec:
      type: string
      nullable: true
      description: Renseigné uniquement si statut = ECHEC
    horodatage:
      type: string
      format: date-time
```