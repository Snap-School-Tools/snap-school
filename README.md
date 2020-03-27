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
apt-get install python3 pyton3-pip
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

## Reverse Proxy und von SSL-Zertifikate

Konfiguration und Anfrage von Zertifikaten findet automatisch durch Einbindung der Projekte [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) und [letsencrypt-nginx-proxy-companion](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion/) statt.

Soll das Anfragen der Zertifikate nur getestet werden, kann in nginx/nginx_proxy_env_template (bzw. nach der Generierung der Config in nginx/.nginx_proxy_env) die folgende Zeile einkommentiert werden:

```
# ACME_CA_URI=https://acme-staging-v02.api.letsencrypt.org/directory
```

Dadurch wird die Staging Umgebung von Letsencrypt verwendet. Root- und Intermediate-Zertifikat werden damit von Browsern nicht akzeptiert. Wird benötigt, da nach zu häufigen Anfragen letsencrypt den Anfrager für einige Zeit sperrt. Solange das nicht gemacht wird, werden echte Zertifikate angefordert und konfiguriert.

## Konfiguration des Blueprints

```bash
cd /srv
git clone https://github.com/Digital-Learning-fur-Schuler/hosted-blueprint.git
cd hosted-blueprint
./configure
Enter an email address that will be used for letsencrypt (will receive warnings if certificates expire)
admin@example.org
Enter all Ilias instance's hostnames (one host name per line, for example ilias.my-school.com, ilias.my-school.provider.com, etc.)
ilias-schule-1.example.org
ilias-schule-2.example.org
ilias-schule-3.example.org

['ilias-schule-1.example.org', 'ilias-schule-2.example.org', 'ilias-schule-3.example.org']
Admin email: admin@example.org
Configuration will be generated for the following domains:
Ilias:
['ilias-schule-1.example.org', 'ilias-schule-2.example.org', 'ilias-schule-3.example.org']
Is that correct? (Y/N)
y
Nginx-proxy configuration is being generated
Nginx-proxy configuration ready
Ilias configuration is being generated
ilias-schule-1.example.org Mysql root password: "<pw1>"
ilias-schule-2.example.org Mysql root password: "<pw2>"
ilias-schule-3.example.org Mysql root password: "<pw3>"
Ilias configuration ready.
```

## Start
```bash
./start
Creating elearning docker network
20038849ff0ba2d27a49a2d7f3d92d145f0c883f3637b5154583d6858e79726d
Starting Nginx reverse proxy...
Creating volume "nginx_nginx-proxy-certs" with default driver
Creating volume "nginx_nginx-proxy-vhosts" with default driver
Creating volume "nginx_nginx-proxy-html" with default driver
Creating nginx-proxy ... done
Creating nginx-letsencrypt ... done
Waiting Nginx to launch on Port 80...
Waiting an additional 10 seconds, just to be safe...
Nginx launched
Starting docker image with env file: ilias/.ilias_env_ilias-schule-1.example.org
Creating volume "ilias-schule-1exampleorg_ilias_www" with default driver
Creating volume "ilias-schule-1exampleorg_ilias_data" with default driver
Creating volume "ilias-schule-1exampleorg_ilias_mysql" with default driver
Creating ilias-schule-1exampleorg_mysql_1 ... done
Creating ilias-schule-1exampleorg_ilias_1 ... done
Starting docker image with env file: ilias/.ilias_env_ilias-schule-2.example.org
Creating volume "ilias-schule-2exampleorg_ilias_www" with default driver
Creating volume "ilias-schule-2exampleorg_ilias_data" with default driver
Creating volume "ilias-schule-2exampleorg_ilias_mysql" with default driver
Creating ilias-schule-2exampleorg_mysql_1 ... done
Creating ilias-schule-2exampleorg_ilias_1 ... done
Starting docker image with env file: ilias/.ilias_env_ilias-schule-3.example.org
Creating volume "ilias-schule-3exampleorg_ilias_www" with default driver
Creating volume "ilias-schule-3exampleorg_ilias_data" with default driver
Creating volume "ilias-schule-3exampleorg_ilias_mysql" with default driver
Creating ilias-schule-3exampleorg_mysql_1 ... done
Creating ilias-schule-3exampleorg_ilias_1 ... done

# Der erste Start kann eine ganze Weile dauern
# Das Login-Passwort für Ilias steht im Logfile nach der ersten Installation, z. B. für schule-3.example.org aus dem Beispiel:
docker-compose --file ilias/docker-compose.yml --env-file ilias/.ilias_env_ilias-schule-3.example.org logs

[...]
ilias_1  | ILIAS installed successfully!
ilias_1  | Log in using the following credentials:
ilias_1  | 
ilias_1  | User:           root
ilias_1  | Password:       jasiDja789afas868
ilias_1  | Setup Password: gsddfs690aNsklfnal
[...]
```

