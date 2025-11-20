# Étape 1 : Build avec Node.js
FROM node:20-alpine AS build

WORKDIR /app

# Argument de build pour l'API URL
ARG VITE_API_BASE_URL=""
ENV VITE_API_BASE_URL=${VITE_API_BASE_URL}

# Copier les fichiers de dépendances
COPY package.json package-lock.json* ./

# Installer les dépendances
RUN npm ci

# Copier le code source
COPY . .

# Définir la variable d'environnement Vite au moment du build
ARG VITE_API_BASE_URL=""
ENV VITE_API_BASE_URL=${VITE_API_BASE_URL}

# Build
RUN npm run build

# Étape 2 : Serveur Nginx
FROM nginx:alpine

# Copier le build React dans le répertoire Nginx
COPY --from=build /app/dist /usr/share/nginx/html

# Copier la configuration Nginx custom pour SPA
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Exposer le port 80
EXPOSE ${WEBAPP_PORT}

# Démarrer Nginx
CMD ["nginx", "-g", "daemon off;"]
