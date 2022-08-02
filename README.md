# disable-ssl-pinning
Désactivation du SSL Pinning en utilisant Frida et Objection

![image](https://user-images.githubusercontent.com/83721477/182347358-73faa728-d03f-462c-927d-9a7e2478dbd3.png)
![image](https://user-images.githubusercontent.com/83721477/182347372-aa4f333d-fb94-42dd-916a-d5ce9309dd1b.png)

Pré-requis: Python, Java, Android Studio

### Qu'est-ce que Frida?
```
Frida est une boîte à outils d'instrumentation de code dynamique qui permet d'injecter des extraits de JavaScript ou
d'une bibliothèque dans des applications natives sur iOS et Android (mais pas seulement) et de déboguer un processus
en cours d'exécution.
```

### Qu'est-ce que Objection?
```
objection est une boîte à outils d'exploration mobile, crée par Frida, conçue pour vous aider à évaluer la sécurité de
vos applications mobiles, sans avoir besoin d'un jailbreak.
```

## Principe de Fonctionnement

Frida fonctionne de 3 façon différentes:

* Injecté (Injection)
  * Connnexion client/server via frida-server
  * Processus frida-server en background (daemon) écoutant sur `localhost:27042`
  * Injection de la librairie `frida-agent` via `ptrace` en attachant ou en créant un processus
  * Nécessite des privilèges root pour ptrace
* Embarqué (Embedded)
  * Appareil Non-root
  * Patch d'application en intégrant la librairie `frida-gadget`
  * `frida-gadget` expose une interface compatible `frida-server` , écoutant sur `localhost:27042`
  * Le nom du processus sera toujours `Gadget` et l'identifiant de l'application installée est toujours `re.frida.Gadget`
* Préchargé (Preloaded)

### Utilitaires Frida
`Frida CLI` : interface REPL qui a pour but d'émuler un grand nombre de fonctionnalités intéressantes d'IPython (ou Cycript), qui essaie de vous rapprocher de votre code pour un prototypage rapide et un débogage facile.<br>https://frida.re/docs/frida-cli/<br><br>
`frida-ps` : outil en ligne de commande pour lister les processus (très utile pour interagir avec un système distant).<br>https://frida.re/docs/frida-ps/<br><br>
`frida-trace` : outil pour tracer dynamiquement les appels de fonctions.<br>https://frida.re/docs/frida-trace/<br><br>
`frida-discover` : outil pour découvrir les fonctions internes d'un programme, qui peuvent ensuite être suivies en utilisant frida-trace.<br>https://frida.re/docs/frida-discover/<br><br>
`frida-ls-devices` : outil en ligne de commande pour lister les périphériques attachés (très utile quand on interagit avec plusieurs périphériques).<br>https://frida.re/docs/frida-ls-devices/<br><br>
`frida-kill` : outil en ligne de commande pour tuer les processus.<br>https://frida.re/docs/frida-kill/

# Injection

On installe la librairie frida-tools pour python.
`pip install frida-tools`

Avant l'installation du serveur, il faut sélectionner le serveur Frida (`frida-server`) en fonction des différentes architectures de processeur.<br><br>
On change de répertoire afin d'utiliser ADB (par défaut disponible sur Android Studio):<br>
`cd %APPDATA%\Local\Android\Sdk\platform-tools`<br><br>
Puis on consulte l'architecture du processeur:<br>
`adb shell getprop ro.product.cpu.abi`

Par la suite il faut télécharger le serveur correspondant à l'architecture<br>
https://github.com/frida/frida/releases

Une fois l'archive décompresser on transmet le fichier frida-server sur l'appareil et on change les permissions:
```cmd
adb root
adb push "emplacement du fichier frida-server" /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/nom_du_fichier_frida_server
```

Pour infos: https://chmodcommand.com/chmod-755/

On lance ensuite le processus en arrière-plan:<br>
`adb shell "/data/local/tmp/nom_du_fichier_frida_server &"`

Par la on lance le processus adb:<br>
adb devices -l
