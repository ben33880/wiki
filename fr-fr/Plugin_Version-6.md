
# Explications pour passer de la version 5 à la version 6 plugin.

</br>

__Note :__ Ces explications sont valables pour une installation sous Linux.



* Ouvrir le terminal
* Aller dans le répertoire du plugin. La commande est normalement <code>cd domoticz/plugins/Domoticz-Zigate</code>
* Exécuter la commande : `git remote set-url origin https://github.com/zigbeefordomoticz/Domoticz-Zigbee`
* Installer les paquets python nécessaire avec la commande : `sudo pip3 install voluptuous pycrypto aiosqlite crccheck pyusb attr attrs aiohttp pyserial-asyncio`

Le temps de la phase de développement, il faut passer sur la nouvelle branche beta6 : `git checkout beta6`

* Mettre à jour le plugin avec la commande : `git pull`
* Installer les librairies python manquantes avec la commande : `git submodule update --init --recursive`
* Rendre le fichier __plugin.py__ exécutable en lançant la commande : `sudo chmod +x plugin.py`
* Redémarrer DomoticZ.

Normalement, le nom du plugin dans matériel est devenu __ZigBee for DomoticZ__.

A partir de maintenant, le terme ZiGate est remplacé par __coordinateur__, plus générique.

Si vous avez déjà un plugin configuré avec une ZiGate comme coordinateur, vous n'avez rien à faire le plugin doit continuer a fonctionner normalement.

## Le paramétrage

Il y a 4 modèles de coordinateurs possibles :

* ZiGate : aucune modification sur le fonctionnement du plugin existant
* ZiGate + : aucune modification sur le fonctionnement du plugin existant
* ZiGate (via zigpy) : le plugin communique avec la ZiGate avec les librairies zigpy. C'est uniquement expérimental et n'est pas supporté.
* Texas Instruments ZNP : pour les nouveaux coordinateurs de marque TI



## IMPORTANT Mise à jour du plugin

Le `git pull` n'est plus suffisant, il faut maintenant faire la commande `git pull --recurse-submodules`.