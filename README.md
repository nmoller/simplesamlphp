# Utilisation de traefik pour simplesamlphp
Setup from https://blog.kilian.io/server-setup/

Créer réseau:
```
docker network create web
```

Démarrer traefik:
```
cd traefik
docker-compose up -d
```

### Simplesamphp

https://gist.github.com/pradtke/a63e843a568b9fa4b956668d0b3c0447

```
cd apache
# Si première fois
git clone git@github.com:simplesamlphp/simplesamlphp.git
# travailler avec dernier release
cd simplesamlphp
git branch -a
git checkout origin/simplesamlphp-1.17
```

Traitement composer:
```
# dans dossier clone de simplesamlphp (apache/simplesamlphp)
docker run -it --rm -u $(id -u):$(id -g) \
-e COMPOSER_HOME=/opt/simplesamlphp/.composer \
-v $(pwd):/opt/simplesamlphp \
-w /opt/simplesamlphp \
nmolleruq/phpcomposer:7.2 composer install --no-dev
```

Modifier `/etc/hosts` (de manière consistente avec label dans `apache/docker-compose.yml traefik.frontend.rule`:
```
echo '127.0.0.1   idp.nmoller.io' >> /etc/hosts
```

Partir apache:
```
cd apache
docker-compose up -d
```

S'assurer que ça fonctionne:

http://idp.nmoller.io/simplesaml

Si cela fonctionne, on configure `simplesamlphp`:
```
apache/simplesamlphp ((05e9943a...))$ cp config-templates/*.php config

# on suit instructions de :
# https://gist.github.com/pradtke/a63e843a568b9fa4b956668d0b3c0447
apache/simplesamlphp ((05e9943a...))$ mkdir cert
apache/simplesamlphp ((05e9943a...))$ openssl req -newkey rsa:2048 -new -x509 -days 3652 -nodes -out cert/saml.crt -keyout cert/saml.pem
```