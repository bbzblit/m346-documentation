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



## Nr 4 Factorial als FaaS in Azure
1. Erstellen einer neuen Funktions App <br/>
Alls erstes musste ich bei Azure eine neue Function-App erstellen. Das ging in meinem Fall ganz einfach über die home page. Alls Programmiersprache habe ich Python ausgewählt, da man dabei fast keine Einschränkungen hat beim Rechnen mit Zahlen (keine Typisierung oder limits wie bei JS).  
![image](https://user-images.githubusercontent.com/99135388/197341160-b398b329-58ef-4abd-acef-dec02832ef7c.png)


2. Erstellen der Funktion <br/>
Als nächstes habe ich zur Funktions app eine neue Funktion hinzugefügt. Als Trigger der Funktion habe ich `http request` ausgewählt. Zudem hbe ich noch eingestellt, dass ich das ganze im Azure Portal entwicklen werde. Ich persönlich finde es angenehmen für einfache Funktionen diese direckt im Portal zu entwicklen. Im letzten Punkt (`Authorization level`) habe ich `function` ausgewählt. Dadurch muss man einen Code beim Aufrufen der URL mitsenden. Dadurch können nur berechtigte Personen auf die Funktion zugreifen. 
![image](https://user-images.githubusercontent.com/99135388/197341196-0050ee5a-aeea-4469-a0b4-8da12aef5a31.png)

Danach habe ich ein Script geschrieben, dass die Fakultät von einer belibigen Nummer ausrechnet. Wie bereits gesagt, hat gibt es in Python keine Limets was Zahlen angehet. Aus diesem Grund validiere ich auch ob die Zahl kleiner oder gleich 200 ist. Das solte einfach verhinderen, dass die Funktion mit extrem grossen Zahlen aufgerufen wird, die möglicherweise zu lange dauern um ausgeführt zu werden und dadurch zu hohe Kosten verursachen. 

```python
import logging

import math

import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    number = req.params.get('number', "")
    if number == "":
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            number = req_body.get('number', "")
    if number == "":
        return func.HttpResponse("You have to provide a number", status_code=400)
    if not number.isnumeric():
        return func.HttpResponse("The number must be an integer", status_code=422)
    if len(number) > 3:
        return func.HttpResponse(f"The number must be positive and less than 200 your number {number} (len error) ", status_code=422)
    number = int(number)
    if number < 0 or number > 200:
        return func.HttpResponse(f"The number must be positive and less than 200 your number {number} ", status_code=422)

    return func.HttpResponse(f"The factorial of {number}! is {math.factorial(number)}")
```
Mir ist durchaus bewusst, dass man das ganze auch gut als recursive Funktion schreiben. Allerings bin ich nicht so ein Freund von recursiven Funktionen und ausserdem ist da noch [dieser Grund](https://www.electropages.com/blog/2021/01/how-start-received-75000-bill-2-hours-google-cloud-services)

Nun sollte die Funktion fertig sein. Sie hat die beiden URL-Paramter `code` und `number`. Mit `code` muss der access code mitgegeben werden und mit `number` die Zahl von der man die Fakultäte ausrechnen möchte.
```bash
https://python-faas.bbzbl-it.dev/api/fakultaet?code=<Access-Code>&number=<Number>
```
## Nr 4 Part 2
Die Funktion ist nun voll funktionsfähig. Allerdings finde ich es nicht sonderlich schön eine Subdomain von Azure zu verwenden. Viel lieber möchte ich meine eigene Domain hinzufügen. 
<br/>
1. Hinzufügen einer Domain bei Azure <br/>
In Azure kann man bei einer Funktions App unter dem Tap `Benutzerdefinierte Domain` eine eigene Domain hinzufügen. Nach dem hinzufügen muss man einen sogenannten `CNAME` eintrag auf die bestehende Azure subdomain machen. Da ich auch in dieser Aufgabe wieder meinen gesammten Traffic über CloudFlare Proxien lassen werde, kann Azure nicht verfizieren, ob die Domain auch wirklich hinzugefügt worden ist. Um das zu verfizieren muss ich noch einen sogennanten `TXT` eintrag machen. Der Context vom `TXT` eintrag ist ein ein Prüfcode. 

2. Hnzufügen eines Zertifikates <br/>
Der Traffic von PC zu CloudFlare ist automatisch verschlüsselt. Allerdings braucht die Funktions App auch ein eigenes SSL Zertifikat, damit der Traffic zwischen CloudFlare und Azure auch verschlüsselt ist. Natürlich könnte ich jetzt ein LetsEncrypt Zertifikat beantragen und über `TXT` Einträge bestätigen, dass ich Eigentümmer der Domain bin. Allerdings währe das Overkill. Cloudflare bietet sogenannte `Origin Server` Zertifikate an. Dabei handel es sich um Zertifikate, welche nur zwischen Cloudflare und Azure gültig sind. Diese werden für alle anderen PCs als nicht vertrauenswürdig angezeigt. Der Voreil besteht darin, dass im Falle eines leakes des Zertifikates dieses ohne Probleme ausgetauscht werden kann und das ohne dass ein Valdes Zertifikat im Umlauf ist. 

![image](https://user-images.githubusercontent.com/99135388/197341902-fa4fc09f-46e5-4da9-bee6-98f02d0cf7fb.png)


Dieses Zertifikat musste ich anschlissend nur noch in ein PFX Certificat umwndeln und auf Azure hochladen. Das umwalden in ein pfx file kann man mithilfe von OpenSSL glüclicherweise ganz einfach anpassen. 
```bash
openssl pkcs12 -export -out ./azure_cert.pfx -inkey ./cloudflare_cert.pem -in ./cloudflare_cert.crf -legacy
```
Anfangs hat Azure mein `pfx` Zertifikat nicht erkannt. Ich dachte zuerst es läge daran, dass ich das Password falsch eingegeben habe. Nach einer kurzen Recherche ist mir dann alerdings aufgefallen, dass OpenSSL 3+ nicht mehr defaultmässig `DES encryption` benutzt. Um dies denoch anwenden zu könnten kann man die Flag `-legacy` mitgeben.




## NR 5. Primefaktor Zerlegungs Funktion schreiben
1.  Erstellen eines Funktion <br/>
Im ersten Schritt habe ich eine neue Funktion zu meiner Funktions App welche ich in der letzen Aufgabe erstellt habe hinzugefügt.  Auch hier benutzte ich als Trigger für die Fuktion `http` request. 
![image](https://user-images.githubusercontent.com/99135388/197406220-f560b117-0d13-43e3-af24-1e6bfc384629.png)

2. Erstellen des Scriptes <br/>
Im nächsten Schritt habe ich ein Script erstellt, welches alle Primfaktoren von einer gegebenen Zahl ausrechnet. Das Script sollte relativ selbsterklärend sein. Als erstes wird validiert, ob die Funktion im richtigen Format ist. Falls dies nicht der Fall sein sollte kommt eine Response mit einer Anweisung, was falsch gelaufen ist. Im unteren Teil des Scriptes werden dann die Primefaktoren ausgerechnet. Die Spezialität besteht darin, dass ich zuerst soweit teile, bis die Zahl nicht mehr durch `2` (erster Primfaktor) teilbar ist. Anschlissend iterier ich durch alle Primfaktoren zwischen 3 und der wurzel aus der Zahl in 2er Schritten. Nun könnte man sich natürlich fragen, wie ich validiere ob es sich bei der Zahl um eine Primzahl handelt. Die Antwort lautet garnicht. Bei der Primfaktorenzerlegung stellt man eine Zahl so dar, dass möglichst viele Natürliche Zahlen miteinander verrechnet werden. Dadurch **muss** es sich immer um Primzahlen handeln. Im letzen Schritt Schau ich dann noch, ob es sich bei der Zahl selber um eine Primzahl handelt. Und schon habe ich alle Primfaktoren ausgerechnet. 
```python
import logging

import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    number = req.params.get('number', "")
    if not number:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            number = req_body.get('number', "")

    if not number:
        return func.HttpResponse(
                "Please pass a number on the query string or in the request body",
                status_code=400     
        )
    if not number.isnumeric():
        return func.HttpResponse(
                "Please pass a number on the query string or in the request body",
                status_code=400     
        )
    number = int(number)
    if number < 0:
        return func.HttpResponse(
                "Please pass a positive number on the query string or in the request body",
                status_code=400     
        )
    
    #Calculation Factors
    factors = []
    while number % 2 == 0:
        factors.append(2)
        number //= 2
    for i in range(3, int(number ** 0.5) + 1, 2):
        while number % i == 0:
            factors.append(i)
            number //= i
    if number > 2:
        factors.append(number)
    return func.HttpResponse(str(factors))
```

3. Hinzufügen eines SSL Zertifikates <br/>
In dieser Aufgabe musste ich kein SSL Zertifikat erstellen, da ich die Funktionsapp von der letzen Aufgabe verwende und dadurch die gleiche Domain Verwende. Ansonsten habe ich unter `NR 4 Part 2` eine Ausführliche Beschreibung, wie ich das SSL Zertifikat hinzugefügt habe.
<br/>
Das Fertige Produkt ist unter Folgenden URL Erreichbar:
```
https://python-faas.bbzbl-it.dev/api/primefactor?code=<Access-Code>&number=<Number>
```

## NR 7
1. Erstellen von mehreren Funktions Apps Inklusive SSL Zertifikaten <br/>
Als erstes musste ich mehrere neue Funktions Apps erstellen (Für jede Programiersprache eines). Das ganze habe ich schon bei `NR 4 Part 1` (Erstellen von einer Funktion) und bei `NR 4 Part 2` (Hinzufügen einer Domain/SSL Zertifikat) documentiert. Aus diesem Grund werde ich hier nichtmehr genauer darauf eingehen

2. Erstellen des FizzBuzz Scripts <br/>
Im nächsten Schritt habe ich mich daran gemacht FizzBuzz in allen Programiersprachen zu implementieren. Im allen Scripts validiere ich am Anfang, ob die gegebene Zahl im richtigen Format ist. Anschlissend caste ich diese (falls nötig abhängig von der Sprache). Im letzen Schritt überprüfe ich mithilfe des Modulus Opperators `%` ob die Zahl teilbar ist. Der Modulusoperator ist ein Operator, mit dessen Hilfe man in der Lage ist wie gross der Restanteil bei einer Division ist. Dadurch eigenet er sich ideal, um zu testen ob eine Zahl durch eine andere Zahl teilbar ist.
```
10 % 2 -> 0
11 % 2 -> 1
11 % 3 -> 2
```
<br/>
<br/>
Grösere Probleme sind mir bei der Aufgabe nicht begegenet. Je nach Programiersprache hat sich immer mal wieder einen tippfehler eingeschlichen, denn ich aber fürher oder später behoben habe. Ein kleineres Problem ist bei Java aufgetreten. Java ist die einzige Programiersprache, die ich lokal programmieren musste, da der Webeditor bei Azure sie nicht supportet hat. Dadurch musste ich zuerst lokal ein Projekt erstellen und dieses anschlissend auf Azure puschen. Das Erstellen und Pushen des Projektes verlief auch ohne Schwirigkeiten. Nach dem Deployen ist allerdings meine Funktionen beim Aufrufen immer in ein Timeout gelaufen. Zuerst hat das ganze für mich gar keinen Sinn gegeben, da Local alles funktioniert hat. Allso musste ich herausfinden, was sich geändert hat. Nach einer kuzen Vergleich ist mir aufgefallen, dass ich beim Projekt die Falsche JavaVersion angegeben habe. Nachdem ich dies behoben habe hat alles wieder normal funktioniert. Bei JavaScript habe ich mir zudem noch einen kleinen Spass erlaubt und das ganze in JSFuck implementiert (natürlich durch ein encoder). 

### Bonus
Als kleiner Bonus habe ich das ganze noch mit PowerShell implementiert. Mein erster Gedanke war, dass es extrem nervig sein wird das ganze mit Powershell zu implementieren. Nachdem ich mir allerdings ein paar beispiele angeschaut habe ist mir aufgefallen, dass es vom Syntax her gar nicht so speziell ist (nicht so wie bash). Mit der Hilfe von Stackoverflow ist mir dann das ganze Umschreiben auch relativ einfach gefallen.


## Python
```python
import logging

import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    number = req.params.get('number', "")
    if not number:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            number = req_body.get('number', "")

    if not number:
        return func.HttpResponse(
                "Please pass a number on the query string or in the request body",
                status_code=400     
        )
    if not number.isnumeric():
        return func.HttpResponse(
                "Please pass a number on the query string or in the request body",
                status_code=400     
        )
    number = int(number)
    if number < 0:
        return func.HttpResponse(
                "Please pass a positive number on the query string or in the request body",
                status_code=400     
        )
    factors = []
    while number % 2 == 0:
        factors.append(2)
        number //= 2
    for i in range(3, int(number ** 0.5) + 1, 2):
        while number % i == 0:
            factors.append(i)
            number //= i
    if number > 2:
        factors.append(number)
    return func.HttpResponse(str(factors))
```

## JavaScript (in JS Fuck)

```jsfuck
[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(+[![]]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(!+[]+!+[]+!+[]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[!+[]+!+[]])+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]])()([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+([]+[])[(![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]()[+!+[]+[!+[]+!+[]]]+((!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[!+[]+!+[]]+([][[]]+[])[+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+[]]+([][[]]+[])[+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[+!+[]]+[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+[]]+([][[]]+[])[+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+([][[]]+[])[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+([][[]]+[])[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]]+[+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+([][[]]+[])[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+([][[]]+[])[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(![]+[])[+!+[]]+(!![]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[+!+[]]+[+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[+[]]+[!+[]+!+[]]+([][[]]+[])[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[+[]]+[!+[]+!+[]]+([][[]]+[])[+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(+!+[]+[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+[!+[]+!+[]]+[+[]])+[])[+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([][[]]+[])[!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]]+(!![]+[])[+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]])[(![]+[])[!+[]+!+[]+!+[]]+(+(!+[]+!+[]+[+!+[]]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[+!+[]])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]]((!![]+[])[+[]])[([][(!![]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]](([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(![]+[+[]])[([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()[+!+[]+[+[]]]+![]+(![]+[+[]])[([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()[+!+[]+[+[]]])()[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((![]+[+[]])[([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()[+!+[]+[+[]]])+[])[+!+[]])+([]+[])[(![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]()[+!+[]+[!+[]+!+[]]])())
```
## JavaScript (Befor encoden in JSFuck)
```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    let number = (req.query.number || (req.body && req.body.number));
    let responseMessage = "";
    let statusCode = 200;
    if (!number) {
        statusCode = 422;
        responseMessage = "Please pass a number on the query string or in the request body";
    }
    const format = RegExp('^[0-9]$');
    if (!format.test(number)) {
        statusCode = 422;
        responseMessage = "The number is not in the correct format (0-9)";
    }
    number = parseInt(number);
    if (number % 15 == 0) {
        responseMessage = "FizzBuzz";
    }
    else if (number % 3 == 0) {
        responseMessage = "Fizz";
    }
    else if (number % 5 == 0) {
        responseMessage = "Buzz";
    }
    else {
        responseMessage = number.toString();
    }
    context.res = {
        body: responseMessage
    };
}
```
## PowerShell

```powershell
using namespace System.Net

param($Request, $TriggerMetadata)

Write-Host "PowerShell HTTP trigger function processed a request."

$number = $Request.Query.Number
if (-not $number) {
    $number = $Request.Body.Number
}


if (-not $number -or -not ($number -match "^[0-9]*$")) {
    $body = "Please pass a number on the query string or in the request body in the format ^[0-9]*$ "
    $StatusCode = [HttpStatusCode]::UnprocessableEntity
}
else {
    if ($number % 15 -eq 0) {
        $body = "FizzBuzz"
    }
    elseif ($number % 3 -eq 0) {
        $body = "Fizz"
    }
    elseif ($number % 5 -eq 0) {
        $body = "Buzz"
    }
    else {
        $body = $number
    }
    $StatusCode = [HttpStatusCode]::OK
}

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = $StatusCode
    Body = $body
})

```
## Java
```java
package dev.bbzblit;

import com.microsoft.azure.functions.ExecutionContext;
import com.microsoft.azure.functions.HttpMethod;
import com.microsoft.azure.functions.HttpRequestMessage;
import com.microsoft.azure.functions.HttpResponseMessage;
import com.microsoft.azure.functions.HttpStatus;
import com.microsoft.azure.functions.annotation.AuthorizationLevel;
import com.microsoft.azure.functions.annotation.FunctionName;
import com.microsoft.azure.functions.annotation.HttpTrigger;

import java.util.Optional;

/**
 * Azure Functions with HTTP Trigger.
 */
public class Function {


    @FunctionName("fizzbuzz")
    public HttpResponseMessage fizzBuzz(
            @HttpTrigger(
                name = "fizzbuzz",
                methods = {HttpMethod.GET, HttpMethod.POST},
                authLevel = AuthorizationLevel.FUNCTION)
                HttpRequestMessage<Optional<String>> request,
            final ExecutionContext context) {
        context.getLogger().info("Java HTTP trigger processed a request.");

        // Parse query parameter
        final String query = request.getQueryParameters().get("number");
        final String number = request.getBody().orElse(query);

        if (number == null) {
            return request.createResponseBuilder(HttpStatus.UNPROCESSABLE_ENTITY).body("Please pass a number on the query string or in the request body").build();
        }
        if (!number.matches("^\\d+$")) {
            return request.createResponseBuilder(HttpStatus.UNPROCESSABLE_ENTITY).body("Please pass a number on the query string or in the request body").build();
        }
        int n = Integer.parseInt(number);
        if (n % 15 == 0) {
            return request.createResponseBuilder(HttpStatus.OK).body("FizzBuzz").build();
        }
        if (n % 3 == 0) {
            return request.createResponseBuilder(HttpStatus.OK).body("Fizz").build();
        }
        if (n % 5 == 0) {
            return request.createResponseBuilder(HttpStatus.OK).body("Buzz").build();
        }
        return request.createResponseBuilder(HttpStatus.OK).body(number).build();
    }
}

```

Die Funktionen sind nun unter diesen URLs auffindbar. Bei allen Endpunkten muss man mit dem `code` parameter einen accesscode mitgeben, um überhaupt die Funktion benutzen zu können.
```
python-faas.bbzbl-it.dev/api/fizzbuzz?code=<AccessCode>&number=<Your Number>

java-faas.bbzbl-it.dev/api/fizzbuzz?code=<AccessCode>&number=<Your Number>

js-faas.bbzbl-it.dev/api/fizzbuzz?code=<AccessCode>&number=<Your Number>

python-faas.bbzbl-it.dev/api/fizzbuzz?code=<AccessCode>&number=<Your Number>
```

## 8. Eine API Aufrufen aus einer Azure funktion
Ich habe mir lange Zeit überlegt, welche API ich aus einer Azure funktion heraus aufrufen könnte. Ich wollte nicht so lösen, dass ich einfach eine Simple api habe die irgendwelche Daten, welche ich dann fast 1 zu 1 an den Benutzer der API sende. Als ich meine alten Projekte durchgegangen bin ist mir eine Idee gekommen. Für ein früheres Projekt (M293) habe ich bereits einmal ein Tool mit JS geschrieben, mit dem man Bitcoin addressen generieren kann. Dabei habe ich gelernt, dass addressen, welche generiert wurden in der Theorie auch nocheinmal generiert werden könnten. Die warscheindlichkeit dafür beträgt allerdings nur etwas weniger als `106000000 zu 2^160 oder 1.461501637×10^48`. Das klingt zwar erstmal extrem unwarscheindlich (was es auch ist). Wenn man es allerdings mit der Warscheindlichkeit vergleicht,  das ein normales 52 Kartenspiel in einer bestimmten Reienfolge gemischt wird ist es viel warscheindlicher (`52!` oder `8.065817517×10^67`).

1. Erstellen einer Azure Funktion <br/>
Im ersten Schritt erstelle ich eine neue Azure funktion wie ich es auch schon mehrmals in den letzen beiden Aufgaben Dokumentiert habe. Ich habe mich dazu entschieden in diesem Beispiel wieder Python einzusetzen. Klar hätte ich auch mein JavaScript Projekt aus dem letzen Modul etwas abändern könnten und einfach in die Cloud stellen. Das währe aber relativ langweilig gewesen.

2. Erstellen eines Scriptes <br/>


<!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
<!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
<!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
<!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->

## NR ? Aufsetzen eines Minecraft Server
Ich benutze für den Minecraft Server ein server mit `1` vCPU `2 GB` Ram und `20 GB` SSD Storage. Als  Betriebsystem verwende ich auch hier `Debian 11`. Die ersten 3 Schritte die ich auf dem Server ausgeführt haben sind exakt die gleichen wie bei `NR. `3` weshabl ich sie nicht nocheinmal dokumentiere.

4. Installieren von Java <br/>
Da Minecraft Server eine Java application ist musste ich zuerst Java installieren. Ich möchte am Ende die neuste Version von Minecraft installieren wesshalb ich die Java version `17` installieren muss.
```bash
sudo apt-get install openjdk-17-jdk
```

5. Herunterladen von Minecraft Server <br/>
Im nächsten Schritt habe ich die executable für Minecraft Server heruntergelade. Das geht ganz einfach über `wget`
```bash
wget https://piston-data.mojang.com/v1/objects/f69c284232d7c7580bd89a5a4931c3581eae1378/server.jar
```

6. Ausführen vom Minecraft Server <br/>
Wenn man über ssh auf einen Server zugreift ist das in etwa so wie wenn man auf einem Pc eine neue Konsole öffnet. Sobald die SSH Verbindung geschlossen wird wird auch automatisch die Konsole geschlossen. Das ergibt im Normalfall auch durchaus sinn. In diesem Fall allerdings möchte man, dass der Server weiterhin läuft auch wenn man die SSH Verbindung schlisst. Die Lösung für das Problem lautet `tmux`. Mit `tmux` kann man eine konsole in einer konsole erstellen die im Hintergrund läuft. Diese wird auch wenn die SSH Verbindung geschlossen wird nicht beendet. 
```bash
sudo apt-get install tmux
```
Eine neue `tmux` session kann man ganz einfach mit dem command `tmux` erstellen. Dass man sich in einer `tmux` session befindet sieht man daran, dass unten an der Konsole ein grüner Strich ist. Den server kann ich dann ganz einfach aufstarten. Die beiden Parameter `-Xmx` und `-Xms` definieren dabei die maximalen und die initiale heap size
```bash
java -Xmx1024M -Xms1024M -jar server.jar nogui
```
Nachdem der Server das erste mal gestartet ist bricht er schnell nach dem Startvorgang ab. Das liegt daran, dass man die minecraft `Eula` (end user agreement) abkzetieren muss. Das kann man ganz einfach dadurch machen, dass man das file `eula.txt` editiert und `eula=false` auf `eula=true` setzt. Danach kann man den Server einfach erneut aufstarten. <br/>
Nachdem der Server komplet aufgestartet ist kann man `tmux` wieder mit dem Shortcut `ctrl + b` und anschlissend `d` verlassen. Alle aktiven `tmux` sessions kann man nun mit dem command `tmux ls` auflisten und falls man wieder in eine session zurückkehren möchte kann man das mit dem command `tmux attach -t <session id>` machen.

7. Einrichten einer Domain <br/>
Nun da der Server nun läuft habe ich schnell noch eine Subdomain eingerichtet die auf den Server zeigt. Wenn man seine Domains bei Cloudflare verwaltet ist es wichtig, dass man Proxy durch Cloudflare ausschaltet. Das liegt daran, dass in der Freeversion diese Protokole/Ports nicht abgedekt werden.

8. Einrichten der Firewall. <br/>
Da ich diesen Server bei Hetzner eingerichtet habe ich automatisch auch Zungang zu einer kostenlosen externen Firewall und muss nicht `ufw` benutzen. FÜr den Minecraft Server muss ich TCP Traffik auf dem Port `22` für `ssh` und dem port `25565` für den Minecraft Server zulassen
![image](https://user-images.githubusercontent.com/99135388/197341759-93f6d9a4-2762-4061-b24a-64d83342a90b.png)
