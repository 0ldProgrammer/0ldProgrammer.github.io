---
title: "HackTheBox - Ophiuchi"
description: pickle-fr.jpg
tags: ["Dans cet article je vous présente la machine que j'ai créée, une machine consacré au détéction de malware."]
---

Bonjour à tous, après un bon moment, je suis content de vous retrouver à nouveau sur mon site Web pour vous présenter la machine Ophiuchi de [HackTheBox](https://www.hackthebox.eu/).

Résumé :

- Une vulnérabilité de type désérialisation YAML (encore) est présente dans le service `Tomcat`.
- Le mot de passe de l'utilisateur est en clair dans le fichier de configuration de `Tomcat`, donc j'ai pu me connecter en tant que `admin`.
- Il est possible d'exécuter des commandes en tant que root grâce au fichier `sudoers`.

