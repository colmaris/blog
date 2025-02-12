+++
title = "Gérer ses dotfiles avec git"
date = "2014-05-19"
Categories = ["adminsys"]
tags = ["dotfiles","git","mr","vcsh"]
+++

L’utilisation de plusieurs ordinateurs sous Linux, peut devenir un vrai cauchemar lorsque l’on veut maintenir la même configuration à jour. <!--more-->Dans mon quotidien il m’arrive d’utiliser trois ordinateurs différents. Et j’aime bien retrouver mes marques quel que soit la machine utilisée. Au départ je « trimballais » partout une clef usb avec tous mes fichiers de configurations, mais cette solution a atteint sa limite très rapidement.

Elle était surtout très contraignante car, je devais en permanence avoir ma clef usb sur moi et en cas de modification de configuration penser à faire une copie (ce qui arrivait une fois sur mille) sur la dite clef pour pouvoir mettre à jour mes autres ordinateurs. Du coup je me retrouvais avec des configurations très hétéroclites.

J’ai donc décidé de créer un dépôt git pour les différentes configurations que je souhaite déployer sur mes ordinateurs. Certains diront pourquoi ne pas tout grouper dans un même dépôt, je préfère les séparer en cas problème sur un cela ne me bloquera pas le reste.

C’est alors que j’ai entendu parler de vcsh et mr (Merci à Brice camarade libriste qui m’a expliqué les bases). Ses deux petits programmes couplés avec git m’ont permit de centraliser tous mes fichiers de configurations sur mon serveur gitlab et ainsi de pouvoir installer mon environnement sur tous les ordinateurs que j’utilise.

### VCSH – Version Control System for $HOME – multiple Git repositories in $HOME

