# 🏗️ Guide Infrastructure - Déploiement de Sites Web

**Comment déployer des sites web sur mon serveur avec Docker + Traefik + Portainer**

---

## 📋 Vue d'ensemble

### Stack technique
- **Docker** : Conteneurisation des applications
- **Traefik** : Reverse proxy automatique + SSL (Let's Encrypt)
- **Portainer** : Interface de gestion + déploiement automatique Git
- **Nginx** : Serveur web dans les containers

### Principe de fonctionnement
1. Code poussé sur Git
2. Portainer détecte les changements
3. Build automatique des images Docker
4. Déploiement des containers
5. Traefik configure automatiquement le routage + SSL

---

## 🐳 Architecture Docker

### Structure d'un projet

```
MON-PROJET/
├── docker-compose.yml       # Configuration principale
├── Dockerfile              # Image Docker du site principal
├── nginx.conf              # Configuration Nginx
├── index.html              # Fichiers du site
├── style.css
├── script.js
├── images/
│
└── sous-site/              # Site satellite (optionnel)
    ├── Dockerfile
    ├── nginx.conf
    ├── index.html
    └── ...
```

---

## 📝 docker-compose.yml

### Configuration de base (1 site)

```yaml
version: '3.9'

services:
  # Site principal
  nginx:
    build:
      context: .
      dockerfile: Dockerfile
    image: mon-projet:latest
    container_name: mon-projet
    labels:
      - traefik.http.routers.mon-projet.rule=Host(`www.mon-site.fr`)
      - traefik.http.routers.mon-projet.tls=true
      - traefik.http.routers.mon-projet.tls.certresolver=lets-encrypt
    restart: unless-stopped
    networks:
      - web

networks:
  web:
    external: true
    name: web
```

### Configuration multi-sites (plusieurs domaines)

```yaml
version: '3.9'

services:
  # Site principal - www.mon-site.fr
  nginx:
    build:
      context: .
      dockerfile: Dockerfile
    image: mon-projet:latest
    container_name: mon-projet
    labels:
      - traefik.http.routers.mon-projet.rule=Host(`www.mon-site.fr`)
      - traefik.http.routers.mon-projet.tls=true
      - traefik.http.routers.mon-projet.tls.certresolver=lets-encrypt
    restart: unless-stopped
    networks:
      - web

  # Site satellite - sous-domaine.mon-site.fr
  sous-site:
    build:
      context: ./sous-site
      dockerfile: Dockerfile
    image: mon-projet-sous-site:latest
    container_name: mon-projet-sous-site
    labels:
      - traefik.http.routers.sous-site.rule=Host(`sous-domaine.mon-site.fr`)
      - traefik.http.routers.sous-site.tls=true
      - traefik.http.routers.sous-site.tls.certresolver=lets-encrypt
    restart: unless-stopped
    networks:
      - web

networks:
  web:
    external: true
    name: web
```

### Explication des labels Traefik

```yaml
labels:
  # Domaine à utiliser
  - traefik.http.routers.MON-ROUTER.rule=Host(`www.exemple.fr`)
  
  # Activer TLS (HTTPS)
  - traefik.http.routers.MON-ROUTER.tls=true
  
  # Certificat Let's Encrypt automatique
  - traefik.http.routers.MON-ROUTER.tls.certresolver=lets-encrypt
```

**Notes importantes :**
- `MON-ROUTER` : Nom unique pour identifier le routeur (ex: `mon-projet`, `acces-connecte`)
- Le nom du routeur doit être identique dans les 3 labels
- Le réseau `web` doit être créé au préalable sur le serveur

---

## 🐋 Dockerfile

### Template de base (site statique)

```dockerfile
FROM nginx:alpine

# Copier les fichiers du site
COPY index.html /usr/share/nginx/html/
COPY style.css /usr/share/nginx/html/
COPY script.js /usr/share/nginx/html/

# Copier les ressources
COPY images /usr/share/nginx/html/images

# Copier la configuration Nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Exposer le port 80
EXPOSE 80

# Démarrer Nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Points clés
- **Image de base** : `nginx:alpine` (légère, 25MB)
- **Port** : Toujours exposer le port 80 (Traefik gère le HTTPS)
- **Fichiers** : Copier dans `/usr/share/nginx/html/`
- **Config** : Remplacer `/etc/nginx/conf.d/default.conf`

---

## ⚙️ nginx.conf

### Configuration optimisée

```nginx
server {
    listen 80;
    listen [::]:80;
    
    server_name www.mon-site.fr mon-site.fr;
    
    root /usr/share/nginx/html;
    index index.html;
    
    # Compression Gzip
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript 
               application/x-javascript application/xml+rss 
               application/javascript application/json;
    
    # Cache des fichiers statiques
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Route principale
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Headers de sécurité
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Bloquer l'accès aux fichiers cachés
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
    
    # Page d'erreur 404
    error_page 404 /index.html;
}
```

### Fonctionnalités incluses
- ✅ Compression Gzip (réduction de 70% de la taille)
- ✅ Cache navigateur (1 an pour les assets)
- ✅ Headers de sécurité
- ✅ Support IPv4 et IPv6
- ✅ SPA routing (Single Page App)

---

## 🚀 Workflow de déploiement

### Étape 1 : Configuration DNS
Pointer le domaine vers l'IP du serveur :

```
Type    Nom                    Valeur
A       www.mon-site.fr        123.45.67.89
A       sous-domaine.mon-site.fr  123.45.67.89
```

### Étape 2 : Créer le dépôt Git
```bash
# Sur votre machine locale
cd mon-projet/
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/user/mon-projet.git
git push -u origin main
```

### Étape 3 : Configurer Portainer

1. **Se connecter à Portainer** (https://portainer.mon-serveur.fr)

2. **Créer une nouvelle Stack** :
   - Stacks → Add Stack
   - Nom : `mon-projet`
   - Build method : **Repository**
   - Repository URL : `https://github.com/user/mon-projet`
   - Reference : `refs/heads/main`
   - Compose path : `docker-compose.yml`
   - ✅ Automatic updates
   - Fetch interval : `5m` (check toutes les 5 minutes)
   - ✅ Re-pull image
   - Deploy

3. **Vérifier le déploiement** :
   - Logs → Vérifier que tout démarre correctement
   - Containers → Vérifier l'état (running)

### Étape 4 : Test
```bash
# Tester le site
curl https://www.mon-site.fr

# Vérifier le certificat SSL
curl -vI https://www.mon-site.fr 2>&1 | grep -i "SSL"
```

### Étape 5 : Mise à jour continue

**Pour déployer des modifications :**

```bash
# Modifier les fichiers
vim index.html

# Push vers Git (Portainer déploiera automatiquement)
git add .
git commit -m "Update homepage"
git push
```

**Portainer va automatiquement :**
1. Détecter le push (dans les 5 minutes)
2. Pull le nouveau code
3. Rebuild les images Docker
4. Redéployer les containers
5. Traefik mettra à jour les routes

---

## 🔧 Configuration serveur (pré-requis)

### 1. Réseau Docker pour Traefik

```bash
# Créer le réseau partagé (une seule fois)
docker network create web
```

### 2. Traefik (reverse proxy)

**docker-compose.yml de Traefik :**

```yaml
version: '3.9'

services:
  traefik:
    image: traefik:latest
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/traefik.yml:ro
      - ./acme.json:/acme.json
    networks:
      - web

networks:
  web:
    external: true
```

**traefik.yml :**

```yaml
api:
  dashboard: true

entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: ":443"

certificatesResolvers:
  lets-encrypt:
    acme:
      email: contact@mon-site.fr
      storage: acme.json
      httpChallenge:
        entryPoint: web

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: web
```

**Permissions :**
```bash
chmod 600 acme.json
```

---

## 🎯 Exemple complet : Annecy Bricolage

### Structure du projet

```
ANNECY-BRICOLAGE/
├── docker-compose.yml
├── Dockerfile
├── nginx.conf
├── index.html
├── style.css
├── script.js
├── images/
│   └── ...
└── acces-connecte/
    ├── Dockerfile
    ├── nginx.conf
    ├── index.html
    ├── style.css
    └── images/
```

### docker-compose.yml

```yaml
version: '3.9'

services:
  # Site principal - www.annecy-bricolage.fr
  nginx:
    build:
      context: .
      dockerfile: Dockerfile
    image: annecy-bricolage:latest
    container_name: annecy-bricolage
    labels:
      - traefik.http.routers.annecy-bricolage.rule=Host(`www.annecy-bricolage.fr`)
      - traefik.http.routers.annecy-bricolage.tls=true
      - traefik.http.routers.annecy-bricolage.tls.certresolver=lets-encrypt
    restart: unless-stopped
    networks:
      - web

  # Site satellite - acces-connecte.annecy-bricolage.fr
  acces-connecte:
    build:
      context: ./acces-connecte
      dockerfile: Dockerfile
    image: annecy-bricolage-acces-connecte:latest
    container_name: acces-connecte-annecy-bricolage
    labels:
      - traefik.http.routers.acces-connecte.rule=Host(`acces-connecte.annecy-bricolage.fr`)
      - traefik.http.routers.acces-connecte.tls=true
      - traefik.http.routers.acces-connecte.tls.certresolver=lets-encrypt
    restart: unless-stopped
    networks:
      - web

networks:
  web:
    external: true
    name: web
```

### Déploiement

1. **DNS configuré** :
   - `www.annecy-bricolage.fr` → IP serveur
   - `acces-connecte.annecy-bricolage.fr` → IP serveur

2. **Push Git** :
   ```bash
   git add .
   git commit -m "Update sites"
   git push
   ```

3. **Portainer déploie automatiquement** (5 minutes max)

4. **Résultat** :
   - https://www.annecy-bricolage.fr → Site principal
   - https://acces-connecte.annecy-bricolage.fr → Site satellite
   - Certificats SSL automatiques
   - HTTP → HTTPS automatique

---

## 🔍 Debugging

### Voir les logs d'un container

```bash
# Via Docker CLI
docker logs mon-projet
docker logs -f mon-projet  # Mode suivi

# Via Portainer
Containers → mon-projet → Logs
```

### Vérifier Traefik

```bash
# Voir les routes configurées
docker logs traefik | grep -i "router"

# Accéder au dashboard Traefik
# https://traefik.mon-serveur.fr
```

### Tester le build localement

```bash
# Build l'image
docker build -t test:latest .

# Lancer le container
docker run -p 8080:80 test:latest

# Tester
curl http://localhost:8080
```

### Problèmes fréquents

**1. Certificat SSL non généré**
- Vérifier que le port 80 est ouvert (challenge HTTP-01)
- Vérifier le DNS (A record pointant vers le serveur)
- Vérifier `acme.json` (permissions 600)

**2. Container ne démarre pas**
- Voir les logs : `docker logs nom-container`
- Vérifier le Dockerfile
- Vérifier nginx.conf (syntaxe)

**3. Site non accessible**
- Vérifier que le container est `running`
- Vérifier les labels Traefik (typo ?)
- Vérifier que le réseau `web` existe
- Vérifier les logs Traefik

---

## 📊 Avantages de cette architecture

✅ **SSL automatique** : Let's Encrypt + renouvellement auto  
✅ **Zero-downtime** : Traefik route le trafic intelligemment  
✅ **Déploiement Git** : Push = déploiement automatique  
✅ **Multi-sites** : Plusieurs sites sur le même serveur  
✅ **Isolation** : Chaque site dans son container  
✅ **Scalable** : Facile d'ajouter de nouveaux sites  
✅ **Monitoring** : Portainer + logs centralisés  

---

## 🎓 Checklist nouveau site

- [ ] Créer le dossier projet avec Dockerfile + docker-compose.yml
- [ ] Configurer nginx.conf
- [ ] Créer les fichiers HTML/CSS/JS
- [ ] Tester localement (`docker-compose up`)
- [ ] Créer le dépôt Git + push
- [ ] Configurer le DNS (A record)
- [ ] Créer la Stack dans Portainer
- [ ] Activer les mises à jour automatiques
- [ ] Vérifier le déploiement (logs)
- [ ] Tester HTTPS + certificat SSL
- [ ] Tester une mise à jour (push Git)

---

## 📚 Ressources

- [Documentation Traefik](https://doc.traefik.io/traefik/)
- [Documentation Docker Compose](https://docs.docker.com/compose/)
- [Documentation Nginx](https://nginx.org/en/docs/)
- [Documentation Portainer](https://docs.portainer.io/)
- [Let's Encrypt](https://letsencrypt.org/)

---

**Dernière mise à jour** : 11 mars 2026  
**Version** : 1.0
