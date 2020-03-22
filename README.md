# Hosted Blueprint für Digital Learning für Schüler
## Allgemeine Informationen
Dieser Blueprint entsteht im Rahmen des #wirvsvirus Hackathons in der Gruppe #1_241_a_digitallearningfürschüler. Zielstellung ist aufzuzeigen, wie aus dem Free and Open Source Umfeld selbst gehostete Lösungen im Bildungsumfeld sofort genutzt werden könnten.

Folgende Free and Open Source Tools werden dazu genutzt:

* [Ilias eLearning Plattform](https://www.ilias.de/)
* [OpenMeetings Web-Konferenz Plattform](https://openmeetings.apache.org/)
* [Mattermost Messaging Plattform](https://mattermost.com/)

## Anleitung
* Bereitstellung eines (virtuellen) Servers (z. B. über einen Anbieter oder unter Nutzung eigener Infrastruktur in der Schule)
    * Betriebssystem z. B. Debian oder Ubuntu (Blueprint getestet mit Ubuntu 16.04)
    * Root-Zugang wird benötigt
    * Server muss je nach Anzahl SchülerInnen und LehrerInnen unterschiedlich dimensioniert werden (nicht Teil dieser Anleitung)
* Installation der Voraussetzungen

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
# Installieren von nginx als Reverse Proxy und certbot für SSL-Zerifikate von letsencrypt (Beispiel für Ubuntu 16.04, Anleitungen für andere Betriebssyteme gibt es hier: https://certbot.eff.org/)
apt-get install nginx
apt-get update
apt-get install software-properties-common
add-apt-repository universe
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install certbot python-certbot-nginx
```

* Installation des Blueprints

```bash
cd /srv
git clone --recurse-submodules https://github.com/Digital-Learning-fur-Schuler/hosted-blueprint.git
cd hosted-blueprint
./configure
# Aktuell manuelles Ändern der Passwörter in mattermost-docker/docker-compose.yml notwendig (externes Git Repository, wird hier nur eingebunden)
# Sicheres Speichern der generierten und manuell festgelegten Passwörter (z. B. in einem Passwort Safe wie KeePass)
```

* Konfiguration eines Reverse Proxies und von SSL-Zertifikaten

TODO (idealerweise in einem weiteren Docker Container)

* Start der Container
```bash
./start 
# Der erste Start kann eine ganze Weile dauern
# Das Login-Passwort für Ilias steht im Logfile nach der ersten Installation
docker-compose -f ilias/docker-compose.yml logs

[...]
ilias_1  | ILIAS installed successfully!
ilias_1  | Log in using the following credentials:
ilias_1  | 
ilias_1  | User:           root
ilias_1  | Password:       jasiDja789afas868
ilias_1  | Setup Password: gsddfs690aNsklfnal
[...]
```

## Update des Blueprints
Diese Anleitung ist vorbehaltlich größerer Änderungen bei Applikationen zu verstehen. In besonderen Fällen wird in den Release Notes dieses Blueprints ausgewiesen, wenn etwas zusätzlich beachtet werden muss.

```bash
./stop
# Backup der persistenten Daten (Anleitung fehlt noch)
git pull --recurse-submodules
./start
```

# Offene TODOs

* Aktuell statisch auf Testserver vorliegende Nginx Konfiguration in `configure` Skript aufnehmen (inkl. Templates)
* Reverse Proxy Konfiguration für OpenMeetings erzeugen
* Nginx evtl. ebenfalls als Docker Container bereitstellen (z. B. als Option)
* Mattermost Passwort mit env-File setzen