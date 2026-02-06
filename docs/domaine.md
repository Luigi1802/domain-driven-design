# Scénario choisi

Billetterie & réservation

# Contexte métier

Notre sujet porte sur un système de vente en ligne de billets pour prendre une place à un évènement en France (concert, spectacle, conférence, ...). La plateforme recense les évènements prenant place dans les différentes salles de spectacle.

L'enjeu est de metttre à disposition une plateforme dans laquelle les utilisateurs peuvent choisir un évènement parmi la programmation, réserver une place, payer la place et recevoir son billet.

Comme contraintes, nous avons la gestion des vagues d'affluence à la réservation, mais aussi la fiabilité des informations liées aux évènement (date, places restantes, authenticité et unicité des billets).

# Rôles utilisateurs

Role 1: Spectateur (client B2C); c'est l'utilisateur venant sur la plateforme pour acheter un billet.

Role 2: Programmateur (client B2B); c'est l'utilisateur venant saisir un nouvel évènenement et y renseigner les informations (artiste, date, salle, nombre de places, tarifs)

Role 3: Développeur (opérationnel) ; c'est le réalisateur technique des évolution et de la maintenance des fonctionnalités du site. C'est le point de contact du role Programmateur

Role 4: Directeur commercial (direction) ; c'est le donneur d'ordre sur les fonctionnalités opérationnelles de la plateforme.

# Problématiques métier

Liste des problèmes:

- gestion des vagues d'affluence quand un nouvel évènement est mis en ligne

- erreur de fiabilité sur les places vendues

- rétention des places dans le panier et gestion du paiement

# Scénario fil rouge

Prenons le scénario de mise en ligne d'un concert de rock. Un programmateur annonce au public la venue d'un groupe pour un concert dans sa salle de spectacle le 15 novembre 2026, sur réservation à partir du 27 juin 2026. Le programmateur va saisir l'évènement sur la plateforme de réservation avec les informations liées à l'évènement:

- date
- lieu
- nombre de place
- artiste
- gamme de tarifs
- date d'ouverture des réservations
- description de l'évènement

A la mise en ligne de l'évènement, le spectateur peut consulter l'évènement sans réserver avant la date communiquée. La direction commerciale consulte les évènements mis en ligne pour projeter l'activité et l'affluence à venir sur le site, nécessitant un dialogue avec les développeurs. La direction commerciale est aussi en charge de la promotion des évènement, la mise en avant sur la plateforme et le suivi des ventes, nécessitant un dialogue avec les programmateurs. Le développeur s'assure du dimensionnnment l'infrastructure et la robustesse de la plateforme. Le jour de l'ouverture de la réservation, l'utilisateur se connecte et choisit l'évènement pour acheter sa place