---
title: "Comment j'ai modernisé mon blog en migrant de Wordpress à Hugo"
date: 2018-03-07T22:31:08+01:00
draft: false
author: VaLouille
type: post
categories:
  - Hugo
tags:
  - hugo
  - gitlab
  - gitlab-ci
  - wordpress
  - disqus
  - pages
  - migration
  - CI
  - site
  - statique
  - blog
---

J'ai eu envie de donner un coup de jeune à mon blog, dont le design commençait à être relativement ancien, et qui ne s'affichait pas toujours correctement sur mobile et tablette. De plus, Wordpress est un bon moteur de blog, mais la gestion des thèmes, des plugins, les mises à jour fréquentes, et le fait qu'une stack LAMP complète soit nécessaire ne me convenait plus. J'ai donc décidé de changer de moteur de blog, pour utiliser un générateur de sites statiques : [Hugo][1].

# Hugo

Hugo est installé en local sur mon ordinateur, et permet de générer des fichiers HTML à partir d'une arborescence de fichiers (configuration, images, posts, thème). Il suffit ensuite de placer les fichiers HTML générés derrière un serveur web quelconque, ou même directement sur sur stockage objet de type "S3". Les articles sont écrits au [format Markdown][5], et sont donc réutilisables dans d'autres moteurs de blog.

Il existe d'autres générateurs de sites statiques (Hexo, Jekyll, Ghost, Middleman ...) mais Hugo, en plus d'avoir un communauté active et grandissante, présente l'avantage d'être codé en Go et ne demande donc pas de dépendance particulière pour fonctionner. L'installation est donc très aisée. Sous macOS, il suffit de taper la commande suivante :

```
valouille@Valintosh ~ » brew install hugo
==> Downloading https://homebrew.bintray.com/bottles/hugo-0.37.high_sierra.bottle.tar.gz
==> Downloading from https://akamai.bintray.com/18/184e4bb5606d61dd91de7c88562035a05f8b84b9f08c6b69458e90ad7cdafea3?__gda__=exp=1520288059~hmac=54a5eddcd11a6d86a0524ef85bf1f6097a0cf09c2b7eb05d103e068c1d
######################################################################## 100.0%
==> Pouring hugo-0.37.high_sierra.bottle.tar.gz
==> Caveats
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d
==> Summary
🍺  /usr/local/Cellar/hugo/0.37: 32 files, 27.5MB
```

Ensuite, on crée la structure minimale en utilisant la commande suivante :

```
valouille@Valintosh ~ » hugo new site hugo
Congratulations! Your new Hugo site is created in /Users/valouille/hugo.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
```

Le site a été créé dans le répertoire hugo. On peut se placer dedans :

```
valouille@Valintosh ~ » cd hugo
```

Par défaut, Hugo n'embarque pas de thème. Il faut donc sur rendre à [l'adresse suivante][2] et en choisir un. J'ai choisi le thème ["Even"][3] que je trouve simple et élégant. Un fois le thème choisi, on se place dans le répertoire `themes` et on clone le dépôt :

```
valouille@Valintosh ~/hugo » cd themes
valouille@Valintosh ~/hugo/themes » git clone https://github.com/olOwOlo/hugo-theme-even.git even
Cloning into 'even'...
remote: Counting objects: 891, done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 891 (delta 1), reused 6 (delta 1), pack-reused 871
Receiving objects: 100% (891/891), 1.14 MiB | 1.72 MiB/s, done.
Resolving deltas: 100% (496/496), done.
```

> L'installation du thème peut légèrement varier d'un thème à l'autre, il est nécessaire de lire le fichier README.md fourni.

Dans le cas du thème Even, il est nécessaire de copier le fichier `exampleSite/config.toml` à la racine du site (à la place de celui par défaut), pour ensuite le modifier :

```
valouille@Valintosh ~/hugo » cp themes/even/exampleSite/config.toml ~/hugo/config.toml
```

Ensuite, on peut visualiser le site dans son navigateur à l'adresse `http://localhost:1313` en lançant la commande suivante :

