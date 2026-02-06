# Ubiquitous Language – Version 1

## Spectateur
Définition métier: 
Un utilisateur de la plateforme dont l'objectif est d'effectuer une Réservation et de Consulter des Évènements
Exemple concret:
Un fan de rock souhaitant réserver une place pour un concert de son groupe préféré.

## Programmateur
Définition métier: 
Un utilisateur de la plateforme dont l'objectif est de Référencer des Évènements pour la ou les Salles dont il est responsable.
Exemple concret:
Un gérant de salle de théâtre veut permettre la réservations en ligne aux pièces de sa salle.

## DirecteurCommercial
Définition métier: 
Un opérateur/dirigeant de la plateforme dont l'objectif est de Promouvoir des Évènements référencés, en établissant un contact avec des Programmateurs afin de lui facturer des promotions payantes pour des Évènements, et en modifiant l'AffichagePromotionnel.
Exemple concret:
Un directeur commercial de l'entreprise qui se tient à jour des tendances de la plateforme et des évènements référencés pour obtenir des revenus promotionnels en mettant en avant des évènements.

## AgentBilleterie
Définition métier: 
Un opérateur de la plateforme dont l'objectif est de traiter les problèmes rencontrés par les Spectateurs (paiement, réception du billet, ...).
Exemple concret:
Un agent de billeterie de l'entreprise qui traite des tickets [...]

## Billet
Définition métier: 
Document permettant à un Spectateur d'attester de sa Réservation pour un Évènement.
Exemple concret:
Le billet d'un concert résumera les informations du concert et du spectateur qui l'a acheté, il sera reçu sous forme numérique à la fin du processus de Réservation

## CommuniquerBillet
Définition métier: 
La transmission du Billet d'une Réservation à un Spectateur, par email.
Exemple concret:
Après avoir complété son paiement de réservation, le spectateur reçoit le billet par email.

## Place
Définition métier: 
Une place correspond à une unité, un emplacement de Réservation potentielle dans un Salle, elle appartient à une CatégoriePlace et ne peut être réservée que par un seul Spectateur pour un Évènement donné.
Exemple concret: 
Un spectateur doit sélectionner un siège pour le spectacle qu'il veut réserver.

## CatégoriePlace
Définition métier: 
Un ensemble regroupant des Places au sein d'une Salle. Elle représente une tranche tarifaire pour la Réservation de l'Évènement,et une zone géographique au sein de la Salle.
Exemple concret: 
Un spectateur choisit de sélectionner une place dans la Catégorie A pour avoir la meilleure visibilité possible sur la scène. 

## Date
Définition métier: 
Le repère temporel complet pour un Évènement : jour, mois, année, heure précise
Exemple concret: 
Des concerts du même artiste ont lieu à de multiple reprises, à des dates différentes.

## Salle
Définition métier: 
Le repère géographique pour un Évènement : on donne son nom dans la description d'un Évènement et elle peut être divisée en CatégoriePlaces et Places lors du processus de Réservation.
Exemple concret: 
Un artiste se produit dans différentes salles, ses fans réservent tous leur place au sein de celle-ci pour assiter aux concerts.

## Artiste
Définition métier: 
L'acteur qui propose une prestation lors d'un Évènement, il peut être de plusieurs natures (artiste solo, groupe, compagnie) et proposer des prestations sous différentes formes (pièce de théâtre, concert, spectacle).
Exemple concret: 
Une compagnie de théâtre se produisant dans une salle pour plusieurs pièces est considérée comme un Artiste.

## Panier
Définition métier: 
Un ensemble d'une ou plusieurs places séléctionnées lors d'une Réservation. Il doit être payé pour valider la Réservation en cours.
Exemple concret: 
Pierre a sélectionné deux places en catégorie B pour lui et sa conjointe, après validation il est face à un panier à payer.

## Réservation
Définition métier: 
Une Réservation est l'aboutissement d'un processus de réservation, elle attribue à un ou plusieurs Spectateur une Place chacun pour un Évènement donné.
Exemple concret: 
Pierre a effectué une réservation pour lui et sa conjointe, qui contient deux places pour un concert de hip-hop.

## OuvertureRéservations
Définition métier: 
Correspond à la phase du cycle de vie d'un Évènement ou celui-ci devient disponible à la Réservation par les Spectateurs. Elle est définie à une date de début renseignée pendant le Réferencement par le Programmateur. Avant cette phase l'Évènement est en consultation seule.
Exemple concret: 
L'ouverture des réservations pour cette pièce de Molière est fixée au 11 décembre 2026. Les spectateurs pourront alors réserver leur place. 

## Évènement
Définition métier: 
Concept clé du domaine, correspond à une prestation d'Artiste donné dans une Salle à une Date donnée, peut être référencé et promu sur la plateforme.
Exemple concret: 
L'évènement le plus populaire ce mois-ci est un concert de rap à la LDLC Arena le 18 octobre.

## RéférencerÉvènement
Définition métier: 
Action consistant à ajouter un Évènement dans le réferencement de la plateforme, elle est la plupart du temps exercée par un Programmateur, en renseignant ses différentes informations.
Exemple concret: 
Le gérant d'une salle référence un prochain concert sur la plateforme en indiquant la date, l'artiste, la description et la date d'ouverture des réservations.

## ConsulterÉvènement
Définition métier: 
Action consistant à accéder aux informations d'un Évènement référencé à partir de l'AffichagePromotionnel ou de l'AffichageRéferentiel de la plateforme.
Exemple concret: 
Un spectateur curieux des prochains évènements dans sa région clique sur un concert mentionné sur la page d'accueil de la plateforme.

Paiement
VerrouillagePlace

## AffichagePromotionnel
Définition métier:
La liste des Evènements bénéficiant dune Promotion visant à améliorer leur visibilité et leur Réservation. Cette liste est gérée par le DirecteurCommercial par dialogue avec les Programmateurs ou par un suivi des performances de Réservation.
Exemple concret:
Un Evènement E est affiché sur la page d'acceuil de la plateforme, permettant une visibilité maximale auprès spectateurs se rendant sur la plateforme.

## AffichageRéferentiel
Définition métier:
La liste des Evènements référencés. L'Evènement peut être ouvert à la réservation ou non, il figure dans l'AffichageRéférentiel dès lors qu'il a été référencé par un Programmateur. Cette liste est consultable par le Spectateur pour faire son choix de Réservation.
Exemple concret:
Un Evènement E a été référencé ce jour, un spectateur peut désormais le trouver par une recherche sur la plateforme, ou dans l'AffichagePromotionnel 

## Promotion
Définition métier: 
Actions commerciales entreprises pour la mise en avant d'un Evènement. Cela permet de donner une meilleure visibilité à l'Evènement et pourquoi pas inciter un spectateur à prendre une Reservation pour cet Evenement.
Exemple concret: 
Un Evenement E qui a lieu dans cinq jours a des Places de disponible à la Réservation. Le DirecteurCommercial décide de le mettre dans l'AffichagePromotionnel pour une meilleure visibilité. 