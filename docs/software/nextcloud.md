# Nextcloud

![logo nextcloud](../_media/logo_nextcloud.png "Provided by nextcloud.com")

Mit Nextcloud speichern Sie Dateien datenschutzkonform auf Ihrem eigenen Server.

?> Die aktuellen Beschreibungen basieren auf Nextcloud in der Version 23.

> **Wichtige Funktionen der Nextcloud**
> - Umgang mit Dateien
>   - Vielfältige Datei-Zugriffsmöglichkeiten: Web, WebDAV (Netzlaufwerke), App, Synchronisationsclient, Webzugriff
>   - Verschlüsselung: sowohl auf Transport- als auch auf Dateiebene möglich
>   - personalisierter Zugriff je Benutzer
>   - gemeinsame Verzeichnisse auf Basis von Gruppenzugehörigkeiten
>   - granulare Einstellung von Zugriffsberechtigungen je nach Einsatzzweck (nur lesen, schreiben, zeitliche Begrenzung, ...)
> - Möglichkeit der Anbindung an vorhandenes Benutzer- und Gruppenverzeichnis (via LDAP)
> - Möglichkeit der Erweiterung der Nextcloud mit zahlreichen Apps (z. B. für Chat, E-Mail, Kalender, Kontakte, Aufgaben uvm.)

