# Rapport Devops AISSAT Djedjiga

# **TP1**

### 1-1 : Documentez les essentiels du conteneur de la base de données (commands and Dockerfile).

Explication du Dockerfile :
&nbsp;

ENV POSTGRES_DB=mydb \ POSTGRES_USER=postgres \ POSTGRES_PASSWORD=postgres : les variables d'environnement définissent les paramètres de configuration pour la base de données. on spécifie le nom de la base de données (mydb), le nom d'utilisateur (postgres) et le mot de passe (postgres).
&nbsp;

COPY 01-CreateScheme.sql /docker-entrypoint-initdb.d/ : Cette commande copie le fichier SQL 01-CreateScheme.sql dans le répertoire /docker-entrypoint-initdb.d/ du conteneur.
&nbsp;

COPY 02-InsertData.sql /docker-entrypoint-initdb.d/ : Cette commande copie le fichier SQL 02-InsertData.sql dans le répertoire /docker-entrypoint-initdb.d/ du conteneur. Ce fichier contient les instructions pour insérer des données dans la base de données.
&nbsp;

EXPOSE 5432 : indique que le container expose le port 5432

#### Étape 1 : Database

*  Construction de l'image de la bd :
``` docker build -t mydatabase . ```
*  Démarrage du container de la bd :
```docker run -d -p 5432:5432 --name database mydatabase```


#### Étape 2 : Init database

* Création du network : ``` docker network create app-network ```
* Reconstruction de l'image de la bd : ``` docker build -t mydatabase . ```
* Démarrage du container de la bd en spécifiant le réseau :
```docker run -d -p 5432:5432 --name database --network app-network mydatabase ```

#### Étape 3 : Persist data

* Construction de l'image de la db avec un volume : ``` docker build -t mydatabase . ```
* Démarrage du conteneur de la db avec un volume monté :
``` docker run -d -p 5432:5432 --name database --network app-network -v mydatabase_data:/var/lib/postgresql/data mydatabase ```


----------------

### 1-2 : Documentez les essentiels du containeur Backend API (commands and Dockerfile).

Les commandes essentielles sont la compilation du code Java (javac), la création de l'image (docker build) et le démarrage du container (docker run).
&nbsp;
Le Dockerfile doit spécifier l'image Java appropriée, copier le code Java compilé dans le container et exécuter le code avec la commande java -jar.


### Étape 4 : Backend API

* Construction de l'image du backend : ``` docker build -t mybackend . ```
* Démarrage du container du backend API en spécifiant le network :
``` docker run -d -p 8080:8080 --name backend --network app-network mybackend ```

----------------

### 1-3 : Explication du multistage build et de chaque étape du Dockerfile.

Explication du Dockerfile :
Utilisation d'un build multistage pour simplifier la construction de l'application Java.

&nbsp; 
Etapes du dockerfile :

&nbsp;
la première étape pour construire le code Java et une pour exécuter le code avec une image JRE.

&nbsp;
Le code Java compilé est copié de l'étape précédente à l'aide de COPY --from=0.

&nbsp;
Exécution de l'application Java avec la commande java -jar.

&nbsp;
Le multistage build permet de séparer la construction du code Java de son exécution dans des images différentes.
#### Étape 5 : Multistage build

* Construction de l'image avec le build multistage : ``` docker build -t myapp . ```
* Démarrage du container de l'application : ``` docker run -d -p 8080:8080 --name app myapp ```

#### Étape 6 : HTTP Server

* Construction de l'image pour le serveur HTTP : ``` docker build -t myhttpserver . ```
* Démarrage du container du serveur HTTP en spécifiant le réseau :
``` docker run -d -p 80:80 --name httpserver --network app-network myhttpserver ```

----------------
### 1-3 Document docker-compose most important commands. 1-4 Document your docker-compose file.

#### Étape 7 : Docker-compose 

* Exécution de la commande ``` docker-compose up ``` pour démarrer tous les containers en arrière-plan.

----------------
### 1-5 Document your publication commands and published images in dockerhub.

