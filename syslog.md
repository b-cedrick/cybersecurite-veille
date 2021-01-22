
# Syslog

La **journalisation** ( historique des événements) est un moyen que les systèmes d’exploitation utilise pour enregistrer toutes les actions d’un événement qui a eu lieu (exécution d’un processus, activité réseau etc ), permettant ainsi de localiser plus rapidement les défaillance d’un système.

Ce processus est géré par un système de journalisation appelé  **syslog**.

**Syslog** : C' est un protocole développé dans les années 1980, qui permet de collecter les messages des services qui tournent sur linux et les enregistrer dans des fichiers appelés « **fichiers log** » ( logs files en anglais ), c’est fichiers sont placé dans **/var/log.**

Syslog permet de filtrer les messages et les stocke dans  **/var/log**  par type de message.  
Liste des fichiers logs minimals présents dans /var/log :
```
-/var/log/secure

-/var/log/maillog

-/var/log/cron

-/var/log/boot.log

-/var/log/messages
```
**/var/log/secure :**  
Syslog stocke dans le fichier de log « secure » tous les messages liée à la sécurité y compris ceux de l’authentification.

**/var/log/maillog:**  
Syslog stocke les message liée au messagerie.

**/var/log/cron :**  
Contient tous les message liée aux informations sur les tâches cron. Enregistrement à chaque fois que le démon cron (ou anacron) commence une tâche.

**/var/log/boot.log :**  
Contient tous les message liée au démarrage du système.

**/var/log/messages :**  
La plupart des messages log sont enregistré dans le fichier /var/log/messages, sauf les type de messages qu’on a vu précédemment.

Tabl
## Fonctionnement de syslog

**Facility & priority :**

Pour comprendre le fonctionnement de syslog nous allons étudier son fichier de configuration :  **/etc/syslog.conf,** en voici un extrait :
```
# Log all kernel messages, authentication messages of
# level notice or higher and anything of level err or
# higher to the console.
# Don't log private authentication messages!
***.err;kern.*;auth.notice;authpriv.none /dev/console** 
# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
***.info;mail.none;authpriv.none         /var/log/messages**

# The authpriv file has restricted access.
**authpriv.*                             /var/log/secure**

# Log all the mail messages in one place.
**mail.*                                 /var/log/maillog**

# Everybody gets emergency messages, plus log them on
# another machine.
***.emerg                    ***
***.emerg                    @arpa.berkeley.edu**

# Root and Eric get alert and higher messages.
*.alert                    root,eric

# Save mail and news errors of level err and higher in a
# special file.
**uucp,news.crit             /var/log/spoolerr**
```
Comme vous pouvez le constater, les lignes du fichier ‘syslog.conf’ est de cette forme : **XXXX.YYYY,** dont **:** _XXXX=Facility et_ _YYYY=Priorité_

_Exemple :_  
**authpriv.* /var/log/secure**  
**mail.* /var/log/maillog**

**Que signifie Facility et Priorité ?**
Les programmes utilisent syslog pour enregistre leurs événements et ces éventement sont caractérisé par  **Facility** et  **Priorité.**

La Priorité indique la criticité du message généré par un programme, Le tableau ci-dessous présentes la liste des priorité syslog :

|  Code| Priorité |	Description|
|--|--|--|
| 0 | Emergencie | Le système est unitilisable |
| 1 | Alert | Une action immediate est requise | 
| 2 | Critical | Condition critique | 
| 3 | Errors | Erreurs détéctés | 
| 4 | Warning | Avertissement | 
| 5 | Notice | Evenement normal mais significatif | 
| 6 | Info | Message d'information | 
| 7 | Debug | Débogage | 

La facilty indique le type de message généré par un programme, le tableau ci-dessous présente ces types de message :
|  Type| 	Description|
|--|--|
| kern | Utilisé pour les messages du noyau |
| user | Utilisateur | 
| mail | Messagerie |
| cron | Le planificateur de tâches | 
| auth | Utilisé pour plusieurs événements de sécurité | 
| authpriv | Utilisé pour les controles d'accès | 
| daemon | Utilisé par les process système et les daemon | 
| mark | Pour les messages générés par syslog lui même contenant un horodatage et la chaine de caractère "--MARK--" | 


**NB :**

> **Noter que Facility et Priority d’un message est déterminé par le programme qui génère le message et pas par syslog.**

Devant chaque  _« Facility.Priority »_  est indiqué le fichier sur lequel syslog enregistre les messages.

**NB :** 

> _un wildcard ( * ) peut être utilisé pour indiquer toute les Facility ou Priority :_ mail.***

Voyons voir des exemples pour que les choses soit bien claire :

**-authpriv.\* /var/log/secure :** Ici toute les messages liée à l’authentification ( **authpriv**  ) et quelque soient leur sévérité (  *****  )seront être enregistré sur  **/var/log/secure**.

**-cron.crit /var/cron :** Les messages  **critique** ( crit ) de type Cron seront enregistrer sur  **/var/cron.**

## **La rotation de log :**

Comme nous avons vu précedement, le système et les application installés sur le serveur génèrent des logs et avec le temps vous pouvez vous retrouver  avec des dizaines voir des centaines de gigaoctets de logs  donc des partition satuées.

Pour éviter ce désagréments, une  **Rotation de log**  est effectuée.

_**Qu’est ce que la Rotation de log ?**_

Lorsqu’une rotation de log est effectuée, le fichier log est renommé avec une extention indiquant la date de la rotation, par exemple le fichier  **/var/log/messages**  devient /var/log/messages-20181018, cela veut dire que la rotation du fichier a été effectué le 2018-10-18. une fois la rotation est effectué un nouveau fichier log est créé (**/var/log/messages**) pour permettre au programme ou bien le système de continuer d’écrire sur le fichier.

Après un certain nombre de rotation, le fichier log le plus ancien est supprimé afin de libérer de l’espace et éviter ainsi la saturation du système du fichier.

La rotation de log est géré par un programe appelé  **Logrotate**  et un cron job exécute le programme Logrotate quotidiennement pour vérifier s’il y a besoin de faire la rotation.

## Analyser les enregistrements de syslog :

Tous les enregistrements sur les fichiers log gérés par syslog sont dans un format standard. 
Ex:
```Nov 14 12:01:23   localhost   run-parts(/etc/cron.daily)[1962] : starting logrotate```
1.  L’horodatage : date et heure de l’entréé.
2.  Le host depuis lequel le message est envoyé ( _ici la machine local_  ).
3.  Le programme ou le service qui a envoyé le message (  _ici c’est Cron_  ).
4.  Le message envoyé.

## Envoyer un message syslog avec « logger » :
L'envoi d'un message syslog avec **logger** est  très utile lorsque on veut dévolopper du des logiciels ou applis et tout simplement pour tester si le syslog fonctionne.

Pour envoyer un message syslog, on utlise la commande  **logger**.  
Par défaut logger envoie un message en utilisant la facility « user » avec la séverité « notice » mais vous pouvez spécifier ce que vous voulez avec l’option  **-p .**

Ex:
```logger -p local7.warning "mon message dans syslog"```
