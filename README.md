# M346-leistungsbeurteilung
## Meine Ziele
Mein Ziel ist es, dass ich alle "Projekte" alle Kompetenzen mit ausnahme von `Nr. 16 Microsoft Certified: Azure Fundamental (AZ-900) absolvieren` erledige. Ich möchte mich damit persönlich im bereich der Cloud weiterbilden. Ich denke, dass ich die ersten Aufgaben durch mein Vorissen schon relativ einfach erledigen kann. ALlerdings habe ich bis jetzt noch keine Erfahrung im Bereich von Serverless Funktionen usw.

## NR 1 Eine statische Webseite erstellen und mit PaaS in der Cloud publizieren.
Ich wollte dafür nicht meine Website aus dem letzen Modul deployen, da ich das bereits in einer anderen Task erledige (NR 3). Aus diesem Grund habe ich mich dafür entschieden einfach diese Dokumentation als IaaS auf GitHub zu deployen. Auch wenn das vielleicht nich ganz dem Ziel entschpricht spielt es keine Rolle, da ich auch ohne diese Aufgabe bereits mehr als genügend gelöst haben sollte.

1. Erstellen einer GitHub Page <br/>
Alls erstes musste ich meine GitHub Page aktivieren. Dass geht ganz einfach unter `Settings` > `Pages`. Als Branch habe ich den Main Branch und den Root folder ausgewählt.  
![image](https://user-images.githubusercontent.com/99135388/197852634-87a6298a-f683-408e-82f4-7163c88fda82.png)

2. Hinzufügen einer Subdomain <br/>
Nach dem Aktivieren von GithubPages sollte das Projekt bereits erreichbar sein. Allerdings ist es relativ langweilig eine Subdomain von Github zu verwenden. Eine eigene Subdomain kann ich ganz einfach unter der gleichen Setting seite hinzufügen, wo ich zuvor auch meinen page akktiviert habe. Nach dem Hinzufügen brauche ich nurnoch einen DNS Eintrag einrichten der auf die alte Subdomain von Github zeigt und fertig ist meine GitHub Page. Es kann sein, dass die Domain anfangs noch nicht erreichbar ist, da Github noch nicht validieren konnte, dass ich der Eigentümmer der Domain sein. Nach spätestens einer halben Stunde (Bei cloudflare dns sonst auch gerne mal 24h) sollte sie aber schon erreichbar sein.

```
m346.bbzbl-it.dev -> github.com
```

3. Aändern des Themes <br/>
Da meine Page aus Markdown generiert wird kann ich auch bedingt mitbestimmen, wie es designt werden sollte (z.B. Hintergrundfarbe usw). Ich bevorzuge `cayman-dark` Theme, welches ich auch für meine anderen Dokumentation verwende. Um dieses hinzuzufügen und zu aktivieren füge ich folgende Zeile code in meinem `_config` file auf GitHUb hinzu.

```
remote_theme: lewismiddleton/cayman-dark
```



## Nr. 2 Einbinden eines Networkdrives
Ich benutze für die Aufgabe einen bereits existierenden Server, auf dem ich aktuell meine Backups von den Server speichere. Ich werde das Filesystem auf meinem lokalen PC mounten. Das aktuelle Betriebsystem von meinem PC ist `Ubuntu 22.04.1 LTS`.

1. Erstellen des Mountpoints <br/>
Als erstes habe ich einen neuen Folder erstellt, an dessen Stelle ich den Networkdrive mounten möchte.

```bash
sudo mkdir /media/M346
```

Damit ich anschlissend auch mit dem Folder interagieren kann, musste ich mir noch die nötigen Berechtigungen geben. Das geht ganz einfach mit dem `chmod` Command.

```bash
sudo chmod 777 /media/M346/
```

2. Umstellen von Passwort Authentifikation zu Public Key Authentifikation <br/>
Im nächsten Schritt könnte ich theoretisch direckt das Filesystem mounten. Allerdings ist der neu erstellte User defaultmässig über ein Passwort geschützt. Dass möchte ich ändern und die Authentifikation über einen Public Key durchführen. Das ist sicherer gegen Phishingattaken und ich muss mir das Passwort nicht merken. Auf dem Server gibt es bereits ein Skript namens `install-ssh-key` mit dem man automatisch SSH Keys installieren kann. Dadurch kann ich über OpenSSH das Ganze ganz einfach installieren. 

```bash
cat ~/.ssh/id_ecdsa.pub | ssh <User>@<ServerIP> install-ssh-key
```
3. Mounten des Filesystems <br/>
Für das Mounten des Filesystems benutze ich `sshfs`. Wie der Name schon sagt, basiert es auf dem Secure Shell Protocoll und ist dadurch nicht die schnellste Möglichkeit. Allerdings ist es relativ sicher und daher gut geeignet um Daten über das Internet zu transportieren.

```bash
sudo sshfs <username>@<server>:<server-directory> /media/M346
```
 

## NR 3 Hosten einer statischen Website in der Cloud
Für diese Aufgabe benutzte ich eine VPS mit `10 GB` Speicher `1` CPU Core und `0.5 GB` RAM. Das sollte für eine kleine Website problemlos ausreichend sein. Als OS benutzte ich `Debian 11`.

1. Login via SSH <br/>
Im ersten Schritt verbinde ich mich mit der VPS via SSH. Als Authentification benutze ich ein 521 Bit langer ECDSA Key.
```bash
ssh root@<ip>
```
2. Erstellen eines neuen Users <br/>
Da es ein Sicherheitsrisiko ist, wenn man sich mit dem Defaultuser anmelden kann, erstelle ich einen neuen User und gebe ihm Sudorechte. Ab diesem Moment  werde ich mich nur noch mit dem neuen User auf dem Server anmelden.

```bash
sudo useradd -m <username>
usermod -aG sudo <username>
mkdir /home/<username>/.ssh
cp .ssh/authorized_keys /home/<username>/.ssh
sudo chown <username> /home/debian/.ssh/authorized_keys
```
Da ich mich auch bei diesem Mal via SSH und Public Key Authentifikation anmelde, muss ich sicherstellen, dass ich beim Ausführen von `sudo` Befehlen nicht nach einem Passwort gefragt werde, welches es gar nicht gibt. Dazu führe ich den Befehl `sudo visudo` aus. Nun sollte sich eine Datei öffnen, in der ich eine Zeile hinzufüge, die meinen neu erstellten User berechtigt `sudo` Befehle ohne eine Passworteingabe auszuführen.
```bash
<user> ALL=(ALL) NOPASSWD: ALL
```
Nachdem ich getestet habe, dass ich mich auch wirklich mit dem User anmelden kann, und ich auch alle nötigen Rechte habe, muss ich noch sichergehen, dass man sich nicht mehr mit dem `root` User anmelden kann. Dazu editiere ich mit vim das File `/etc/ssh/sshd_config`. In ihm sind alle Konfigurationen für den SSH Daemon drinnen. Ich füge nun eine Zeile hinzu, welche das Login für den root User via `ssh` verbietet.
```bash
PermitRootLogin no
```
Nun muss ich nur noch den `ssh` Dämon restarten, damit die Changes aktiv werden.
```bash
sudo systemctl restart sshd
```
3. Update Packagemanager <br/>
Linux benutzt ein Packagemanager. Im Falle von Debian ist das `apt` (Advanced Packaging Tool). Der Packagemanager besitzt eine lokale Liste von allen Packages, welche auf den Server, die er kennt verfügbar sind. Diese Liste ändert sich von Zeit zu Zeit. Um nun immer die aktuellste Liste zu haben, kann man den Befehl `sudo apt-get update` ausführen. Dadurch wird die aktuellste Liste von dem Server gesynct. 

4. Installieren vom Webserver <br/>
Als Webserver benutzte ich `nginx`. Dieser kann man  einfach Mithilfe des Packagemanagers installieren.

```bash
sudo apt-get install nginx
```
Nachdem die Installation durchgelaufen ist, sollte eine default Page von NGINX ersichtlich sein, wenn man die Serverip aufruft. 

5. Hinzufügen einer Domain <br/>
In meinem nächsten Schritt habe ich eine Subdomain eingerichtet, welche auf die Server Ip Adresse zeigt. Meine Domains verwalte ich über Cloudflare, dadurch kann ich auch gleich den Traffic durch Cloudflare proxien lassen, um DDoS Angriffe abzuwehren. Ich habe also einen `A` Eintrag hinzugefügt, welcher auf die IPv4 zeigt und einen `AAAA` der auf die IPv6 erzeugt.

6. Configurieren der Domain <br/>
Nachdem ich die Domain `nr3.bbzbl-it.dev` so konfiguriert habe, dass sie auf den Server zeigt habe ich mich daran gemacht, diese auf dem Server zu konfigurieren. Dadurch habe ich zuerst eine neue Konfigurationsdatei für NGRX hinzugefügt (`/etc/nginx/sites-available/nr3.bbzbl-it.conf`). In dem File füge ich dann die Konfiguration für den Server hinzu.
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html/nr3.bbzbl-it.dev;
    server_name nr3.bbzbl-it.dev;
}
```
Als nächstes muss ich noch die defaultconfig entfernen die sich im File `/var/www/html/default`befinden, da es sonst zu Konflikten mit meiner Konfigruation kommen könnte. Nach einem Restart `sudo systemctl restart nginx` sollten nun die neuen Konfigruationen aktive sein. 

7. Enfügen der Website <br/>
Im nächsten Schritt muss ich meine Website die aktuell auf [GitHub](https://github.com/bbzblit/modul-293-project) ist, irgendwie auf den Webserver bringen. Glücklicherweise kann man mit `apt` ganz einfach die git commandline Tools installieren, um nachher das Projekt zu clonen
```bash
sudo apt-get install git
git clone https://github.com/bbzblit/modul-293-project.git
sudo mv modul-293-project/src/ /var/www/html/nr3.bbzbl-it.dev/
```
Wenn man nun den Befehl `curl localhost` ausführt, sollte die Website  erreichbar sein. Allerdings ist sie noch nicht von CloudFlare aus erreichbar. 

8. Hinzufügen eines SSL Zertifikates  <br/>
Im nächsten Schritt muss man noch ein SSL Zertifikat konfigurieren, das die Website auch über https her erreichbar ist. Natürlich sollte das kein selbstsigniertes Zertifikat sein. Deshalb werde ich Lets Encrypt verwenden, um kostenlos ein signiertes Zertifikat zu erstellen. Dafür muss ich zuerst die nötigen Packages installieren. 

```
sudo apt-get install certbot -y
sudo apt-get install python3-certbot-nginx -y
```
Danach generiere ich mir mit Certbot ein SSL Zertifikat. Der Vorteil davon ist, dass es auch gleich die nötigen Konfigurationen für den Ngrix Server einfügt. Wichtig ist, dass die Domaineinträge dafür schon exisiteren, da sonst nicht validiert werden kann. Und damit man auch Bestätigen kann, dass man der Eigentümmer der Domain ist.
```
sudo certbot --nginx -d nr3.bbzbl-it.dev
```
Nach dem erfollgreichen Erstellen sollte nun die Website auch über CloudFlare erreichbar sein.

9. Einrichten der Firewall <br/>
Damit meine Website von aussen erreichbar ist, muss nur auf einen Port `443` zugegriffen werden. In der aktuellen Konfiguration ohne Firewall sind allerdings alle Ports geöffnet. Das kann zum Problem werden, wenn eine Applikation im Hinzergrund läuft von der wir nichts wissen. Desshalb sollte man nur Ports öffnen welche man zwingend braucht. Als Firewall verwende ich hier `ufw` Uncomplicated Firewall da diese sehr leicht zu konfigurieren und für eine Website ohne Problem ausreichend ist.
```bash
sudo apt-get install ufw
```
Als erstes erlauben wir `ssh` und `https`. Es ist sehr wichtig zu beachten, dass man ssh erlaubt, bevor die Firewall aktiviert wird, da defaultmässig **allen** eingehender Traffik blockiert wird.
```bash
sudo ufw allow ssh
sudo ufw allow https
sudo ufw enable
```
Dadurch sollte nun die Website unter [https://nr3.bbzbl-it.dev/](https://nr3.bbzbl-it.dev/) erreichbar sein.



## Nr 4 Factorial als FaaS in Azure
1. Erstellen einer neuen Funktions App <br/>
Alls erstes musste ich bei Azure eine neue Function-App erstellen. Das ging in meinem Fall ganz einfach über die Homepage. Als Programmiersprache habe ich Python ausgewählt, da man dabei fast keine Einschränkungen hat beim Rechnen mit Zahlen (keine Typisierung oder limits wie bei JS).  
![image](https://user-images.githubusercontent.com/99135388/197341160-b398b329-58ef-4abd-acef-dec02832ef7c.png)


2. Erstellen der Funktion <br/>
Als nächstes habe ich in der Funktions App eine neue Funktion hinzugefügt. Als Trigger der Funktion habe ich `http request` ausgewählt. Zudem habe ich noch eingestellt, dass ich das Ganze im Azure Portal entwickeln werde. Ich persönlich finde es angenehmer, einfache Funktionen direkt im Portal zu entwicklen. Im letzten Punkt (`Authorization level`) habe ich `function` ausgewählt. Dadurch muss man einen Code beim Aufrufen der URL mitsenden. Dadurch können nur berechtigte Personen auf die Funktion zugreifen. 
![image](https://user-images.githubusercontent.com/99135388/197341196-0050ee5a-aeea-4469-a0b4-8da12aef5a31.png)

Danach habe ich ein Script geschrieben, dass die Fakultät von einer belibigen Nummer ausrechnet. Wie bereits gesagt, gibt es in Python keine Limets was Zahlen angeht. Aus diesem Grund validiere ich auch, ob die Zahl kleiner oder gleich 200 ist. Das sollte einfach verhinderen, dass die Funktion mit extrem grossen Zahlen aufgerufen wird, die möglicherweise zu lange dauern, um ausgeführt zu werden und dadurch zu hohe Kosten verursachen. 

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
Mir ist durchaus bewusst, dass man das Ganze auch gut als recursive Funktion schreiben. Allerings bin ich nicht so ein Freund von recursiven Funktionen und ausserdem ist da noch [dieser Grund](https://www.electropages.com/blog/2021/01/how-start-received-75000-bill-2-hours-google-cloud-services)

Nun sollte die Funktion fertig sein. Sie hat die beiden URL-Paramter `code` und `number`. Mit `code` muss der Access Code mitgegeben werden und mit `number` die Zahl von der man die Fakultäte ausrechnen möchte.
```bash
https://python-faas.bbzbl-it.dev/api/fakultaet?code=<Access-Code>&number=<Number>
```
## Nr 4 Part 2
Die Funktion ist nun voll funktionsfähig. Allerdings finde ich es nicht sonderlich schön eine Subdomain von Azure zu verwenden. Viel lieber möchte ich meine eigene Domain hinzufügen. 
<br/>
1. Hinzufügen einer Domain bei Azure <br/>
In Azure kann man bei einer Funktions App unter dem Tap `Benutzerdefinierte Domain` eine eigene Domain hinzufügen. Nach dem hinzufügen muss man einen sogenannten `CNAME` eintrag auf die bestehende Azure Subdomain machen. Da ich auch in dieser Aufgabe wieder meinen gesammten Traffic über CloudFlare Proxien lassen werde, kann Azure nicht verfizieren, ob die Domain auch wirklich hinzugefügt worden ist. Um das zu verifizieren, muss ich noch einen sogenannten `TXT` Eintrag machen. Der Context vom `TXT` Eintrag ist ein Prüfcode. 

2. Hinzufügen eines Zertifikates <br/>
Der Traffic von PC zu CloudFlare ist automatisch verschlüsselt. Allerdings braucht die Funktions App auch ein eigenes SSL Zertifikat, damit der Traffic zwischen CloudFlare und Azure auch verschlüsselt ist. Natürlich könnte ich jetzt ein LetsEncrypt Zertifikat beantragen und über `TXT` Einträge bestätigen, dass ich Eigentümmer der Domain bin. Allerdings wäre das Overkill. Cloudflare bietet sogenannte `Origin Server` Zertifikate an. Dabei handelt es sich um Zertifikate, welche nur zwischen Cloudflare und Azure gültig sind. Diese werden für alle anderen PCs als nicht vertrauenswürdig angezeigt. Der Vorteil besteht darin, dass im Falle eines Leakes des Zertifikates dieses ohne Probleme ausgetauscht werden kann. Zudem ist es nach dem Austausches nicht mehr gültig, weshalb es nutzlos wird.

![image](https://user-images.githubusercontent.com/99135388/197341902-fa4fc09f-46e5-4da9-bee6-98f02d0cf7fb.png)


Dieses Zertifikat musste ich anschliessend nur noch in ein PFX Zertifikat umwandeln und auf Azure hochladen. Das Umwandeln in ein pfx File kann man Mithilfe von OpenSSL glücklicherweise ganz einfach anpassen. 
```bash
openssl pkcs12 -export -out ./azure_cert.pfx -inkey ./cloudflare_cert.pem -in ./cloudflare_cert.crf -legacy
```
Anfangs hat Azure mein `pfx` Zertifikat nicht erkannt. Ich dachte zuerst es läge daran, dass ich das Password falsch eingegeben habe. Nach einer kurzen Recherche ist mir dann allerdings aufgefallen, dass OpenSSL 3+ nicht mehr defaultmässig `DES encryption` benutzt. Um dies dennoch anwenden zu können, kann man die Flag `-legacy` mitgeben.




## NR 5. Primefaktor Zerlegungs Funktion schreiben
1.  Erstellen einer Funktion <br/>
Im ersten Schritt habe ich eine neue Funktion zu meiner Funktions App, welche ich in der letzen Aufgabe erstellt habe, hinzugefügt.  Auch hier benutzte ich als Trigger für die Fuktion `http` request. 
![image](https://user-images.githubusercontent.com/99135388/197406220-f560b117-0d13-43e3-af24-1e6bfc384629.png)

2. Erstellen des Scriptes <br/>
Im nächsten Schritt habe ich ein Script erstellt, welches alle Primfaktoren von einer gegebenen Zahl ausrechnet. Das Script sollte relativ selbsterklärend sein. Als erstes wird validiert, ob die Funktion im richtigen Format ist. Falls dies nicht der Fall sein sollte, kommt eine Response mit einer Anweisung, was falsch gelaufen ist. Im unteren Teil des Scriptes werden dann die Primfaktoren ausgerechnet. Die Spezialität besteht darin, dass ich zuerst soweit teile, bis die Zahl nicht mehr durch `2` (erster Primfaktor) teilbar ist. Anschliessend iteriere ich durch alle ungeraden Zahlen zwischen 3 und der Wurzel aus der Zahl in 2er Schritten. Nun könnte man sich natürlich fragen, wie ich validiere, ob es sich bei der Zahl um eine Primzahl handelt. Die Antwort lautet gar nicht. Bei der Primfaktorenzerlegung stellt man eine Zahl so dar, dass möglichst viele natürliche Zahlen miteinander verrechnet werden. Dadurch **muss** es sich immer um Primzahlen handeln. Im letzen Schritt schau ich dann noch, ob es sich bei der Zahl selber um eine Primzahl handelt. Und schon habe ich alle Primfaktoren ausgerechnet. 
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
1. Erstellen von mehreren Funktions Apps inklusive SSL Zertifikaten <br/>
Als erstes musste ich mehrere neue Funktions Apps erstellen (für jede Programmiersprache eine). Das ganze habe ich schon bei `NR 4 Part 1` (Erstellen von einer Funktion) und bei `NR 4 Part 2` (Hinzufügen einer Domain/SSL Zertifikat) dokumentiert. Aus diesem Grund werde ich hier nicht mehr genauer darauf eingehen.

2. Erstellen des FizzBuzz Scripts <br/>
Im nächsten Schritt habe ich mich daran gemacht FizzBuzz in allen Programmiersprachen zu implementieren. In allen Scripts validiere ich am Anfang, ob die gegebene Zahl im richtigen Format ist. Anschliessend caste ich diese (falls nötig abhängig von der Sprache). Im letzten Schritt überprüfe ich mit Hilfe des Modulus Opperators `%`, ob die Zahl teilbar ist. Der Modulusoperator ist ein Operator, mit dessen Hilfe man in der Lage ist festzustellen, wie gross der Restanteil bei einer Division ist. Dadurch eigenet er sich ideal, um zu testen ob eine Zahl durch eine andere Zahl teilbar ist.
```
10 % 2 -> 0
11 % 2 -> 1
11 % 3 -> 2
```
<br/>
<br/>
Grössere Probleme sind mir bei der Aufgabe nicht begegenet. Je nach Programmiersprache hat sich immer mal wieder ein Tippfehler eingeschlichen, den ich aber früher oder später behoben habe. Ein kleineres Problem ist bei Java aufgetreten. Java ist die einzige Programmiersprache, die ich lokal programmieren musste, da der Webeditor bei Azure sie nicht supportet hat. Dadurch musste ich zuerst lokal ein Projekt erstellen und dieses anschlissend auf Azure puschen. Das Erstellen und Pushen des Projektes verlief ohne Schwierigkeiten. Nach dem Deployen ist allerdings meine Funktion beim Aufrufen immer in ein Timeout gelaufen. Zuerst hat das Ganze für mich gar keinen Sinn gegeben, da local alles funktioniert hat. Also musste ich herausfinden, was sich geändert hat. Nach kurzem Vergleich ist mir aufgefallen, dass ich beim Projekt die falsche JavaVersion angegeben habe. Nachdem ich dies behoben habe, hat alles wieder normal funktioniert. Bei JavaScript habe ich mir zudem noch einen kleinen Spass erlaubt und das ganze in JSFuck implementiert (natürlich durch einen Encoder). 

### Bonus
Als kleiner Bonus habe ich das Ganze noch mit PowerShell implementiert. Mein erster Gedanke war, dass es extrem nervig sein wird, das Ganze mit Powershell zu implementieren. Nachdem ich mir allerdings ein paar Beispiele angeschaut habe, ist mir aufgefallen, dass es vom Syntax her gar nicht so speziell ist (nicht so wie bash). Mit der Hilfe von Stackoverflow ist mir dann das ganze Umschreiben auch relativ einfach gefallen.


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

Die Funktionen sind nun unter diesen URLs auffindbar. Bei allen Endpunkten muss man mit dem `code` Parameter einen Accesscode mitgeben, um überhaupt die Funktion benutzen zu können.
```
python-faas.bbzbl-it.dev/api/fizzbuzz?code=<AccessCode>&number=<Your Number>

