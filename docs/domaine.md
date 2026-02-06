# Scénario choisi

Billetterie & réservation

# Contexte métier

Nous avons choisit le contexte de vente de place d'évènements culturels (concert, spectacle, conférence, ...) .Le domaine de la billetterie événementielle concerne la mise à disposition de places pour des événements organisés dans des salles de spectacle. Cela implique une coordination entre les orgnisateurs et les spectateurs.

L'enjeu est de metttre en avant des évènements pour lesquels les utilisateurs peuvent choisir de réserver une place, payer la place et recevoir son billet.

Comme contraintes, nous avons l'accès équitable à la réservation dans les périodes de forte demande. Nous avons aussi la fiabilité des informations liées aux évènement (date, places restantes, authenticité et unicité des billets). Une erreur de gestion peut entrainer une survente, une mauvaise expérience client ou une perte de chiffre d'affaires

# Rôles utilisateurs

Role 1: Spectateur (client B2C); c'est l'acteur en recherche de billet à réserver.

Role 2: Programmateur (client B2B); c'est l'utilisateur ayant un évènement à promouvoir, il apporte la connaissance des les informations liées (artiste, date, salle, nombre de places, tarifs)

Role 3: Agent de billeterie (opérationnel) ; gère les problèmes liés aux réservations, paiements échoués, annulations et réclamations client.

Role 4: Directeur commercial (direction) ; c'est l'acteru responsable de la stratégie de vente et de promotion. Il suit les ventes et anticipe l'affluence des spectateur sur les évènements à venir

# Problématiques métier

Liste des problèmes:

- gestion de l'accès à la réservation quand un nouvel évènement est mis en ligne

- erreur de fiabilité sur les places vendues (surventes, annulations, réclamations)

- rétention des places dans le panier et gestion du paiement

# Scénario fil rouge

Prenons le scénario de mise en ligne d'un concert de rock. Un programmateur annonce au public la venue d'un groupe pour un concert dans sa salle de spectacle le 15 novembre 2026, sur réservation à partir du 27 juin 2026. Le programmateur va saisir l'évènement sur une plateforme de réservation avec les informations liées à l'évènement:

- date
- lieu
- nombre de place
- artiste
- gamme de tarifs
- date d'ouverture des réservations
- description de l'évènement

A la mise en ligne de l'évènement, le spectateur peut consulter l'évènement sans pouvoir réserver avant la date communiquée. La direction commerciale consulte les évènements mis en ligne pour projeter l'activité et l'affluence à venir sur la plateforme, nécessitant un dialogue avec les agents de billeterie. La direction commerciale est aussi en charge de la promotion des évènement, la mise en avant sur la plateforme et le suivi des ventes, nécessitant un dialogue avec les programmateurs. L'agent de billeterie  gère les problèmes liés aux réservations, paiements échoués, annulations et réclamations client. Le jour de l'ouverture de la réservation, l'utilisateur se connecte et choisit l'évènement pour:

- choisir l'évènement
- choisir sa place
- ajouter sa place dans son panier
- valider son panier
- payer
- recevoir son ticket