## Update
Diese Anleitung beinhaltet nicht, was eventuell bei Updates der verschiedenen Software-Komponenten zu beachten ist. Bitte immer vorher deren entsprechende Release Notes prüfen, ob etwas zu beachten ist.

```bash
./update
The following operation will update all container images defined in docker-compose files of the projects. You should backup or create a VM snapshot before doing that. Do you want to start? (Y/N) y
Pulling nginx images...
Pulling nginx-proxy             ... done
Pulling nginx-proxy-letsencrypt ... done
Pulling ilias images (you can ignore warning messages that variables are not set)...
WARNING: The ILIAS_AUTO_SETUP variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_DB_USER variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_DB_PASSWORD variable is not set. Defaulting to a blank string.
WARNING: The ILIAS_HOST_NAME variable is not set. Defaulting to a blank string.
WARNING: The VIRTUAL_HOST variable is not set. Defaulting to a blank string.
WARNING: The LETSENCRYPT_HOST variable is not set. Defaulting to a blank string.
WARNING: The MYSQL_ROOT_PASSWORD variable is not set. Defaulting to a blank string.
WARNING: The MYSQL_DATABASE variable is not set. Defaulting to a blank string.
Pulling mysql ... done
Pulling ilias ... done
Recreating containers...
Creating elearning docker network
Starting Nginx reverse proxy...
nginx-proxy is up-to-date
nginx-letsencrypt is up-to-date
Waiting Nginx to launch on Port 80...
Waiting an additional 10 seconds, just to be safe...
Nginx launched
Starting docker image with env file: ilias/.ilias_env_ilias-schule-1.example.org
ilias-schule-1exampleorg_mysql_1 is up-to-date
ilias-schule-1exampleorg_ilias_1 is up-to-date
Starting docker image with env file: ilias/.ilias_env_ilias-schule-2.example.org
ilias-schule-2exampleorg_mysql_1 is up-to-date
ilias-schule-2exampleorg_ilias_1 is up-to-date
Starting docker image with env file: ilias/.ilias_env_ilias-schule-3.example.org
ilias-schule-3exampleorg_mysql_1 is up-to-date
ilias-schule-3exampleorg_ilias_1 is up-to-date
Updated successfully
```
## Purge / Löschen
Um alles vom Server wieder zu löschen kann `purge` genutzt werden. Damit wird alles gelöscht, inkl. der persistenten Daten der Docker Container. Bitte nur machen, wenn alles gesichert wurde oder noch getestet wird. Da `docker system prune` genutzt wird, kann das auch andere Daten anderer Docker-Projekte enthalten, die nicht mehr im System referenziert sind.

```
./purge
The following operation will purge your docker system to start from scratch (can include data from other docker projects, if no references exist anymore). Do you know what you do and want to start from scratch? (Y/N) y
Stopping Nginx reverse proxy...
Stopping nginx-letsencrypt ... done
Stopping nginx-proxy       ... done
Removing nginx-letsencrypt ... done
Removing nginx-proxy       ... done
Network elearning is external, skipping
Stopping Ilias instances...
Stopping docker multi-container with env-file ilias/.ilias_env_ilias-schule-1.example.org
Stopping ilias-schule-1exampleorg_ilias_1 ... done
Stopping ilias-schule-1exampleorg_mysql_1 ... done
Removing ilias-schule-1exampleorg_ilias_1 ... done
Removing ilias-schule-1exampleorg_mysql_1 ... done
Network elearning is external, skipping
Stopping docker multi-container with env-file ilias/.ilias_env_ilias-schule-2.example.org
Stopping ilias-schule-2exampleorg_ilias_1 ... done
Stopping ilias-schule-2exampleorg_mysql_1 ... done
Removing ilias-schule-2exampleorg_ilias_1 ... done
Removing ilias-schule-2exampleorg_mysql_1 ... done
Network elearning is external, skipping
Stopping docker multi-container with env-file ilias/.ilias_env_ilias-schule-3.example.org
Stopping ilias-schule-3exampleorg_ilias_1 ... done
Stopping ilias-schule-3exampleorg_mysql_1 ... done
Removing ilias-schule-3exampleorg_ilias_1 ... done
Removing ilias-schule-3exampleorg_mysql_1 ... done
Network elearning is external, skipping
Deleted ilias/.ilias_env_ilias-schule-1.example.org
Deleted ilias/.ilias_env_ilias-schule-2.example.org
Deleted ilias/.ilias_env_ilias-schule-3.example.org
Deleted nginx/.nginx_proxy_env
Deleted Networks:
elearning

Total reclaimed space: 0B
Deleted Volumes:
ilias-schule-3exampleorg_ilias_www
nginx_nginx-proxy-certs
nginx_nginx-proxy-html
ilias-schule-1exampleorg_ilias_www
ilias-schule-3exampleorg_ilias_mysql
60abfc41a6a1ded771e06d68fd2cdc26382167c41ff37a146c45435662ce77dc
ilias-schule-3exampleorg_ilias_data
ilias-schule-1exampleorg_ilias_data
ilias-schule-2exampleorg_ilias_www
ilias-schule-2exampleorg_ilias_data
ilias-schule-2exampleorg_ilias_mysql
nginx_nginx-proxy-vhosts
ilias-schule-1exampleorg_ilias_mysql

Total reclaimed space: 674.2MB
Error: No such network: elearning
```
## Backup
Mithilfe des Skriptes `./backup` wird ein Backup aller Projekte angelegt.

