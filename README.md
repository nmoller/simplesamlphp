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


### SimpleSamlPhp config
`authsources.php` :

```
/*
Sans ça id du service n'est pas défini... Essai d'abord dans 'example-userpass'
*/
'SimpleSAML\Module\exampleautth\Auth\Sourc\External.AuthId' => [
        'exampleauth:External',
],

'example-userpass' => [
    'exampleauth:UserPass',

    // Give the user an option to save their username for future login attempts
    // And when enabled, what should the default be, to save the username or not
    //'remember.username.enabled' => false,
    //'remember.username.checked' => false,
    
    'student:studentpass' => [
        'uid' => ['test'],
        'eduPersonAffiliation' => ['member', 'student'],
        'name' => ['Student Name'],
        'mail' => ['somestudent@example.org'],
        'type' => ['student'],
    ],
    'employee:employeepass' => [
        'uid' => ['employee'],
        'eduPersonAffiliation' => ['member', 'employee'],
    ],
    
],
```

Pour configurer idp:
`metadata/saml20-idp-hosted.php`:
```
<?php
$metadata['__DYNAMIC:1__'] = [
    /*
     * The hostname for this IdP. This makes it possible to run multiple
     * IdPs from the same configuration. '__DEFAULT__' means that this one
     * should be used by default.
     */
    'host' => '__DEFAULT__',

    /*
     * The private key and certificate to use when signing responses.
     * These are stored in the cert-directory.
     */
    'privatekey' => 'saml.pem',
    'certificate' => 'saml.crt',

    /*
     * The authentication source which should be used to authenticate the
     * user. This must match one of the entries in config/authsources.php.
     */
    'auth' => 'example-userpass',
    // 'auth' => 'SimpleSAML\Module\exampleautth\Auth\Sourc\External.AuthId'
];
```

### Installer auth_saml2

Cloner moodle, cloner configphp (checkout branche k8s, commenter `$CFG->sslproxy = true;`)

Dossier clone moodle
```
cd auth
git@github.com:catalyst/moodle-auth_saml2.git saml2
```
Installer et configurer pour que ça marche avec simplesaml.