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
 
