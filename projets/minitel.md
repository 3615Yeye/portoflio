---
description: >
  This chapter shows how to upgrade Hydejack to a newer version. The method depends on how you've installed Hydejack.
hide_description: true
---

# Minitel

**En cours de rédaction.**

Le Minitel est de retour pour contrôler une Raspberry Pi avec son clavier et son écran.

![Alt text](/assets/img/Minitel_Side.jpg)

Un grand merci aux différentes personnes qui ont partagé au fil des années leurs expérimentations et succès dans l'utilisation du Minitel avec une Raspberry. Voici les deux principales sources qui m'ont permises d'y arriver :

- blog.uggy.fr, [cet article en particulier](http://blog.uggy.org/?post/2015/02/22/Minitel-et-Raspberry)
- Le blog de Pila, [cet article en particulier](http://pila.fr/wordpress/?p=425)

=> Adaptation à la dernière Raspbian et des essais avec les dernières possibilitées de la Raspberry modèle 3

Sommaire :

{:.no_toc}
* this unordered seed list will be replaced by toc as unordered list
{:toc}

## Pré-requis

- Un minitel avec une fiche DIN
- Adaptateur USB
- A raspberry with raspbian

## Hardware

Matériel nécessaire :

- Un convertisseur USB / série PL2303
- Une fiche DIN à 5 broches
- Cables électriques similaire à ceux utilisé dans les montages Arduino
- Un fer à souder pour fixer les cables à la fiche DIN

Le blog d'Uggy décrit comment lier les 3 cables du convertisseur USB / série à la fiche DIN : [dans cet article](http://blog.uggy.org/?post/2015/02/22/Minitel-et-Raspberry)

### Test

Identifier l'IP de la raspberry et s'y connecter.

``` bash
ping raspberrypi.local

ssh 192.168.1.10
```

Valider la liaison Raspberry Pi <-> Minitel et identifier le périphérique série qui commence par tty, ici ttyUSB0 :

Allumer le Minitel :

- Fnct + T puis A pour activer le videotext
- Fnct + T puis E désactive l'écho local
- Fnct + P puis 4 pour régler la vitesse de transmission série à 4800 bauds

``` bash
# La commande suivante doit afficher quelques carrés blanc sur le Minitel
echo 'Test affichage minitel' > /dev/ttyUSB0
```

[Photo du résultat d'un echo]

## Software

Création du service systemd qui proposera un login au Minitel :

``` bash
# Création du service :

sudo vim /etc/systemd/system/serial-getty-minitel@.service
    #  This file is part of systemd.
    #
    #  systemd is free software; you can redistribute it and/or modify it
    #  under the terms of the GNU Lesser General Public License as published by
    #  the Free Software Foundation; either version 2.1 of the License, or
    #  (at your option) any later version.

    [Unit]
    Description=Serial Getty on %I
    Documentation=man:agetty(8) man:systemd-getty-generator(8)
    Documentation=http://0pointer.de/blog/projects/serial-console.html
    BindsTo=dev-%i.device
    After=dev-%i.device systemd-user-sessions.service
    After=rc-local.service

    # If additional gettys are spawned during boot then we should make
    # sure that this is synchronized before getty.target, even though
    # getty.target didn't actually pull it in.
    Before=getty.target
    IgnoreOnIsolate=yes

    [Service]
    ExecStart=-/sbin/getty -L -i -I "\033\143" %i 4800 minitel1b-80
    Type=idle
    Restart=always
    UtmpIdentifier=%I
    TTYPath=/dev/%I
    TTYReset=yes
    TTYVHangup=yes
    KillMode=process
    IgnoreSIGPIPE=no
    SendSIGHUP=yes

    [Install]
    WantedBy=getty.target

# Mise à disposition du service pour l'interface série précédemment identifiée :
sudo ln -s /etc/systemd/system/serial-getty-minitel@.service /etc/systemd/system/getty.target.wants/serial-getty-minitel@ttyUSB0.service
```

Redémarrer systemd pour la prise en compte du nouveau service 
``` bash
sudo systemctl daemon-reload

# Activer le nouveau service :
sudo systemctl enable serial-getty-minitel@ttyUSB0.service

sudo systemctl start serial-getty-minitel@ttyUSB0.service
```

Le prompt de login apparait sur le Minitel et c'est parti !
