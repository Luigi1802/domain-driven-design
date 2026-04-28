# Vision de déploiement Docker

## Description des services

Le système de billetterie sera déployé avec une architecture microservices où chaque Bounded Context correspond à un conteneur Docker distinct. **ContexteRéférencement** offre une API REST pour gérer les événements ; il s'exécute sur le port 3001 avec une base PostgreSQL dédiée et expose les endpoints `/events` pour la consultation. **ContexteInterface** est une application web frontale qui consomme les APIs des autres services ; conteneurisée en Node.js, elle s'exécute sur le port 3000 et reçoit les variables d'environnement pour les URLs des services backend. **ContexteRéservation** gère la logique de réservation via une API REST sur le port 3002, communique avec ContexteRéférencement par HTTP synchrone et publie les événements `RéservationConfirmée` sur RabbitMQ. **ContextePaiement** exécute son service sur le port 3003 ; il traite les demandes de paiement, intègre des APIs bancaires externes et publie les résultats (`PaiementResultat`) sur le message broker. **ContexteNotification** s'exécute sur le port 3004 et écoute les événements RabbitMQ pour déclencher l'envoi d'emails via SendGrid.

Tous les services partagent une infrastructure commune : une instance RabbitMQ sur le port 5672 pour la communication asynchrone inter-contextes, une base PostgreSQL centralisée (ou des bases isolées selon les besoins), et un réseau Docker nommé `billetterie-network` pour la communication inter-conteneurs. Chaque service expose des variables d'environnement pour configurer les URLs des dépendances, les identifiants de base de données, et les clés API externes.

## Exemple de Dockerfile

```Docker
# Dockerfile ContexteRéservation

FROM node:18-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm ci --only=production

COPY src ./src

RUN npm run build

FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/src ./src
COPY --from=builder /app/dist ./dist

EXPOSE 3002

ENV NODE_ENV=production
ENV SERVICE_PORT=3002
ENV RABBITMQ_URL=amqp://guest:guest@rabbitmq:5672

CMD ["node", "dist/index.js"]

```