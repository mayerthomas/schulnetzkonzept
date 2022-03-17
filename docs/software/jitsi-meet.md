# Jitsi Meet

![logo jitsi-meet](../_media/logo_jitsi-meet.png "Provided by jitsi.org")


Jitsi ist eine Sammlung freier Software u. a. für Videokonferenzen und Instant Messaging. Unter bestimmten Voraussetzungen (Bandbreite, Server-Kapazität) können parallel stattfindende Videokonferenzen mit sehr vielen Teilnehmern durchgeführt werden.

Die nachfolgend dargestellte Installation hat mehrere Tests mit 60 gleichzeitigen Teilnehmern ohne Probleme absolviert. Bei mehrheitlich aktivierten Kameras der Teilnehmer lag die Auslastung von CPU und RAM bei ca. 70 %, bei überwiegend deaktivierten Kameras bei ca. 10 %. Der eingehende Datenverkehr belief sich bei einem 45-minütigen Test auf ca. 13 GByte bei einer durchschnittlichen Bandbreite von 16 Mbit/s. Der ausgehende Datenverkehr betrug ca. 22 GByte bei einer Bandbreite von durchschnittlich 43 MBit/s. Ich vermute, dass das System mit den hier angegebenen Mindestanforderungen mit insgesamt 100 gleichzeitigen Teilnehmern - verteilt auf einen oder mehrere Räume - gut umgehen kann.

Das System ist so konfiguriert, dass Konferenzen mit zwei Teilnehmern zwar über Jitsi initiiert werden, technisch dann aber direkt über die beiden Teilnehmer laufen und den Jitsi-Server somit nicht belasten. Erst ab drei Teilnehmern in einem Raum greift die Jitsi-Videobridge. Dies hat den enormen Vorteil, dass Events mit einer sehr großen Zahl an Zwei-Teilnehmer-Gesprächen (z. B. bei virtuellen Elternabenden) problemlos über das System abgewickelt werden können.

Für viele gleichzeitige Videokonferenzen mit mehr als 100 Teilnehmern kann man das System zudem sehr gut skalieren, indem mehrere Instanzen der Jitsi-Videobridge eingebunden werden (hier nicht gezeigt).

> **Wichtige Funktionen von Jitsi Meet**
> - Videotelefonie
> - Chat während Videotelefonie
> - Initialisierung von Videotelefonaten nur durch bestimmte Personengruppen
> - Bildschirmfreigabe
> - Hohe Kapazität
> - Break-Out-Räume

