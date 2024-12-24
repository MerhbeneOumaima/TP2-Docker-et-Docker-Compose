# Utiliser une image de base légère de Node.js
FROM node:lts-alpine

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers package.json et package-lock.json
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste des fichiers
COPY . .

# Construire l'application React
RUN npm run build

# Installer un serveur HTTP léger pour servir les fichiers
RUN npm install -g serve

# Exposer le port 3000
EXPOSE 3000

# Commande pour démarrer le serveur HTTP
CMD ["serve", "-s", "build"]
