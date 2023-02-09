# Hébergement Symfony – React

Voici un petit tuto basé sur mes maigres connaissances en déploiement. N’ayant déployé que sur Hostinger, il se peut que les étapes soient légèrement différentes sur d’autres hébergeurs.

 Pour commencer, il vous faut un hébergeur ! Ça peut être un hébergeur comme OVH, Hostinger, … ou encore vous avec un Raspberry Pi. 

 ⚠️ ⚠️ ⚠️ ⚠️

*Avant de prendre le premier hébergeur qui vous passe sous la main, renseignez-vous sur l’environnement installé.*

*Exemple : si vous utilisez aussi Node.js côté back, vous ne pourrez pas héberger votre projet sur un serveur mutualisé chez Hostinger, vous serez obligé de choisir un serveur VPS.*

 ⚠️ ⚠️ ⚠️ ⚠️

Le cas du Raspberry Pi ne sera pas évoqué dans ce tutoriel.

Maintenant que vous avez votre hébergeur, il vous faut un accès au serveur **depuis un terminal** et une **BDD**.

---

## Accéder au serveur depuis un terminal

 Chercher la clé SSH que votre hébergeur vous donne, ajouter un MDP pour cette clé, il vous sera demandé pour se connecter au serveur depuis le terminal.
 
  Exécuter cette commande : `ssh -p < votreCleSSH >`

⚠️ ⚠️ ⚠️ ⚠️

*Attention, ce n’est pas le MDP de votre compte sur l’hébergeur qu’il faut saisir, mais bien celui préalablement créé pour la clé.*

 ⚠️ ⚠️ ⚠️ ⚠️

 A partir de maintenant, vous devriez pouvoir naviguer dans les dossiers de votre serveur avec la commande `cd`.

 ---

## Import du back sur le serveur

Pour commencer, vérifier la version de php : `php -v`

Si ce n’est pas la bonne, vous pouvez surement modifier la version depuis l’interface graphique de l’hébergeur ou installer la bonne version en ligne de commande si vous avez un serveur VPS.

Maintenant que vous avez la bonne version de PHP, il faut récupérer votre code depuis la plateforme de versioning. Pour récupérer ce code, vous allez avoir besoin, encore une fois, d’une clé SSH.

### Création de la clé

`ssh-keygen -t ed25519 -C “votre-email@exemple.fr “ ` <= à changer par votre vraie adresse mail

Afficher et copier votre clé : `cat ~/.ssh/id_ed25519.pub`

Ajout de la clé sur votre plateforme de versioning (ici Github) : `Settings -> SSH and GPG keys -> News SSH key`

### Clonage du repository Git

Naviguer jusqu’à la racine du dossier portant votre nom de domaine grâce à `cd`. Si un dossier vide est présent, ne le supprimer pas tout de suite, il aura son utilité plus tard.

Exécuter : `git clone dossierBack`

Vous avez maintenant votre dossier créé par le git clone et potentiellement votre dossier vide à côté.

### Mettre à jour composer

Vérifier la version de composer installer sur votre serveur, la commande peut être exécutée n’importe où : `php composer --version`

Si la version de votre composer ne vous convient pas, placez-vous à l’**INTERIEUR** de votre dossier créé par le git clone, puis exécuter :
```shell
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```
Ces commandes permettent d’installer composer dans sa dernière version compatible avec votre version de PHP. Si vous faites un `ls`, un fichier `composer.phar` a été créé. Il doit être au même niveau que vos fichiers « bin », « migrations », …

⚠️*IMPORTANT: À partir de maintenant votre commande `php composer ` devient `php composer.phar` pour utiliser la version que vous venez d'installer et non la version installée en global sur votre serveur!*

---

##  Import des dépendances et création de la BDD

On commence par importer les dépendances : `php composer.phar update` (je vous conseil le `update` car chez certain hébergeur le `install` bug avec `composer.phar`)

Création du `.env.local`: `touch .env.local`

Ajouter toutes vos variables d’environnement nécessaires au bon fonctionnement de votre projet : `nano .env.local`

