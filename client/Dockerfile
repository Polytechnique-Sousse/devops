# Utiliser l'image de base Node.js (LTS) avec Alpine pour réduire la taille de l'image
FROM node:lts-alpine

# Définir le répertoire de travail dans le conteneur
WORKDIR /app

# Copier les fichiers package.json et package-lock.json dans le conteneur
COPY package*.json ./

# Installer les dépendances avec npm
RUN npm install

# Copier le reste des fichiers du projet dans le conteneur
COPY . .

# Construire l'application React
RUN npm run build

# Installer un serveur HTTP léger pour servir les fichiers de build
RUN npm install -g serve

# Exposer le port 3000
EXPOSE 3000

# Démarrer le serveur pour servir les fichiers de build sur le port 3000
CMD ["serve", "-s", "build", "-l", "3000"]
