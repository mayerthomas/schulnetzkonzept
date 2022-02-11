# Debian

![logo debian](../_media/logo_debian.png "Provided by debian.org")

Linux Debian ist ein weit verbreitetes und überaus stabiles Betriebssystem, welches sich für viele in diesem Projekt beschriebenen Programme als Basis eignet. 

> **Hilfreiche Links**
> - [Debian Downloadseite](https://www.debian.org/CD/http-ftp/)
> - [Wiki des Debian-Forums](https://wiki.debianforum.de/Hauptseite)

## Installation

### Virtuelle Maschine erzeugen

1.  Offizielles stabiles amd64-Image von der Debian-Downloadseite herunterladen.
2.  Heruntergeladene ISO-Datei auf auf dem Virtualisierungs-Host ablegen.
3.  Auf dem Virtualisierungsserver eine neue virtuelle Maschine mit den in der jeweiligen Anleitung dargestellten virtuellen Hardwarekomponenten erzeugen.
4.  In den Einstellungen zur virtuellen Maschine als CD-Laufwerk das Debian-Image wählen und die virtuelle Maschine von diesem booten lassen.
5.  Die virtuelle Maschine einschalten und das Konsolenfenster öffnen. Nun sollte der Installationsassistent von Linux-Debian zu sehen sein.

### Grundinstallation von Debian

Bei der Grundinstallation von Debian sind folgende Schritte zu durchlaufen:

*   Auswahl der Installationsmethode: grafische Installation
*   Auswahl der Installationssprache
*   Auswahl des Installationsstandorts
*   Auswahl des Tastaturlayouts
*   Vergabe des Hostnamens
    *   Ich verwende als Hostnamen nextcloud, samba bzw. fog.
*   Angabe des Domänennamens
    *   Alle auf diese Weise installierten Systeme und OPNsense müssen den gleichen Domänennamen haben. Es empfiehlt sich, einen öffentlich erreichbaren Domänennamen zu vergeben. Am besten eignet sich hierzu eine Subdomain jener Domäne, welche Sie für Ihre Schulwebseite registriert haben.
    *   Beispiel für einen Domänennamen: directory.ihre-schule.de
*   Passwortvergabe für den Benutzer root
    *   Künftig melden Sie sich für alle beschriebenen Arbeiten an der Konsole als Benutzer root und dem vergebenen Passwort an.
*   Anlage eines weiteren Benutzers und Passwortvergabe
    *   Dieser Benutzer wird im Weiteren nicht benötigt, muss aber angelegt werden.
*   Festplattenpartitionierung
    *   Ich verwende hier: "Geführt - vollständige Festplatte verwenden" und belasse im weiteren Partitionierungsverlauf alle Einstellungen auf den bereits hinterlegten Auswahlmöglichkeiten.
*   Konfiguration des Paketmanagers
    *   Es wird keine weitere CD/DVD benötigt.
    *   Als Spiegelserver verwende ich die voreingestellte Auswahlmöglichkeit (Deutschland, deb.debian.org).
    *   HTTP-Proxy: sofern Sie im Netzwerk des Servers keinen oder einen transparenten Proxy verwenden, ist hier keine Angabe erforderlich
    *   Die Teilnahme an der Paktetverwendungserfassung ist optional.
*   Softwareauswahl - Ich verwende hier nur (alle anderen abwählen):
    *   SSH Server
    *   Standard-Systemwerkzeuge
*   Grub-Bootloader auf der Festplatte installieren
    *   Die Frage nach der Installation des Bootloaders im Master-Boot-Record mit Ja beantworten.
*   Erster Start des neu installierten Systems.
*   Das virtuelle Laufwerk mit dem Debian-Image wird nun nicht mehr benötigt und kann von der virtuellen Maschine entfernt werden.

### Debian auf den neusten Stand bringen und grundlegende Pakete installieren

```bash
# Paketlisten aktualisieren und installierte Pakete updaten
apt update && apt upgrade -y
   
# Texteditor vim installieren
apt install vim -y
```


### Netzwerkeinstellungen festlegen

```bash
# Bearbeiten der Datei für die Netzwerkeinstellungen mit dem Editor vim
vim /etc/network/interfaces
```

```bash
# Datei /etc/network/interfaces

# Die Zeile: iface <Interface-Name> inet dhcp abändern zu:
iface <Interface-Name> inet static

# Unter der abgeänderten Zeile "iface ..." folgende Angaben ergänzen:
# ACHTUNG: bei "address" die gewünschte IP angeben - z. B.:
address 10.1.1.1
netmask 255.255.240.0
gateway 10.1.0.1
dns-nameservers 10.1.0.1
```

## SSH-Zugriff für User root freischalten

!> **Achtung:** Das beschriebene Vorgehen hebelt die eigentlich erforderliche Anmeldung des Benutzers root unter Zuhilfenahmen von Zertifikaten aus. Ich für mich habe entschieden, dass die root-Anmeldung über SSH ohne Zertifikate im Alltag der praktikablere Weg ist. \
Möchten Sie alternativ die besonders sichere Variante der Anmeldung über Zertifikate verwenden, kann Ihnen [diese Beschreibung](https://www.df.eu/de/support/df-faq/cloudserver/anleitungen/ssh-logins-auf-public-private-key-umstellen/) helfen.\
Verzichten Sie in Folge auf die nachfolgenden Veränderungen an der Datei /etc/ssh/sshd\_conf.

```bash
vim /etc/ssh/sshd_config
```

```bash
# Datei /etc/ssh/sshd\_config

# Bei der Zeile "#PermitRootLogin prohibit-password" das vorangestellte 
# Rautezeichen entfernen und die Zeile wie folgt abändern:
PermitRootLogin yes

# Den SSH-Server neu staten, damit die Änderungen wirsam werden:
service ssh restart
```

Von nun an ist es möglich, auf den Debian-Server von anderen Clients im Netzwerk aus als Benutzer root via SSH (Konsolenzugriff z. B. mit Putty) und SCP (Dateitransfers z. B. mit WinSCP) zuzugreifen.


## Wartung

### Update des Debian-Systems
Das Update ist mit einem einzigen Befehl schnell erledigt. Vor Updates empfiehlt sich das Anfertigen einer Sicherung, z. B. durch das Erstellen eines Snapshots auf dem Virtualisierungs-Host.

```bash
# Paketlisten aktualisieren und installierte Pakete updaten
apt update && apt upgrade -y
```