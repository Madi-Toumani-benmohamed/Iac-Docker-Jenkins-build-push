# Pipeline CI/CD avec Jenkins, GitHub et Docker

Ce dépôt contient une application dotée d'une pipeline CI/CD complète utilisant **Jenkins**, **GitHub** et **Docker**. L'objectif est d'automatiser le processus de **build**, **test** et **déploiement** de l'application à chaque modification du code sur GitHub.

## Table des matières

- [Prérequis](#prérequis)
- [Structure du projet](#structure-du-projet)
- [Configuration de GitHub](#configuration-de-github)
- [Installation et configuration de Jenkins](#installation-et-configuration-de-jenkins)
- [Création du pipeline Jenkins](#création-du-pipeline-jenkins)
- [Intégration avec Docker](#intégration-avec-docker)
- [Automatisation des tests et du déploiement](#automatisation-des-tests-et-du-déploiement)
- [Optimisation et sécurité du pipeline](#optimisation-et-sécurité-du-pipeline)
- [Contribution](#contribution)
- [Licence](#licence)

## Prérequis

Avant de commencer, assurez-vous d'avoir installé :

- **Git** pour le contrôle de version.
- **Docker** pour la conteneurisation.
- **Jenkins** pour l'automatisation CI/CD.
- Un compte **GitHub** pour héberger le code.
- Un compte sur **Docker Hub** ou un registre Docker privé.

## Structure du projet

- **main** : Branche principale pour la production.
- **dev** : Branche de développement pour les tests.
- **Jenkinsfile** : Fichier de définition du pipeline Jenkins.
- **Dockerfile** : Fichier pour construire l'image Docker de l'application.
- **src/** : Dossier contenant le code source de l'application.
- **tests/** : Dossier contenant les tests unitaires ou d'intégration.

## Configuration de GitHub

### 1. Créer le dépôt GitHub

- Rendez-vous sur GitHub et créez un nouveau dépôt pour le projet.
- Clonez le dépôt sur votre machine locale :
  ```bash
  git clone https://github.com/votre-utilisateur/votre-depot.git
  ```
- Créez les branches `main` et `dev` :
  ```bash
  git checkout -b main
  git push origin main
  git checkout -b dev
  git push origin dev
  ```

### 2. Stratégies de gestion

- **Pull Requests** : Utilisez des pull requests pour fusionner les modifications des branches fonctionnelles vers `dev`, puis vers `main`.
- **Commits** : Faites des commits atomiques avec des messages clairs et descriptifs.
- **Merges** : Favorisez les merges après revue de code pour maintenir un historique propre.

### 3. Configurer un webhook pour Jenkins

- Allez dans **Settings > Webhooks** du dépôt GitHub.
- Cliquez sur **Add webhook** et saisissez l'URL de Jenkins : `http://votre-jenkins-url/github-webhook/`.
- Sélectionnez les événements à surveiller, comme `Push` ou `Pull Request`.

## Installation et configuration de Jenkins

### 1. Installer Jenkins

- **Sur un serveur** :
  - Suivez les instructions officielles : [Installation de Jenkins](https://www.jenkins.io/doc/book/installing/).
- **Avec Docker** :
  ```bash
  docker run -d -p 8080:8080 --name jenkins jenkins/jenkins:lts
  ```

### 2. Installer les plugins nécessaires

- **GitHub Plugin** : Pour l'intégration avec GitHub.
- **Docker Plugin** : Pour construire et déployer des images Docker.
- **Pipeline Plugin** : Pour créer des pipelines as code.

### 3. Créer un job Jenkins connecté à GitHub

- Dans Jenkins, créez un nouveau **Pipeline**.
- Sous **Pipeline script from SCM**, configurez le dépôt GitHub et spécifiez la branche `dev`.
- Activez le déclencheur **GitHub hook trigger for GITScm polling**.

## Création du pipeline Jenkins

Créez un fichier `Jenkinsfile` à la racine du projet avec le contenu suivant :

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev', url: 'https://github.com/votre-utilisateur/votre-depot.git'
            }
        }
        stage('Test') {
            steps {
                sh './run-tests.sh' // Script pour exécuter les tests
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("votre-utilisateur/votre-image:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push to Docker Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials-id') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }
    post {
        success {
            echo 'Déploiement réussi.'
        }
        failure {
            echo 'Le build ou le déploiement a échoué.'
        }
    }
}
```

**Remarque :** Remplacez `votre-utilisateur`, `votre-depot`, `votre-image` et `dockerhub-credentials-id` par vos informations.

## Intégration avec Docker

### 1. Installer Docker sur le serveur Jenkins

- Suivez les instructions : [Installation de Docker](https://docs.docker.com/engine/install/).

### 2. Créer un Dockerfile pour l'application

Exemple pour une application Node.js :

```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### 3. Configurer Jenkins pour gérer Docker

- Assurez-vous que l'utilisateur Jenkins a les permissions nécessaires pour exécuter Docker.
- Si Jenkins est dans un conteneur, exécutez-le avec l'option `--privileged` :
  ```bash
  docker run --privileged ...
  ```

## Automatisation des tests et du déploiement

### 1. Intégrer les tests dans le pipeline

- Ajoutez un script de test (`run-tests.sh`) qui exécute vos tests.
- Modifiez l'étape `Test` dans le `Jenkinsfile` pour appeler ce script.

### 2. Déployer l'application

- Utilisez `docker-compose` pour orchestrer le déploiement si plusieurs services sont impliqués.
- Exemple de `docker-compose.yml` :

  ```yaml
  version: '3'
  services:
    app:
      image: votre-utilisateur/votre-image:${BUILD_NUMBER}
      ports:
        - "80:3000"
  ```

## Optimisation et sécurité du pipeline

### 1. Optimisation des builds Docker

- **Cache Docker** : Organisez le `Dockerfile` pour maximiser l'utilisation du cache.
- **Images légères** : Utilisez des images de base légères comme `node:14-alpine`.

### 2. Sécuriser le pipeline

- **Gestion des secrets** : Utilisez le gestionnaire de **Credentials** de Jenkins pour stocker les identifiants en toute sécurité.
- **Permissions** : Limitez les accès aux jobs Jenkins et aux ressources sensibles.
- **Sécurisation des webhooks** : Configurez des webhooks sécurisés avec des jetons secrets.

### 3. Configurer les notifications

- Ajoutez des notifications par email ou via des outils comme Slack.
- Exemple pour les emails dans le `Jenkinsfile` :

  ```groovy
  post {
      success {
          mail to: 'equipe@example.com',
               subject: "Succès du build ${env.BUILD_NUMBER}",
               body: "Le build et le déploiement ont réussi."
      }
      failure {
          mail to: 'equipe@example.com',
               subject: "Échec du build ${env.BUILD_NUMBER}",
               body: "Le build ou le déploiement a échoué."
      }
  }
  ```

## Contribution

Les contributions sont les bienvenues ! Pour contribuer :

1. **Fork** le projet.
2. Créez votre branche de fonctionnalité (`git checkout -b feature/AmazingFeature`).
3. **Commitez** vos changements (`git commit -m 'Add some AmazingFeature'`).
4. **Poussez** vers la branche (`git push origin feature/AmazingFeature`).
5. Ouvrez une **Pull Request**.

## Licence

Ce projet est sous licence MIT - voir le fichier [LICENSE](LICENSE) pour plus de détails.

---

Pour toute question ou assistance supplémentaire, n'hésitez pas à ouvrir une issue ou à me contacter.