```
valouille@Valintosh ~/hugo » hugo server

                   | EN
+------------------+----+
  Pages            |  8
  Paginator pages  |  0
  Non-page files   |  0
  Static files     | 32
  Processed images |  0
  Aliases          |  1
  Sitemaps         |  1
  Cleaned          |  0

Total in 77 ms
Watching for changes in /Users/valouille/hugo/{content,data,layouts,static,themes}
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

{{% figure class="center" src="/images/hugo_even.png" alt="Site Hugo avec le thème Even" title="Site Hugo avec le thème Even" %}}

Ce mode lance un serveur web en local, et est pratique pour tester des modifications ou visualiser le rendu avant de le publier. On peut également ajouter l'option `-D` pour visualiser les posts taggués "draft".

Pour personaliser le blog, il est nécessaire de modifier le fichier `config.toml`.

# Import à partir de Wordpress

Possédant déjà un blog technique depuis 2013, comprenant 70 articles, je ne voulais pas perdre ce contenu, et devais donc l'importer de Wordpress vers Hugo. J'ai également souhaité importer les commentaires.

## Articles

Il existe un [plugin Wordpress][4] qui permet d'exporter le contenu de son site Wordpress en Markdown, qu'on peut ensuite réutiliser tel quel dans Hugo. Ce plugin prend également en compte les catégories, les tags, les URLs, les images ...

Son installation s'effectue comme n'importe quel plugin Wordpress :
* Télécharger le [fichier ZIP du plugin][6]
* Placer le plugin dans le répertoire `/wp-content/plugins/`
* S'assurer que l'extention `PHP ZIP` est bien activée
* Activer le plugin dans l'interface d'administation de WordPress (`Extensions`, `Extensions installées`, `WordPress to Hugo Exporter`, `Activer`)
* Dans le menu `Outils`, sélectionner `Export to Hugo`

Un fichier `hugo-export.zip` va être généré. Un fois extrait, on copie les répertoires `posts` (qui contient les articles) et `wp-content` (qui contient les médias) dans le répertoire `content` de Hugo :

```
valouille@Valintosh ~/Downloads/hugo-export » cp -r posts wp-content ~/hugo/content/
```

Normalement, les articles devraient s'afficher :

{{% figure class="center" src="/images/hugo_even_import.png" alt="Site Hugo avec le thème Even" title="Site Hugo avec le thème Even et les articles importés" %}}

## Commentaires

L'utilisation d'un site 100% statique implique qu'il n'y a pas de base de données pour stocker les commentaires. Il est donc nécessaire de passer par un service externe pour les gérer. J'ai fait le choix de partir vers la solution [Disqus][7] qui est un service managé, car je ne souhaite pas gérer un service sur mon serveur uniquement pour gérer les commentaires. De plus, la possibilité de poster des commentaires engendre le fait que des robots postent du SPAM via les formulaires. C'est donc de la gestion, et Disqus permet de s'absoudre de cela. Enfin, dans le cadre d'un site sans pub et à but non lucratif, Disqus n'ajoute pas de pubs ([plan "Free"][8]).

Pour utiliser Disqus, il est nécessaire de [créer un compte][9] pour obtenir un `shortname` qui sera ensuite paramétré dans le site pour le lier à Disqus.

Pour migrer les commentaires de Wordpress vers Disqus, on peut passer par [le plugin Disqus pour Wordpress][10]. Une fois le plugin installé, avec le `shortname` configuré, dans l'interface d'administation de Wordpress, on va directement dans le catégorie `Disqus`. Un bouton `Export Comments` permet de lancer l'export automatique des commentaires vers Disqus.

> Quand Hugo est lancé via la commande `hugo server`, les commentaires Disqus ne s'affichent pas sur les pages. Il est nécessaire de déployer son site sur l'URL finale pour voir les commentaires sur les pages. Cependant, dans l'administration de Disqus, on peut voir les commentaires importés dans la partie `Admin`, `Modération`, `Approuvés`.

# Rédaction des articles

Pour rédiger de longs articles, j'ajoute le tag suivant afin de pouvoir déployer le site sans que l'article ne soit visible tant qu'il n'est pas fini de rédiger :

```
draft: true
```

Dans Wordpress, on a la possibilité d'écrire avec un éditeur de texte "[WYSIWYG][11]", on voit donc directement le texte mis en forme tel qu'il sera une fois publié. Dans Hugo, il faut écrire les articles au format Markdown, et donc utiliser une certaine syntaxe. Pour visualiser l'article, on peut utiliser `hugo server -D` et ouvrir `http://localhost:1313`.

Il existe cependant des éditeurs de texte qui permettent de prévisualiser en temps réel le rendu des fichiers Markdown. J'utilise personnellement [Atom][12], avec le plugin [markdown preview][15] qui permet d'avoir une vue splitée. Un des avantages d'Atom est que grace au plugin [Git-Plus][16], on peut faire des commit et des push directement à partir de l'éditeur de texte.

# Publication des modifications

Une autre régression à ne pas apporter quand on passe d'un système tout-en-un comme Wordpress à un système tel que Hugo est de rendre la publication d'un article fastidieuse. Comme j'ai déjà un GitLab installé, j'ai décidé d'utiliser [GitLab CI][13] ainsi que [GitLab Pages][14] pour publier les articles automatiquement lorsque j'effectue un commit dans le dépôt contenant les fichiers sources de mon blog.

{{% figure class="center" src="/images/gitlab_hugo.png" alt="Workflow de déploiement d'une nouvelle version du blog" title="Workflow de déploiement d'une nouvelle version du blog" %}}

Lorsqu'un commit est poussé dans GitLab :

  * Un job est lancé automatiquement sur un Runner GitLab
  * Le job va utiliser une image Docker contenant Hugo pour générer les fichiers HTML du site à partir du dépot Git
  * La CI va ensuite uploader les fichiers HTML dans GitLab Pages
  * Le site est à présent à jour

Voici le contenu de mon fichier .gitlab-ci.yml qui contient la configuration de la CI :

```
image: jojomi/hugo:0.37

variables:
  GIT_SUBMODULE_STRATEGY: recursive

pages:
  script:
  - hugo
  artifacts:
    paths:
    - public
  only:
  - master

test:
  script:
  - hugo
  except:
  - master
```

J'espère que ce post vous donnera des idées si vous souhaitez vous lancer dans la création d'un blog ou dans la migration du vôtre !

[1]: http://hohugo.io
[2]: https://themes.gohugo.io/
[3]: https://themes.gohugo.io/hugo-theme-even/
[4]: https://github.com/SchumacherFM/wordpress-to-hugo-exporter
[5]: https://fr.wikipedia.org/wiki/Markdown
[6]: https://github.com/SchumacherFM/wordpress-to-hugo-exporter/archive/master.zip
[7]: https://disqus.com/
[8]: https://disqus.com/pricing/
[9]: https://disqus.com/register/
[10]: https://disqus.com/admin/wordpress/
[11]: https://fr.wikipedia.org/wiki/What_you_see_is_what_you_get
[12]: https://atom.io/
[13]: https://about.gitlab.com/features/gitlab-ci-cd/
[14]: https://about.gitlab.com/features/pages/
[15]: https://github.com/atom/markdown-preview
[16]: https://atom.io/packages/git-plus
