# LAB-13-Bypass-de-la-D-tection-de-Root-Android-avec-Objection
Ce lab est dans la continuité du LAB 11 sur Frida — mais cette fois on monte d'un niveau en utilisant Objection, un toolkit d'exploration mobile construit par-dessus Frida. Là où Frida demande d'écrire soi-même ses scripts d'injection, Objection propose une interface interactive qui automatise les opérations les plus courantes, dont le bypass de détection de root.
L'application cible reste dans la famille OWASP UnCrackable, et l'environnement de travail est identique : un émulateur Android, ADB pour la communication, et frida-server côté Android.

Lab réalisé dans un environnement contrôlé à des fins strictement pédagogiques.


Objectifs

Installer Objection et Frida côté PC.
Déployer et lancer frida-server sur l'émulateur Android.
Vérifier la communication entre le PC et l'appareil.
Attacher Objection à l'application cible.
Exécuter le bypass de détection de root via Objection.
Valider le résultat avant et après le bypass.
Comprendre les limites du bypass Java et comment Frida peut prendre le relais pour les checks natifs C/C++.


Environnement
ÉlémentDétailSystème PCWindowsAppareil AndroidÉmulateur AndroidOutilsADB, Frida, frida-server, ObjectionLangagesPython, JavaScriptCibleApplication Android avec détection de root

Étape 1 — Installation d'Objection
L'installation recommandée passe par pipx, qui isole les outils Python dans leurs propres environnements et évite les conflits de dépendances :
bashpip install --user pipx
pipx ensurepath
pipx install objection
Ou plus directement avec pip :
bashpip install --upgrade objection
On vérifie ensuite que l'installation s'est bien passée :
bashobjection --version
objection --help


Étape 2 — Préparation de l'émulateur Android
Vérification de la connexion ADB
bashadb devices
L'émulateur doit apparaître avec le statut device. S'il affiche unauthorized, il faut accepter la fenêtre de débogage USB depuis l'interface de l'émulateur.
Identification de l'architecture
bashadb shell getprop ro.product.cpu.abi
Cette étape est importante : la version de frida-server à télécharger doit correspondre exactement à l'architecture de l'émulateur (x86_64, arm64-v8a, etc.) et à la version de Frida installée sur le PC.
bashpip install --upgrade frida-tools
frida --version
Déploiement de frida-server
On transfère frida-server sur l'émulateur, on lui donne les droits d'exécution, et on le lance en arrière-plan :
bashadb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server &"
Si la commande en une ligne pose problème dans PowerShell, on peut ouvrir un shell interactif :
bashadb shell
cd /data/local/tmp
./frida-server
Dans ce cas, ce terminal doit rester ouvert pendant toute la durée du lab.
Vérification finale
bashadb forward tcp:27042 tcp:27042
frida-ps -Uai
La liste des applications installées sur l'émulateur doit s'afficher — c'est la confirmation que Frida communique bien avec frida-server.

Étape 3 — Lancement d'Objection sur l'application cible
On attache Objection directement au processus de l'application et on exécute le bypass de détection de root en une seule commande :
bashobjection --gadget "owasp.mstg.uncrackable1" explore
Puis dans le shell Objection :
bashandroid root disable
<img width="1465" height="441" alt="Objection attaché et bypass exécuté" src="https://github.com/user-attachments/assets/ab702817-0dc1-4521-be44-829d408277ae" />

Étape 4 — Résultat après bypass
Avec le bypass actif, l'application se lance normalement sans détecter l'environnement rooté. L'écran principal est accessible et l'application fonctionne comme prévu.
<img width="403" height="828" alt="Application déverrouillée après bypass" src="https://github.com/user-attachments/assets/dc5c92e5-121e-4c89-9de7-378ef18318be" />

Limites à connaître
Le bypass via Objection (android root disable) agit au niveau Java — il intercepte les appels des APIs Android classiques utilisées pour détecter le root. C'est suffisant pour la grande majorité des applications.
Cependant, certaines apps implémentent leurs vérifications directement en code natif C/C++ via le NDK. Dans ce cas, Objection seul ne suffit pas, et il faut compléter avec un script Frida personnalisé qui hook les fonctions natives au niveau du binaire.
