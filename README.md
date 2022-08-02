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

# Injection (Root)

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
`adb shell "./data/local/tmp/nom_du_fichier_frida_server"`

Par la on lance le processus adb:<br>
`adb devices -l`

Pour vérifier les processus en cours sur l'appareil:
`frida-ps -U`

A ce stade l'agent Frida est installé, on peux désormais se connecter de plusieurs façons:
* En utilisant Frida CLI (frida --help)
* En utilisant Objection (objection --help)
* En utilisant Python (pip install frida)

## Frida CLI
```
usage: frida [options] target

positional arguments:
  args                  extra arguments and/or target

options:
  -h, --help            show this help message and exit
  -D ID, --device ID    connect to device with the given ID
  -U, --usb             connect to USB device
  -R, --remote          connect to remote frida-server
  -H HOST, --host HOST  connect to remote frida-server on HOST
  --certificate CERTIFICATE
                        speak TLS with HOST, expecting CERTIFICATE
  --origin ORIGIN       connect to remote server with “Origin” header set to ORIGIN
  --token TOKEN         authenticate with HOST using TOKEN
  --keepalive-interval INTERVAL
                        set keepalive interval in seconds, or 0 to disable (defaults to -1 to auto-select based on
                        transport)
  --p2p                 establish a peer-to-peer connection with target
  --stun-server ADDRESS
                        set STUN server ADDRESS to use with --p2p
  --relay address,username,password,turn-{udp,tcp,tls}
                        add relay to use with --p2p
  -f TARGET, --file TARGET
                        spawn FILE
  -F, --attach-frontmost
                        attach to frontmost application
  -n NAME, --attach-name NAME
                        attach to NAME
  -N IDENTIFIER, --attach-identifier IDENTIFIER
                        attach to IDENTIFIER
  -p PID, --attach-pid PID
                        attach to PID
  -W PATTERN, --await PATTERN
                        await spawn matching PATTERN
  --stdio {inherit,pipe}
                        stdio behavior when spawning (defaults to “inherit”)
  --aux option          set aux option when spawning, such as “uid=(int)42” (supported types are: string, bool, int)
  --realm {native,emulated}
                        realm to attach in
  --runtime {qjs,v8}    script runtime to use
  --debug               enable the Node.js compatible script debugger
  --squelch-crash       if enabled, will not dump crash report to console
  -O FILE, --options-file FILE
                        text file containing additional command line options
  --version             show program's version number and exit
  -l SCRIPT, --load SCRIPT
                        load SCRIPT
  -P PARAMETERS_JSON, --parameters PARAMETERS_JSON
                        parameters as JSON, same as Gadget
  -C USER_CMODULE, --cmodule USER_CMODULE
                        load CMODULE
  --toolchain {any,internal,external}
                        CModule toolchain to use when compiling from source code
  -c CODESHARE_URI, --codeshare CODESHARE_URI
                        load CODESHARE_URI
  -e CODE, --eval CODE  evaluate CODE
  -q                    quiet mode (no prompt) and quit after -l and -e
  -t TIMEOUT, --timeout TIMEOUT
                        seconds to wait before terminating in quiet mode
  --no-pause            automatically start main thread after startup
  -o LOGFILE, --output LOGFILE
                        output to log file
  --eternalize          eternalize the script before exit
  --exit-on-error       exit with code 1 after encountering any exception in the SCRIPT
  --auto-perform        wrap entered code with Java.perform
  --auto-reload         Enable auto reload of provided scripts and c module (on by default, will be required in the
                        future)
  --no-auto-reload      Disable auto reload of provided scripts and c module
```

### Basic frida hooking
`frida -l disableRoot.js -f owasp.mstg.uncrackable1`

### Hooking before starting the app
`frida -U --no-pause -l disableRoot.js -f owasp.mstg.uncrackable1`

## Objection
```
Options:
  -N, --network            Connect using a network connection instead of USB.
  -h, --host TEXT          [default: 127.0.0.1]
  -p, --port INTEGER       [default: 27042]
  -ah, --api-host TEXT     [default: 127.0.0.1]
  -ap, --api-port INTEGER  [default: 8888]
  -g, --gadget TEXT        Name of the Frida Gadget/Process to connect to.
                           [default: Gadget]
  -S, --serial TEXT        A device serial to connect to.
  -d, --debug              Enable debug mode with verbose output. (Includes
                           agent source map in stack traces)
  --help                   Show this message and exit.

Commands:
  api          Start the objection API server in headless mode.
  device-type  Get information about an attached device.
  explore      Start the objection exploration REPL.
  patchapk     Patch an APK with the frida-gadget.so.
  patchipa     Patch an IPA with the FridaGadget dylib.
  run          Run a single objection command.
  signapk      Zipalign and sign an APK with the objection key.
  version      Prints the current version and exists.
  ```

### Basic objection hooking
Par défaut, objection fait spawn un process afin de s'y connecter (en mode non-root on parlera de connexion à un gadget)<br>
`objection -g com.app.app explore`

### Hooking before starting the app
`objection -g com.app.app explore --startup-command "android sslpinning disable"`

Plus d'infos: https://github.com/sensepost/objection/wiki

# Embarqué (Non-root)
Dans un premier temps, on installe apktool:<br>
https://ibotpeaches.github.io/Apktool/install/

Par la suite on viendra placer `apktool.jar` ainsi que `apktool.bat` dans le dossier suivant: <br>
`%APPDATA%\Local\Android\Sdk\platform-tools`

On ajoute nos repertoires dans la variable d'environnement `Path`:<br><br>
`%APPDATA%\Local\Android\Sdk\build-tools\33.0.0` - Contenant aapt, apksigner et zipalign<br><br>
`%APPDATA%\Local\Android\Sdk\platform-tools` - Contenant adb et apktool<br>

Liste des dépendances: https://github.com/sensepost/objection/wiki/Patching-Android-Applications#patching---dependencies

Par la suite java est requis afin d'utiliser apktool:
https://www.java.com/fr/download/manual.jsp

On patch notre application avec objection:
`objection patchapk -s "C:\Users\CHAUVIN ANTOINE\Downloads\sub.apk"`

Si vous rencontrez des problèmes, certaines solutions comme `aapt2` sont à envisager:
https://stackoverflow.com/questions/50725735/invalid-resource-directory-navigation
https://github.com/sensepost/objection/wiki/Android-APK-Patching#debugging-failed-patches
