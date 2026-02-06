# domain overview

## Vue d'ensemble du domaine

Notre domaine s'étend de la mise en ligne d'un évènement à la réception du billet acheté.

## Liste des fonctionnalités

- Référencer un évènement
- Promouvoir un évènement
- Consulter un évènement
- Choisir sa place
- réserver sa place
- payer sa réservation
- communiquer le billet réservé

## Classification des sous domaines

| fonctionnalité       | Type | Justification |
|----------------------|------|---------------|
| Service de référencement d'évènements | Core | Ce service est au coeur de la pluvalue métier, il permet de rendre disponible des places pour que le spectateur prenne connaissance de l'offre d'évènement. Sans disponibilité d'ajout d'évènement, pas de réservation possible. |
| Consulter un évènement | Core | Pour c=que le spectateur fasse une réservation, il doit pouvoir visualiser les information de l'évènement ciblé. Sans cela il est difficile pour lui de se projeter dans une réservation, et donc un paiement. La perte de cette fonctionalité amènerait une perte de revenus. |
| Panier de réservation | Support | En plus d'un rôle de récapitulatif pour le spectateur, le panier assure un rôle de gestion des stocks de place. Par la notion de place verouillée quand une place est dans un panier, on assure la fiabilité des informations de disponibilité |
| cartographie des sièges de la salle | Support | Cette fonctionalité est purement une amélioration de l'expérience utilisateru dans la démarche de réservation. Sans elle, il est toujours possible de réserver une place non placée et aller au bout de la réservation |
| Service de paiement | Generic | Service clef pour la validation d'une réservation, non propre au métier ce qui donne la possibilité de faire appel  à une solution externe |
| Service d'envoie de mail | Generic | Service clef pour la finalisation d'une réservation, non propre au métier ce qui donne la possibilité de faire appel  à une solution externe |