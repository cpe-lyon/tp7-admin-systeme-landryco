# Exercice 1. Personnalisation de GRUB

2. __Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’affiche pendant 10 secondes ;
passé ce délai, le premier OS du menu doit être lancé automatiquement.__

GRUB_TIMEOUT=10 ; GRUB_TIMEOUT_STYLE=menu ;

5. __On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres
à rajouter au fichier grub.__

GRUB_GFXMODE=1280x1024,1024x768x32 ; GRUB_GFXPAYLOAD_LINUX=keep ;

6. __On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : grub2-splash-images
(après installation, celles-ci sont disponibles dans /usr/share/images/grub).__

Il faut utiliser le paramètre GRUB_BACKGROUND="<chemin d'accès>".

8. __Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer.__

/etc/grub.d/40_custom :

```
menuentry ’Arrêt’ {
halt
}
menuentry ’Redémarrage’ {
reboot
}
```

9. __Configurer GRUB pour que le clavier soit en français__

```
sudo mkdir /boot/grub/layouts
sudo grub-kbdcomp -o /boot/grub/layouts/fr.gkb fr
```

/etc/default/grub :

GRUB_TERMINAL_INPUT=at_keyboard ;

/etc/grub.d/40_custom :

```
insmod keylayouts
keymap fr
```

# Exercice 2. Noyau

5. __Chargez le module ; vérifiez dans le journal du noyau que le message ”La fonction init_module() est
appelée” a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod.__

Il faut utiliser insmod

# Exercice 3. Interception de signaux

1. __Commencez par écrire un script qui recopie dans un fichier tmp.txt chaque ligne saisie au clavier par
l’utilisateur__

```
#!/bin/bash
while :; do
read a
echo $a >> tmp.txt
done
```

2. __Lancez votre script et appuyez sur CTRL+Z. Que se passe-t-il ? Comment faire pour que le script pour-
suive son exécution ?__

Le script est mis en pause.<br/> 
Pour le relancer il faut faire fg \<num\>, num étant le numéro affiché quand on suspend le script

3. __Toujours pendant l’exécution du script, appuyez sur CTRL+C. Que se passe-t-il ?__

Le script s'arrête

4. __Modifiez votre script pour redéfinir les actions à effectuer quand le script reçoit les signaux SIGTSTP
(= CTRL+Z) et SIGINT (= CTRL+C) : dans le premier cas, il doit afficher ”Impossible de me placer en
arrière-plan”, et dans le second cas, il doit afficher ”OK, je fais un peu de ménage avant” avant de
supprimer le fichier temporaire et terminer le script.__

```
trap 'echo impossible de me placer en arrière-plan !' TSTP
trap 'echo OK, je fais juste un peu de ménage avant ; exit' INT
trap 'rm tmp.txt' EXIT
```

6. __Relancez votre script et faites immédiatement un CTRL+C : vous obtenez un message d’erreur vous
indiquant que le fichier tmp.txt n’existe pas. A l’aide de la commande interne test, corrigez votre
script pour que ce message n’apparaisse plus.__

`trap 'if [ -e tmp.txt ]; then rm tmp.txt; fi' EXIT`

# Exercice 4. Surveillance de l’activité du système

1. __Dans tty1, lancez la commande htop, puis tapez la commande w dans tty2. Qu’affiche cette commande ?__

Affiche les utilisateurs connectés et leurs tâches en cours

2. __Comment afficher l’historique des dernières connexions à la machine ?__

`last`

3. __Quelle commande permet d’obtenir la version du noyau ?__

`uname -r`

4. __Comment récupérer toutes les informations sur le processeur, au format JSON ?__

`lshw -class Processor -json`

5. __Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?
Comment afficher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot ?__

```
journalctl --list-boots
journalctl -b -1
```

6. __Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ?__

`journalctl --list-boots`

7. __Faites en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message
à l’écran d’une maintenance le 26 mars à minuit.__

/etc/motd
```
============================================

ATTENTION : MAINTENANCE LE 26 MARS A MINUIT

============================================
```

8. __Ecrivez un script bash qui permet de calculer le k-ième nombre de Fibonacci : Fk = Fk−1 + Fk−2,
avec F0 = F1 = 1. Lancez le calcul de F100 puis lancez la commande tload depuis un autre terminal
virtuel. Que constatez-vous ? Interrompez ensuite le calcul avec CTRL+C et observez la conséquence sur
l’affichage de tload.__

```
function fib(){
  if [ $1 -le 1 ]; then
    echo 1
  else
    echo $[`fib $[$1-2]` + `fib $[$1 - 1]` ]
  fi
}

fib $1
```

Le calcul de F100 demande beaucoup de puissance ce qui entraine l’utilisation du processeur. Et retombe très rapidement après arrêt du skript
