# Observabilité


## Correlation ID

Dans notre système de billetterie, le Correlation ID (au format UUID v4) est généré à la frontière du système, dès l'entrée d'une requête via nos Adapters (qu'il s'agisse d'un appel REST sur `POST /reservations` ou de la consommation d'un message depuis le broker). Cet identifiant est ensuite passé systématiquement en paramètre de nos cas d'usage dans la couche Application, puis injecté dans les logs de nos Repositories (ex: appels PostgreSQL). Enfin, lorsqu'un événement métier comme `RéservationConfirmée` est publié, le Correlation ID est intégré dans les headers du message, permettant de tracer le flux de bout en bout, de l'intention du Spectateur jusqu'à l'envoi du billet par le `ContexteNotification`.

## Métriques métier

| Nom de la métrique | Description | Type |
|--------------------|-------------|------|
| **reservations_creees_total** | Nombre total de Réservations initiées au statut `EN_ATTENTE_PAIEMENT`, segmenté par `evenement_id` et `categorie_place`. Permet de monitorer en temps réel l'engouement lors de l'ouverture des ventes. | Counter 
| **reservations_expirees_total** | Nombre total de Réservations ayant dépassé le délai de verrouillage de 15 minutes sans validation de paiement. Indique d'éventuelles frictions techniques ou ergonomiques dans le `ContextePaiement`. | Counter 
| **delai_emission_billet_secondes** | Mesure du délai entre l'autorisation du paiement externe et l'envoi effectif du Billet au Spectateur par le `ContexteNotification`. Permet de garantir le respect de nos accords de niveau de service (SLA). | Histogram


## Logging structuré

Tous nos Bounded Contexts émettent des logs au format JSON pour faciliter l'indexation. Ils contiennent systématiquement le Correlation ID, le contexte d'origine, et des métadonnées liées aux agrégats de la billetterie.

```json
{
  "timestamp": "2026-04-27T10:29:51.000Z",
  "level": "INFO",
  "correlation_id": "550e8400-e29b-41d4-a716-446655440000",
  "bounded_context": "ContexteRéservation",
  "service": "CréerRéservation",
  "event": "reservation_created",
  "message": "Réservation créée avec succès pour le Spectateur S-0012",
  "resource_id": "RSV-9041",
  "metadata": {
    "evenement_id": "EV-88",
    "place_id": "P-1042",
    "statut": "EN_ATTENTE_PAIEMENT",
    "verrouillage_expire_a": "2026-04-27T10:44:51.000Z"
  }
}
```