> **Hilfreiche Links**
> - [Nextcloud Downloadseite](https://nextcloud.com/install/)
> - [Nextcloud Dokumentationen](https://nextcloud.com/support/)
> - [Nextcloud Forum](https://help.nextcloud.com/)

## Installation

### Notwendige Pakete installieren

```bash
# Notwendige Tools
apt install sudo zip unzip -y
    
# Aktuelleres PHP (8.0) als das vom Debian-Repository (7.3) installieren
sudo apt -y install lsb-release apt-transport-https ca-certificates wget
sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
apt update
apt install php8.0 php8.0-common php8.0-cli

# Vorgeschlagene Pakete aus Nextcloud-Dokumentation
apt install apache2 mariadb-server libapache2-mod-php8.0
apt install php8.0-gd php8.0-mysql php8.0-curl php8.0-mbstring php8.0-intl
apt install php8.0-gmp php8.0-bcmath php-imagick php8.0-xml php8.0-zip
apt install imagemagick

# Zusätzlich für unser Konzept notwendig:
apt install php8.0-ldap php8.0-apcu
```

### Datenbankserver (MariaDB/MySQL) einrichten

Zunächst starten Sie einen textbasieren Assistenten zur Absicherung Ihres Datenbankservers. Dabei vergeben Sie bitte ein eigenes Passwort für den Datenbank-User root. Wenn Sie im weiteren Verlauf in einem Konsolenbefehl die Zeichenfolge \<dbuser-root-pw\> sehen, dann ersetzen Sie diese bitte mit Ihrem selbst vergebenen Passwort.

```bash
mysql_secure_installation

# Bitte tätigen Sie folgende → Angaben:

# Passwort für Datenbank-User root festlegen
# Remove anonymous users? [Y/n] → y
# Disallow root login remotely? [Y/n] → y
# Remove test database and access to it? [Y/n] → y
# Reload privilege tables now? [Y/n] → y
```

Nun erzeugen Sie einen Datenbank-User mit dem Namen nextcloud und eine gleichnamige Datenbank. Dabei vergeben Sie bitte ein eigenes Passwort für den Datenbank-User nextcloud. Wenn Sie im weiteren Verlauf in einem Konsolenbefehl die Zeichenfolge \<dbuser-nextcloud-pw\> sehen, dann ersetzen Sie diese bitte mit Ihrem selbst vergebenen Passwort.

Schließen Sie jede der folgenden Zeilen einzeln mit Eingabe der Enter-Taste ab:

```bash
mysql -uroot -p
<dbuser-root-pw>

CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY '<dbuser-nextcloud-pw>';
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost';
FLUSH PRIVILEGES;
quit;
```

### Webserver (Apache) einrichten

Zunächst erzeugen Sie eine Konfigurationsdatei für die Nextcloud-Webseite. Beachten Sie, dass die Datei als letzte Zeile eine Leerzeile hat.

```bash
vim /etc/apache2/sites-available/nextcloud.conf
```

```apacheconf
# Datei /etc/apache2/sites-available/nextcloud.conf

<VirtualHost *:80>
  DocumentRoot /var/www/nextcloud/
  ServerName  nextcloud.ihre-schule.de
  ServerAlias www.nextcloud.ihre-schule.de

  <Directory /var/www/nextcloud/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews

    <IfModule mod_dav.c>
      Dav off
    </IfModule>

  </Directory>
  
  <IfModule mod_headers.c>
      Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains; preload"
  </IfModule>
  
</VirtualHost>

#wichtig: Leerzeile als letzte Zeile der Datei!

```

Die Default-Konfiguration des Webservers deaktivieren und die Konfiguration für die Nextcloud aktivieren:

```bash
a2dissite 000-default.conf
a2ensite nextcloud.conf
```

In den PHP-Einstellungen für Apache muss noch der Default-Wert für das Memory-Limit auf 512 MB erhöht werden:

```bash
vim /etc/php/8.0/apache2/php.ini
```

```ini
# Datei /etc/php/8.0/apache2/php.ini

# Die nachfolgenden Zeilen finden und wie folgt abändern:
upload_max_filesize = 16G
post_max_size = 16G
memory_limit = 512M
output_buffering = 0
```

```bash
vim /etc/php/8.0/cli/conf.d/20-apcu.ini
```

```ini
# Datei /etc/php/8.0/cli/conf.d/20-apcu.ini

# einfügen:
apc.enable_cli=1
```

Im Anschluss aktivieren Sie die für den Betrieb von Nextcloud notwendige Apache-Module und starten den Webserver neu:

```bash
a2enmod rewrite
a2enmod headers
a2enmod env
a2enmod dir
a2enmod mime
systemctl restart apache2
```

### Nextcloud-Dateien herunterladen und entpacken

Suchen Sie auf der Downloadseite von Nextcloud nach dem aktuellsten Downloadlink für die ZIP-Datei von Nextcloud und laden diese auf der Konsole mit dem Programm wget herunter:

```bash
# In das www-Verzeichnis des Webservers wechseln
cd /var/www/

# Achtung: hier den selbst recherchierten
# aktuellen Link zur aktuellen ZIP-Datei verwenden:
wget https://download.nextcloud.com/server/releases/latest.zip

# ZIP-Datei entpacken - Achtung verwenden Sie den richten Namen:
unzip latest.zip

# Die ZIP-Datei wird nicht mehr benötigt und kann gelöscht werden:
rm latest.zip
```

### Dateiberechtigungen anpassen

```bash
chown -R www-data:www-data /var/www/nextcloud/
find /var/www/nextcloud/ -type d -exec chmod 750 {} \;
find /var/www/nextcloud/ -type f -exec chmod 640 {} \;
```

### Die Benutzerdateien außerhalb des Ordners /var/www aufbewahren

Zur Steigerung der Sicherheit erstellen wir den Ordner zur Aufbewahrung der Dateien aller Nextcloud-Benutzer außerhalb des www-Verzeichnisses:

```bash
mkdir /srv/nextcloud-data
chown www-data:www-data /srv/nextcloud-data/
```

### Beenden der Nextcloud-Installation

*   Rufen Sie die URL Ihrer Nextcloud auf, z. B.: nextcloud.ihre-schule.de
*   Legen Sie ein Administrator-Konto für Ihre Nextcloud an
*   Tätigen Sie folgende Angaben:
    *   Datenverzeichnis: /srv/nextcloud-data
    *   Datenbank: MySQL/MariaDB
    *   Datenbank-Benutzer: nextcloud
    *   Datenbank-Passwort: \<dbuser-nextcloud-pw\>
    *   Datenbank-Name: nextcloud
    *   Datenbank-Host: localhost

## Konfiguration

### Sinnvolle Anpassungen an der Nextcloud-Konfiguration

```bash
vim /var/www/nextcloud/config/config.php
```

```php
# Datei /var/www/nextcloud/config/config.php

# Folgende Einstellungen - falls noch nicht vorhanden - ergänzen/abändern:
'trusted_domains' => 
  array (
	  0 => 'nextcloud.ihre-schule.de',
	  # ggf. wollen Sie die www-Subdomäne ergänzen:
	  1 => 'www.nextcloud.ihre-schule.de',
  ),
# Zeit in Minuten, bis gelöschte LDAP-User auch in der Nextcloud gelöscht werden
'ldapUserCleanupInterval' => 15,
'memcache.local' => '\OC\Memcache\APCu',
'overwrite.cli.url' => 'https://nextcloud.ihre-schule.de/',
'overwriteprotocol' => 'https',
# index.php entfernen
'htaccess.RewriteBase' => '/',
# Wenn Sie einen Reverse-Proxy verwenden, tragen Sie dessen interne IP ein:
'trusted_proxies' => 
  array (
    0 => '10.1.100.1',
  ),
'forwarded_for_headers' => 
  array (
    0 => 'HTTP_X_FORWARDED_FOR',
  ),
# Mit nachfolgenden beiden Angaben landen alle Freigaben für einen Benutzer automatisch 
# im Ordner 'Mit mir geteilt', ohne dass der Nutzer hierzu explizit zustimmen muss.
'share_folder' => '/Mit mir geteilt',
'sharing.enable_share_accept' => false,
```

### Memory-Limit setzen
```bash
vim /var/www/nextcloud/.user.ini
```

```ini
# Datei /var/www/nextcloud/.user.ini

# einfügen:
memory_limit=512M
```

### Update der .htaccess-Datei

Damit der in der Datei config.php eingestellte Änderung in Bezug auf die Entfernung der index.php aus Nextcloud-URLs greift, muss die .htaccess-Datei mit folgendem Befehl aktualisiert werden:

```bash
cd /var/www/nextcloud
sudo -u www-data php occ maintenance:update:htaccess
```

### Installation und Konfiguration der LDAP-Verbindung zum Samba-Server

Sofern Sie die Nextcloud mit den Benutzern und Gruppen des Samba-Servers betreiben möchten, gehen Sie wie folgt vor:

*   Melden Sie sich bei Ihrer Nextcloud als Administrator an, klicken Sie auf Ihr Benutzer-Icon und wählen den Menüpunkt "Apps".
*   Suchen Sie unter App-Pakete nach "LDAP user and group backend" und aktivieren Sie dieses.
*   Stellen Sie vor dem nächsten Schritt sicher, dass der Samba-Server über eine verschlüsselte Verbindung erreichbar ist ([s. dazu Anleitung Samba](/software/samba)).
*   Nun können Sie mit der LDAP-Einrichtung beginnen. Begeben Sie sich hierzu zu den Einstellungen und darin zum Punkt LDAP / AD Integration:
    *   Server
        *   Host: ldaps://ldap.ihre-schule.de  
        *   Port: 636
        *   Benutzer-DN: CN=svc\_nextcloud,OU=services,OU=Users,OU=School,DC=schulnetz,DC=intra\
            (Passen Sie die Angaben an Ihre Installation an!)
        *   Passwort: <sambauser-Administrator-pw>
        *   Basis-DN: DC=schulnetz,DC=intra\
            (Passen Sie die Angaben an Ihre Installation an!)
    *   Benutzer
        *   Sofern Sie im Samba-Server einer Gruppe "nextcloud" angelegt haben und die Nextcloud nur für zu dieser Gruppe zugeordneten Benutzern zulassen wollen, wählen Sie bei "Nur aus diesen Gruppen:" die Gruppe "nextcloud" aus.
    *   Anmelde-Attribute
        *   Haken bei: LDAP-/AD-Benutzername:
    *   Gruppen
        *   Nur diese Objektklassen: group
        *   Wählen Sie bei "Nur aus diesen Gruppen" jene Gruppen aus, welche für die Datei- bzw. Freigabeverwaltung für Sie relevant sind, z. B. schueler, lehrer, 5a, ...
    *   Fortgeschritten
        *   Ordnereinstellungen:
            *   Anzeigenamen des Benutzers: displayName
            *   Basis-Benutzerbaum:  OU=Users,OU=School,DC=schulnetz,DC=intra\
                (Passen Sie die Angaben an Ihre Installation an!)                  
            *   Anzeigenamen der Gruppe: cn
            *   Basis-Gruppenbaum: OU=Groups,OU=School,DC=schulnetz,DC=intra\
                (Passen Sie die Angaben an Ihre Installation an!)  
            *   Assoziation zwischen Gruppe und Benutzer: member(AD)
        *   Spezielle Eigenschaften
            *   Kontingent Feld: postOfficeBox\
                (= Quota; wird im LDAP-Account-Manager je Benutzer bei Postfach hinterlegt; Beispiele: 500 MB, 1 GB, ...)
            *   E-Mail-Feld: mail
        *   Experte
            *   Interner Benutzername: userPrincipalName

### Einrichtung der Hintergrundaufgaben

Damit die Nextcloud reibungslos funktioniert muss sie im Hintergrund diverse Aufgaben erledigen. Der effektivste Weg dafür ist das Ausführen der Aufgaben als Benutzer www-data über einen Cronjob vom System anzustoßen:

```bash
crontab -u www-data -e
```

```bash
# Datei crontab

# am Ende der Datei einfügen:
# Ausführung von cron.php alle 5 Minuten:
*/5  *  *  *  * php -f /var/www/nextcloud/cron.php
```

Nun müssen Sie als Administrator in den Einstellungen der Nextcloud diese Variante der Hintergundaufgaben-Erledigung noch aktivieren:

*   Begeben Sie sich zum Punkt Grundeinstellungen.
*   Wählen Sie unter Hintergrund-Aufgaben die Option "Cron".

### Anlage von Ordnern und Freigaben

Zur gemeinsamen Zusammenarbeit an Dateien und Ordnern in der Schule schlage ich folgende Ordnerstruktur (samt Freigaben) vor, welche Sie als Nextcloud-Administrator anlegen:

*   Schule
    *   Klassen (Schreibzugriff für Gruppe Lehrer)
        *   5a (Schreibzugriff für Gruppe 5a)
        *   5b (Schreibzugriff für Gruppe 5b)
        *   ...
    *   Für Lehrer (Schreibzugriff für Gruppe Lehrer)
    *   Für Schüler (Schreibzugriff für Gruppe Lehrer, Lesezugriff für Gruppe Schüler)

Damit Benutzer an sie geteilte Ordner nicht an andere weiterverteilen können (z. B. Schüler der 5a teilt den Klassenordner an Schüler der 5b) empfiehlt es sich, bei den Freigabeeinstellungen des jeweiligen Ordners den Haken bei "Weiterverteilen erlauben" zu entfernen.

Über die oben dargestellten Freigaben hinaus kann jeder Benutzer eigene Dateien und Ordner (außerhalb des Ordners "Mit mir geteilt") mit beliebigen Personen und Gruppen teilen.

Wenn ein Benutzer einen Freigegebenen Ordner (versehentlich) entfernt hat, findet er diesen wieder im linken Seitenmenü unter "Freigaben → Mit Ihnen geteilt", "Freigaben → Gelöschte Freigaben" oder "Freigaben → Ausstehende Freigaben".

## Nextcloud verschlüsselt über vollqualifizierte Domänennamen erreichbar machen

Damit die Nextcloud nicht über Ihre IP sondern über z. B. https://nextcloud.ihre-schule.de sowohl von außen als auch von innerhalb des Schulhauses aufgerufen werden kann, lesen Sie in der Anleitung zu OPNsense folgende Abschnitte:

*   Vollqualifizierte Domänennamen für Schulnetzdienste
*   Installation und Konfiguration des Reverse-Proxy-Servers
*   Installation und Konfiguration der automatisierten Zertifikatsverwaltung

## Wartung

### Update der Nextcloud

```bash
cd /var/www/nextcloud
sudo -u www-data php updater/updater.phar
# Die Fragen zum Update mit Y (Yes) beantworten.
# Die Frage nach Beibehaltung des Wartungsmodus mit N (No) beantworten.

# Sollte es während des Upgrade-Prozesses zu Fehlern kommen, kann dieser mit folgendem Befehl erneut angestoßen werden:
sudo -u www-data php occ upgrade
```

### Gelöschte Samba- bzw. LDAP-Benutzer auch in der Nextcloud löschen

Wenn Sie auf dem mit der Nextcloud über LDAP verbundenen Samba-Server einen Benutzer löschen, kann sich dieser zwar nicht mehr in der Nexcloud anmelden, aber seine Daten sind nach wie vor im Ordner /srv/nextcloud-data vorhanden.

Zur Anzeige der Benutzer, welche die Nextcloud als "zu löschend" erkannt hat, führen Sie folgenden Befehl aus:

```bash
cd /var/www/nextcloud
sudo -u www-data php occ ldap:show-remnants
```

Um die User zu löschen führen Sie den nachfolgenden Befehl aus:
```bash
cd /var/www/nextcloud
sudo -u www-data php occ ldap:show-remnants | while read pipe user line ; do echo $user ; sudo -u www-data php occ user:delete $user ; done
```
?> Bitte beachten Sie, dass es bei obigem Löschbefehl zu einigen Fehlermeldungen kommt. Diese sind lediglich der Tatsache geschuldet, dass der Befehl zeilenweise die Nextcloud-Ausgabe der zu löschenden User abarbeitet. Da die Ausgabe auch Zeilen ohne einen Benutzer beinhaltet (Überschrift, Trennzeilen), kommt es in diesen zu Fehlern. Diese beeinflussen die Löschung der Benutzer aber nicht.