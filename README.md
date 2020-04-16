[![Join the chat at https://gitter.im/Digital-Learning-for-Students/community](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/Digital-Learning-for-Students/community)
![#wirvsvirus Logo](resources/wirvsvirus-logo.png)
# Hosted Blueprint für Digital Learning für Schüler
## Allgemeine Informationen
Dieser Blueprint entstand im Rahmen des #wirvsvirus Hackathons in der Gruppe #1_241_a_digitallearningfürschüler. Zielstellung ist aufzuzeigen, wie aus dem Free and Open Source Umfeld selbst gehostete Lösungen im Bildungsumfeld sofort genutzt werden könnten.

Folgende Free and Open Source Tools werden dazu genutzt bzw. sind in Planung:

In Test:
* [Ilias eLearning Plattform](https://www.ilias.de/)

In Planung (diese Tools oder ähnliche):
* [Matrix Chat Server](https://matrix.org/) oder [Mattermost self hosted Slack-Alternative](https://mattermost.com/) 
* [OpenMeetings Web-Konferenz Plattform](https://openmeetings.apache.org/), neueste Version 5 aktuell noch in Testphase

Im Fall von Matrix Chat kann je Server nur eine Domäne betrieben werden. Wenn auf einem Server Software für verschiedene Schulen betrieben werden soll, müssten sich mehrere Schulen diese Instanz teilen.

## Anleitung
## Voraussetzungen
* Bereitstellung eines (virtuellen) Servers (z. B. über einen Anbieter oder unter Nutzung eigener Infrastruktur in der Schule)
    * Betriebssystem z. B. Debian oder Ubuntu (Blueprint getestet mit Ubuntu 16.04)
    * Root-Zugang wird benötigt
    * Server muss je nach Anzahl SchülerInnen und LehrerInnen unterschiedlich dimensioniert werden (nicht Teil dieser Anleitung)

## DNS-Konfiguration

Wenn Subdomains für verschiedene Schulen genutzt werden sollen, z. B. `schule-1.example.org`, `schule-2.example.org` müssen alle diese Subdomains im DNS konfiguriert werden oder, wenn gewünscht, ein Wildcard-Eintrag für die Ziel-IP (im Beispiel 123.123.123.123) gemacht werden. Hier ein einfaches Beispiel:

```
Zone: example.org

Host            Type    MX  Destination
schule-1        A           123.123.123.123
schule-2        A           123.123.123.123
```

oder

```
Zone: example.org

Host            Type    MX  Destination
*               A           123.123.123.123
```
## Installation der Voraussetzungen

```bash
# Verbinden mit SSH (bei Linux und Mac OS X mit dem verfügbaren ssh Kommando, bei Windows wird z. B. Putty verwendet)
apt-get install python3 python3-pip
# Docker Installation nach Vorgabe in https://docs.docker.com/install/linux/docker-ce/ubuntu/ (zum Teil werden von Hosting-Anbietern auch Images mit vorinstalliertem Docker angeboten)
apt-get remove docker docker-engine docker.io containerd runc
apt-get update
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# Fingerprint Check (siehe Docker Doku)
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
# Installation von docker-compose nach Vorgabe in https://docs.docker.com/compose/install/
curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
# Sicherstellen, dass auf dem Server kein weiterer Webserver läuft (das Projekt beinhaltet einen Nginx Reverse Proxy und benötigt freie Ports 80 und 443)
apt-get remove nginx apache2
# oder
systemctl disable nginx
systemctl stop nginx
systemctl disable apache2
systemctl stop apache2
```

## Reverse Proxy und SSL-Zertifikate (https)

Sowohl das Erstellen der Reverse Proxy Konfiguration als auch die Anfrage und automatische Aktualisierung von SSL-Zertifikaten findet automatisch durch Einbindung der Projekte [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) und [letsencrypt-nginx-proxy-companion](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion/) statt.

Soll das Anfragen der Zertifikate nur getestet werden, kann in `nginx/nginx_proxy_env_template` (bzw. nach der Generierung der Config in `nginx/.nginx_proxy_env`) die folgende Zeile aktiviert werden (standardmäßig auskommentiert):

```
# ACME_CA_URI=https://acme-staging-v02.api.letsencrypt.org/directory
```

Dadurch wird die Staging Umgebung von Letsencrypt verwendet. Root- und Intermediate-Zertifikat werden damit von Browsern nicht akzeptiert. Wird während Tests benötigt, da nach zu häufigen Anfragen letsencrypt den Anfrager für einige Zeit sperrt. Solange das nicht gemacht wird, werden echte Zertifikate angefordert und konfiguriert.

## Installation des Blueprints
Die im Folgenden dargestellten Schritte müssen mit einem für Docker berechtigten User durchgeführt werden. Getestet wurden sie mit Root-User.

```bash
cd /srv #Dieses Verzeichnis kann frei gewählt werden und ist nur ein Vorschlag (bitte Befehle entsprechend anpassen, wenn in ein anderes Verzeichnis installiert wird)
git clone https://github.com/Digital-Learning-fur-Schuler/hosted-blueprint.git
cd hosted-blueprint
# Pfad zur PATH-Variable hinzufügen, damit das Kontroll-Skript von überall aus aufgerufen werden kann
echo "PATH=$PATH:." >> ~/.bashrc
source ~/.bashrc # Alternativ aus- und wieder einloggen
bpctl -h # Hilfe für das Skript anzeigen
usage: bpctl [-h] {configure,start,stop,update,backup,purge} ...

Control script to control the digital learning environment

positional arguments:
  {configure,start,stop,update,backup,purge}
    configure           Configures the environment. Configure tools to use and
                        (sub) domains to install them on.
    start               Starts all docker containers
    stop                Stops all docker containers
    update              Updates all docker images and recreates changed
                        containers
    backup              Creates backups of all docker volumes and recreates
                        changed containers. Will pause containers during
                        backup operation.
    purge               Purges configuration, images, volumes but not backups
                        of the complete environment. Purge operation can
                        include other docker data that is not referenced
                        anymore (calls docker system prune)

optional arguments:
  -h, --help            show this help message and exit
```

## Konfiguration des Blueprints
Nach erfolgreicher Installation können nun die Tools, die verwendet werden sollen, konfiguriert werden.

```bash
bpctl configure
Enter an email address that will be used for letsencrypt (will receive warnings if certificates expire)
webmaster@example.org
Enter the client you want to activate during installation (default/school)
school
Enter all ILIAS instance's hostnames (one host name per line, for example ilias.my-school.com, ilias.my-school.provider.com, etc.)
ilias-school1.example.org
ilias-school2.example.org
ilias-school3.example.org

['ilias-school1.example.org', 'ilias-school2.example.org', 'ilias-school3.example.org']
Admin email: webmaster@example.org
ILIAS client: school
Configuration will be generated for the following domains:
ILIAS:
['ilias-school1.example.org', 'ilias-school2.example.org', 'ilias-school3.example.org']
Is that correct? (Y/N)
y
Nginx-proxy configuration is being generated
Nginx-proxy configuration ready
Ilias configuration is being generated
ilias-school1.example.org Mysql root password: "<pw1>"
ilias-school2.example.org Mysql root password: "<pw2>"
ilias-school3.example.org Mysql root password: "<pw3>"
ILIAS configuration ready.
[✓] Configuration finished successfully
```

## Start

Die Umgebung wird wie folgt gestartet.

**Nach dem ersten Start *müssen* die ILIAS Root-Passwörter aus dem Logfile abgerufen und beispielsweise in einem Passwort Safe hinterlegt werden. Beim nächsten Neustart werden diese Passwörter nicht mehr angezeigt.**

```bash
bpctl start

Starting hosted blue print...
[*] Creating elearning docker network
70e530ea068a303980858d6725effbce0c0dd5e916dce810b2aa366cc1c5a8a3
[*] Starting Nginx reverse proxy...
Creating volume "nginx_nginx-proxy-certs" with default driver
Creating volume "nginx_nginx-proxy-vhosts" with default driver
Creating volume "nginx_nginx-proxy-html" with default driver
Creating nginx-proxy ... done
Creating nginx-letsencrypt ... done
[*] Waiting Nginx to launch on Port 80...
[*] Starting docker image with env file: ilias/env_ilias-school1.example.org
Creating volume "ilias-school1exampleorg_ilias_www" with default driver
Creating volume "ilias-school1exampleorg_ilias_data" with default driver
Creating volume "ilias-school1exampleorg_ilias_mysql" with default driver
Pulling ilias (digitallearningforschools/ilias-school:5.4)...
5.4: Pulling from digitallearningforschools/ilias-school
68ced04f60ab: Already exists
1d2a5d8fa585: Already exists
5d59ec4ae241: Already exists
d42331ef4d44: Already exists
408b7b7ee112: Already exists
570cd47896d5: Already exists
2419413b2a16: Already exists
c66b5dbcbf27: Already exists
fbe2e5e30914: Already exists
3657e888189a: Already exists
df6a9028e5fe: Already exists
31e8b5dd7268: Already exists
0b37cac42026: Already exists
3dfcad9611ac: Already exists
a1e292ea4302: Pull complete
98a6a90f7fa6: Pull complete
55421a3d4a90: Pull complete
caf8ea66ad0c: Pull complete
fb24c9610fca: Pull complete
a0d462320ee8: Pull complete
66ee6194624c: Pull complete
6f2302675cbf: Pull complete
8c014f074efa: Pull complete
1738562f3b49: Pull complete
b186250ae142: Pull complete
Digest: sha256:63e7b32273778862556de33548ca27ee9798ae85d13cdedbf9c09d846933a1e5
Status: Downloaded newer image for digitallearningforschools/ilias-school:5.4
Creating ilias-school1exampleorg_mysql_1 ... done
Creating ilias-school1exampleorg_ilias_1 ... done
[*] Starting docker image with env file: ilias/env_ilias-school2.example.org
Creating volume "ilias-school2exampleorg_ilias_www" with default driver
Creating volume "ilias-school2exampleorg_ilias_data" with default driver
Creating volume "ilias-school2exampleorg_ilias_mysql" with default driver
Creating ilias-school2exampleorg_mysql_1 ... done
Creating ilias-school2exampleorg_ilias_1 ... done
[*] Starting docker image with env file: ilias/env_ilias-school3.example.org
Creating volume "ilias-school3exampleorg_ilias_www" with default driver
Creating volume "ilias-school3exampleorg_ilias_data" with default driver
Creating volume "ilias-school3exampleorg_ilias_mysql" with default driver
Creating ilias-school3exampleorg_mysql_1 ... done
Creating ilias-school3exampleorg_ilias_1 ... done
[✓] All containers in the environment started successfully.

# Der erste Start kann eine ganze Weile dauern
# Das Login-Passwort für Ilias steht im Logfile nach der ersten Installation, z. B. für school1.example.org aus dem Beispiel:
cd ilias
docker-compose --env-file env_ilias-school1.example.org logs

[...]
ilias_1  | =======================================================
ilias_1  | ILIAS installed successfully!
ilias_1  | Log in using the following credentials:
ilias_1  | 
ilias_1  | User:           root
ilias_1  | Password:       <root_pw>
ilias_1  | Setup Password: <setup_pw>
ilias_1  | =======================================================
[...]
```

## Update
Das Update-Kommando aktualisiert die Images bezogen auf ihre aktuellen Versions-Tags (damit i. d. R. nur Hotfixes oder Security Fixes enthalten). Erst das Aktualisieren des Git Repositories (oder das Auschecken einer anderen Version) ändert auch die Release-Version der jeweiligen Software. Bitte immer die Release Notes beachten, bevor die Version gewechselt wird.

```bash
git pull # Bei Release Updates
bpctl update
The following operation will update all container images defined in docker-compose files of the projects. You should backup or create a VM snapshot before doing that. Do you want to start? (Y/N) y
[*] Pulling nginx images...

Pulling nginx-proxy             ... done
Pulling nginx-proxy-letsencrypt ... done
[*] Pulling ilias images (you can ignore warning messages that variables are not set)...
WARNING: The ILIAS_AUTO_SETUP variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_DB_USER variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_DB_PASSWORD variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_HOST_NAME variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_CLIENT_NAME variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_DB_DUMP variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_DEFAULT_SKIN variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_DEFAULT_STYLE variable is not set. Defaulting to a blank string.
WARNING: The VIRTUAL_HOST variable is not set. Defaulting to a blank string.
WARNING: The LETSENCRYPT_HOST variable is not set. Defaulting to a blank string.
WARNING: The MYSQL_ROOT_PASSWORD variable is not set. Defaulting to a blank string.
WARNING: The MYSQL_DATABASE variable is not set. Defaulting to a blank string.
Pulling mysql ... done
Pulling ilias ... done
[*] Recreating containers...
Starting hosted blue print...
[*] Creating elearning docker network
[*] Starting Nginx reverse proxy...
nginx-proxy is up-to-date
nginx-letsencrypt is up-to-date
[*] Waiting Nginx to launch on Port 80...
[*] Starting docker image with env file: ilias/env_ilias-school1.example.org
ilias-school1exampleorg_mysql_1 is up-to-date
ilias-school1exampleorg_ilias_1 is up-to-date
[*] Starting docker image with env file: ilias/env_ilias-school2.example.org
ilias-school2exampleorg_mysql_1 is up-to-date
ilias-school2exampleorg_ilias_1 is up-to-date
[*] Starting docker image with env file: ilias/env_ilias-school3.example.org
ilias-school3exampleorg_mysql_1 is up-to-date
ilias-school3exampleorg_ilias_1 is up-to-date
[✓] All containers in the environment started successfully.
[✓] Updated successfully
```
## Purge / Löschen
Um alles vom Server wieder zu löschen kann `purge` genutzt werden. Damit wird alles gelöscht, inkl. der persistenten Daten der Docker Container. Bitte nur machen, wenn alles gesichert wurde oder noch getestet wird. Da `docker system prune` genutzt wird, kann das auch Daten anderer Docker-Projekte enthalten, die nicht mehr von Docker referenziert werden. Mit dem Kommando `backup` gesicherte Daten werden nicht gelöscht.

```
bpctl purge

The following operation will purge your docker system to start from scratch (can include data from other docker projects, if no references exist anymore). Do you know what you do and want to start from scratch? (Y/N) y
[*] Stopping Nginx reverse proxy...
Stopping nginx-letsencrypt ... done
Stopping nginx-proxy       ... done
Removing nginx-letsencrypt ... done
Removing nginx-proxy       ... done
Network elearning is external, skipping
[*] Stopping Ilias instances...
[*] Stopping docker multi-container with env-file ilias/env_ilias-school1.example.org
Stopping ilias-school1exampleorg_ilias_1 ... done
Stopping ilias-school1exampleorg_mysql_1 ... done
Removing ilias-school1exampleorg_ilias_1 ... done
Removing ilias-school1exampleorg_mysql_1 ... done
Network elearning is external, skipping
[*] Stopping docker multi-container with env-file ilias/env_ilias-school2.example.org
Stopping ilias-school2exampleorg_ilias_1 ... done
Stopping ilias-school2exampleorg_mysql_1 ... done
Removing ilias-school2exampleorg_ilias_1 ... done
Removing ilias-school2exampleorg_mysql_1 ... done
Network elearning is external, skipping
[*] Stopping docker multi-container with env-file ilias/env_ilias-school3.example.org
Stopping ilias-school3exampleorg_ilias_1 ... done
Stopping ilias-school3exampleorg_mysql_1 ... done
Removing ilias-school3exampleorg_ilias_1 ... done
Removing ilias-school3exampleorg_mysql_1 ... done
Network elearning is external, skipping
[✓] All containers in the environment stopped successfully.
[*] Deleted ilias/env_ilias-school1.example.org
[*] Deleted ilias/env_ilias-school2.example.org
[*] Deleted ilias/env_ilias-school3.example.org
[*] Deleted nginx/.nginx_proxy_env
Deleted Networks:
elearning

Total reclaimed space: 0B
Deleted Volumes:
ilias-school3exampleorg_ilias_data
ilias-school1exampleorg_ilias_data
ilias-school2exampleorg_ilias_www
ilias-school1exampleorg_ilias_www
ilias-school3exampleorg_ilias_www
nginx_nginx-proxy-vhosts
ilias-school1exampleorg_ilias_mysql
ilias-school2exampleorg_ilias_mysql
nginx_nginx-proxy-certs
nginx_nginx-proxy-html
ilias-school3exampleorg_ilias_mysql
ilias-school2exampleorg_ilias_data
d5521a4ee4ea33bb0ba6a436d198b2d693e33c461825d2d3f11cbfe24a7b3754

Total reclaimed space: 1.444GB
[✓] Purge operation complete.
```
## Backup
Mithilfe des Skriptes `./backup` wird ein Backup aller Projekte angelegt.

```
bpctl backup
Pausing nginx-proxy       ... done
Pausing nginx-letsencrypt ... done
[+] Backing up nginx project to backups/2020-03-30_16-20
    - Saving config to ./docker-compose.yml
    - Saving database dumps to ./dumps
[*] Backing up nginx__nginx-proxy to ./nginx-proxy...
    - Saving config to ./nginx-proxy/config.json
    - Saving logs to ./nginx-proxy/docker.out/err
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-certs/_data volume to ./nginx-proxy/volumes/var/lib/docker/volumes/nginx_nginx-proxy-certs/_data
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-html/_data volume to ./nginx-proxy/volumes/var/lib/docker/volumes/nginx_nginx-proxy-html/_data
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-vhosts/_data volume to ./nginx-proxy/volumes/var/lib/docker/volumes/nginx_nginx-proxy-vhosts/_data
    - Saving /var/run/docker.sock volume to ./nginx-proxy/volumes/var/run/docker.sock
    - Saving /var/lib/docker/volumes/d5521a4ee4ea33bb0ba6a436d198b2d693e33c461825d2d3f11cbfe24a7b3754/_data volume to ./nginx-proxy/volumes/var/lib/docker/volumes/d5521a4ee4ea33bb0ba6a436d198b2d693e33c461825d2d3f11cbfe24a7b3754/_data
[*] Backing up nginx__nginx-proxy-letsencrypt to ./nginx-proxy-letsencrypt...
    - Saving config to ./nginx-proxy-letsencrypt/config.json
    - Saving logs to ./nginx-proxy-letsencrypt/docker.out/err
    - Saving /var/run/docker.sock volume to ./nginx-proxy-letsencrypt/volumes/var/run/docker.sock
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-vhosts/_data volume to ./nginx-proxy-letsencrypt/volumes/var/lib/docker/volumes/nginx_nginx-proxy-vhosts/_data
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-html/_data volume to ./nginx-proxy-letsencrypt/volumes/var/lib/docker/volumes/nginx_nginx-proxy-html/_data
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-certs/_data volume to ./nginx-proxy-letsencrypt/volumes/var/lib/docker/volumes/nginx_nginx-proxy-certs/_data
[*] Compressing backup folder to backups/2020-03-30_16-20.tar.gz
tar: backups/2020-03-30_16-20/nginx-proxy-letsencrypt/volumes/var/run/docker.sock: socket ignored
tar: backups/2020-03-30_16-20/nginx-proxy/volumes/var/run/docker.sock: socket ignored
Total bytes written: 122880 (120KiB, 3.2MiB/s)
[✓] Finished Backing up nginx to backups/2020-03-30_16-20.tar.gz.
Unpausing nginx-letsencrypt ... done
Unpausing nginx-proxy       ... done
Backing up docker-compose project with env_file ilias/env_ilias-school1.example.org
Pausing ilias-school1exampleorg_mysql_1 ... done
Pausing ilias-school1exampleorg_ilias_1 ... done
[+] Backing up ilias project to backups/env_ilias-school1.example.org/2020-03-30_16-20
    - Saving config to ./docker-compose.yml
    - Saving database dumps to ./dumps
[*] Backing up ilias__mysql to ./mysql...
    - Saving config to ./mysql/config.json
    - Saving logs to ./mysql/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-school1exampleorg_ilias_mysql/_data volume to ./mysql/volumes/var/lib/docker/volumes/ilias-school1exampleorg_ilias_mysql/_data
[*] Backing up ilias__ilias to ./ilias...
    - Saving config to ./ilias/config.json
    - Saving logs to ./ilias/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-school1exampleorg_ilias_data/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-school1exampleorg_ilias_data/_data
    - Saving /var/lib/docker/volumes/ilias-school1exampleorg_ilias_www/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-school1exampleorg_ilias_www/_data
[*] Compressing backup folder to backups/env_ilias-school1.example.org/2020-03-30_16-20.tar.gz
Total bytes written: 497930240 (475MiB, 44MiB/s)
[✓] Finished Backing up ilias to backups/env_ilias-school1.example.org/2020-03-30_16-20.tar.gz.
Unpausing ilias-school1exampleorg_ilias_1 ... done
Unpausing ilias-school1exampleorg_mysql_1 ... done
Backing up docker-compose project with env_file ilias/env_ilias-school2.example.org
Pausing ilias-school2exampleorg_mysql_1 ... done
Pausing ilias-school2exampleorg_ilias_1 ... done
[+] Backing up ilias project to backups/env_ilias-school2.example.org/2020-03-30_16-21
    - Saving config to ./docker-compose.yml
    - Saving database dumps to ./dumps
[*] Backing up ilias__mysql to ./mysql...
    - Saving config to ./mysql/config.json
    - Saving logs to ./mysql/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-school2exampleorg_ilias_mysql/_data volume to ./mysql/volumes/var/lib/docker/volumes/ilias-school2exampleorg_ilias_mysql/_data
[*] Backing up ilias__ilias to ./ilias...
    - Saving config to ./ilias/config.json
    - Saving logs to ./ilias/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-school2exampleorg_ilias_www/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-school2exampleorg_ilias_www/_data
    - Saving /var/lib/docker/volumes/ilias-school2exampleorg_ilias_data/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-school2exampleorg_ilias_data/_data
[*] Compressing backup folder to backups/env_ilias-school2.example.org/2020-03-30_16-21.tar.gz
Total bytes written: 497930240 (475MiB, 47MiB/s)
[✓] Finished Backing up ilias to backups/env_ilias-school2.example.org/2020-03-30_16-21.tar.gz.
Unpausing ilias-school2exampleorg_ilias_1 ... done
Unpausing ilias-school2exampleorg_mysql_1 ... done
Backing up docker-compose project with env_file ilias/env_ilias-school3.example.org
Pausing ilias-school3exampleorg_mysql_1 ... done
Pausing ilias-school3exampleorg_ilias_1 ... done
[+] Backing up ilias project to backups/env_ilias-school3.example.org/2020-03-30_16-21
    - Saving config to ./docker-compose.yml
    - Saving database dumps to ./dumps
[*] Backing up ilias__mysql to ./mysql...
    - Saving config to ./mysql/config.json
    - Saving logs to ./mysql/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-school3exampleorg_ilias_mysql/_data volume to ./mysql/volumes/var/lib/docker/volumes/ilias-school3exampleorg_ilias_mysql/_data
[*] Backing up ilias__ilias to ./ilias...
    - Saving config to ./ilias/config.json
    - Saving logs to ./ilias/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-school3exampleorg_ilias_data/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-school3exampleorg_ilias_data/_data
    - Saving /var/lib/docker/volumes/ilias-school3exampleorg_ilias_www/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-school3exampleorg_ilias_www/_data
[*] Compressing backup folder to backups/env_ilias-school3.example.org/2020-03-30_16-21.tar.gz
Total bytes written: 497930240 (475MiB, 46MiB/s)
[✓] Finished Backing up ilias to backups/env_ilias-school3.example.org/2020-03-30_16-21.tar.gz.
Unpausing ilias-school3exampleorg_ilias_1 ... done
Unpausing ilias-school3exampleorg_mysql_1 ... done
```

# Todos
* Anleitung / Skript für das Wiederherstellen aus dem Backup
* Weitere Tools dem Blueprint hinzufügen (aktuelle Version beinhaltet nur ILIAS)

# Test protocols
[Backup/Restore](docs/backup_restore_test.md)

# Acknowledgements

Dieses Projekt setzt folgende Open Source Software-Komponenten ein bzw. ermöglicht ihre Nutzung:

* [ILIAS - eLearning Plattform](https://www.ilias.de/)
* [ILIAS Docker Image der studer + raimann ag](https://github.com/studer-raimann/docker-ilias)
* [nginx-proxy - nginx reverse proxy für Docker Container](https://github.com/nginx-proxy/nginx-proxy)
* [letsencrypt-nginx-proxy-companion - leichtgewichtiger companion container für nginx-proxy um mithilfe von Letsencrypt Zertifikate zu generieren](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion/)
* [Gist des Github Users "pirate" (Nick Sweeting) um ein docker-compose Projekt zu sichern](https://gist.github.com/pirate/265e19a8a768a48cf12834ec87fb0eed)
