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

# Scan de port NMAP

    # Nmap 7.91 scan initiated Tue Mar 23 17:21:45 2021 as: nmap -p22,8080 -sC -sV -oA nmap/ophiuchi 10.10.10.227
    Nmap scan report for 10.10.10.227
    Host is up (0.040s latency).

    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   3072 6d:fc:68:e2:da:5e:80:df:bc:d0:45:f5:29:db:04:ee (RSA)
    |   256 7a:c9:83:7e:13:cb:c3:f9:59:1e:53:21:ab:19:76:ab (ECDSA)
    |_  256 17:6b:c3:a8:fc:5d:36:08:a1:40:89:d2:f4:0a:c6:46 (ED25519)
    8080/tcp open  http    Apache Tomcat 9.0.38
    |_http-title: Parse YAML
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    # Nmap done at Tue Mar 23 17:21:54 2021 -- 1 IP address (1 host up) scanned in 9.24 seconds
    
 Il s'avère que le port `22/tcp` est ouvert qui tourne sur `OpenSSH`, nous avons également le port `8080/tcp` qui tourne sur `Tomcat 9.0.38`. J'ai regardé un peu les dates des versions des ports ouverts, et il s'avère que c'est assez récent, donc il est peu probable que il y a une vulnérabilité ou des vulnérabilités dans un de ces services.