java-faas.bbzbl-it.dev/api/fizzbuzz?code=<AccessCode>&number=<Your Number>

js-faas.bbzbl-it.dev/api/fizzbuzz?code=<AccessCode>&number=<Your Number>

python-faas.bbzbl-it.dev/api/fizzbuzz?code=<AccessCode>&number=<Your Number>
```

## 8. Eine API Aufrufen aus einer Azure funktion
Ich habe mir lange Zeit überlegt, welche API ich aus einer Azure Funktion heraus aufrufen könnte. Ich wollte es nicht so lösen, dass ich einfach eine Simple api habe, die irgendwelche Daten sendet, welche ich dann fast 1 zu 1 an den Benutzer der API sende. Als ich meine alten Projekte durchgegangen bin, ist mir eine Idee gekommen. Für ein früheres Projekt (M293) habe ich bereits einmal ein Tool mit JS geschrieben, mit dem man Bitcoin Adressen generieren kann. Dabei habe ich gelernt, dass Adressen, welche generiert wurden in der Theorie auch noch einmal generiert werden könnten. Die Wahrscheinlichkeit dafür beträgt allerdings etwas weniger als `106000000 zu 2^160 oder 1.461501637×10^48`. Das klingt zwar erstmals extrem unwahrscheinlich (was es auch ist). Wenn man es allerdings mit der Wahrscheinlichkeit vergleicht, wenn ein normales 52-Kartenspiel in einer bestimmten Reihenfolge gemischt wird, ist es viel wahrscheinlicher (`52!` oder `8.065817517×10^67`).

1. Erstellen einer Azure Funktion <br/>
Im ersten Schritt erstelle ich eine neue Azure Funktion, wie ich es auch schon mehrmals in den letzen beiden Aufgaben dokumentiert habe. Ich habe mich dazu entschieden, in diesem Beispiel wieder Python einzusetzen. Klar hätte ich auch mein JavaScript Projekt aus dem letzen Modul etwas abändern und einfach in die Cloud stellen können. Das wäre aber relativ langweilig gewesen.

2. Erstellen eines Scriptes <br/>
Im nächsten Schritt habe ich das Script erstellt. Dieses Script ist relativ komplex. Aus diesem Grund werde ich nur auf den Teil eingehen, in dem ich checke, ob eine generierte Bitcoin Adkönntedresse bereits Bitcoins auf ihr hat. Mit der Funktion `getAmount(address)` lese ich die Amount an Bitcoins aus. Dabei verwende ich die API von `blockchain.info`. Der Vorteil ist, dass sie 100% kostenlos ist und man sich nicht zu regisitrieren braucht. Der Nachteil ist, dass man nicht zu viele Requests stellen darf, da man sonst für ein paar Minunten ein Timeout bekommt. Die genaue Amount an Satoshis (100000000 Satoshis = 1 BTC) bekomme ich über den Endpukt `https://blockchain.info/q/addressbalance/<addresse>`. Anschliessend reche ich das Ganze noch in Bitcoins um und schon kann ich sagen, wie viel Bitcoins die Adresse besitzt.


