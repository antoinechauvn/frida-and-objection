# disable-ssl-pinning
Désactivation du SSL Pinning en utilisant Frida et Objection

![image](https://user-images.githubusercontent.com/83721477/182347358-73faa728-d03f-462c-927d-9a7e2478dbd3.png)
![image](https://user-images.githubusercontent.com/83721477/182347372-aa4f333d-fb94-42dd-916a-d5ce9309dd1b.png)

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
  * Injection de l'agent via `ptrace` en attachant ou en créant un processus
  * Nécessite des privilèges root pour ptrace
* Embarqué (Embedded)
  * Appareil Non-root
  * Patch d'application en intégrant la librairie `frida-gadget`
* Préchargé (Preloaded)

### Utilitaires Frida
`Frida CLI` : interface REPL qui a pour but d'émuler un grand nombre de fonctionnalités intéressantes d'IPython (ou Cycript), qui essaie de vous rapprocher de votre code pour un prototypage rapide et un débogage facile.<br>https://frida.re/docs/frida-cli/<br><br>
`frida-ps` : outil en ligne de commande pour lister les processus (très utile pour interagir avec un système distant).<br>https://frida.re/docs/frida-ps/<br><br>
`frida-trace` : outil pour tracer dynamiquement les appels de fonctions.<br>https://frida.re/docs/frida-trace/<br><br>
`frida-discover` : outil pour découvrir les fonctions internes d'un programme, qui peuvent ensuite être suivies en utilisant frida-trace.<br>https://frida.re/docs/frida-discover/<br><br>
`frida-ls-devices` : outil en ligne de commande pour lister les périphériques attachés (très utile quand on interagit avec plusieurs périphériques).<br>https://frida.re/docs/frida-ls-devices/<br><br>
`frida-kill` : outil en ligne de commande pour tuer les processus.<br>https://frida.re/docs/frida-kill/

![image](https://user-images.githubusercontent.com/83721477/182355512-32b54896-3c92-4fe2-bb50-62ee2d890762.png)