```
./backup
Pausing nginx-proxy       ... done
Pausing nginx-letsencrypt ... done
[+] Backing up nginx project to backups/2020-03-27_21-48
    - Saving config to ./docker-compose.yml
    - Saving database dumps to ./dumps
[*] Backing up nginx__nginx-proxy to ./nginx-proxy...
    - Saving config to ./nginx-proxy/config.json
    - Saving logs to ./nginx-proxy/docker.out/err
    - Saving /var/run/docker.sock volume to ./nginx-proxy/volumes/var/run/docker.sock
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-vhosts/_data volume to ./nginx-proxy/volumes/var/lib/docker/volumes/nginx_nginx-proxy-vhosts/_data
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-html/_data volume to ./nginx-proxy/volumes/var/lib/docker/volumes/nginx_nginx-proxy-html/_data
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-certs/_data volume to ./nginx-proxy/volumes/var/lib/docker/volumes/nginx_nginx-proxy-certs/_data
    - Saving /var/lib/docker/volumes/b8c4d3a4e2b2b56ac09b4d4c879157c781251b8ffbc400117d3e6cd6afa8af01/_data volume to ./nginx-proxy/volumes/var/lib/docker/volumes/b8c4d3a4e2b2b56ac09b4d4c879157c781251b8ffbc400117d3e6cd6afa8af01/_data
[*] Backing up nginx__nginx-proxy-letsencrypt to ./nginx-proxy-letsencrypt...
    - Saving config to ./nginx-proxy-letsencrypt/config.json
    - Saving logs to ./nginx-proxy-letsencrypt/docker.out/err
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-html/_data volume to ./nginx-proxy-letsencrypt/volumes/var/lib/docker/volumes/nginx_nginx-proxy-html/_data
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-vhosts/_data volume to ./nginx-proxy-letsencrypt/volumes/var/lib/docker/volumes/nginx_nginx-proxy-vhosts/_data
    - Saving /var/lib/docker/volumes/nginx_nginx-proxy-certs/_data volume to ./nginx-proxy-letsencrypt/volumes/var/lib/docker/volumes/nginx_nginx-proxy-certs/_data
    - Saving /var/run/docker.sock volume to ./nginx-proxy-letsencrypt/volumes/var/run/docker.sock
[*] Compressing backup folder to backups/2020-03-27_21-48.tar.gz
tar: backups/2020-03-27_21-48/nginx-proxy-letsencrypt/volumes/var/run/docker.sock: socket ignored
tar: backups/2020-03-27_21-48/nginx-proxy/volumes/var/run/docker.sock: socket ignored
Total bytes written: 122880 (120KiB, 42MiB/s)
[√] Finished Backing up nginx to backups/2020-03-27_21-48.tar.gz.
Unpausing nginx-letsencrypt ... done
Unpausing nginx-proxy       ... done
Backing up docker-compose project with env_file ilias/.ilias_env_ilias-schule-1.example.org
Pausing ilias-schule-1exampleorg_mysql_1 ... done
[+] Backing up ilias project to backups/.ilias_env_ilias-schule-1.example.org/2020-03-27_21-48
    - Saving config to ./docker-compose.yml
    - Saving database dumps to ./dumps
[*] Backing up ilias__mysql to ./mysql...
    - Saving config to ./mysql/config.json
    - Saving logs to ./mysql/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-schule-1exampleorg_ilias_mysql/_data volume to ./mysql/volumes/var/lib/docker/volumes/ilias-schule-1exampleorg_ilias_mysql/_data
[*] Backing up ilias__ilias to ./ilias...
    - Saving config to ./ilias/config.json
    - Saving logs to ./ilias/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-schule-1exampleorg_ilias_www/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-schule-1exampleorg_ilias_www/_data
    - Saving /var/lib/docker/volumes/ilias-schule-1exampleorg_ilias_data/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-schule-1exampleorg_ilias_data/_data
[*] Compressing backup folder to backups/.ilias_env_ilias-schule-1.example.org/2020-03-27_21-48.tar.gz
Total bytes written: 230942720 (221MiB, 72MiB/s)
[√] Finished Backing up ilias to backups/.ilias_env_ilias-schule-1.example.org/2020-03-27_21-48.tar.gz.
Unpausing ilias-schule-1exampleorg_mysql_1 ... done
Backing up docker-compose project with env_file ilias/.ilias_env_ilias-schule-2.example.org
Pausing ilias-schule-2exampleorg_mysql_1 ... done
Pausing ilias-schule-2exampleorg_ilias_1 ... done
[+] Backing up ilias project to backups/.ilias_env_ilias-schule-2.example.org/2020-03-27_21-48
    - Saving config to ./docker-compose.yml
    - Saving database dumps to ./dumps
[*] Backing up ilias__mysql to ./mysql...
    - Saving config to ./mysql/config.json
    - Saving logs to ./mysql/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-schule-2exampleorg_ilias_mysql/_data volume to ./mysql/volumes/var/lib/docker/volumes/ilias-schule-2exampleorg_ilias_mysql/_data
[*] Backing up ilias__ilias to ./ilias...
    - Saving config to ./ilias/config.json
    - Saving logs to ./ilias/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-schule-2exampleorg_ilias_data/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-schule-2exampleorg_ilias_data/_data
    - Saving /var/lib/docker/volumes/ilias-schule-2exampleorg_ilias_www/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-schule-2exampleorg_ilias_www/_data
[*] Compressing backup folder to backups/.ilias_env_ilias-schule-2.example.org/2020-03-27_21-48.tar.gz
Total bytes written: 242339840 (232MiB, 71MiB/s)
[√] Finished Backing up ilias to backups/.ilias_env_ilias-schule-2.example.org/2020-03-27_21-48.tar.gz.
Unpausing ilias-schule-2exampleorg_ilias_1 ... done
Unpausing ilias-schule-2exampleorg_mysql_1 ... done
Backing up docker-compose project with env_file ilias/.ilias_env_ilias-schule-3.example.org
Pausing ilias-schule-3exampleorg_mysql_1 ... done
Pausing ilias-schule-3exampleorg_ilias_1 ... done
[+] Backing up ilias project to backups/.ilias_env_ilias-schule-3.example.org/2020-03-27_21-48
    - Saving config to ./docker-compose.yml
    - Saving database dumps to ./dumps
[*] Backing up ilias__mysql to ./mysql...
    - Saving config to ./mysql/config.json
    - Saving logs to ./mysql/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-schule-3exampleorg_ilias_mysql/_data volume to ./mysql/volumes/var/lib/docker/volumes/ilias-schule-3exampleorg_ilias_mysql/_data
[*] Backing up ilias__ilias to ./ilias...
    - Saving config to ./ilias/config.json
    - Saving logs to ./ilias/docker.out/err
    - Saving /var/lib/docker/volumes/ilias-schule-3exampleorg_ilias_data/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-schule-3exampleorg_ilias_data/_data
    - Saving /var/lib/docker/volumes/ilias-schule-3exampleorg_ilias_www/_data volume to ./ilias/volumes/var/lib/docker/volumes/ilias-schule-3exampleorg_ilias_www/_data
[*] Compressing backup folder to backups/.ilias_env_ilias-schule-3.example.org/2020-03-27_21-48.tar.gz
Total bytes written: 242780160 (232MiB, 75MiB/s)
[√] Finished Backing up ilias to backups/.ilias_env_ilias-schule-3.example.org/2020-03-27_21-48.tar.gz.
Unpausing ilias-schule-3exampleorg_ilias_1 ... done
Unpausing ilias-schule-3exampleorg_mysql_1 ... done
```

# Todos
* Anleitung / Skript für das Wiederherstellen aus dem Backup
* Weitere Tools dem Blueprint hinzufügen (aktuelle Version beinhaltet nur Ilias)

# Acknowledgements

Dieses Projekt setzt folgende Open Source Software-Komponenten ein bzw. ermöglicht ihre Nutzung:

* [Ilias - eLearning Plattform](https://www.ilias.de/)
* [nginx-proxy - nginx reverse proxy für Docker Container](https://github.com/nginx-proxy/nginx-proxy)
* [letsencrypt-nginx-proxy-companion - leichtgewichtiger companion container für nginx-proxy um mithilfe von Letsencrypt Zertifikate zu generieren](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion/)
* [Gist des Github Users "pirate" (Nick Sweeting) um ein docker-compose Projekt zu sichern](https://gist.github.com/pirate/265e19a8a768a48cf12834ec87fb0eed)