#### Étape 8 : Publish

* Connexion au registre Docker : ``` docker login ```
* Publication des images individuellement :
```docker push mydatabase```
```docker push mybackend```
```docker push myhttpserver```
```docker push myapp```

----------------

# **TP2**

### 2-1 What are testcontainers?

testcontainers sont des bibliothèques Java qui permettent d'éxecuter un container Docker
pendant les tests . Ils sont généralement utilisés pour valider que les tests d'intégration qui 
nécessitent des db par exemple, fonctionnent correctement 

### 2-2 Document your Github Actions configurations.

- création du répertoire .github/workflows et mettre à l'intérieur le fichier main.yml
- Dans le fichier main.yml :
  - declencher les pipeline sur la branche main
  - pour le job test-backend : récupérer d'abord le code
    ```  uses: actions/checkout@v2.5.0```   
  
   ensuite j'ai configuré le jdk17  ```actions/setup-java``` 
  
  &nbsp;
  

   puis construire le projet avec les tests inclus avec ```mvn```

   &nbsp;
   l'analyse du code avec SonarCloud en utilisant  ```sonar:sonar -Dsonar.projectKey=api-simple-student -Dsonar.organization=api-simple-student -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}```
  - pour le job build-and-push-docker-image : comme le job précédent on récupére d'abord le code avec un checkout
    &nbsp;
    ensuite se connecter à DockerHub 
    ```     
            name: Login to DockerHub
            run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}  
    ```
    
    - construire et publier les images pour chaque partie : backend, db, httpd, frontend
      &nbsp;
      exemple sur ce que j'ai fait pour publier l'image db :
      ```
          - name: Build image and push database
            uses: docker/build-push-action@v3
            with:
              context: ./db/
              tags: ${{secrets.DOCKERHUB_USERNAME}}/db
              push: ${{ github.ref == 'refs/heads/master' }} 
      ``` 

### 2-3 Document your quality gate configuration.
- dans le fichier main.yml décrit précedemment la commande mvn inclus l'étape SonarCloud analysis  

----------------

# **TP3**

### 3-1 Document your inventory and base commands
- l'inventory est donc dans le fichier devopsformation/ansible/inventories/setup.yml
- ansible_user est modifié avec centos 
- ansible_ssh_private_key_file : la variable avec le chemin vers le fichier qui contient la clé privée utilisée pour l'authentification 
- prod: contient le nom de domaine qui permet d'acceder au serveur 

Base commands :
- tester l'inventory avec ping  ```ansible all -i inventories/setup.yml -m ping```
- avoir des information sur le hostes on a utilisé : ```ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"```
- pour supprimer le serveur hhtpd du host on a utilisé : ```ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become```

### 3-2 Document your playbook

pour initialiser un role on lance la commande : ```ansible-galaxy init roles/docker```

le playbook se trouve dans : devopsformation/ansible/playbook.yml
le playbook spécifie l'éxecution de plusierus rôles : doker, network, proxy, app, database

```
- hosts: all
  gather_facts: false
  become: yes

  roles:
    - docker
    - network
    - database
    - app
    - proxy

```
La commande ``` ansible-playbook -i inventories/setup.yml playbook.yml ```
est utilisée pour exécuter le playbook Ansible à l'aide du fichier playbook.yml


### 3-3 Document your docker_container tasks configuration.
exemple : Pour exécuter un container Docker de db
&nbsp;
```
- name: Run database
  docker_container:
    name: mydb
    image: djedjiga18/db
    env:
      POSTGRES_USER: "postgres"
      POSTGRES_PASSWORD: "postgres"
      POSTGRES_DB: "mydb"
    networks:
      - name: my-network
    volumes:
      - "shared-docker:/path/in/container"

```
&nbsp;
docker-container : module Ansible utilisé pour gérer les containers Docker

&nbsp;
image : image docker à utiliser déja publiée au précédent tp sur Docker Hub

&nbsp;
networks: Réseaux auxquels le conteneur doit être connecté.

&nbsp;