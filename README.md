# Hackathon "Agricola Numerica" 

Bienvenue au hackathon "Agricola Numerica" ! 

**L'Objectif :** En 5 jours, développer collectivement une version web fonctionnelle et jouable du jeu de société Agricola (basé sur les règles du fichier `a7-agricola-2016-regle.pdf`), en utilisant une architecture microservices de pointe.

## 1. Architecture Globale de la Solution

Nous allons construire une application distribuée basée sur une architecture microservices avec la stack technique suivante :

*   **Backend :** Java 17+ & Spring Boot 3.x
*   **Architecture Backend :** Spring Cloud (Gateway, Eureka)
*   **Communication Asynchrone :** RabbitMQ
*   **Gestion de l'État de Jeu :** Redis
*   **Base de Données (pour les services simples) :** H2 / PostgreSQL
*   **Communication Temps Réel :** WebSockets (via STOMP)
*   **Frontend :** Framework JavaScript moderne (React, Vue, ou Angular)
*   **Conteneurisation :** Docker & Docker Compose

### 1.1. Architecture Frontend (Single Page Application)

L'interface utilisateur sera une **Application Web Monopage (Single Page Application - SPA)**. Ce choix architectural est crucial pour offrir une expérience de jeu fluide et réactive, digne d'une application de bureau. Au lieu de recharger des pages entières, la SPA mettra à jour dynamiquement les composants de l'interface (le plateau, l'inventaire, la ferme du joueur) en temps réel. Elle communiquera avec le backend via deux canaux distincts : des appels **API REST** pour envoyer les actions du joueur (ex: "placer un fermier") et une connexion **WebSocket** pour recevoir instantanément les mises à jour de l'état du jeu, garantissant que tous les joueurs voient la même information au même moment.

### 1.1. Schéma d'Architecture Détaillé

Le schéma ci-dessous illustre le flux de communication entre les différents composants de notre application, du navigateur de l'utilisateur jusqu'aux services de backend et aux systèmes de stockage.

```

  +--------------------------------------------------------------------------------------------------+
  |                                        Navigateur de l'Utilisateur                               |
  |                                                                                                  |
  |    +------------------------------------------------------------------------------------------+  |
  |    |                                     Frontend (React/Vue/Angular)                         |  |
  |    |                                                                                          |  |
  |    |  (Équipe 6: Plateau)   (Équipe 7: Ferme Joueur)   (Équipe 8: Inventaire)                  |  |
  |    +------------------------------------------------------------------------------------------+  |
  |         |             ^                                                                          |
  |         | HTTP/REST   | WebSocket                                                                |
  +---------|-------------|--------------------------------------------------------------------------+
           |             |
           |             |
  +--------v-------------|--------------------------------------------------------------------------+
  |                       |                                        Docker Network                  |
  |  +--------------------+---------------------------------+                                        |
  |  | API Gateway (Spring Cloud Gateway) - Équipe 1        |                                        |
  |  | (Point d'entrée unique, routage, sécurité de base)   |                                        |
  |  +------------------------------------------------------+                                        |
  |      |         |         |         |         |                                                   |
  |      | REST    | REST    | REST    | REST    | REST                                              |
  |      v         v         v         v         v                                                   |
  |  +---------+ +---------+ +---------+ +---------+ +----------------------+                        |
  |  | User    | | Game    | | Game    | | Game    | | Notification Service |<-----(Écoute)-----------+
  |  | Service | | Lobby   | | Engine  | | State   | | (Équipe 9)           |                        |
  |  | (Éq. 11)| | Service | | (Éq. 2-4) | Service | +----------------------+                        |
  |  +---------+ | (Éq. 10)| +---------+ | (Éq. 5) |      ^ WebSocket (STOMP)                        |
  |      |       |         |      |      +---------+      | (Pousse les mises à jour)                |
  |      |       |         |      |           |            +----------------------------------------+
  |      |       |         |      |           |
  |      |       |         |      |           |
  |      v       v         |      v           v
  |  +---------+ +---------+ |  +--------------------------+  +--------------------+
  |  | H2 /    | | H2 /    | |  | RabbitMQ (Messagerie)    |  | Redis (Cache)      |
  |  | Postgre | | Postgre | |  | (Événements de jeu)      |  | (État du jeu)      |
  |  | SQL     | | SQL     | +->| `GameStateUpdatedEvent`  |->| (GameState Object) |
  |  +---------+ +---------+    +--------------------------+  +--------------------+
  |                                      ^                            ^
  |                                      | (Publie l'événement)       | (Sauvegarde/Lecture)
  |                                      +----------------------------+
  |                                                                                                  |
  |  +--------------------------------------------------------------------------------------------+  |
  |  | Service Discovery (Eureka) - Équipe 1                                                      |  |
  |  | (Tous les services backend s'enregistrent ici. La Gateway l'utilise pour le routage.)      |  |
  |  +--------------------------------------------------------------------------------------------+  |
  +--------------------------------------------------------------------------------------------------+

```


