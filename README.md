# Projet Docker - Stack Full-Stack avec Reverse Proxy

**Repository :** https://github.com/hippot02/projet_docker

## Équipe

* Hippolyte
* Yvelin

## Objectif du projet final

Assembler et exécuter une **application web complète** composée de trois services :

* **Backend :** API REST Spring Boot
* **Frontend :** application React ou Vue
* **Base de données :** PostgreSQL

L’objectif est de conteneuriser chaque service, les orchestrer avec **Docker Compose**, et garantir la persistance des données ainsi que la bonne communication entre les services.

---



## Commandes pour Builder et Lancer

### Démarrage complet

```bash
docker compose up -d --build
```

### Rebuild d'un service spécifique

```bash
docker compose build --no-cache <nom_container>
docker compose up -d <nom_container>
```

### Arrêt de la stack

```bash
docker compose down
```
---

## Endpoints et URLs

### Frontend
- **URL principale** : http://localhost
- Accessible via le reverse-proxy

### API Backend
- **Health check** : http://localhost/api/health
- **Liste des items** : http://localhost/api/items (GET)
- **Ajouter un item** : http://localhost/api/items (POST)

### Base de données
- **Host** : localhost:5432 (non exposé publiquement)
- **Database** : mydb
- **User** : myuser

---

## Variables d'Environnement

Un fichier `.env` est utilisé à la racine du projet :

Adapter le .env.example pour faire fonctionner l'app

---

## Choix Techniques

### 1. **Reverse Proxy Nginx**
**Pourquoi :** 
- Point d'entrée unique sur le port 80
- Évite les problèmes CORS (même origine pour frontend et API)
- Masque les ports internes des services
- Configuration simple avec `proxy_pass`

### 2. **Build Multi-Stage pour le Frontend**
**Pourquoi :**
- **Étape 1 (Node.js)** : Compile React avec Vite (`npm run build`)
- **Étape 2 (Nginx)** : Sert uniquement les fichiers statiques
- **Avantages** : Image finale légère, pas de Node.js en production

**Dockerfile webapp :**
```dockerfile
FROM node:20-alpine AS build
# Build de l'app React
RUN npm run build

FROM nginx:alpine
# Copie uniquement le dist/
COPY --from=build /app/dist /usr/share/nginx/html
```

### 3. **Variables d'Environnement Vite avec ARG**
**Pourquoi :**
- Les variables Vite (`VITE_*`) doivent être définies **au moment du build**
- Utilisation de `ARG` dans le Dockerfile pour passer la valeur depuis docker-compose
- Permet de changer l'URL de l'API selon l'environnement

**Configuration :**
```yaml
# docker-compose.yml
webapp:
  build:
    args:
      VITE_API_BASE_URL: "http://localhost"
```

### 4. **Réseaux Docker Séparés**
**Pourquoi :**
- **Réseau `public`** : Communication entre reverse-proxy, webapp et spring-api
- **Réseau `backend`** : Isole PostgreSQL (accessible uniquement par spring-api)
- **Sécurité** : La base de données n'est pas accessible depuis le reverse-proxy ou webapp

### 5. **Configuration Nginx pour SPA**
**Pourquoi :**
- React Router nécessite que toutes les routes redirigent vers `index.html`
- Configuration `try_files $uri /index.html;` dans nginx.conf
- Évite les erreurs 404 lors du refresh sur une route React
---

## Problèmes Rencontrés et Solutions

### 1. **Variable VITE_API_BASE_URL ignorée**
**Problème :** Le frontend compilé contenait toujours `http://localhost:8080` au lieu de l'URL configurée.

**Cause :** 
- Les variables Vite sont résolues au moment du build Docker
- Un fichier `.env` local était copié dans le conteneur et prenait la priorité
- L'ARG n'était pas déclaré dans le Dockerfile

**Solution :**
- Ajout de `ARG VITE_API_BASE_URL` et `ENV VITE_API_BASE_URL` dans le Dockerfile avant le build
- Suppression du fichier `.env` local (ou ajout au `.dockerignore`)
- Passage de la valeur via `docker-compose.yml` :
  ```yaml
  build:
    args:
      VITE_API_BASE_URL: "http://localhost"
  ```

### 2. **Erreur 404 sur /api/items**
**Problème :** Le reverse-proxy ne routait pas correctement vers l'API Spring.

**Cause :** Configuration Nginx avec `proxy_pass http://backend/;` (avec `/` final) enlevait le préfixe `/api/` du chemin.

**Solution :** Utiliser `proxy_pass http://backend;` (sans `/` final) pour conserver le chemin complet.

### 3. **Cache navigateur persistant**
**Problème :** Après rebuild, l'ancienne version JavaScript était toujours chargée.

**Solution :** 
- Vider le cache navigateur avec Ctrl+Shift+R ou Ctrl+F5
- Utiliser la navigation privée pour tester
- Rebuild avec `--no-cache` pour forcer une reconstruction complète

### 4. **Ports internes vs externes**
**Problème :** Confusion entre les ports internes Docker et les ports exposés.

**Solution :**
- Les services communiquent entre eux via les ports internes (8080, 8081)
- Seul le reverse-proxy expose le port 80 publiquement
- Suppression de l'exposition des ports dans webapp et spring-api du docker-compose

---

## Tests et Validation

### 1️⃣ Lancer la stack

```bash
docker compose up -d --build
```

### 2️⃣ Vérifier que tout fonctionne

* **Frontend** : http://localhost
* **API Health** : http://localhost/api/health
* **Liste items** : http://localhost/api/items
* **PostgreSQL** : Persistance via volume `postgres_data`

### 3️⃣ Tester l'ajout d'items

1. Ouvrir http://localhost
2. Ajouter un item via le formulaire
3. Vérifier que l'item apparaît dans la liste
4. Redémarrer la stack : `docker compose restart`
5. Vérifier que les données sont toujours présentes (persistance)

### 4️⃣ Consulter les logs

```bash
docker compose logs -f
docker compose logs -f spring-api
docker compose logs -f reverse-proxy
```

---