> **Hilfreiche Links**
> - [Jitsi-Projekt-Webseite](https://jitsi.org/jitsi-meet/)
> - [Jitsi Installationshandbuch](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-quickstart)
> - [Jitsi Benutzerhandbuch](https://jitsi.github.io/handbook/docs/user-guide/user-guide-start)
> - [Beschreibung Jitsi und LDAP](https://github.com/jitsi/jitsi-meet/wiki/LDAP-Authentication)

## Voraussetzungen

Um die für Jitsi Meet notwendige Bandbreite (ca. 2 MBit/s je Teilnehmer) sicherstellen zu können, empfiehlt sich die datenschutzkonforme Installation der erforderlichen Komponenten auf einer virtuellen Maschine bei einem gut angebundenen deutschen Hoster und nicht im eigenen Schulnetz.

Die Mindestanforderungen sollten sich mit ca. 10 bis 15 Euro monatlich abdecken lassen:

* mindestens 4 CPUs
* mindestens 8 GB Arbeitsspeicher
* mindestens 32 GB Festplattenspeicher
* Internetanbindung mit 1 Gbit/s duplex

Selbstverständlich können bei entsprechender Bandbreite und Serverinfrastruktur die erforderlichen Komponenten auch im eigenen Schulnetz installiert werden. Hier müssten dann gem. der [OPNsense-Anleitung](betriebssysteme/opnsense.md) des Schulnetzkonzepts entsprechende Einstellungen im HAProxy und für die Zertifikatsvergabe getätigt werden.

Weitere Voraussetzungen:

* Debian-Server mit SSH-Zugriff
* Eine Subdomain, welche mittels A- oder CNAME-Record auf den Debian-Server zeigt, z. B. meet.ihre-schule.de

## Installation

### Paketquellen aktualisieren und erforderlicher Pakete installieren

```bash
apt update
apt install gnupg2 nginx-full sudo curl wget openjdk-11-jdk apt-transport-https -y
```

### Jitsi-Repository hinzufügen und Paketquellen aktualisieren

```bash
curl https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list > /dev/null

sudo apt update
```

### Installation und Konfiguration der Firewall

Da der virtuelle Server direkt beim Anbieter im Internet läuft, empfiehlt sich die Installation einer Firewall.

```bash
# Installation der Firewall:
apt install ufw -y

# Konfiguration und Aktivierung der Firewall:
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 10000/udp
sudo ufw allow 22/tcp
sudo ufw allow 3478/udp
sudo ufw allow 5349/tcp
# Port 5222 für XMPP-Clients:
sudo ufw allow 5222/tcp
sudo ufw enable
```

### Installation von Jitsi Meet

```bash
sudo apt install jitsi-meet

# Beim Installationsprozess wird man nach dem Hostnamen gefragt.
# Dieser muss mit der oben festgelegten Subdomain übereinstimmen: 
# z. B. meet.ihre-schule.de
# Weiterhin muss für die verschlüsselte Kommunikation ein Zertifikat
# installiert werden. Hier wird empfohlen die Option 
# "Generate a new self-signed certificate" zu verwenden.
# Im weiteren Installationsverlauf wird dieses Zertifikat durch ein
# Let's-Encrypt-Zertifikat ersetzt.
```

### Installation eines Let's-Encrypt-Zertifikats

Damit der Zugriff auf denn Jitsi Meet-Dienst verschlüsselt erfolgt, empfiehlt sich die Installation eines Zertifikats. Am einfachsten und zudem kostenlos erfolgt dies über Let's Encrypt.

Während der Zertifikatsinstallation ist darauf zu achten, dass man eine gültige E-Mail-Adresse für Benachrichtigungen und die richtigen Hostnamen (z. B. meet.ihre-schule.de) angibt.

```bash
sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

### Weitere Konfiguration

Damit nur authentifizierte Benutzer die Möglichkeit haben, eine Videokonferenz zu initiieren sind diverse Einstellungen nötig.

Zunächst wird beschrieben, wie die Absicherung für in Jitsi angelegte Benutzer erfolgt. Weiter unten wird diese Beschreibung um die Möglichkeit ergänzt, die Benutzer eines vorhandenen [Samba-Servers](software/samba.md) via LDAP einzubinden.

Auch die Konfiguration von Breakout-Rooms wird in diesem Abschnitt erläutert.

### Authentifizierung, Gäste-Login und Breakout-Rooms aktivieren

Damit nicht jeder Besucher Ihrer Jitsi-Seite eine Videokonferenz initiieren kann, muss die Authentifizierung aktiviert werden. Damit aber trotzdem nicht authentifizierte Gäste an einer Konferenz teilnehmen können, muss zudem der Gäste-Login eingerichtet werden.

```bash
# Achtung: passen Sie im nachfolgenden Befehl die Domänenangabe an!
vim /etc/prosody/conf.avail/[meet.ihre-schule.de].cfg.lua
```

```bash
# Datei /etc/prosody/conf.avail/\[meet.ihre-schule.de\].cfg.lua

# Suchen Sie den zu Ihrer Domäne passenden Virtual-Host-Eintrag und ändern sie 
# den Wert bei authentication von anonymous zu internal_hashed:
VirtualHost "meet.ihre-schule.de"
    authentication = "internal_hashed"
    # ...

# Für Breakout-Rooms innerhalb von modules_enabled {}
# die Zeile "Muc_breakout_rooms"; ergänzen:
modules_enabled = {
	# ...
	"muc_breakout_rooms";
}

# Für Breakout-Rooms unterhalb der Zeile main_muc
# die Zeile breakout_rooms_muc... ergänzen:
main_muc = "conference.meet.ihre-schule.de"
breakout_rooms_muc = "breakout.meet.ihre-schule.de"

# Zur Aktivierung des Gäste-Logins fügen Sie unterhalb des 
# Virtual-Host "meet.ihre-schule.de" den nachfolgenden Block ein. 
# Passen sie dabei den Domänennamen entsprechend an:
VirtualHost "guest.meet.ihre-schule.de"
    authentication = "anonymous"
    c2s_require_encryption = false

# Für Breakout-Rooms am Ende der Datei folgende Angaben einfügen:
Component "breakout.meet.ihre-schule.de" "muc"
    restrict_room_creation = true
    storage = "memory"
    admins = { "focus@auth.meet.ihre-schule.de" }
    muc_room_locking = false
    muc_room_default_public_jids = true
```

### Anpassungen an der Jitsi Meet-Konfiguration

```bash
# Achtung: passen Sie im nachfolgenden Befehl die Domänenangabe an!
vim /etc/jitsi/meet/[meet.ihre-schule.de]-config.js
```

```bash
# Datei /etc/jitsi/meet/\[meet.ihre-schule.de\]-config.js

# Ergänzen Sie unter Hots den Wert anonymousdomain, und passen Sie dabei den 
# Domänennamen entsprechend an:
var config = {
    hosts: {
            domain: 'jitsi-meet.example.com',
            anonymousdomain: 'guest.meet.ihre-schule.de',
            # ...
        },
    # ...
    # Damit jeder Teilnehmer zu Beginn einer Videokonferenz aufgefordert wird,
    # seinen Namen einzugeben, suchen Sie die Zeile
    # "// requireDisplayName: true," und entfernen Sie die Kommentierungszeichen:
    requireDisplayName: true,
    # ...
}
```

### Anpassungen an der Jicofo-Konfiguration

```bash
vim /etc/jitsi/jicofo/sip-communicator.properties
```

```bash
# Datei /etc/jitsi/jicofo/sip-communicator.properties

# Ergänzen Sie die nachfolgende Zeile und passen Sie dabei 
# den Domänennamen entsprechend an:
org.jitsi.jicofo.auth.URL=XMPP:meet.ihre-schule.de
```

### User in Prosody verwalten

Sofern Sie den Jitsi-Server nicht via ldap am Samba-Server anbinden wollen, müssen die Benutzer in Jitsi selbst verwaltet werden.

Wenn Sie den Samba-Server einbinden wollen, überspringen Sie diesen Punkt und lesen weiter unten weiter.

```bash
# Ersetzen Sie bei den nachfolgenden Befehlen <username>, <password>
# meet.ihre-schule.de, meet%2eihre-schule%2ede durch entsprechende Werte.

# Benutzer anlegen
sudo prosodyctl register <username> meet.ihre-schule.de <password>

# Benutzer auflisten
ls /var/lib/prosody/meet%2eihre-schule%2ede/accounts

# Benutzer löschen
sudo prosodyctl unregister <username> meet.ihre-schule.de

# Nach der Anlage und dem Löschen von Usern 
# müssen die Dienste des Jitsi-Servers neu gestartet werden:
systemctl restart prosody
systemctl restart jicofo
systemctl restart jitsi-videobridge2

# Da die Passwörter bei der Benutzeranlage unverschlüsselt über Konsolenbefehle 
# eingegeben werden, empfiehlt sich nach jeder Benutzeranlage 
# das Löschen der Bash-History:
history -c
```

### Wie Benutzer ihr Passwort selbst ändern

Leider bietet die Jitsi-Web-Oberfläche dem Benutzer nicht die Möglichkeit, sein Passwort selbst zu ändern. Da diese Möglichkeit gemäß Datenschutzgrundverordnung aber erforderlich ist, muss der User hierfür einen kleinen Umweg über einen XMPP-Client gehen, welcher hier am Beispiel des Programms Pidgin skizziert wird:

* Programm Pidgin installieren
* Konto hinzufügen
    * Protokoll: XMPP
    * Jitsi-Benutzernamen eintragen
    * Domain: z. B. meet.ihre-schule.de
    * Jitsi-Passwort eintragen
* Unter Konten die Funktion zum Passwort ändern aufrufen

## Zur Authentifizierung den Samba-Server via LDAP einbinden

Wenn für die Authentifizierung am Jitsi-Server der Samba-Server angebunden werden soll, sind zu dem oben beschriebenen Vorgehen zusätzlich folgende Dinge zu erledigen.

Folgende Voraussetzungen müssen für die nachfolgende Beschreibung erfüllt sein:

* Der LDAP-Server ist via ldaps und Port 636 vom Internet aus erreichbar.
* Der LDAP-Server ist verschlüsselt und über ein gültiges Zertifikat (z. B. via ldap.ihre-schule.de) erreichbar.

### Erforderliche Pakete installieren

```bash
apt install sasl2-bin libsasl2-modules-ldap lua-cyrussasl
```
```bash
# Datei /etc/prosody/conf.avail/\[meet.ihre-schule.de\].cfg.lua

# Suchen Sie den zu Ihrer Domäne passenden Virtual-Host-Eintrag und ändern sie 
# den Wert bei authentication zu cyrus:
VirtualHost "meet.ihre-schule.de"
    authentication = "cyrus"
    # ...
    
# Ergänzen Sie unmittelbar oberhalb der Sektion moduls_enabled
# die folgenden beiden Zeilen und fügen Sie auth_cyrus innerhalb der Sektion
# moduls_enabled ein:

cyrus_application_name = "xmpp"
allow_unencrypted_plain_auth = true

modules_enabled = {
    # ...
    "auth_cyrus";
}
```

### saslauthd konfigurieren

```bash
mkdir /etc/sasl
vim /etc/sasl/xmpp.conf
```

```bash
# Datei /etc/sasl/xmpp.conf

# Fügen Sie die folgenden Zeilen ein:
pwcheck_method: saslauthd
mech_list: PLAIN
```

```bash
vim /etc/saslauthd.conf
```

```bash
# Datei /etc/saslauthd.conf

# Passen Sie die folgenden Angaben gemäß Ihrer Samba-Installation an:
ldap_servers: ldaps://ldap.ihre.schule.de
ldap_port: 636
ldap_search_base: OU=School,dc=schulnetz,dc=intra
ldap_bind_dn: CN=svc_jitsi,OU=services,OU=Users,OU=School,DC=schulnetz,DC=intra
ldap_bind_pw: PassW0rd
ldap_filter: (samaccountname=%u)
ldap_version: 3
ldap_auth_method: bind
```

Sollen am Jitsi-Server nur bestimmte Benutzergruppen das Recht haben, eine Videokonferenz starten zu können, kann dies über den Parameter ldap_filter eingestellt werden. 

**Beispiel:**

* Alle User, welche das Recht zum Initiieren einer Jitsi-Videokonferenz haben sollen, gehören folgender Gruppe an: jitsi
* Die Gruppe jitsi liegt in der Samba-Domäne an folgender Stelle (DN): CN=jitsi,OU=Groups,OU=School,DC=schulnetz,DC=intra

    ldap_filter: (&(samaccountname=%u)(memberof=CN=jitsi,OU=Groups,OU=School,DC=schulnetz,DC=intra))

### Datei saslauthd editieren

```bash
vim /etc/default/saslauthd
```

```bash
# Datei /etc/default/saslauthd

# Ändern Sie den Wert bei START auf yes:
START=yes

# Ändern Sie den Wert von MECHANISM auf "ldap":
MECHANISMS="ldap"

# Ändern sie den Wert bei MECH_OPTIONS auf "/etc/saslauthd.conf"
MECH_OPTIONS="/etc/saslauthd.conf"
```

### Den Benutzer prosody der Gruppe sasl hinzufügen

```bash
usermod -aG sasl prosody
```

### Dienste neu starten

```bash
service saslauthd restart
service prosody restart
```

## Fehlersuche

### Browser

Jitsi funktioniert grundsätzlich mit allen modernen Browsern. Offiziell empfohlen wird Google Chrome. Ggf. kann ein Schließen und erneutes Öffnen des Browsers oder der Wechsel des Browsers bei Problemen helfen.

### Log-Dateien

Auch ein Blick auf die Log-Dateien des Jitsi-Servers kann hilfreich sein. Hier die wichtigsten Dateien hierzu:

* /var/log/jitsi/jvb.log
* /var/log/jitsi/jicofo.log
* /var/log/prosody/prosody.log
* /var/log/prosody/prosody.err