# Installation de Nginx, de Php-fpm et Mysql sur macOSx

## Sommaire
- [Installation de Nginx, de Php-fpm et Mysql sur macOSx](#installation-de-nginx-de-php-fpm-et-mysql-sur-macosx)
  - [Sommaire](#sommaire)
  - [Introduction](#introduction)
  - [Installation homebrew](#installation-homebrew)
  - [Installation php-fpm](#installation-php-fpm)
  - [Installation nginx](#installation-nginx)
    - [Configuration nginx](#configuration-nginx)
      - [Dossier de travail](#dossier-de-travail)
      - [Nginx.conf](#nginxconf)
      - [Création des dossiers de conf](#création-des-dossiers-de-conf)
  - [Installation serveur Mysql](#installation-serveur-mysql)
- [Sequel Pro](#sequel-pro)
  - [Configuration Sequel Pro](#configuration-sequel-pro)
  - [Aller plus loin](#aller-plus-loin)
    - [Configuration nginx avec nom de domaine](#configuration-nginx-avec-nom-de-domaine)
    - [Configuration fichier hosts](#configuration-fichier-hosts)
    - [SSL](#ssl)
      - [Certificat](#certificat)
      - [Snippets](#snippets)
      - [Audrey.ooo.conf](#audreyoooconf)

## Introduction
Par défaut macOSx à un serveur non actif Apache pré-installé ainsi qu'un serveur php (il n'y a pas de serveur sql de pré-installé). Si vous êtes comme moi, je préfère de loin un serveur Nginx. De plus les versions sont pas forcément à jour sur votre mac. Autant de raisons qui font que ne nous voulons pas les activer.<br />
Grâce à **Homebrew** nous pouvons installer un serveur **Nginx**, serveur **Php-fpm** et serveur **Mysql** à jour et facile à maintenir.


## Installation homebrew
Rendez-vous sur [https://brew.sh/index_fr](https://brew.sh/index_fr).<br />
Ou pour les fénéants ouvrez votre terminal et coller cette commande :
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

brew install wget
```
Pourquoi WGET... ? Je ne fais que suivre le tutoriel officiel d'installation de homebrew. :relaxed: <br /><br />
**Bon à savoir** : Si vous êtes plus à l'aise avec le terminal, les installations de Nginx ou Php se retrouverons par défault dans le répertoire : `/opt/homebrew/etc/` :kissing_heart:

## Installation php-fpm
Maintenant que nous avons installé homebrew, entrons dans le vif du sujet.
J'ai besoin d'installer une version la plus à jour possible. <br />
> Normal !! Me diriez-vous.

Ok ! Très bien ! comment trouve-t-on cette version à jour. Si ca se trouve, il en existe plusieurs versions plus ou moins récentes. Comment les connaître ?

```
brew search php
```
Vous y verrez afficher en résultat une liste de ce type :
```
==> Formulae
brew-php-switcher          php-cs-fixer               phplint                phpstan
php                        php@7.2                    phpmd                  phpunit
php-code-sniffer           php@7.3                    phpmyadmin
```
Nous pouvons voir que la version la plus récente est la version **php@7.3**. Il nous reste plus qu'à l'installer.<br /><br />
**Bon à savoir** : Même si php-fpm n'est pas affiché distinctement la version de php@7.3 est bien une **fpm**.
```
brew install php@7.3
```

Félicitation votre serveur **Php** est maintenant installé et à jour. :relaxed:

## Installation nginx
Attaquons-nous à Nginx. Pas besoin de faire de recherche car il n'y a qu'une seule version maintenue à jour.<br />
Copiez la commande suivante :
```
brew install nginx
```
Félicitation votre serveur **Nginx** est maintenant installé et à jour. :relaxed:

### Configuration nginx
Voici la partie la plus délicate. Rien de bien compliqué en revanche aller trop vite peut créer des problèmes. Donc soyez encore plus sérieux dans cette partie.

> Vous avez la pression hein ! :scream: :sob:

J'aime reprendre les bests practices de linux. Et comme par hasard sur macOSx c'est presque pareil...enfin presque...

#### Dossier de travail
Tout d'abord nous allons créer un répertoire de travail pour le développement. Tout vos sites internets seront dans ce répertoire et facile à trouver.

Nous allons créer un dossier dans votre répertoire utilisateur. (la nouille)
```
cd ~/
mkdir -p Sites
```

Puis créer un fichier php dans ce dossier :
```
cd Sites/
nano index.php
```
Ajoutez cette ligne suivante puis **ctrl + x** pour quitter et **Y** ou **O** pour enregistrer avant de quitter :
```
<?php phpinfo(); ?>
```
Voici un aperçu de ce que vous devez avoir avec **Finder**.

![Aperçu Finder](/images/apercu-finder-site.png)

Félicitation votre nouveau répertoire de déveleppement est près. :relaxed:

#### Nginx.conf
Maintenant que nous avons configuré notre nouveau répertoire de développement, continuons à configurer Nginx. Nous allons créer le répertoire de logs, puis dupliquer le fichier nginx.conf pour en faire une sauvergarde.
```
cd /opt/homebrew/etc/nginx/
mkdir logs
mv nginx.conf nginx.conf.bak
nano nginx.conf
```
Insérez les lignes du fichiers [nginx.conf](nginx.conf) ou copiez le fichier directement dans le répertoire **nginx/**, c'est comme vous le souhaitez.

```
user                    <user> staff;
worker_processes        1;

error_log               /opt/homebrew/etc/nginx/logs/error.log;
# error_log             /opt/homebrew/etc/nginx/logs/error.log;  notice;
# error_log             /opt/homebrew/etc/nginx/logs/error.log;  info;

events {
    worker_connections  1024;
}

http {
    include             mime.types;
    default_type        application/octet-stream;
    sendfile            on;
    keepalive_timeout   65;
    gzip                on;
    include             sites-enabled/*.conf;
}
```

**Bon à savoir** : \<user> dois être remplacé pour votre nom. Pour savoir ou est votre nom ouvrez votre terminal et observé le début de ligne.<br />

Juste avant que vous écrivez vos commandes. Par exemple pour moi c'est :
```
 geekoun@MacBook-Pro-de-geekoun  ~
 ```
 Le nom juste avant le arobase est celui que vous devez insérer. :wink:

#### Création des dossiers de conf
Je n'aime pas avoir un fichier extrêment long à lire. Je préfère en avoir plusieurs, classés dans des dossiers et reliés entre eux.
Si vous avez été attentif la dernière ligne du fichier **nginx.conf** est un **include** et il inclus dans un répertoire qui n'existe pas encore. Nous allons tous créer maintenant. :punch:

```
cd /opt/homebrew/etc/nginx/
rm -r servers/
mkdir sites-enabled
```
**mkdir** signifie créer un dossier/répertoire. La commande ci-dessus permet de créer 2 dossiers **sites-enabled** et **sites-available**.
Nou allons créer nos autres fichiers de conf dans le dossier **sites-enabled**.

La preuve par l'exemple sera plus pertinente.

```
cd sites-enabled
nano default.conf
```
Insérez les lignes du fichiers [default.conf](sites-available/default.conf) ou copiez le fichier directement dans le répertoire **sites-enabled/**, c'est comme vous le souhaitez.
```
server {
    listen       80;
    server_name  localhost;
    root         /Users/<user>/Sites/;
    index        index.php index.html index.htm;

    location ~ \.php {
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
    }
}

```
Vous voyez la ligne ou il y a écrit **root** ? Eh ben cette ligne sert à désigner le nouveau répertoire de travail que nous avons créer au tout début du tutoriel.<br /><br />
**Bon à savoir** : <user> dois être remplacé pour votre nom.
<br /><br />
Nous allons enfin créer le lien dans le dossier **sites-enabled/** :
```
ln -s default.conf ../sites-enabled
```
Vous devriez voir ceci dans votre dossier **sites-enabled/** :
```
cd ../sites-enabled
ls -la
lrwxr-xr-x   1 <user>  admin   31  5 jan 12:40 default.conf -> ../sites-available/default.conf
```
Vous voyez cette petite flèche. Elle indique que vous avez créer un lien. <br /><br />
Nous y sommes presque pour une configuration de base. Il va falloir relancer **Nginx** pour appliquer les nouveaux paramètres. <br />
Comme ceci :
```
brew services restart nginx
```
Féléciation !! Vous pouvez dès à présent lancer votre navigateur préféré safari, Chrome, Firefox, etc. et écrire dans votre barre d'adresse **[http://localhost](http://localhost)** :relaxed:

![Aperçu localhost](/images/localhost.png)

**Bon à savoir** : Pensez à vider le cache de votre navigateur avant. Et si ca ne fonctionne pas redémarrez votre mac. Parfois il en a besoin !

## Installation serveur Mysql
Je ne suis pas pour un phpmyadmin. Ca demande trop de ressources inutiles juste pour avoir accès à une base de donnée.
Pour ma part, j'utilise un serveur **Mysql** et **Sequel Pro** pour y accèder.<br />

Je le télécharge depuis le site officiel sur ce [lien](https://dev.mysql.com/downloads/mysql/). Prenez la version **dmg** se sera plus simple.
Ouvrez votre fichier **dmg**.

![Ouverture du dmg](/images/ouverture-install-mysql.png)

Il y a un petit avertissement. Cliquez sur **Continuer**.

![Avertissement installation de Mysql](/images/avertissement-install-mysql.png)

Acceptez la **licence**, la **destination** jusqu'à **type d'installation**. A cette étape si vous cliquez sur le bouton **Personnaliser** vous pourrez voir ce qui sera installé.

![Personnalisation de l'installation de Mysql](/images/personnaliser-installation-mysql.png)

Continuer en en cliquant sur **Installer**.
Une fois l'installation terminée, vous devez générer un mot de passe pour vous connectez à la base de donnée. Faites attention à bien cocher le bouton : **Use Legacy Password Encryption** !!<br /><br />
Identifiant par défaut : **root**<br />
Mot de passe : **votre mot de passe**
<br /><br />
**Bon à savoir** : Pour avoir plus d'infos suivre les instructions sur ce [lien](https://dev.mysql.com/doc/mysql-osx-excerpt/5.7/en/preface.html)<br />

Voici ce que vous devez voir après l'installation.

![Aperçu préférence système](/images/apercu-preference-systeme.png)

![Mysql](/images/mysql.png)

**Bon à savoir** :fist: : Vous pouvez réinitialiser **Mysql**. Ce qui aura pour effet de remettre à zéro et aussi de changer de mot de passe en cliquant sur le bouton : **Initialise Database**.<br />

Félicitation !! Vous venez de faire une installation rapide et simple d'un serveur **Mysql**. :relaxed:

# Sequel Pro
Rendez-vous sur le site officiel et [téléchargez-le](https://sequelpro.com/download).
Une fois téléchargé, copiez-le dans **Applications**.

## Configuration Sequel Pro
Ouvrez-le ensuite et configurer-le comme ceci :

![Sequel Pro Configuration](/images/sequel-pro-conf.png)
1. Cliquez sur le petit bouton **+**
2. Entrez un nom par exemple : **Local**
3. Insérez les informations de connexion à votre base de donnée. Le mot de passe est celui que vous avez entré pendant l'installation de **Mysql**.

![Aperçu de la base de donnée](/images/apercu-de-la-bdd.png)

Félicitation !! Vous venez de faire une installation rapide et simple d'un interpreteur pour lire vos bases de données en 2 temps 3 mouvements. :relaxed:

## Aller plus loin
Ok ! Je sent qu'il y a des motivés. :stuck_out_tongue_winking_eye:<br />
Je vais donc  vous proposez de configurer des noms de domaines supplémentaires ainsi que de les faire pointer dans le dossier que vous désirez. Et en tout dernier on fera une configuration pour mettre en https. Car si ces messieurs codeurs font développement e-shop par exemple il leur faudra un certificat pour faire des testes. :stuck_out_tongue_winking_eye:

### Configuration nginx avec nom de domaine
Vous vous souvenez du fameux dossier **sites-available/** ? Sinon je vous invite à retrouver ce cours plus [haut](#création-des-dossiers-de-conf).
Donc rendez-vous dans le dossier **sites-available/**. Nous allons dupliquer le fichier **default.conf**, le modifier et créer un lien vers le dossier **sites-enabled/**.<br />

Et comme vous êtes des fous pour être arrivé si loin :metal:. Je vais utiliser mes [alias](https://github.com/geekoun/Installation-Nginx-php-fpm-sur-macOS/blob/master/.bash_profile) :fire:

```
..nginx
cd sites-available/
cp default.conf www.audrey.ooo
ln -s www.audrey.ooo ../sites-enabled/
nano www.audrey.ooo
```
Voici à quoi doit ressembler après modification le fichier.
```
server {
    listen       80;
    server_name  audrey.ooo;
    root         /Users/<user>/Sites/www.audrey.ooo/;
    index        index.php index.html index.htm;

    location ~ \.php {
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
    }
}
```
**Bon à savoir** : N'oubliez pas de modifier le \<user\>... je dis ça... je dis rien... :metal: :fire: :sunglasses:<br /><br />

> Vas y quoi ! T'as oublié de créer le dossier dans Sites/ !!

Ahh oui pas faux ! Nous allons procéder comme suit :
```
..~
cd Sites/
mkdir -p www.audrey.ooo
cp index.php www.audrey.ooo/
```

Il ne reste plus qu'a recharger le **Nginx** :
```
brew services restart nginx
```

### Configuration fichier hosts
Bon ben il reste encore à modifier notre fichier **hosts** pour faire pointer notre nom de domain (www.audrey.ooo) vers notre macOSx.

> Mais What !!!

Ben oui par défaut http://localhost pointe automatiquement. Et nous allons devoir aider un peut pour que ca pointe correctement sur notre nouveau nom de doamine.<br /> Comme ceci :
```
hosts
```
Voici ce que le nano affiche après ajout de la ligne magique :
```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost

127.0.0.1 audrey.ooo
```

### SSL
Aller hop dernière petite étape et non des moindres...

> Pfff ils sont fous ces romains !

Pour ce faire nous allons créer 2 dossiers supplémentaires dans **Nginx**. Un qui sera nommé **snippet** l'autre pour générer le certificat ssl auto-signé, nommé **ssl**. hop hop !!
```
..nginx
mkdir -p {snippets,ssl}
```

#### Certificat
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ssl/nginx-selfsigned.www.audrey.ooo.key -out ssl/nginx-selfsigned.www.audrey.ooo.crt
openssl dhparam -out ssl/dhparam.www.audrey.ooo.pem 2048
```

#### Snippets
Nous allons ensuite créer nos snippets et les intégrer en dernière partie dans **toutouyoutou.conf**
```
cd snippets/
nano /etc/nginx/snippets/self-signed.www.audrey.ooo.conf
```

Ajoutez ceci ou copiez le [fichier](snippets/self-signed.www.audrey.ooo.conf) c'est comme vous le souhaitez :
```
ssl_certificate /usr/local/etc/nginx/ssl/nginx-selfsigned.www.audrey.ooo.crt;
ssl_certificate_key /usr/local/etc/nginx/ssl/nginx-selfsigned.www.audrey.ooo.key;
```
Nous allons créer le snippet pour le fichier **pem** :
```
nano /etc/nginx/snippets/ssl-params.www.audrey.ooo.conf
```
ET ajoutez ceci ou copiez le [fichier](snippets/ssl-params.www.audrey.ooo.conf) c'est comme vous le souhaitez :
```
ssl_session_cache    shared:SSL:1m;
ssl_session_timeout  5m;

ssl_ciphers  HIGH:!aNULL:!MD5;
ssl_prefer_server_ciphers  on;
ssl_dhparam /usr/local/etc/nginx/ssl/dhparam.www.audrey.ooo.pem;
```
Vou êtes maintenant prêt à passer à la suite. :+1:

#### Audrey.ooo.conf
Enfin notre dernière étape. Je suis déjà trsite à l'idée de vous quittez. Mais que voulez-vous les bonnes chose ont une fin :disappointed_relieved: :relaxed:<br />

Allons modifier notre fichier **www.audrey.ooo.conf** ou copiez le fichier directement dans le répertoire comme vous le souhaitez.
En prime nous allons également forcer la redirection du port 80 (http) vers le 443 (https) :
```
server {
    listen 80;
    server_name audrey.ooo;
    return 301 https://$server_name$request_uri;
}

server {
    listen       443 ssl http2;
    server_name  audrey.ooo;
    root         /Users/<user>/Sites/www.audrey.ooo/;
    index        index.php index.html index.htm;

    include /usr/local/etc/nginx/snippets/self-signed.www.audrey.ooo.conf;
    include /usr/local/etc/nginx/snippets/ssl-params.www.audrey.ooo.conf;

    location ~ \.php {
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
    }
}
```
Il ne reste plus qu'a recharger le **Nginx** :
```
brew services restart nginx
```

ETTTTT tadammm :

![https://audrey.ooo](/images/audrey.ooo.png)

Cliquez sur : **Continuer vers le site audrey.ooo (dangereux)**. Comme le certificat n'est pas valide le navigateur affichera un message d'alerte. Vous pouvez enregistrer le certificat sur votre mac et le valider pour éviter d'autre soucis. :relaxed:

![https://audrey.ooo](/images/apercu-audrey.ooo.png)

**Bon à savoir** : Pensez à vider le cache de votre navigateur avant. Et si ca ne fonctionne pas redémarrez votre mac. Parfois il en a besoin !

Thx !
