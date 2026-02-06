# Bounded-contexts


### Liste des contextes

- ContexteRéférencement
- ContexteInterface
- ContexteRéservation
- ContextePaiement
- ContexteNotification

## Tableau descriptif

| Bounded Context | Type | Rôle |
|-----------------|------|------|
| ContexteRéférencement | Core | Stocker les données liées aux évènements. Modèle relationnel de données, règle d'unicité de l'évènement, standardisation des champs admis pour décrire l'évènement. Interface: API et base de données accessible pour l'écriture (ajout) et lecture (consultation).
| ContexteInterface | Support | Mettre en forme la donnée et structurer la saisie d'évènement, mise en avant de l'affichage promotionnel. Règles d'accès basé sur le rôle de l'utilisateur. Droits de référencer (programmateur), de réserver (spectateur), de promouvoir (directeru commercial). Interface: application web liée au BC "Référencement d'évènement". |
| ContexteRéservation | Core | Attribuer une place à un spectateur. Règle de verouillage des places, intégrité du stock de places disponibles et regulation de la demande spectateur. Interfacé au BC "Référencement d'évènement" pour la validation des réservations possibles. |
| ContextePaiement | Generic | Accorder la place au spectateur en contrepartie du montant demandé. Modèle générique de saisie d'informations bancaires, règle de validation via un tunnel de paiement, interface bancaire (3D Secure, AIS, ...) |
| ContexteNotification | Generic | Notifier le spectateur du statut de sa réservation. Modèle générique configuré pour le besoin métier. Avancement de la réservation, rappel de réservation en attente, confirmation de réservation, envoie de facture, envoie de billet. Interface: envoie de mail automatisé à destination de la boite mail du spectateur. |
