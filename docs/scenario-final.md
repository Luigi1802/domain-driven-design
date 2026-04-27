# Scénario complet inter-contextes

## Description narrative du scénario : Réservation, Paiement et Émission du Billet

Ce scénario illustre le parcours critique d'achat sur notre plateforme de billetterie, impliquant une orchestration fluide entre le `ContexteRéservation`, le `ContextePaiement` et le `ContexteNotification`. Tout commence lorsqu'un Spectateur consulte le plan de salle d'un Évènement et décide de sélectionner une Place spécifique pour l'ajouter à son panier. Le `ContexteRéservation` vérifie immédiatement la disponibilité de cette place. Si elle est libre, l'agrégat Réservation est créé avec le statut initial "En attente de paiement". À cet instant précis, un verrouillage temporaire est appliqué sur la Place afin de garantir qu'aucun autre utilisateur ne pourra la sélectionner pendant la phase de paiement.

Le Spectateur est ensuite redirigé vers l'interface de règlement. Le `ContextePaiement` prend alors le relais en initiant une transaction auprès de l'adapter bancaire externe (ex: Stripe). Le Spectateur saisit ses coordonnées bancaires et valide son achat. Une fois l'autorisation bancaire reçue, le `ContextePaiement` acte que les fonds sont sécurisés et publie le succès de la transaction.

Cette confirmation financière déclenche une réaction immédiate : le `ContexteRéservation` est informé et met à jour l'état de la Réservation, qui bascule définitivement sur le statut "Confirmée". Le verrouillage temporaire de la Place se transforme alors en une affectation permanente.

Enfin, la confirmation de la Réservation interpelle le `ContexteNotification`. Ce dernier récupère les informations nécessaires pour générer le Billet (qui contient le QR code unique, les détails de l'Évènement et les coordonnées de la Place), puis l'envoie par email au Spectateur. Le processus s'achève avec succès : le spectateur détient son Billet, la place est vendue, et le système est prêt à accueillir le public. Ce flux piloté par les événements (Event-Driven) garantit la consistance de la donnée, même lors des pics de charge typiques de l'ouverture des ventes.

## Liste des événements déclenchés (ordre chronologique)

1. `ReservationInitiee` : Émis par le `ContexteRéservation` lorsque le Spectateur valide son panier.

2. `PlaceVerrouillee` : Émis par le `ContexteRéservation` pour signaler que la place est bloquée pour 15 minutes.

3. `PaiementInitie` : Émis par le `ContextePaiement` lors de l'ouverture du tunnel de paiement.

4. `PaiementAutorise` : Émis par le `ContextePaiement` lorsque l'API bancaire valide la transaction.

5. `ReservationConfirmee` : Émis par le `ContexteRéservation` en réaction au paiement, transformant le verrouillage en vente définitive.

6. `BilletGenere` : Émis par le `ContexteNotification` (ou un contexte de génération documentaire) une fois le PDF du billet créé.

7. `BilletEnvoye` : Émis par le `ContexteNotification` confirmant la bonne transmission de l'email au Spectateur.

## Rappel des invariants concernés
- Disponibilité stricte de la Place : Une Place ne peut faire l'objet d'une nouvelle Réservation ou d'un verrouillage que si son statut actuel est strictement "Disponible". Deux Spectateurs ne peuvent en aucun cas verrouiller la même Place simultanément.

- Délai de verrouillage (Time-to-Live) : Le verrouillage d'une Place n'est valide que pour une durée stricte de 15 minutes. Si l'événement `PaiementAutorise` n'est pas traité dans ce laps de temps, la Réservation expire automatiquement et la Place redevient "Disponible" pour le reste du public.

- Consistance de l'émission du Billet : Un Billet d'accès ne peut être généré et expédié au Spectateur que si et seulement si l'agrégat Réservation associé a atteint le statut "Confirmée" (garantissant que le paiement a été intégralement validé au préalable).