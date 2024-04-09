# Webservice Mail de l'Abes

Configuration docker pour déployer le web service d'envoi de mail utilisé en interne à l'Abes par nos applications. Le but est de simplifier l'envoi des mails depuis les applications, et de centraliser la configuration à un endroit unique.

[![Docker Pulls](https://img.shields.io/docker/pulls/abesesr/wsmail.svg)](https://hub.docker.com/r/abesesr/wsmail/)

## URLs du WS de mail Abes 
- dev : https://apicom-dev.sudoc.fr/wsmail/
- test : https://apicom-test.sudoc.fr/wsmail/
- prod : https://apicom.sudoc.fr/wsmail/

## Prérequis

- docker
- docker compose
- réglages ``vm.max_map_count`` pour elasticsearch (cf [FAQ pour les détails du réglage](README-faq.md#comment-r%C3%A9gler-vmmax_map_count-pour-elasticsearch-))

## Installation

On commence par récupérer la configuration du déploiement depuis le github :
```bash
cd /opt/pod/ # à adapter en local car vous pouvez cloner le dépàt dans votre homedir
git clone https://github.com/abes-esr/ws-mail-docker.git
```

Ensuite on configure notre déploiement en prenant exemple sur le fichier [``.env-dist``](https://github.com/abes-esr/ws-mail-docker/blob/develop/.env-dist) qui contient toutes les variables utilisables avec les explications :
```bash
cd /opt/pod/ws-mail-docker/
cp .env-dist .env
# personnalisez alors le .env en partant des valeurs exemple présentes dans le .env-dist
# il faut au minimum renseigner les variables commençant par WS_SPRING_MAIL... les autres sont facultatives
```

Finalement on peut démarrer l'application :
```bash
cd /opt/pod/ws-mail-docker/
docker compose up -d
```

A partir de là, l'application tournera sur : http://localhost:8080/


## Démarrage et arrêt

Pour démarrer l'application :
```bash
cd /opt/pod/ws-mail-docker/
docker compose up
# ajouter -d si vous souhaitez démarrer l'application en tache de fond
# dans le cas contraire, utilisez CTRL+C pour ensuite quitter l'application
```

## Déploiement continu

Les objectifs des déploiements continus de theses-docker sont les suivants (cf [poldev](https://github.com/abes-esr/abes-politique-developpement/blob/main/01-Gestion%20du%20code%20source.md#utilisation-des-branches)) :
- git push sur la branche ``develop`` provoque un déploiement automatique sur le serveur ``diplotaxis2-dev``
- git push (le plus couramment merge) sur la branche ``main`` provoque un déploiement automatique sur le serveur ``diplotaxis2-test``
- git tag X.X.X (associé à une release) sur la branche ``main`` permet un déploiement (non automatique) sur le serveur ``diplotaxis2-prod``

Le déploiement automatiquement de ``theses-docker`` utilise l'outil [watchtower](https://containrrr.dev/watchtower/). Pour permettre ce déploiement automatique avec watchtower, il suffit de positionner à ``false`` la variable suivante dans le fichier ``/opt/pod/ws-mail-docker/.env`` :
```env
WSMAIL_WATCHTOWER_RUN_ONCE=false
```
Le fait de passer ``THESES_WATCHTOWER_RUN_ONCE`` à ``false`` va faire en sorte d'exécuter périodiquement watchtower. Par défaut cette variable est à ``true`` car ce n'est pas utile voir cela peut générer du bruit dans le cas d'un déploiement sur un PC en local.

## Architecture

Le WS est une application relativement simple, écrite en Java 21 (LTS) et Spring Boot 3.
La documentation est publiée à l'aide de Spring Docs (OpenAPI 3).
Le code est testé à chaque build à l'aide de JUnit.