### 1.2. Le Rôle Indispensable de Docker

Dans un projet de cette envergure, avec 11 microservices développés par 74 personnes, **Docker n'est pas une simple commodité, c'est le pilier qui garantit la cohésion et le succès du projet.** Son utilisation est indispensable pour les raisons suivantes :

*   **Standardisation de l'Environnement :** Docker résout le problème classique du "ça marche sur ma machine". En définissant chaque service et ses dépendances dans un `Dockerfile` et en orchestrant l'ensemble avec `docker-compose`, nous nous assurons que chaque développeur, quel que soit son système d'exploitation (Windows, macOS, Linux), exécute **exactement le même environnement**. Cela élimine les bugs liés à la configuration locale et permet de se concentrer sur le code.

*   **Gestion Simplifiée des Dépendances :** Notre architecture repose sur des services tiers comme RabbitMQ et Redis. Les installer et les configurer manuellement sur chaque poste serait une source infinie d'erreurs et de perte de temps. Avec `docker-compose`, l'ajout de ces services se résume à quelques lignes de YAML. Tout le monde dispose des mêmes versions, configurées de la même manière, en lançant une seule commande.

*   **Isolation et Autonomie des Services :** Chaque microservice tourne dans son propre conteneur isolé. Cela garantit que les dépendances d'un service (par exemple, une librairie spécifique) n'entrent pas en conflit avec celles d'un autre. C'est l'incarnation même du principe d'autonomie des microservices.

*   **Fiabilité de l'Intégration et de la Démo Finale :** Le fichier `docker-compose.yml` est notre "contrat d'orchestration". Il définit comment les 11 services, le frontend, RabbitMQ et Redis démarrent et communiquent entre eux. Pour la démo finale, lancer l'application entière se fera via une seule commande : `docker-compose up`. C'est la garantie d'une présentation fiable, reproductible et professionnelle, prouvant que nous avons livré un système intégré et non un assemblage fragile.

En résumé, Docker abstrait la complexité de l'infrastructure, permettant à toutes les équipes de collaborer efficacement pour construire un produit unifié, comme le feraient des équipes dans une entreprise technologique moderne.

### 1.3. ddddd

La création d'un squelette pour le fichier `docker-compose.yml` est même l'une des toutes premières tâches du hackathon.

C'est de la responsabilité de l'Équipe 1 (Les Architectes) de le définir dès le Jour 1. Ce fichier de base est essentiel pour que toutes les autres équipes puissent démarrer leur développement dans un environnement cohérent.

Le squelette initial doit contenir les services d'infrastructure fondamentaux sur lesquels tout le monde s'appuiera :

- Le service de découverte Eureka
- La passerelle API Gateway
- Le broker de messages RabbitMQ
- Le cache Redis

