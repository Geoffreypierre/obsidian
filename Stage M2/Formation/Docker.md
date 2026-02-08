#docker #ci #cd
# CI/CD

## CI

CI ou intégration continue regroupe les outils et pipeline utilisés pour la compilation et  vérification durant les étapes de développement du projet
### CD

Déploiement continue regroupe les outils et pipeline utilisés pour la compilation et  vérification lors de la mise en production effective du projet

# Docker

Docker est une plateforme logicielle permettant de faire tourner des logiciels dans des conteneurs.
### Dockerfile

#### Syntaxe

```dockerfile
FROM python:3-slim                # Spécifie l'environnement et la version
WORKDIR /app                      # Précise le dossier racine 
COPY server.py                    # Copie le fichier dans le conteneur 
EXPOSE 8000                       # informe le port (ne le modifie pas)
CMD ["python", "-u", "server.py"] # réalise la commande "python -u server.py". les [] évitent des problèmes de syntaxes de commandes suivant les systèmes d'exploitations
```

#### Commandes

```bash 
docker build -t mon-serveur .
```

Permet de build le conteneur, ``-t`` nomme l'image avec le nom ``mon-serveur``

```bash 
docker run --rm mon-serveur  
```

``--rm`` permet de supprimer automatiquement le conteneur une fois qu’il s’arrête

```bash 
docker run --rm -p 32010:8000 mon-serveur  
```

``-p`` assigne le port **32010** de mon pc au port **8000** du conteneur Docker. C'est à dire que une fois le conteneur lancé, le contenu publié est disponible soit à l'adresse ```http://localhost:32010``` de mon ordinateur ou bien  à l'adresse ```http://localhost:8000``` une fois rentré dans le conteneur docker via la commande  ```docker exec -it mon-conteneur``` 

### Docker compose

#### Syntaxe

```yaml
services:
  wordpress:                       # nom du service                
    image: wordpress:6.9-apache    # image et version utilisée           
    ports: 
      - "8080:80"                  # portPC : portDocker
    environment:                   # les 4 champs sont présent à chaque fois
      - WODRPRESS_DB_HOST=db
      - WODRPRESS_DB_USER=root
      - WODRPRESS_DB_PASSWORD=rootpw
      - WODRPRESS_DB_NAME=wordpress
  
  db:
    image: mariadb:5.7
    environment:
      - MARIADB_ROOT_PASSWORD=rootpw
      - MYSQL_DATABASE=wordpress
    volumes:
      - db_data:/var/lib/mysql

  python-app:
    image: gitlab.codein.fr:5050/axel/docker-2026/server
    ports:
      - "8081:8000"

volumes:
  db_data:
```

Le volume permet de garder en copie physique la base de données par exemple pour éviter de la perdre à chaque fois que l'on éteint le conteneur.