Pour la création de la BDD, si vous en avez la possibilité, je vous conseille de passer par l'interface graphique de votre hébergeur. Sinon vous devrez récupérer l’`utilisateur root` ou créer un `utilisateur sudo` pour le renseigner dans le `.env.local`.

 ⚠️ ⚠️ ⚠️ ⚠️

*Si vous avez un serveur mutualisé, n'oubliez pas l'id de votre serveur devant le nom de votre BDD et de l'admin : `idServeur_nomBDD` / `idServer_Admin`* dans le `.env.local`.

⚠️ ⚠️ ⚠️ ⚠️

Si vous n’avez pas créé votre BDD à la main depuis l’interface graphique : `php bin/console d:d:c`

Puis, **pour tout le monde**: `php bin/console d:m:m`.

On installe apache-pack : `php composer.phar symfony/apache-pack` (en cas d'erreur, vérifier que vous avez bien un dossier `public`)

Pour ceux qui ont le dossier vide, vous pouvez renommer votre dossier `public` du projet Symfony comme le dossier vide et supprimer le dossier vide. Pour les autres, garder votre dossier public.

**Pour la suite du tuto, le dossier `public` qui a été renommé sera appelé `public_html`**

*P.S: Le nom du dossier `public` n’est pas important pour symfony, ce qui compte c’est que le chemin du vendor, dans le fichier `index.php`, reste correct. Si vous déplacez le dossier public, il faut modifier le composer.json et faire un lien symbolique. Cf: [Documentation Symfony](https://symfony.com/doc/current/configuration/override_dir_structure.html#override-the-public-directory)*

---

## Ajouter le front

Pour faciliter l’organisation des dossiers/fichiers dans le dossier `public_html`, je vous conseille de rassembler toutes les ressources (css , js, images,…) du back dans un dossier assets, pour éviter de supprimer les mauvais dossiers lors de futur mise à jour du Front.

### Dans le cas ou le back se débrouille pour récupérer le code du front

Vérifier que vous avez la même version de Node.js que vos collègues: `node -v`

Si besoin, mettre à jour node: 
```shell
sudo npm cache clean -f
sudo npm install -g n
sudo n stable
```
Vérifier que vous avez la même version de yarn que vos collègues: `yarn -v`

si besoin, mettre à jour yarn: `npm install --global yarn`.

Maintenant que tout est à jour:

Récupérer le repository git : `git clone dossierFront`

Aller dans le dossier, puis: `yarn install`

Si un dossier `dist` est déjà présent, le supprimer.

Remplacer tout les endpoints `localhost` par votre nom de domaine, attention à préciser `https` si vous avez demandé le certificat à votre hébergeur. Sur vscode: `CTRL + MAJ + F` pour modifier toutes les occurences présentes dans le projet.

Exécuter: `yarn build`.

Cette commande va vous créer un dossier `dist`. Il va falloir importer le contenu de ce dossier dans votre dossier `public_html`.

Soit vous disposez d'une interface graphique et vous le faites manuellement, soit vous utilisez des lignes de commandes, mais je ne les connais pas. Vous pouvez commencer vos recherches avec `wget` et `scp`.

---

## Gestion du .htaccess

Maintenant que le back et le front sont sur le serveur, il faut configurer le .htaccess. Voici un exemple à adapter à votre projet si besoin:
```apache
# Use the front controller as index file. It serves as a fallback solution when
# every other rewrite/redirect fails (e.g. in an aliased environment without
# mod_rewrite). Additionally, this reduces the matching process for the
# start page (path "/") because otherwise Apache will apply the rewriting rules
# to each configured DirectoryIndex file (e.g. index.php, index.html, index.pl).
DirectoryIndex index.html

# By default, Apache does not evaluate symbolic links if you did not enable this
# feature in your server configuration. Uncomment the following line if you
# install assets as symlinks or if you experience problems related to symlinks
# when compiling LESS/Sass/CoffeScript assets.
# Options +FollowSymlinks

# Disabling MultiViews prevents unwanted negotiation, e.g. "/index" should not resolve
# to the front controller "/index.php" but be rewritten to "/index.php/index".
<IfModule mod_negotiation.c>
    Options -MultiViews
</IfModule>

<IfModule mod_rewrite.c>
    # This Option needs to be enabled for RewriteRule, otherwise it will show an error like
    # 'Options FollowSymLinks or SymLinksIfOwnerMatch is off which implies that RewriteRule directive is forbidden'
    Options +FollowSymlinks

    RewriteEngine On

    # Determine the RewriteBase automatically and set it as environment variable.
    # If you are using Apache aliases to do mass virtual hosting or installed the
    # project in a subdirectory, the base path will be prepended to allow proper
    # resolution of the index.php file and to redirect to the correct URI. It will
    # work in environments without path prefix as well, providing a safe, one-size
    # fits all solution. But as you do not need it in this case, you can comment
    # the following 2 lines to eliminate the overhead.
    RewriteCond %{REQUEST_URI}::$0 ^(/.+)/(.*)::\2$
    RewriteRule .* - [E=BASE:%1]

    # Sets the HTTP_AUTHORIZATION header removed by Apache
    RewriteCond %{HTTP:Authorization} .+
    RewriteRule ^ - [E=HTTP_AUTHORIZATION:%0]

    # Redirect to URI without front controller to prevent duplicate content
    # (with and without `/index.php`). Only do this redirect on the initial
    # rewrite by Apache and not on subsequent cycles. Otherwise we would get an
    # endless redirect loop (request -> rewrite to front controller ->
    # redirect -> request -> ...).
    # So in case you get a "too many redirects" error or you always get redirected
    # to the start page because your Apache does not expose the REDIRECT_STATUS
    # environment variable, you have 2 choices:
    # - disable this feature by commenting the following 2 lines or
    # - use Apache >= 2.3.9 and replace all L flags by END flags and remove the
    #   following RewriteCond (best solution)
    RewriteCond %{ENV:REDIRECT_STATUS} =""
    RewriteRule ^index\.php(?:/(.*)|$) %{ENV:BASE}/$1 [R=301,L]

    # If the requested filename exists, simply serve it.
    # We only want to let Apache serve files and not directories.
    # Rewrite all other queries to the front controller.
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-l
    RewriteRule ^ index.php [QSA,L]
    # RewriteRule ^ %{ENV:BASE}/index.php [L]
</IfModule>

<IfModule !mod_rewrite.c>
    <IfModule mod_alias.c>
        # When mod_rewrite is not available, we instruct a temporary redirect of
        # the start page to the front controller explicitly so that the website
        # and the generated links can still be used.
        RedirectMatch 307 ^/$ /index.html/
        # RedirectTemp cannot be used instead
    </IfModule>
</IfModule>
```
Si vous trouvez des abérations dans le `.htaccess`, merci de m'en informer. Je comprends globalement ce qu'il fait mais pas en détail.

A ce stade, vous devez avoir vos routes symfony qui fonctionnent et la navigation à partir de votre navbar front qui fonctionne.

Mais si vous écrivez directement une route Front dans l'url (ex : `https://domaine.com/contact`) vous allez avoir une erreur Symfony comme quoi la route n'existe pas. C'est normal, si vous tapez une URL en dure, c'est le back qui traite la demande et Symfony ne connait pas les routes du router React.

---

## Routing Symfony - React

Pour régler le problème, il faut capturer toutes les routes possibles de votre projet **SAUF** celles écritent dans Symfony (sinon votre API ne fonctionne plus) et faire une « redirection » vers React. On va donc créer une route juste pour ça:

`@Route("/{reactRouting}", name="app_react", priority="-1", defaults={"reactRouting": null}, requirements={"reactRouting"=".+"})`

- `/{reactRouting}`  couplé à `requirements={"reactRouting"=".+"}` permet de capturer tout ce qui est inscrit après le premier « / » qui suit le nom de domaine
- `priority="-1"` permet de passer la priorité de cette route en dernière pour que nos endpoints de l'API ne soit pas capturés par cette route
- `defaults={"reactRouting": null}` permet de ne pas avoir d'erreur sur la page d'accueil de votre site, car le paramètre `reactRouting` ne sera pas défini
  
Victoire ! On n'a plus d'erreurs comme quoi Symfony ne trouve pas la route. Mais on fait comment pour renvoyer vers React ?

---

## Redirection vers React

On va associer notre nouvelle route à une méthode. Cette méthode va afficher une vue twig:

```PHP
// Dans App\Controller\ReactController.php

    /**
     * @Route("/{reactRouting}", name="app_react", priority="-1", defaults={"reactRouting": null}, requirements={"reactRouting"=".+"})
     */
    public function index()
    {
        return $this->render('react/index.html.twig');
    } 

```
On importera les fichiers js et css créer par le `yarn build` dans notre vue twig et on y ajoutera `<div id="root"></div>`. Ce qui donne:
```twig
{# Dans Templates\react\index.html.twig #}

<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="stylesheet" href="{{ asset('css/fichier1.css') }}">
        <link rel="stylesheet" href="{{ asset('css/fichier2.css') }}">

        <script defer src="{{ asset('js/fichier1.js') }}"></script>
        <script defer src="{{ asset('js/fichier2.js') }}"></script>
        <script defer src="{{ asset('js/fichier3.js') }}"></script>
    </head>
    <body>
        <div id="root"></div>
    </body>
</html>
```
Maintenant tout fonctionne, le seul problème c'est que les fichiers CSS et JS générés par le `yarn build` ont des noms aléatoires. Du coup à chaque mise à jour du Front, il va falloir modifier manuellement le nom de chaque fichier importé.

---

## On optimise ?

### Faciliter le switch entre production et développement

Pour que ce soit plus facile de jongler entre `http://localhost:8000` et `https://mon-domaine.com` ainsi que `public` et `nouveauNomDePublic` lorsque l'on passe du développement à la production, on va rajouter deux variables d'environnement dans notre `.env` et `.env.local` afin de ne plus avoir à se demander "dans quels fichiers je dois me rendre pour modifier mes chemains".
```yaml
# dans le .env

# Domaine of the site, dont forget the "/" at the end
SITE_HOST=XXXX
# Name of "public" folder
PUBLIC_FOLDER_NAME=XXXX
```
```yaml
# Dans le .env.local en dev

# Domaine of the site, dont forget the "/" at the end
SITE_HOST="http://localhost:8000/"
# Name of "public" folder, surrounded by "/"
PUBLIC_FOLDER_NAME="public"
```
```yaml
# Dans le .env.local en prod

# Domaine of the site, dont forget the "/" at the end
SITE_HOST="https://mon-domaine.com/"
# Name of "public" folder, surrounded by "/"
PUBLIC_FOLDER_NAME="public_html"
```
Puis dans le `service.yaml`
```yaml
# Dans le service.yaml

parameters:
    app.site_host: '%env(SITE_HOST)%'
    app.public_folder_name: '%env(PUBLIC_FOLDER_NAME)%'

App\Controller\ReactController:
    arguments:
        $publicFolderName: '%app.public_folder_name%'
App\Service\Service1:
    arguments:
        $siteHost: '%app.site_host%'
        $publicFolderName: '%app.public_folder_name%'
App\Service\Service2:
    arguments:
        $siteHost: '%app.site_host%'
        $publicFolderName: '%app.public_folder_name%'
```
Plus qu'à récupérer les valeurs dans les `__construct()` de vos services / controllers.

### Optimisation de l'import des fichiers CSS et JS

On va pimper notre ReactController:
```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ReactController extends AbstractController
{
    private $pathPublicFolder;

    public function __construct(string  $publicFolderName)
    {
        $this->pathPublicFolder = __DIR__ . '/../../' . $publicFolderName;
    }

    /**
     * @Route("/{reactRouting}", name="app_react", priority="-1", defaults={"reactRouting": null}, requirements={"reactRouting"=".+"})
     */
    public function index()
    {
        // Get all js and css files generate by 'yarn build' 
        $fullPathJsFiles = glob($this->pathPublicFolder . '/js/*.js');
        $fullPathCssFiles = glob($this->pathPublicFolder . '/css/*.css');

        // Just take the name of the files
        $allJsFiles = [];
        $allCssFiles = [];
        foreach($fullPathJsFiles as $file){
            $allJsFiles[] = basename($file);
        }
        foreach($fullPathCssFiles as $file){
            $allCssFiles[] = basename($file);
        }

        return $this->render('react/index.html.twig', [
            'jsFiles' => $allJsFiles,
            'cssFiles' => $allCssFiles
        ]);
    }
}
```
Explication du code:
- On récupère le nom du dossier `public_html` ou `public` dans `$publicFolderName`
- On stock le chemin jusqu'au dossier  `public_html` ou `public` dans `$pathPublicFolder`
- On récupère tous les chemins des fichiers JS et CSS grâce à `*.css` et `*.js`
- On boucles sur les tableaux et on ne récupère que le nom des fichiers grâce à `basename`
- On envoie nos deux tableaux à notre vue twig.

Ici deux choix s'offre à vous. Soit vous faites un template avec toute la structure html, soit vous utilisez votre `base.html.twig`et dans ce cas il y a une petite modification de structure pour que le style de votre back-office ne fasse pas la tronche en cas de conflit de CSS.

### Tout dans un même template

```twig
{# Dans Templates\react\index.html.twig #}

<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        {% for cssFile in cssFiles %}
            <link rel="stylesheet" href="{{ asset('css/' ~ cssFile) }}">
        {% endfor %}
        {% block javascripts %}
            {% for jsFile in jsFiles %}
            <script defer src="{{ asset('js/' ~ jsFile) }}"></script>      
            {% endfor %}
        {% endblock %}
    </head>
    <body>
        <div id="root"></div>
    </body>
</html>
```

### On réutilise base.html.twig
```twig
{# Dans Template\base.html.twig #}

<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        {% block stylesheets %}
            {# stylesheets in the index.html.twig and bockoffice.html.twig files  #}
        {% endblock %}
        {% block javascripts %}
            {# scripts in the index.html.twig and bockoffice.html.twig files  #}
        {% endblock %}
    </head>
    <body>

    {% block body %}{% endblock %}

    </body>
</html>
```
```twig
{# Dans Template\backoffice.html.twig #}

{% extends 'base.html.twig' %}

{% block stylesheets %}
    {# bootstrap 5.3 css  #}
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-GLhlTQ8iRABdZLl6O3oVMWSktQOp6b7In1Zl3/Jr59b6EGGoI1aFkw7cmDA6j6gD" crossorigin="anonymous">
    <!-- Bootstrap icons 1.10.3 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.3/font/bootstrap-icons.css">
    <!-- Our custom CSS -->
    <link rel="stylesheet" href="{{ asset('assets/css/style.css') }}">
{% endblock %}

{% block javascripts %}
    {# bootstrap 5.3 js  #}
    <script defer src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js" integrity="sha384-w76AqPfDkMBDXo30jS1Sgez6pr3x5MlQ1ZAGC+nuZB+EYdgRZgiwxhTBTkF7CXvN" crossorigin="anonymous"></script>
{% endblock %}

{% block body %}
    {# Nav container #}
    {% include 'layout/nav.html.twig' %}

    {# Main container #}
    {% block main %}{% endblock %}
{% endblock %}
```
```twig
{# Dans Templates\react\index.html.twig #}

{% extends 'base.html.twig' %}

{% block stylesheets %}
    {% for cssFile in cssFiles %}
        <link rel="stylesheet" href="{{ asset('css/' ~ cssFile) }}">
    {% endfor %}
{% endblock %}

{% block javascripts %}
    {% for jsFile in jsFiles %}
    <script defer src="{{ asset('js/' ~ jsFile) }}"></script>      
    {% endfor %}
{% endblock %}

{% block title %}Votre titre{% endblock %}

{% block body %}
    <div id="root"></div>
{% endblock %}
```
Le tuto touche à sa fin, j'espère qu'il pourra vous aider lors du déploiement de votre projet!