```python
import logging

import azure.functions as func

import hashlib
import random
import http.client

A = 0
B = 7
P = 2**256 - 2**32 - 2**9 - 2**8 - 2**7 - 2**6 - 2**4 - 1
GX = 55066263022277343669578718895168534326250603453777594175500187360389116729240
GY = 32670510020758816978083085130507043184471273380659243275938904335757337482424
G = (GX, GY)

def getAmount(address : str) -> float:
    """
    Return the amount of bicoin in of the address using blockchain.com api
    using http client
    """
    url = "https://blockchain.info/q/addressbalance/" + address
    conn = http.client.HTTPSConnection("blockchain.info")
    conn.request("GET", "/q/addressbalance/" + address)
    res = conn.getresponse()
    data = res.read()
    return float(data.decode("utf-8"))/100000000

def point_add(p1 : tuple, p2 : tuple) -> tuple:
    if p1 != p2:
        lam = (p1[1] - p2[1]) * pow(p1[0] - p2[0], P-2, P)
        x3 = (pow(lam, 2) - p1[0] - p2[0]) % P
        y3 = (lam * (p1[0] - x3) - p1[1])  % P
        return (x3, y3)
    return point_dubl(p1)

def point_dubl(p1 : tuple) -> tuple:
    lam = (3*p1[0]**2 + A) * pow(2*p1[1], P-2, P)
    v = p1[1] - lam*p1[0] % P
    x3 = (lam**2 - 2*p1[0]) % P
    y3 = (lam*x3 + v) * -1 % P
    return (x3, y3)


def calc_publ(private_key):
    publ = None
    Q = G
    binar = private_key
    while binar:
        if binar%2 == 1:
            if publ == None:
                publ = Q
            else:
                publ = point_add(publ, Q)
        Q = point_dubl(Q)
        binar = binar//2    
    hex_value = hex(publ[0])[2:].upper()
    if publ[1]%2 == 1:
        return  "03" + hex_value
    return "02" + hex_value

def generateRandomKey():
    return hashlib.sha256(str(random.getrandbits(256)).encode()  + str(random.randint(1, 1000000000000000000000000000)).encode())


def public_key_to_address(public_key):
    public_key = hashlib.new('ripemd160', hashlib.sha256(bytes.fromhex(public_key)).digest()).digest()
    public_key = "00" + public_key.hex()
    checksum = hashlib.sha256(hashlib.sha256(bytes.fromhex(public_key)).digest()).hexdigest()[:8]
    public_key = public_key + checksum
    return public_key

def base58Check(value : str) -> str:
    """
    Convert a value to a base58Check string
    """
    value = int(value, 16)
    alphabet = '123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz'
    A = ""
    while value > 0:
        value, mod = divmod(value, 58)
        A = alphabet[mod] + A
    return A



def newAddress():
    priv = generateRandomKey()
    publ = calc_publ(int(priv.hexdigest(), 16))
    return {
            "private_key" : priv.hexdigest(),
            "public_key" : publ,
            "address" : "1" + base58Check(public_key_to_address(publ))
            }



def main(req: func.HttpRequest) -> func.HttpResponse:
    address = newAddress()
    amount = getAmount(address["address"])
    if amount > 0:
        return func.HttpResponse(f"Wow you found {amount} BTC in {address['address']} \n\nYour private key {address['private_key']} \n\n\nDetails: \n{address}")
    return func.HttpResponse(f"Nothing found in {address['address']}\n\nYour private key: {address['private_key']} \n\n\nDetails: \n{address} \n\nYou can try it in 5 Secods again")
```

