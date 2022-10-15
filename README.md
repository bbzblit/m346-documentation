# M346-leistungsbeurteilung
## Meine Ziele
Mein Ziel ist es, dass ich alle "Projekte" alle Kompetenzen mit ausnahme von `Nr. 16 Microsoft Certified: Azure Fundamental (AZ-900) absolvieren` erledige. Ich möchte mich damit persönlich im bereich der Cloud weiterbilden. Ich denke, dass ich die ersten Aufgaben durch mein Vorissen schon relativ einfach erledigen kann. ALlerdings habe ich bis jetzt noch keine Erfahrung im Bereich von Serverless Funktionen usw.

## Planung
Bis zum Ende des Modules habe ich noch etwa 6 Wochen Zeit. Ich sollte dahher genügend Zeit haben um problemlos alle Services aufzubauen und zu Documentieren. Ein etwas anderes Problem stellen noch die Kosten dar. Ich habe von GitHub education aus `100 CHF` Azure und weitere `100 CHF` digitalCloud guthaben. Bei gewissen Projekten werde ich auch noch auf meinen Provider, den ich sonnst immer Verwende (Hetzner) switchen. Bei gewissen Augaben muss man auch eine App bei einem der anderen beiden grossen Cloud anbieter `AWS` und `GCP` erstellen. Diese werde ich gegen Ende machen, um grössere Kosten zu vermeiden. 


## NR 1 Eine statische Webseite erstellen und mit PaaS in der Cloud publizieren.

