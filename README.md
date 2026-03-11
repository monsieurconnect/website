# Monsieur Connect

Site one-page pour présenter l'activité de Monsieur Connect : développement d'applications web, automatisation Python, intégration d'IA et systèmes domotiques.

## Structure du projet

```
MR-CONNECT/
├── index.html              # Page HTML principale
├── styles.css              # Styles CSS
├── script.js               # JavaScript (scroll fluide)
├── Dockerfile              # Image Docker
├── docker-compose.yml      # Configuration Docker Compose
├── nginx.conf              # Configuration Nginx
├── brief.md                # Brief du projet
├── guide.md                # Guide d'infrastructure
└── README.md               # Ce fichier
```

## Technologies utilisées

- HTML5
- CSS3 (design minimaliste et responsive)
- JavaScript vanilla (scroll fluide)
- Nginx (serveur web)
- Docker (conteneurisation)

## Déploiement local

### Test avec Docker

```bash
# Build l'image
docker build -t monsieur-connect:latest .

# Lancer le container
docker run -p 8080:80 monsieur-connect:latest

# Accéder au site
open http://localhost:8080
```

### Test sans Docker

Ouvrir simplement `index.html` dans un navigateur.

## Déploiement en production

### Pré-requis

- Serveur avec Docker + Traefik + Portainer
- DNS configuré pour pointer vers le serveur
- Réseau Docker `web` créé

### Étapes

1. **Configurer le DNS**
   ```
   Type A : www.monsieurconnect.fr → IP_SERVEUR
   Type A : monsieurconnect.fr → IP_SERVEUR
   Type A : www.monsieurconnect.net → IP_SERVEUR
   Type A : monsieurconnect.net → IP_SERVEUR
   ```

2. **Créer le dépôt Git et pousser le code**
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/USERNAME/monsieur-connect.git
   git push -u origin main
   ```

3. **Configurer Portainer**
   - Stacks → Add Stack
   - Nom : `monsieur-connect`
   - Build method : Repository
   - Repository URL : URL du dépôt Git
   - Reference : `refs/heads/main`
   - Compose path : `docker-compose.yml`
   - ✅ Automatic updates
   - Fetch interval : `5m`
   - ✅ Re-pull image
   - Deploy

4. **Vérifier le déploiement**
   ```bash
   # Tester le site
   curl https://www.monsieurconnect.fr
   
   # Vérifier le certificat SSL
   curl -vI https://www.monsieurconnect.fr 2>&1 | grep -i "SSL"
   ```

### Mises à jour

Pour déployer des modifications :

```bash
# Modifier les fichiers
vim index.html

# Push vers Git
git add .
git commit -m "Update content"
git push
```

Portainer détectera automatiquement les changements et redéploiera le site.

## Caractéristiques

- ✅ Design minimaliste et professionnel
- ✅ 100% responsive (mobile, tablette, desktop)
- ✅ Scroll fluide vers la section contact
- ✅ Optimisé pour les performances (Gzip, cache)
- ✅ Headers de sécurité configurés
- ✅ SSL automatique avec Let's Encrypt
- ✅ SEO optimisé

## Sections du site

1. **Hero** - Présentation principale
2. **Ce que je fais** - 4 domaines d'activité
3. **Ce que je peux construire** - 3 types de solutions
4. **Technologies** - Stack technique
5. **Méthode de travail** - Process en 4 étapes
6. **Contact** - Formulaire de contact
7. **Footer** - Informations légales

## Contact

Email : contact@monsieurconnect.fr

---

**Dernière mise à jour** : 11 mars 2026
