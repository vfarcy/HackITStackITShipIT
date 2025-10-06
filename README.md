# Hackathon "Agricola Numerica" - Document de Référence Complet

Bienvenue au hackathon "Agricola Numerica" ! Ce document est la source unique de vérité pour l'organisation, l'architecture, les objectifs, les rôles et les modalités d'évaluation de notre projet.

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

!Diagramme d'architecture
*(Note : Ce diagramme est une représentation conceptuelle de l'architecture cible)*

## 2. Répartition des Équipes par Domaine

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

## 3. Organisation Interne des Équipes

Chaque équipe s'organisera en "Squad Agile" avec les rôles suivants pour maximiser l'efficacité :

*   **Product Owner / Chef de Projet (1) :** Gardien de la vision fonctionnelle, expert des règles, gestion du backlog de l'équipe.
*   **Tech Lead / Architecte (1) :** Gardien de la vision technique, responsable de la qualité du code et des choix de conception internes.
*   **Squad Backend (2-3) :** Développe la logique métier et les API du service.
*   **Squad Frontend (2-3) :** (Pour les équipes du pôle Interface) Développe les composants visuels et les interactions.
*   **DevOps / Ingénieur Qualité (1) :** Responsable du `Dockerfile` du service et de son intégration dans le `docker-compose` global.

## 4. Organisation & Communication du Hackathon

Pour assurer une collaboration fluide entre 74 personnes, nous suivrons des rituels agiles :

*   **Scrum des Scrums :** Chaque matin à 9h15, un représentant technique de chaque équipe se réunit avec les organisateurs pour synchroniser, identifier les dépendances et lever les blocages.
*   **Contrats d'API :** La communication entre les services se base sur des contrats formels définis avec **OpenAPI (Swagger)**. L'équipe "Architectes" est la gardienne de la documentation d'API centrale. Aucune intégration ne doit commencer sans un contrat validé.
*   **Outils :** La communication se fera sur une plateforme de chat (Slack/Teams) avec des canaux dédiés (`#api-contracts`, `#frontend-backend-sync`, `#deployment-docker`, etc.).

## 5. Backlog Quotidien (Objectifs Généraux)

*   **Jour 1 (Lundi) : Fondations & Contrats.** Mise en place de l'infrastructure de base (`docker-compose`, services vides), modélisation des données et définition des contrats d'API.
*   **Jour 2 (Mardi) : Le "Hello World" de Bout en Bout.** Un utilisateur doit pouvoir se créer, créer une partie et la rejoindre. L'architecture de base doit être prouvée fonctionnelle.
*   **Jour 3 (Mercredi) : La Première Action de Jeu.** Un joueur doit pouvoir effectuer une action simple (ex: "Prendre 3 bois") et la mise à jour doit être visible par tous en temps réel. **Checkpoint à mi-parcours.**
*   **Jour 4 (Jeudi) : Montée en Puissance.** Implémentation des fonctionnalités complexes (phase de Récolte, construction de clôtures, etc.).
*   **Jour 5 (Vendredi) : Finalisation & Démos.** Gel des fonctionnalités, chasse aux bugs, préparation et passage des démonstrations finales.

## 6. Ressources Techniques (Backend)

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

## 7. Ressources Humaines (Encadrement)

Une équipe de mentors sera présente pour vous guider :

*   **Coordinateur Principal (1) :** Chef d'orchestre de l'événement.
*   **Architecte en Chef (1-2) :** Référence technique suprême, anime le Scrum des Scrums.
*   **Mentors Spécialisés (3-4) :** Experts en Backend, Frontend, et DevOps/Infra pour vous aider au quotidien.
*   **Game Master (1) :** L'expert fonctionnel des règles d'Agricola.

## 8. Évaluation

L'évaluation portera sur le processus autant que sur le résultat final.

1.  **Évaluation Continue :** Les mentors observeront la proactivité, la communication et la collaboration tout au long de la semaine.
2.  **Checkpoint (Mercredi) :** Une mini-démo par pôle pour vérifier l'état de l'intégration.
3.  **Évaluation Finale (Vendredi) :** La note sera une pondération de :
    *   **Démo Finale (40%) :** Le produit final sera jugé sur sa complétude, sa robustesse et la qualité de la présentation.
    *   **Évaluation par les Pairs (30%) :** Chaque étudiant évaluera anonymement la contribution des membres de son équipe (technique, collaboration, fiabilité).
    *   **Évaluation par les Mentors (30%) :** Les encadrants évalueront la performance individuelle observée durant le hackathon (apprentissage, curiosité, esprit d'équipe).

## 9. Stratégie de Test et Qualité

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