### Erstellen der Statischen Website.
Als statische Website verwende ich meine Website aus dem letzen Modul M290. Diese braucht einen Webserver mit `PHP` und eine SQL datenbank. Ich verwende zum Deployen den [Webhostingservice](https://www.hetzner.com/webhosting) von `Hetzner` da ich in dem Basicpaket für `1.90` nichtnur einen Webserver mit PHP zur Verfügung habe sondern dazu auch gleich noch eine MySQL Datenbank, eine Domain bei der ich frei über die DNS EInträge verfügen kann, und einen E-Mail Server. Auch brauch ich mir keine Sorgen über Folgekosten zu machen, da sowohl der Netzwerktransfer wie aber auch Schreib und Leservorgäng in der MySQL Datenbank inbegriffen sind. 

## Nr. 2 Einbinden eines Networkdrives
Ich benutze für das einbinden einen bereits exisitierenden Server, auf dem ich aktuell meine Backubs von den Server speichere. Ich werde das Filesystem auf meinem lokalen PC Mounten. Das aktuelle Betriebsystem von meinem PC ist `Ubuntu 22.04.1 LTS`.

1. Erstellen des Mountpoints <br/>
Als erstes habe ich einen neuen Folder erstellt, an dessen Stelle ich den Networkdrive mounten möchte

```bash
sudo mkdir /media/M346
```

Damit ich anschlissend auch mit dem Folder interagieren kann musste ich mir noch die nötigen Berechtigungen geben. Das geht ganz einfach mit dem `chmod` command.

```bash
sudo chmod 777 /media/M346/
```

2. Umstellen von Passwort Authentifikation zu public key Authentifikation <br/>
Im nächsten Schritt könnte ich theoretisch direckt das Filesystem mounten. Allerdings ist der neu erstellte User defaultmässig über ein Passwort geschützt. Dass möchte ich ändern und die Authentifikation über einen Public Key durchführen. Das ist sicherer gegen Physchingattaken und ich muss mir das Passwort nicht merken. Auf dem server gibt es bereits ein Skript namens `install-ssh-key` mit dem man automatisch SSH Keys installieren kann. Dadurch kann ich über OpenSSH das Ganze ganz einfach installieren. 

```bash
cat ~/.ssh/id_ecdsa.pub | ssh <User>@<ServerIP> install-ssh-key
```
3. Mounten des Filesystems <br/>
Für das mounten des filesystems benutze ich `sshfs`. Wie der name schon sagt basiert es auf dem Secure Shell Protocoll und ist dadurch nicht die Schnellste Möglichkeit allerdings ist es relativ sicher und dahher gut geeignet um Daten über das Internet zu transportieren.

```bash
sudo sshfs <username>@<server>:<server-directory> /media/M346
```
 

## NR 3 Hosten einer statischen Website in der Cloud
Für diese Aufgabe benutzte ich eine VPS mit `10 GB` Speicher `1` CPU Core und `0.5 GB` RAM. Das sollte für eine kleine Website problemlos ausreichend sein. Als OS benutzte ich `Debian 11`.

1. Login via SSH <br/>
Im ersten Schritt verbinde ich mit der VPS via SSH. Als authentification benutze ich ein 521 Bit langer ECDSA Key.
```bash
ssh root@<ip>
```
2. Erstellen eines neuen Users <br/>
Da es ein Sicherheitsrisiko ist, wenn man sich mit dem Defaultuser anmelden kan erstelle ich ein neuen User und gebe ihm Sudorechte. Vom jetztigen Moment an werde ich mich nurnoch mit dem neuen User auf dem Server anmelden.

```bash
sudo useradd -m <username>
usermod -aG sudo <username>
mkdir /home/<username>/.ssh
cp .ssh/authorized_keys /home/<username>/.ssh
sudo chown <username> /home/debian/.ssh/authorized_keys
```
Da ich mich auch bei diesem Mal via SSH Anmelde muss ich sicherstellen dass ich beim ausführen von `sudo` befehlen nicht nach eine passwort gefragt werde, welches mir nicht bekannt ist. Dazu führe ich dem befehl `sudo visudo` aus. Nun sollte sich eine Datei öffnen in der ich eine eile hinzufüge, die meinen neu erstellten User berechtigt `sudo` Befehle ohne eine Passworteingabe auszuführen.
```bash
<user> ALL=(ALL) NOPASSWD: ALL
```
Nachdem ich getestet habe, dass ich micht auch wirklich mit dem User anmelden kann und ich auch alle nötigen Rechte habe muss ich noch sichergehen, dass man sich nichtmehr mit dem `root` User anmelden kann. Dazu editiere ich mit vime das file `/etc/ssh/sshd_config`. In ihm sind alle Konfigurationen für den SSH Client drinnen. Ich füge nun eine Zeile hinzu, welche root login via `ssh` verbietet.
```bash
PermitRootLogin no
```
Nun muss ich nur noch den `ssh` daemon restartet, dammit die changes aktiv werden.
```bash
sudo systemctl restart sshd
```
3. Update Packagemanager <br/>
Linux benutzt ein Packagemanager. Im falle von Debian ist das `apt` (Advanced Packaging Tool). Der Packagemanager besitzt eine lokale Liste von allen Packages, welche auf den Server die er kennt verfügbar sind. Diese Liste ändert sich von Zeit zu Zeit. Um nun immer die aktuellste Liste zu haben kann man den Befehl `sudo apt-get update` ausführen. Dadurch wird die aktuellste Liste von dem Server gesynct. 

4. Installieren vom Webserver <br/>
Als Webserver benutzte ich `nginx`. Dieser kann man ganz einfach mithilfe des Packagemanagers installieren.

```bash
sudo apt-get install nginx
```
Nachdem die Instatlation durchgelaufen ist sollte einen eine default Page von NGINX begrüssen wenn man die Serverip aufruft. 

5. Hinzufügen einer Domain <br/>
In meinem nächsten Schritt habe ich eine Subdomain eingerichtet, bei welchem die Domain auf die Server Ip addrese Zeigt. Meine Domains verwalte ich über Cloudflare, dadurch kann ich auch gleich den Traffic durch Cloudflare proxien lasse um DDoS Angriffe abzuweheren. Ich habe also einen `A` Eintrag hinzugefügt, welcher auf die IPv4 zeit und einen `AAAA` der auf die IPv6 zeugt.

6. Configurieren der Domain <br/>
Nachdem ich die Domain `nr3.bbzbl-it.dev` so konfiguriert habe, dass sie auf den Server zeigt habe ich mich daran gemacht, diese auf dem Server zu konfigurieren. Dadurch habe ich zuerst eine neue Konfigurationsdatei für NGRX hinzugefügt (`/etc/nginx/sites-available/nr3.bbzbl-it.conf`). In dem File füge ich dann die Configuration für den Server hinzu.
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html/nr3.bbzbl-it.dev;
    server_name nr3.bbzbl-it.dev;
}
```
Als nächstes muss ich noch die defaultconfig entfernen die im file `/var/www/html/default`, da es sonst zu conflikten mit meiner configruation kommen könnte. Nach einem restart `sudo systemctl restart nginx` sollte nun die neuen Configruationen aktive sein. 

7. Enfügen der Website <br/>
Im nächsten Schritt muss ich meine Website die aktuell auf [GitHub](https://github.com/bbzblit/modul-293-project) ist irgendwie auf den Webserver bringen. Glücklicherweise kann man mit `apt` ganz einfach die git commandline Tools installieren um nachher das Projekt zu clonen
```bash
sudo apt-get install git
git clone https://github.com/bbzblit/modul-293-project.git
sudo mv modul-293-project/src/ /var/www/html/nr3.bbzbl-it.dev/
```
Wenn man nun den befehl `curl localhost` ausführt sollte die Website nun erreichbar sein. Allerdings ist sie noch nicht von CloudFlare aus erreichbar. 

8. Hinzufügen eines SSL Zertifikates  <br/>
Im nächsten Schritt muss man noch ein SSL Zertifikat konfigurieren, dass die Website auch über https her erreichbar ist. Natürlich sollte das kein Selbstsigniertes Zertifikat sein. Desshabl werde ich Lets Encrypt verwenden, um kostenlos ein Signiertes Zertifikat zu erstellen. Dafür muss ich zuerst die Nötigen packages installieren. 

```
sudo apt-get install certbot -y
sudo apt-get install python3-certbot-nginx -y
```
Danach generiere ich mir mit Certbot ein SSL Zertifikat. Der Vorteil davon ist, dass es auch gleich die nötigen Configurationen für den Ngrix Server einfügt. Wichtig ist, dass die Domaineinträge dafür schon exisiteren, da sonst nicht validiert werden kann dass man auch der Eigentümmer der Domain ist.
```
sudo certbot --nginx -d nr3.bbzbl-it.dev
```
Nach dem erfollgreichen erstellen sollte nun die Website auch über CloudFlare erreichbar sein.

9. Einrichten der Firewall <br/>
Damit meine Website von ausen erreichbar ist muss nur auf einen Port `443` zugegriffen werden. In der aktuellen Konfiguration ohne Firewall sind allerdings alle Ports geöffnet. Das kann zum Problem werden, wenn eine Applikation im Hinzergrund läuft von der wir nichts wissen. Desshalb sollte man nur Ports öffnen welche man zwingend braucht. Als firewall verwende ich hier `ufw` Uncomplicated Firewall da diese sehr leicht zu konfigurieren ist und für eine Website ohne Problem ausreichend ist.
```bash
sudo apt-get install ufw
```
Alls erstes erlauben wir `ssh` und `https`. Es ist sehr wichtig zu beachten dass man ssh erlaubt befor die firewall aktiviert wird, da defaultmässig **allen** eingehender Traffik blockiert wird.
```bash
sudo ufw allow ssh
sudo ufw allow https
sudo ufw enable
```
Dadurch sollte nun die Website unter [https://nr3.bbzbl-it.dev/](https://nr3.bbzbl-it.dev/) erreichbar sein.

## NR ? Aufsetzen eines Minecraft Server
Ich benutze für den Minecraft Server ein server mit `1` vCPU `2 GB` Ram und `20 GB` SSD Storage. Als  Betriebsystem verwende ich auch hier `Debian 11`. Die ersten 3 Schritte die ich auf dem Server ausgeführt haben sind exakt die gleichen wie bei `NR. `3` weshabl ich sie nicht nocheinmal dokumentiere.