Pour aider l'Équipe 1 à démarrer (et pour que tout le monde ait une vision claire), voici à quoi pourrait ressembler un excellent squelette de `docker-compose.yml` pour le Jour 1. Ce fichier pourrait être placé à la racine du projet.


 # Version de la syntaxe Docker Compose
 version: '3.8'
 
 # Définition des services (conteneurs)
 services:
 
   # Service Discovery : L'annuaire de tous nos microservices
   eureka-server:
     # 'build' indique à Docker de construire l'image à partir du Dockerfile
     # situé dans le dossier ./eureka-server
     build: ./eureka-server
     # Nom de l'image qui sera construite (optionnel mais bonne pratique)
     image: hackit-shipit/eureka-server:0.0.1-SNAPSHOT
     ports:
       # Mappe le port 8761 du conteneur au port 8761 de la machine hôte
       - "8761:8761"
     environment:
       # Active le profil Spring 'docker' pour des configurations spécifiques au conteneur
       - SPRING_PROFILES_ACTIVE=docker
     networks:
       - agricola-net
 
   # API Gateway : Le point d'entrée unique pour le frontend
   api-gateway:
     build: ./api-gateway
     image: hackit-shipit/api-gateway:0.0.1-SNAPSHOT
     ports:
       # Le port standard pour les requêtes HTTP de l'application
       - "8080:8080"
     environment:
       - SPRING_PROFILES_ACTIVE=docker
       # Indique à la Gateway où trouver Eureka pour le routage dynamique
       - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-server:8761/eureka
     # 'depends_on' s'assure qu'Eureka est démarré avant la Gateway
     depends_on:
       - eureka-server
     networks:
       - agricola-net
 
   # Messagerie Asynchrone : Pour la communication par événements
   rabbitmq:
     # Utilise une image officielle depuis Docker Hub
     image: rabbitmq:3.13-management
     ports:
       # 5672 est le port pour le protocole AMQP (utilisé par Spring)
       - "5672:5672"
       # 15672 est pour l'interface de management web (très utile pour le debug)
       - "15672:15672"
     environment:
       # Identifiants par défaut (à ne pas utiliser en production, mais parfait pour le dev)
       - RABBITMQ_DEFAULT_USER=guest
       - RABBITMQ_DEFAULT_PASS=guest
     networks:
       - agricola-net
 
   # Cache / Base de données en mémoire : Pour stocker l'état du jeu
   redis:
     image: redis:7.2-alpine
     ports:
       # 6379 est le port par défaut de Redis
       - "6379:6379"
     networks:
       - agricola-net
 
 # Définition du réseau partagé
 # Permet aux conteneurs de communiquer entre eux en utilisant leur nom de service
 # (ex: la Gateway peut joindre Eureka via l'URL http://eureka-server:8761)
 networks:
   agricola-net:
     driver: bridge
 
 


## 2. Organisation

### 2.1. La répartition des équipes par domaines

Le projet est divisé en 11 domaines, chacun sous la responsabilité d'une équipe dédiée. La collaboration et la communication entre ces équipes sont la clé du succès.

| Pôle | Équipe | Nom | Domaine(s) | Responsabilités Clés |
| :--- | :--- | :--- | :--- | :--- |
| **Infrastructure** | Équipe 1 (6) | Les Architectes | API Gateway, Service Discovery | Point d'entrée unique, routage, annuaire des services, documentation API globale. |
| **Logique de Jeu** | Équipe 2 (7) | Les Maîtres du Jeu | Game Engine (Tours & Actions) | Gestion des tours, des phases et des actions de base. |
| | Équipe 3 (7) | Les Bâtisseurs | Game Engine (Construction) | Logique de construction (pièces, clôtures, étables). |
| | Équipe 4 (7) | Les Fermiers | Game Engine (Récolte & Animaux) | Logique de la phase de Récolte et de la gestion des animaux. |
| | Équipe 5 (7) | Les Gardiens de l'État | Game State Service | Définition et gestion de l'état du jeu dans Redis. |
| **Interface** | Équipe 6 (7) | Les Artistes | Frontend (Plateau Principal) | Affichage du plateau de jeu global, gestion de l'état client et des WebSockets. |
| | Équipe 7 (7) | Les Paysagistes | Frontend (Ferme du Joueur) | Affichage de la ferme personnelle de chaque joueur. |
| | Équipe 8 (6) | Les Intendants | Frontend (Inventaire & Actions) | Affichage de l'inventaire et gestion de l'envoi des actions du joueur. |
| **Utilitaires** | Équipe 9 (7) | Les Messagers | Notification Service | Écoute des événements sur RabbitMQ et diffusion via WebSockets. |
| | Équipe 10 (7) | Les Hôtes | Game Lobby Service | Création et gestion des salons de jeu avant le début d'une partie. |
| | Équipe 11 (6) | Les Greffiers | User Service | Gestion simplifiée des profils de joueurs. |

### 2.2. Les Commandements du Hackathon : À faire et à ne pas faire