Comme son nom l’indique il permet de faire de la gestion de version pour le $HOME. Grâce à lui plusieurs dépôts git peuvent cohabiter dans le même répertoire. Il centralise toutes les têtes de dépôts au même endroit, par défaut il les place dans `~/.config/vcsh/repos.d` mais il est tout à fait possible de le changer, de même si l’on désire maintenir plusieurs dépôts git ailleurs que dans le $HOME. Pour plus d’informations je vous invite sur [la page github](https://github.com/RichiH/vcsh) du projet

Passons à son installation, sous Debian,

``` shell 
sudo apt-get install vcsh
```

### mr pour myrepo

`mr` intervient principalement sur l’utilisation et la configuration des dépôts. Dans un premier temps il permet avec une seule ligne de commande, de livrer et pousser les modifications, mettre à jour simultanément tous les dépôts renseignés dans sa configuration. Dans un second temps il permet aussi de gérer la configuration de ses mêmes dépôts. Dans mon cas il me permet de pousser mes modifications à la fois sur mon serveur gitlab mais aussi sur mon compte github pour en garder une sauvegarde. Il me permet de rajouter une url à mon origin dès le clonage des dépôts, ce qui m’évite une configuration post-installation de tous mes dépots. Pour plus d’informations voici la [page](http://myrepos.branchable.com) du projet.

L’installation sur Debian est toujours aussi simple

``` shell 
sudo apt-get install mr 
```

### Préparatifs avant la transformation

Tout d’abords j’ai défini quels fichiers de configurations que je souhaite garder à jour et déployer sur mes différents ordinateurs:

1. La configuration d’openbox
2. La configuration d’emacs
3. La configuration de terminator
4. La configuration de zsh

### Initialisation avec vcsh

J’ai au préalable créé sur gitlab et github un dépôt par configurations. Ensuite j’ai suivi la documentation de vcsh pour créer un par un par les dépôts. Exemple avec openbox :

``` shell 
#initialisation du dépôt
vcsh init openbox
#création du gitignore pour éviter d'avoir des erreures sur les dossiers non #suivi
vcsh write-gitignore openbox
#ajout des fichiers
vcsh openbox add ~/.config/openbox/rc.xml ~/.config/openbox/menu.xml ~/.config/openbox/autostart ~/.gitignore/openbox
vcsh commit -am 'intial commit'
vcsh openbox remot set-url --add origin git@github.com:colmaris/dotfiles-openbox.git
```

J’ai reproduit cette manipulation pour chacun des dépôts à initialiser. Petite astuce si le dépôt git existe déjà, comme ce fut le cas pour moi avec ma configuration d’emacs, dont je ne voulais pas perdre l’historique. Il m’a suffi de créer le chemin vers les fichiers de configurations dans le dépôts git avant la migration vers vcsh.

Pour emacs il faut de l’on retrouve le chemin exact vers le fichier `init.el`.

``` shell
cd ~/.emacs
mkdir .emacs/
git mv init.el .emacs
git add .emacs
git commit -am 'moving file init.el'
git push
```

Ensuite j’ai supprimé totalement le dossier .emacs de mon `$HOME`, pour le cloner avec vcsh.

``` shell
vcsh clone git@git.olivierdelort.net:colmaris/emacs <span class="crayon-e">emacs</span>
```

Ainsi j’ai put garder l’historique de mon dépôt emacs, et je peux maintenant l’utiliser avec vcsh sans problème.

### Configuration de mr

Une fois tous mes dépôts de configuration initialiser avec vcsh, je suis passé à la configuration de mr dont le but premier, dans mon cas, est de pouvoir pousser mes modifications sur mon gitlab et les sauvegarder sur github.

La configuration de *mr* se fait via un fichier .mrconfig directement placé dans le $HOME. Voici le mien

``` shell
[DEFAULT]
git_gc = git gc "$@"
# * Dotfiles Organisation

# ** Emacs
[$HOME/.config/vcsh/repo.d/emacs.git]
checkout =
		vcsh clone git@git.olivierdelort.net:colmaris/emacs.git emacs
		vcsh emacs remote set-url --add origin git@github.com:colmaris/dotfiles-emacs.git

# ** Openbox
[$HOME/.config/vcsh/repo.d/openbox.git]
checkout =
		vcsh clone git@git.olivierdelort.net:colmaris/dotfiles-openbox.git openbox
		vcsh openbox remote set-url --add origin git@github.com:colmaris/dotfiles-openbox.git

# ** Terminator
[$HOME/.config/vcsh/repo.d/terminator.git]
checkout =
		vcsh clone git@git.olivierdelort.net:colmaris/terminator-solarized.git terminator
		vcsh terminator remote set-url --add origin git@github.com:colmaris/terminator-solarized.git

# ** Zsh
[$HOME/.config/vcsh/repo.d/zsh.git]
checkout =
		vcsh clone git@git.olivierdelort.net:colmaris/dotfiles-zsh.git zsh
		vcsh zsh remote set-url --add origin git@github.com:colmaris/dotfiles-zsh.git
```

Petite explication :

``` shell
# ** Emacs
#ici j'indique ou se trouve la tête du dépôt
[$HOME/.config/vcsh/repo.d/emacs.git]
#ici se trouve les actions à réaliser lors du clonage
checkout =
# je clone à partir de mon gitlab
vcsh clone git@git.olivierdelort.net:colmaris/emacs.git emacs
#je rajoute mon compte github à l'origin de mon dépôt
vcsh emacs remote set-url --add origin git@github.com:colmaris/dotfiles-emacs.git
```

Lors du clonage des dépôts mr rajoutera l’url de mon compte github à l’origin déjà configurée.

Ce qui me permet de pousser d’un seul coup tous les dépôts sur mon github.

``` shell
mr push
```

### Déploiement

A partir de maintenant je peux déployer mes configurations sur n’importe quel ordinateur ou git, vcsh et mr sont installés.

Je procède comme suit :

``` shell
#installation des prérequis
sudo apt-get install git vcsh mr
#configuration de mr
git clone git@git.olivierdelort.net:colmaris/dotfiles-mr.git ~/.mrconfig
#clonage 
mr checkout
```

Et voilà en quelques minutes j’ai déployé ma configuration et je suis prêt à travailler. S’il m’arrive de faire des modifications je les livre et les pousse directement dans le dépôt concerné. Et sur mes autres ordinateurs il me suffit de faire une mise à jour avec la commande `mr update` pour qu’elles soient prises en comptent.

``` shell
mr update
mr update: /home/draconis/.config/vcsh/repo.d/apache-autoindex.git
Already up-to-date.
mr update: /home/draconis/.config/vcsh/repo.d/draconis-install.git
Already up-to-date.
mr update: /home/draconis/.config/vcsh/repo.d/emacs.git
Already up-to-date.
mr update: /home/draconis/.config/vcsh/repo.d/eso-theme.git
Already up-to-date.
mr update: /home/draconis/.config/vcsh/repo.d/motd-colmaris.git
Already up-to-date.
mr update: /home/draconis/.config/vcsh/repo.d/mrconfig.git
Already up-to-date.
mr update: /home/draconis/.config/vcsh/repo.d/mytheme-lightdm.git
Already up-to-date.
mr update: /home/draconis/.config/vcsh/repo.d/openbox.git
Already up-to-date.
mr update: /home/draconis/.config/vcsh/repo.d/terminator.git
Already up-to-date.
mr update: /home/draconis/.config/vcsh/repo.d/zsh.git
Already up-to-date.
mr update: finished (10 ok)
```

### Conclusion

Depuis que j’utilise cette méthode je revis littéralement, je ne me soucis plus de savoir si j’ai ma clef usb à jour et avec moi. Tout est centralisé sur mon gitlab et j’ai mon github en sauvegarde. Je l’ai étendu sur d’autre projet sur lesquels je travaille.
