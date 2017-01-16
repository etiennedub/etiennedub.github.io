---
layout: nomenu
title: opusenligne
date: 8/9/2009
---
#OpusEnLigne.ca sur Linux

OpusEnligne.ca est une un service web pour recharger ça carte OPUS en ligne. Il demande l'achat d'un lecteur de carte. Cependant, le service est seulement accessible aux utilisateurs de macOS ou Windows.

![No Linux]({{site.url}}/images/no-linux.png)

Étant un utilisateur de Linux j'ai trouvé ceci très embêtant et en me penchant sur le problème j'ai trouvé une méthode pour le faire fonctionner.

## Démarche 

J'ai commencé par regarder comment leur système fonctionnait sur macOS. Pour utiliser *OpusEnLigne* nous devons d'abord télécharger le plug-in *SmartCardPlugin* pour recharger notre carte. 

La première étape est de trouver comment le site communique avec le plug-in. Les outils de débogage de Firefox nous permettent de rapidement trouver que le site utilise le type MIME suivant : *smartcard://XSCPServer=...* .
L'URL précédant démarre l'application associée à *smartcard://* et la suite va être passée en argument.


![debug]({{site.url}}/images/smartcard-debug.png)

Du côté de l'application, on extrait SmartCardPlugin.app et remarque rapidement qu'un script Bash fait appel à XSCP.jar en Java. Bonne nouvelle ceux-ci devraient être compatible sur Linux. Le script détermine l'emplacement de l'archive *JAR* et d'un log.

![Bash script]({{site.url}}/images/opus-bash.png)

Sur Linux, je tente d'exécuter le script de la même manière qu'avec macOS et malheureusement ça ne fonctionne pas... J'essaie plusieurs options qui ne fonctionne pas non plus alors, je tente de faire croire au programme qu'il est exécuté sur un système macOS avec le code suivant :

```java

public static void main(String[] args) {
		final String propertyName = "os.name";
	    String oldProperty = System.getProperty(propertyName);
	    System.setProperty(propertyName,"mac");
		XSCP.main(args);
		}
```
et ça fonctionne ! Le code précédant est très simple, je change mon nom de système d'exploitation (juste le temps d'exécution du code) plus j'appel la fonction *main* du *XSCP.jar* originale.

Il ne reste plus qu'à compiler le tous comme une nouvelle archive *JAR*.

## Installation

1. Télécharger [XSCP_linux.jar]({{site.url}}/ressources/OpusEnLigne/XSCP_linux.jar) et [smartCardDriver]({{site.url}}/ressources/OpusEnLigne/smartCardDriver).
2. Copier *XSCP_linux.jar* à la racine de votre dossier personnel (Vous pouvez changer le répertoire en éditant *smartCardDriver*). 
3. Copier *smartCardDriver* dans un de vos dossiers *bin* (/usr/bin, /usr/local/bin/, etc).
4. Associer le type MINE ***smartcard://*** pour exécuter *smartCardDriver*. Ceci est différent pour chaque distribution.
Sur ArchLinux :
  * Télécharger ce [desktop entrie]({{site.url}}/ressources/OpusEnLigne/smart-card-driver.desktop) et copier-le dans votre répertoire */usr/share/application*
  * Exécuter la commande suivante :
	
		$ xdg-mime default smart-card-driver.desktop x-scheme-handler/smartcard

5. Le site OpusEnLigne.ca vérifie votre *user-agent* il suffit de le changer pour un valide (la procédure est différente d'un navigateur à l'autre). Personnellement j'utilise le suivant :

    	Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:16.0) Gecko/20100101 Firefox/50.0
	
6. Voilà !

![Profit]({{site.url}}/images/opusEnLigne-arch.png)