Pour que 11 équipes convergent vers un produit unique et fonctionnel, le respect de quelques principes fondamentaux est non-négociable.

*   **1. VALIDEZ LES CONTRATS AVANT DE CODER**
    *   **À FAIRE :** Passez du temps avec les équipes dont vous dépendez (ou qui dépendent de vous) pour définir et valider les contrats d'API (OpenAPI) et les formats d'événements (RabbitMQ). Utilisez le canal `#api-contracts`.
    *   **À NE PAS FAIRE :** Commencer à implémenter la logique métier de votre service en vous basant sur des suppositions sur ce que les autres services vont vous envoyer.
    *   **POURQUOI ?** Un contrat est une promesse. Coder sans contrat, c'est construire une maison sans plan. Vous serez obligé de tout casser et de tout reconstruire quand vous réaliserez que les portes ne sont pas à la bonne taille pour les murs. C'est la cause n°1 d'échec dans les projets microservices.

*   **2. INTÉGREZ TÔT, INTÉGREZ SOUVENT**
    *   **À FAIRE :** Dès que deux services ont un contrat validé, essayez de les faire communiquer, même si c'est pour un simple "Hello World". L'objectif du Jour 2 est précisément de valider ce flux de base.
    *   **À NE PAS FAIRE :** Attendre le Jour 4 ou 5 pour essayer de connecter votre service au reste de l'écosystème. C'est la recette garantie pour une "intégration hell" (l'enfer de l'intégration).
    *   **POURQUOI ?** Découvrir un problème de réseau, de configuration ou de sérialisation le mardi est un petit obstacle. Le découvrir le vendredi à 14h est une catastrophe.

*   **3. NE RÉINVENTEZ PAS LA ROUE**
    *   **À FAIRE :** Partagez les configurations communes ! Si une équipe a trouvé la bonne configuration Maven/Gradle pour RabbitMQ ou Eureka, partagez-la sur le canal `#backend-config`. Idem pour les `Dockerfile` ou les configurations de composants frontend.
    *   **À NE PAS FAIRE :** Chaque équipe passe des heures à résoudre les mêmes problèmes de configuration dans son coin.
    *   **POURQUOI ?** Vous êtes une seule et grande équipe projet. Le temps gagné par une équipe sur un problème technique est du temps gagné pour l'ensemble du projet.

*   **4. PENSEZ DOCKER DÈS LA PREMIÈRE MINUTE**
    *   **À FAIRE :** Chaque équipe est responsable de fournir un `Dockerfile` fonctionnel pour son service. Testez régulièrement que votre service démarre correctement avec `docker-compose up`.
    *   **À NE PAS FAIRE :** Développer pendant 4 jours en local avec votre IDE et vous dire "on verra pour Docker plus tard".
    *   **POURQUOI ?** La démo finale se fera avec `docker-compose up`. Si votre service ne fonctionne que sur votre machine, il ne fonctionne pas pour le projet. Docker est la garantie de la reproductibilité et le filet de sécurité de notre démo.

### 2.3. Organisation Interne des Équipes

Chaque équipe s'organisera en "Squad Agile" avec les rôles suivants pour maximiser l'efficacité :

*   **Product Owner / Chef de Projet (1) :** Gardien de la vision fonctionnelle, expert des règles, gestion du backlog de l'équipe.
*   **Tech Lead / Architecte (1) :** Gardien de la vision technique, responsable de la qualité du code et des choix de conception internes.
*   **Squad Backend (2-3) :** Développe la logique métier et les API du service.
*   **Squad Frontend (2-3) :** (Pour les équipes du pôle Interface) Développe les composants visuels et les interactions.
*   **DevOps / Ingénieur Qualité (1) :** Responsable du `Dockerfile` du service et de son intégration dans le `docker-compose` global.

## 3. Organisation & Communication du Hackathon

Pour assurer une collaboration fluide entre 74 personnes, nous suivrons des rituels agiles :

*   **Scrum des Scrums :** Chaque matin à 9h15, un représentant technique de chaque équipe se réunit avec les organisateurs pour synchroniser, identifier les dépendances et lever les blocages.
*   **Contrats d'API :** La communication entre les services se base sur des contrats formels définis avec **OpenAPI (Swagger)**. L'équipe "Architectes" est la gardienne de la documentation d'API centrale. Aucune intégration ne doit commencer sans un contrat validé.
*   **Outils :** La communication se fera sur une plateforme de chat (Slack/Teams) avec des canaux dédiés (`#api-contracts`, `#frontend-backend-sync`, `#deployment-docker`, etc.).

