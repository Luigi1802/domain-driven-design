# Architecture Runtime

Description des services:

`postgres` est la base de données relationnelle partagée, exposée sur le port 5432. Elle stocke les données des contextes Référencement et Réservation, et son démarrage conditionne l'initialisation des services qui en dépendent.

`rabbitmq` est le message broker asynchrone (port 5672), point central de la communication événementielle. Il reçoit les publications de `reservation-service` et `paiement-service`, et les distribue à `notification-service`.

`referencement-service` (port 3001) expose une API REST pour la gestion des événements culturels. Il s'appuie sur PostgreSQL pour la persistance et sur RabbitMQ pour éventuellement émettre des événements de catalogue.

`reservation-service` (port 3002) orchestre la réservation : il interroge `referencement-service` en HTTP synchrone pour valider la disponibilité, écrit en base, et publie les événements de réservation sur RabbitMQ.

`paiement-service` (port 3003) traite les demandes de paiement via l'API Stripe (clé injectée par variable d'environnement). Il publie l'événement `PaiementResulté` sur RabbitMQ et ne dépend d'aucune base de données propre.

`notification-service` (port 3004) est un consommateur pur : il souscrit aux événements RabbitMQ et déclenche les envois d'e-mails via SendGrid. Il ne possède aucune API entrante et aucune base de données.

`interface-service` (port 3000) est le frontend web. Il agrège les trois APIs backend (`referencement`, `reservation`, `paiement`) dont les URLs lui sont passées en variables d'environnement, ce qui lui permet d'être reconfiguré sans rebuild.

 Un orchestrateur Docker Compose ou Kubernetes gère le démarrage coordonné des conteneurs, les dépendances d'initialisation (ex: la base doit être prête avant le service).

## Configuration à haut niveau (pseudo-code)

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secure_password
      POSTGRES_DB: billetterie
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  rabbitmq:
    image: rabbitmq:3.12-management
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
    ports:
      - "5672:5672"

  # ContexteRéférencement
  referencement-service:
    build: ./services/referencement
    environment:
      DB_HOST: postgres
      DB_PORT: 5432
      DB_NAME: billetterie
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      SERVICE_PORT: 3001
    ports:
      - "3001:3001"
    depends_on:
      - postgres
      - rabbitmq

  # ContexteRéservation
  reservation-service:
    build: ./services/reservation
    environment:
      DB_HOST: postgres
      REFERENCEMENT_API: http://referencement-service:3001
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      SERVICE_PORT: 3002
    ports:
      - "3002:3002"
    depends_on:
      - postgres
      - rabbitmq
      - referencement-service

  # ContextePaiement
  paiement-service:
    build: ./services/paiement
    environment:
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      STRIPE_API_KEY: ${STRIPE_API_KEY}
      SERVICE_PORT: 3003
    ports:
      - "3003:3003"
    depends_on:
      - rabbitmq

  # ContexteNotification
  notification-service:
    build: ./services/notification
    environment:
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672
      SENDGRID_API_KEY: ${SENDGRID_API_KEY}
      SERVICE_PORT: 3004
    ports:
      - "3004:3004"
    depends_on:
      - rabbitmq

  # ContexteInterface (Frontend)
  interface-service:
    build: ./services/interface
    environment:
      REFERENCEMENT_API: http://referencement-service:3001
      RESERVATION_API: http://reservation-service:3002
      PAIEMENT_API: http://paiement-service:3003
      SERVICE_PORT: 3000
    ports:
      - "3000:3000"
    depends_on:
      - referencement-service
      - reservation-service
      - paiement-service

volumes:
  postgres_data:

networks:
  billetterie-network:
    driver: bridge
```

## Schéma


![diagramme](./media/architecture-runtime.png)