Der API Endpunkt ist unter folgenden URL Auffinbar
```
https://python-faas.bbzbl-it.dev/api/good-luck?code=<Access-Code>
```

## 16. Absolwieren von AZ-900
Mein Ziel ist es, dass ich nach dem Modul auch etwas habe (neben meinen Erfahrungen und Wissen aus dem Modul), das ich auch später noch in meinem Arbeitsleben verwenden kann. Aus diesem Grund habe ich mich dazu entschieden die az-900 Zertifizierung zu absolvieren. Die az-900 Zertifizierung beinhaltet die Basics von der Cloud, wodurch es einen guten Einstieg in das Thema Cloud darstellt. Zur Vorbereitung habe ich den [Offiziellen Azure](https://learn.microsoft.com/en-us/certifications/exams/az-900) Guide durchgearbeitet. Mir gefiel an den Trainings vor allem auch, dass man in einer sogennanten `Sandbox` Umgebung das Gelernte auch gleich praktisch anwenden konnte. Dadurch kann man kostenlos die Dienste auspropieren ohne sich Sorgen machen zu müssen, etwas falsch zu machen. Anschliessend habe ich noch ein Video von [Free Code Camp](https://www.youtube.com/watch?v=NKEFWyqJ5XA) angeschaut, um mich zu vergewissern, dass ich auch alles Wichtige kenne. Dadurch fühle ich mich nun ausreichend vorbereitet, um bei der Zertifizierung anzutreten. Ich muss nun nur noch einen  Termin vereinbaren, wann ich die Zertifizierung absolvieren kann. 



## 20 Automatische Uniti Tests
Automatisierte Unit Tests sollen sicherstellen, dass sich keine Bugs in eine Funktion einschleichen können. Bisher habe ich alle Tests immer händisch vorgenommen. Das heisst, dass ich nachdem ich eine Änderung an der Funktion gemacht habe, die Funktion immer mit allen Evantualitäten ausgetestet habe. Für kleinere Projekte wie meine Funktions Apps ist das kein Problem. Allerdings je grösser die App wird desto nerfiger und afwändiger ist es immer wieder alles durchzutesten. Aus diesem Grund gibt es automatisierte Tests. Dabei handelt es sich um vordefinierte Methoden, die automatisiert api calls lossenden und dann auch das ergebnis validieren. Jeder Test läst sich normalerweise in 3 Teile einteilen. Zuerst ist da einmal die Vorbereitung. In dieser Phase werden die ganzen Attribute erstelt, mit denen man den Test durchführt. Anschlissend führt man das ganze durch. Normalerweise in Form eines REST Api calles. Am Ende wird dann noch validiert, ob das Ergebnis auch mit dem erwartet übereinstimmt. <br/>
<br/>
In meinem Fall habe ich das ganze anhand meiner FizzBuzz Funktion implementiert, welche ich in Java geschrieben habe. Diese habe ich noch von der Implementaton auf meinem PC Verfügbar. Dadurch konnte ich gleich die mitgenerierte JavaTestKlasse verwenden. Ein Testcase kann man inform einter Methode erstellen, welche mit `@test` annotiert ist. Durch die Annotation erkennt Azure, dass es sich dabei um einen Test handelt. Diese Tests werden immer vor dem Compilen ausgeführt. In meinem Fall habe ich 3 Testcases implementiert. 2 Davon sollen erfolgreich sein und der 3. soll auf einen Fehlercode prüfen.  Wie ich das ganze implementiert habe sollte relativ klar an den Commentaren im Code erkennbar sein.
<br/><br/>
Jetzt wo ich die Uniti Tests habe wird jedes mal befor ich meinen Code Compiled wird die ganzen Tests einmal ausgeführt. Bei einem Fehler in einem der Tests wird das Compilen sofort abgebrochen. Wenn ich meine Tests nun richtig geschriebe habe sollte und alle Fälle abgedekt habe sollte es nun nurnoch schwirig möglich sein einen kleinen Fehler zu implementieren, ohne dass es auffält.
```java
package dev.bbzblit;

import com.microsoft.azure.functions.*;
import org.mockito.invocation.InvocationOnMock;
import org.mockito.stubbing.Answer;

import java.util.*;
import java.util.logging.Logger;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;


/**
 * Unit test for Function class.
 */
public class FunctionTest {
    /**
     *  Test if class is be able to work with query parameters
     */
    @Test
    public void testHttpTriggerJava() throws Exception {
        // Setup
        final HttpRequestMessage<Optional<String>> req = mock(HttpRequestMessage.class);

        final Map<String, String> queryParams = new HashMap<>();
        queryParams.put("number", "15"); //<- Create a QueryParameter with number=15


        doReturn(queryParams).when(req).getQueryParameters();//<- Says the Mock class to return the QueryParameter 
                                                             //"queryParams" when getQueryParameters() is called in
                                                             // the controller

        final Optional<String> queryBody = Optional.empty(); //<- Same with query body but this time is it empty
        doReturn(queryBody).when(req).getBody();

        doAnswer(new Answer<HttpResponseMessage.Builder>() {
            @Override
            public HttpResponseMessage.Builder answer(InvocationOnMock invocation) {
                HttpStatus status = (HttpStatus) invocation.getArguments()[0];
                return new HttpResponseMessageMock.HttpResponseMessageBuilderMock().status(status);
            }
        }).when(req).createResponseBuilder(any(HttpStatus.class));


        final ExecutionContext context = mock(ExecutionContext.class);
        doReturn(Logger.getGlobal()).when(context).getLogger(); //<- Mocks the Logging Class
                                                                //That logs of the controller are shown 
                                                                //in the console

        final HttpResponseMessage ret = new Function().fizzBuzz(req, context); //<- Invoke the function 
                                                                               //called fizzBuzz

        // Verify
        assertEquals(ret.getStatus(), HttpStatus.OK);
        assertEquals(ret.getBody(), "FizzBuzz");
    }

    /**
     * Test if it also works with a Body parameter instead of a QueryParameter
     */
    @Test
    public void testFizzBuzzWithBody() throws Exception {
        // Setup
        final HttpRequestMessage<Optional<String>> req = mock(HttpRequestMessage.class);

        doReturn(new HashMap<>()).when(req).getQueryParameters();

        final Optional<String> queryBody = new HashMap<String, String>() {{
            put("number", "15");
        }}.entrySet().stream().findFirst().map(Map.Entry::getValue);
        doReturn(queryBody).when(req).getBody();

        doAnswer(new Answer<HttpResponseMessage.Builder>() {
            @Override
            public HttpResponseMessage.Builder answer(InvocationOnMock invocation) {
                HttpStatus status = (HttpStatus) invocation.getArguments()[0];
                return new HttpResponseMessageMock.HttpResponseMessageBuilderMock().status(status);
            }
        }).when(req).createResponseBuilder(any(HttpStatus.class));


        final ExecutionContext context = mock(ExecutionContext.class);
        doReturn(Logger.getGlobal()).when(context).getLogger();

        final HttpResponseMessage ret = new Function().fizzBuzz(req, context);

        // Verify
        assertEquals(ret.getStatus(), HttpStatus.OK);
        assertEquals(ret.getBody(), "FizzBuzz");
    }


    /**
     * Test if it returns a 422 Unprocessable Entity if the number is not in the format of an Integer
     */
    @Test
    public void testWIthInvalideNumber() throws Exception {
        // Setup
        final HttpRequestMessage<Optional<String>> req = mock(HttpRequestMessage.class);

        doReturn(new HashMap<>()).when(req).getQueryParameters();

        final Optional<String> queryBody = new HashMap<String, String>() {{
            put("number", "abc");
        }}.entrySet().stream().findFirst().map(Map.Entry::getValue);
        doReturn(queryBody).when(req).getBody();

        doAnswer(new Answer<HttpResponseMessage.Builder>() {
            @Override
            public HttpResponseMessage.Builder answer(InvocationOnMock invocation) {
                HttpStatus status = (HttpStatus) invocation.getArguments()[0];
                return new HttpResponseMessageMock.HttpResponseMessageBuilderMock().status(status);
            }
        }).when(req).createResponseBuilder(any(HttpStatus.class));


        final ExecutionContext context = mock(ExecutionContext.class);
        doReturn(Logger.getGlobal()).when(context).getLogger();

        final HttpResponseMessage ret = new Function().fizzBuzz(req, context);

        // Verify
        assertEquals(ret.getStatus(), HttpStatus.UNPROCESSABLE_ENTITY);
        assertEquals(ret.getBody(), "Please pass a number on the query string or in the request body");
    }
}

```

## 22 Bowlin Counter
Ich hab mir diese Aufgabe ausgesucht, da ich sehr gerne Coderätzel löse. Ich musste zuerst einmal googlen, worum es sich bei der Aufgabe handelt. Ich hab schnell eine Aufgabenberschreibung gefunden. Mir ist schnell aufgefallen, dass dieses Rätzel relativ bekannt ist. Dadurch kam mir die Idee, dass ich es möglicherweise bereits einmal auf CodeWars gelöst haben könnte oder es zumindes verfügbar ist, [und so war es auch](https://www.codewars.com/kata/5531abe4855bcc8d1f00004c). Dadurch habe ich natürlich einen extrem guten Vorteil, da ich durch die Seite eine genaue Beschreibung von dem Problem bekomme. Ausserdem gibt es da automatische Testcases mit denen ich meine Lösung testen kann. Dadurch muss ich schon eine Aufgabe weniger bewältigen. Als ich mir die Aufgabe durchgelesen habe ist mir die Aufgabe relativ leicht vorgekommen (für ein 4er Katar). Nachdem ich eine erste Lösung habe, welche auch funktionert habe ich mich noch drangesetzt, um diese zu Optimieren, bis ich zufrieden dammit war.
```python
translator = { "x" : 10, "/" : 10,  "1" : 1 , "2" : 2, "3": 3, "4" : 4, "5" : 5, "6" : 6, "7" : 7, "8" : 8, "9" : 9 }
def bowling_score(frames : str) -> int:
    last_char = ""
    frames = frames.replace(" ", "").lower()
    points = 0
    round = 0
    while round < 10:
        char = frames[0]
        frames = frames[1:]
        points += translator.get(char, 0)
        if char == "/":
            points -= translator.get(last_char, 0)
            points += translator.get(frames[0], 0)
        elif char == "x":
            if frames[1] != "/":
                points += translator.get(frames[0], 0)
                points += translator.get(frames[1], 0)
            else:
                points += 10
        last_char = char
        round += {"x" : 1}.get(char , 0.5)
    return points
```
Bei der Methode kann man nun als paramter `frames` eine String mit einer aneinanderreihung von Punktzahlen pro Runde geben (z.B `15 1/ x ...`). In der Methode wird nie validiert, ob die angegebenen Punkte korrekt sind. Allerdings ist das auch nicht Teil der Augabe. Anschlissend wird mithilfe eines while loopes den String in die totale Punktzahl umgerechnet.
<br/>
Für das Deployment habe ich meine bestehende Python app verwendet, welche unter der URL `https://python-faas.bbzbl-it.dev` verfügbar ist.
```
https://python-faas.bbzbl-it.dev/api/bowling-dojo?code=<Acces-Code>&rounds=<RoundString>
```

<!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
<!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
<!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->
<!-- !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! -->

## NR ? Aufsetzen eines Minecraft Server
Ich benutze für den Minecraft Server einen Server mit `1` vCPU `2 GB` Ram und `20 GB` SSD Storage. Als  Betriebsystem verwende ich auch hier `Debian 11`. Die ersten 3 Schritte, die ich auf dem Server ausgeführt habe, sind exakt die Gleichen wie bei `NR. `3`, weshalb ich sie nicht nocheinmal dokumentiere.

4. Installieren von Java <br/>
Da Minecraft Server eine Java Application ist, musste ich zuerst Java installieren. Ich möchte am Ende die neuste Version von Minecraft installieren, weshalb ich die Java Version `17` installieren muss.
```bash
sudo apt-get install openjdk-17-jdk
```

5. Herunterladen von Minecraft Server <br/>
Im nächsten Schritt habe ich die Executable für Minecraft Server heruntergelade. Das geht ganz einfach über `wget`
```bash
wget https://piston-data.mojang.com/v1/objects/f69c284232d7c7580bd89a5a4931c3581eae1378/server.jar
```

6. Ausführen vom Minecraft Server <br/>
Wenn man über ssh auf einen Server zugreift, ist das in etwa so, wie wenn man auf einem PC eine neue Konsole öffnet. Sobald die SSH Verbindung geschlossen wird, wird auch automatisch die Konsole geschlossen. Das ergibt im Normalfall auch durchaus Sinn. In diesem Fall allerdings möchte man, dass der Server weiterhin läuft, auch wenn man die SSH Verbindung schliesst. Die Lösung für das Problem lautet `tmux`. Mit `tmux` kann man eine Konsole in einer Konsole erstellen, die im Hintergrund läuft. Diese wird, auch wenn die SSH Verbindung geschlossen wird, nicht beendet. 
```bash
sudo apt-get install tmux
```
Eine neue `tmux` Session kann man ganz einfach mit dem Command `tmux` erstellen. Dass man sich in einer `tmux` Session befindet, sieht man daran, dass unten an der Konsole ein grüner Strich ist. Den Server kann ich dann ganz einfach aufstarten. Die beiden Parameter `-Xmx` und `-Xms` definieren dabei die maximalen und die initiale heap Size.
```bash
java -Xmx1024M -Xms1024M -jar server.jar nogui
```
Nachdem der Server das erste Mal gestartet ist, bricht er schnell nach dem Startvorgang ab. Das liegt daran, dass man die Minecraft `Eula` (end user Agreement) akzeptieren muss. Das kann man ganz einfach dadurch machen, dass man das File `eula.txt` editiert und `eula=false` auf `eula=true` setzt. Danach kann man den Server einfach erneut aufstarten. <br/>
Nachdem der Server komplett aufgestartet ist, kann man `tmux` wieder mit dem Shortcut `ctrl + b` und anschliessend `d` verlassen. Alle aktiven `tmux` Sessionen kann man nun mit dem Command `tmux ls` auflisten. Falls man wieder in eine Session zurückkehren möchte, kann man das mit dem Command `tmux attach -t <session id>` machen.

7. Einrichten einer Domain <br/>
Nun, da der Server nun läuft, habe ich schnell noch eine Subdomain eingerichtet, die auf den Server zeigt. Wenn man seine Domains bei Cloudflare verwaltet, ist es wichtig, dass man Proxy durch Cloudflare ausschaltet. Das liegt daran, dass in der Freeversion diese Protokole/Ports nicht abgedeckt werden.

8. Einrichten der Firewall. <br/>
Da ich diesen Server bei Hetzner eingerichtet habe, habe ich automatisch auch Zugang zu einer kostenlosen externen Firewall und muss nicht `ufw` benutzen. FÜr den Minecraft Server muss ich TCP Traffik auf dem Port `22` für `ssh` und den Port `25565` für den Minecraft Server zulassen
![image](https://user-images.githubusercontent.com/99135388/197341759-93f6d9a4-2762-4061-b24a-64d83342a90b.png)