## 4. Backlog Quotidien (Objectifs Généraux)
 
Voici une feuille de route détaillée pour guider le travail de chaque équipe jour après jour.

### Jour 1 (Lundi) : Fondations, Modélisation et Contrats
**Objectif Global :** Mettre en place l'infrastructure, définir les modèles de données et établir les contrats d'API. Personne ne code une logique métier complexe, on construit les fondations.
*   **Équipe 1 (Architectes)**: Créer le squelette de l'infrastructure (`Eureka`, `Gateway`, `docker-compose` de base) et définir le processus de documentation OpenAPI.
*   **Pôle Logique de Jeu (Équipes 2, 3, 4)**: Modéliser les concepts clés d'Agricola en objets Java (`Game`, `PlayerFarm`, `Animal`, etc.) en collaboration avec l'équipe 5.
*   **Équipe 5 (Gardiens de l'État)**: Créer le projet `game-state-service`, configurer Redis, et définir la classe principale `GameState` qui sera stockée.
*   **Pôle Frontend (Équipes 6, 7, 8)**: Mettre en place le projet frontend et créer les maquettes visuelles statiques des différents plateaux et inventaires.
*   **Équipe 9 (Messagers)**: Créer le projet `notification-service`, ajouter RabbitMQ au `docker-compose`, et définir le format de l'événement `GameStateUpdatedEvent`.
*   **Équipe 10 (Hôtes)**: Créer le projet `game-lobby-service` et définir ses endpoints REST dans un fichier `openapi.yaml`.
*   **Équipe 11 (Greffiers)**: Créer le projet `user-service` et définir son endpoint `POST /users` dans un fichier `openapi.yaml`.

### Jour 2 (Mardi) : Le "Hello World" de Bout en Bout
**Objectif Global :** Intégrer les services de base pour prouver que l'architecture fonctionne. Un joueur doit pouvoir se créer, créer une partie et la rejoindre.
*   **Équipe 1 (Architectes)**: Intégrer les routes des services `user-service` et `lobby-service` dans la Gateway.
*   **Équipe 2 (Maîtres du Jeu)**: Commencer à implémenter la logique de la phase de "Préparation" (révéler une carte, réapprovisionner).
*   **Équipe 5 (Gardiens de l'État)**: Implémenter les méthodes `save(GameState)` et `findById(gameId)` du service.
*   **Pôle Frontend (Équipes 6, 7, 8)**: Rendre les vues dynamiques en les connectant à l'API Gateway pour créer un joueur et afficher la liste des parties.
*   **Équipe 9 (Messagers)**: Mettre en place un "ping-pong" WebSocket pour tester la connexion temps réel avec le frontend.
*   **Équipe 10 (Hôtes)**: Implémenter la logique des endpoints du lobby définis la veille.
*   **Équipe 11 (Greffiers)**: Implémenter la logique de l'endpoint `POST /users`.

### Jour 3 (Mercredi) : La Première Action de Jeu
**Objectif Global :** Un joueur doit pouvoir effectuer une action simple (ex: "Prendre 3 bois") et la mise à jour doit être visible par tous les joueurs de la partie en temps réel.
*   **Équipe 1 (Architectes)**: Intégrer les routes du `game-engine-service` et stabiliser l'infrastructure.
*   **Équipe 2 (Maîtres du Jeu)**: Implémenter le endpoint `POST /games/{id}/action`, valider une action simple, mettre à jour l'état, et publier l'événement `GameStateUpdatedEvent` sur RabbitMQ.
*   **Équipe 5 (Gardiens de l'État)**: S'assurer que la sauvegarde/lecture de l'état après une action fonctionne parfaitement.
*   **Pôle Frontend (Équipes 6, 7, 8)**: Rendre les cases actions cliquables, envoyer la requête d'action, et gérer la réception du message WebSocket pour rafraîchir TOUTE l'interface.
*   **Équipe 9 (Messagers)**: Implémenter la logique complète : écouter RabbitMQ, recevoir l'événement, et le pousser via WebSocket aux bons clients.
*   **Équipes 3, 4, 10, 11**: Support, débugging et préparation des tâches du lendemain.

### Jour 4 (Jeudi) : Montée en Puissance des Fonctionnalités
**Objectif Global :** Implémenter les logiques de jeu les plus complexes et les afficher.
*   **Équipe 2 (Maîtres du Jeu)**: Gérer la séquence des tours de joueurs et le placement des fermiers.
*   **Équipe 3 (Bâtisseurs)**: Implémenter la logique des actions "Construire des pièces" et "Construire des clôtures".
*   **Équipe 4 (Fermiers)**: Implémenter la logique de la phase de **Récolte** (nourrir, reproduire). C'est le gros morceau de la journée.
*   **Pôle Frontend (Équipes 6, 7, 8)**: Afficher dynamiquement les nouvelles fonctionnalités (ferme qui grandit, indicateur de récolte, etc.).
*   **Toutes les autres équipes**: Débugging, stabilisation, et amélioration de l'existant.

### Jour 5 (Vendredi) : Finalisation, Débugging et Préparation de la Démo
**Objectif Global :** Geler les nouvelles fonctionnalités ("feature freeze"), chasser les bugs, et préparer une présentation fluide.
*   **Matin (Toutes les équipes)**:
    *   Correction intensive des bugs.
    *   Amélioration de l'UX/UI (retours visuels, messages d'erreur clairs).
    *   S'assurer que le projet se lance avec un simple `docker-compose up`.
    *   Finaliser le `README.md` de chaque service.
*   **Après-midi (Toutes les équipes)**:
    *   Répétition de la démo.
    *   Préparation des slides de présentation.
    *   **DÉMOS !**

## 5. Ressources Techniques (Backend)

Chaque service backend partagera une base commune et ajoutera des dépendances spécifiques.

| Technologie | Dépendance Maven/Gradle Clé | Utilisé par |
| :--- | :--- | :--- |
| **API REST** | `spring-boot-starter-web` | Tous les services |
| **Service Discovery** | `spring-cloud-starter-netflix-eureka-client/server` | Équipe 1 et tous les autres |
| **API Gateway** | `spring-cloud-starter-gateway` | Équipe 1 |
| **Messagerie** | `spring-boot-starter-amqp` (pour RabbitMQ) | Équipes 2, 3, 4, 9, 10 |
| **Base de Données** | `spring-boot-starter-data-jpa` + `h2` | Équipe 11 (et 10) |
| **Cache/État de Jeu** | `spring-boot-starter-data-redis` | Équipe 5 |
| **Temps Réel** | `spring-boot-starter-websocket` | Équipe 9 |
| **Documentation API** | `org.springdoc:springdoc-openapi-starter-webmvc-ui` | Tous les services backend |
| **Utilitaires** | `org.projectlombok:lombok`, `spring-boot-devtools` | Tous les services |

## 6. Ressources Humaines (Encadrement du hackathon )

Une équipe de mentors sera présente pour vous guider :

*   **Coordinateur Principal (1) :** Chef d'orchestre de l'événement.
*   **Architecte en Chef (1-2) :** Référence technique suprême, anime le Scrum des Scrums.
*   **Mentors Spécialisés (3-4) :** Experts en Backend, Frontend, et DevOps/Infra pour vous aider au quotidien.
*   **Game Master (1) :** L'expert fonctionnel des règles d'Agricola.

## 7. Évaluation

L'évaluation portera sur le processus autant que sur le résultat final.

1.  **Évaluation Continue :** Les mentors observeront la proactivité, la communication et la collaboration tout au long de la semaine.
2.  **Checkpoint (Mercredi) :** Une mini-démo par pôle pour vérifier l'état de l'intégration.
3.  **Évaluation Finale (Vendredi) :** La note sera une pondération de :
    *   **Démo Finale (40%) :** Le produit final sera jugé sur sa complétude, sa robustesse et la qualité de la présentation.
        *   **Nature de la démo :** Il s'agit d'une **démonstration globale et unifiée de l'application complète**, et non d'une série de démos par équipe. L'objectif est de prouver que les 11 microservices s'intègrent et fonctionnent ensemble pour créer une expérience utilisateur cohérente.
        *   **Format :** La présentation doit raconter l'histoire d'une session de jeu fluide, du début à la fin (création de partie, tours de jeu, actions complexes, phase de récolte).
        *   **Critères :** La note portera sur la **fluidité** du scénario présenté, la **robustesse** de l'application (absence de bugs majeurs), la **complétude** des fonctionnalités et la **clarté** des explications techniques sur le flux de données (API Gateway -> Service -> RabbitMQ -> WebSocket -> Frontend).
        *   **Responsabilité :** Le succès de cette démo est une **responsabilité collective**. Un échec d'intégration impacte l'ensemble du projet, ce qui reflète la réalité des projets logiciels à grande échelle.
    *   **Évaluation par les Pairs (30%) :** Chaque étudiant évaluera anonymement la contribution des membres de son équipe (technique, collaboration, fiabilité).
    *   **Évaluation par les Mentors (30%) :** Les encadrants évalueront la performance individuelle observée durant le hackathon (apprentissage, curiosité, esprit d'équipe).

## 8. Stratégie de Test et Qualité

La qualité est l'affaire de tous. Pour livrer un produit fonctionnel, chaque équipe doit s'assurer que son domaine est robuste et bien testé. Voici les types de tests à réaliser :

### Tests Unitaires (Responsabilité de CHAQUE équipe)

Ces tests valident les plus petites unités de code (une méthode, une classe) en isolation.

*   **Équipes Backend (2 à 5, 9 à 11) :**
    *   **Outils :** JUnit 5, Mockito.
    *   **Quoi tester :** La logique métier dans les classes de service, les validateurs, les classes utilitaires. Les dépendances externes (autres services, bases de données) doivent être "mockées".
*   **Équipes Frontend (6, 7, 8) :**
    *   **Outils :** Jest, React Testing Library (ou équivalent).
    *   **Quoi tester :** Chaque composant en isolation. Valider qu'il s'affiche correctement en fonction des `props` et que les événements utilisateur sont bien gérés.

### Tests d'Intégration (Responsabilité de CHAQUE équipe backend)

Ces tests valident qu'un service fonctionne correctement avec ses dépendances directes, comme sa base de données ou son cache.

*   **Outils :** `@SpringBootTest`, `Testcontainers` (si une vraie BDD/Redis est nécessaire).
*   **Exemple pour l'Équipe 11 (Greffiers) :** Créer un test qui utilise le `UserRepository` pour écrire un utilisateur dans la base de données H2, puis le relire pour vérifier son intégrité.
*   **Exemple pour l'Équipe 5 (Gardiens de l'État) :** Créer un test qui sauvegarde un objet `GameState` complet dans Redis, le récupère et vérifie que toutes les données sont correctes.

### Tests de l'API (Responsabilité de CHAQUE équipe backend)

Ces tests valident que les endpoints REST du service se comportent comme attendu.

*   **Outils :** `MockMvc` dans Spring.
*   **Quoi tester :** Pour chaque endpoint, tester le cas nominal (statut 200 OK), les cas d'erreur (ex: 400 Bad Request si une donnée est manquante), et la sécurité de base.

### Tests de Bout en Bout (E2E - Responsabilité partagée, menée par le Pôle Frontend)

Ces tests simulent un parcours utilisateur complet à travers toute l'application, du clic dans le navigateur jusqu'à la mise à jour des données.

*   **Outils :** Cypress ou Playwright.
*   **Responsabilité principale :** Les **équipes Frontend (6, 7, 8)** écrivent et maintiennent ces tests.
*   **Scénario de test critique (à réaliser pour le Jour 4) :**
    1.  Le test crée un joueur et une partie via des appels API.
    2.  Il se connecte à l'interface en tant que ce joueur.
    3.  Il simule un clic sur l'action "Prendre 3 bois".
    4.  Il vérifie que l'inventaire du joueur dans l'interface est bien mis à jour et que la case action est maintenant occupée.
*   **Collaboration :** La réussite de ces tests est un objectif commun. Si un test E2E échoue, les équipes Frontend, Gateway et Backend concernées doivent collaborer pour trouver et corriger le problème.

Bonne chance à toutes et à tous, et que la meilleure ferme gagne !

## 9. Motivation : L'Énergie du Hackathon

Un projet de cette intensité est un marathon, pas un sprint. Maintenir un haut niveau d'énergie et de motivation est aussi crucial que la qualité du code. Voici comment nous allons transformer ce défi en une expérience stimulante et positive.

### Discours de Lancement (Kick-off)

> Bonjour à tous et bienvenue au hackathon "Agricola Numerica" !
>
> Regardez autour de vous. Aujourd'hui, dans cette salle, nous n'avons pas des étudiants. Nous avons 74 ingénieurs, architectes et chefs de produit. À partir de maintenant, et pour les 5 prochains jours, vous êtes une startup. Et votre mission, si vous l'acceptez, est ambitieuse : construire une version web complète d'Agricola, en utilisant une architecture microservices de pointe.
>
> C'est un défi immense. Mais c'est précisément pour ça que c'est intéressant. Ce que vous allez faire cette semaine, ce n'est pas juste un projet académique. C'est une simulation professionnelle de haut niveau. Les compétences que vous allez maîtriser – Docker, RabbitMQ, Spring Cloud, la collaboration à grande échelle – sont exactement ce que les meilleures entreprises tech recherchent. Ce projet sur votre CV ne sera pas juste une ligne, ce sera une histoire puissante que vous pourrez raconter en entretien.
>
> Maintenant, je sais que le diagramme d'architecture peut sembler intimidant. Onze services, des dizaines de dépendances... C'est normal de ressentir un peu de vertige.
>
> Mais ne vous laissez pas paralyser. Votre guide, c'est le **backlog quotidien**. Oubliez la démo de vendredi. Aujourd'hui, votre seul et unique objectif est de poser les fondations. Créer vos projets, définir vos contrats d'API. C'est tout. Concentrez-vous sur la victoire du jour, et nous construirons ce projet brique par brique, tous ensemble.
>
> Chaque équipe est autonome, chaque rôle est crucial. Le succès ne viendra pas seulement de votre code, mais de votre communication, de votre entraide.
>
> Alors, êtes-vous prêts à construire quelque chose dont vous serez fiers ? Êtes-vous prêts à relever le défi ?
>
> Parfait. Que le hackathon "Agricola Numerica" commence !

### Créer l'Élan (Avant et au Début)

*   **Une Vision Professionnelle :** Vous n'êtes pas juste des étudiants qui codent un jeu. Pendant 5 jours, vous êtes les 74 ingénieurs d'une startup qui doit livrer un produit complexe à un client. Ce projet sur votre CV est un véritable atout qui démontre des compétences techniques et humaines rares (collaboration à grande échelle, résilience, architecture distribuée).
*   **Démystifier la Complexité :** L'architecture peut sembler intimidante, mais le **Backlog Quotidien** est votre guide. Concentrez-vous uniquement sur les objectifs du jour. Lundi, l'objectif n'est pas de finir le jeu, mais de poser les fondations. Chaque jour est une victoire atteignable.
*   **Valoriser Chaque Rôle :** Chaque rôle, du "Product Owner" au "DevOps/Ingénieur Qualité", est essentiel. L'appropriation (`ownership`) de votre domaine est la clé. Le succès de votre équipe dépend de la contribution de chacun, et le succès du projet dépend de la collaboration de toutes les équipes.

### Maintenir l'Énergie (Pendant)

*   **Célébrer les Victoires :** Un canal de chat **`#succes`** sera créé. Utilisez-le pour célébrer publiquement chaque avancée, même les plus petites : "L'API Gateway route vers le User Service !", "Notre composant React affiche les données mockées !", "On a réussi à faire communiquer le service X et Y !". Voir la progression collective est un puissant moteur.
*   **Transformer les Blocages en Entraide :** Un blocage n'est pas un échec, c'est une opportunité de collaborer. N'hésitez jamais à demander de l'aide sur les canaux dédiés. Aider une autre équipe est tout aussi valorisé que faire avancer son propre service.
*   **Gérer l'Humain :** Des développeurs fatigués ne sont pas efficaces. Des pauses régulières seront encouragées. Du café et des snacks seront disponibles pour garder les batteries chargées. L'ambiance doit rester positive et collaborative.

Ce hackathon est une simulation intense mais bienveillante. L'objectif est d'apprendre, de collaborer et de produire collectivement quelque chose dont vous serez tous fiers.

