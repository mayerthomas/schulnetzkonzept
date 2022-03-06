# Raspberry Pi OS Lite

![logo raspberry pi os lite](../_media/logo_raspberry-pi-os.png "Provided by raspberry.com")

Raspberry Pi OS Lite ist ein sehr kleines, auf Debian basierendem System für den Raspberry Pi, einem preiswerten und stromsparenden Einplatinencomputer.

> **Hilfreiche Links**
> - [Offizielle Webseite des Raspberry Pi](https://www.raspberrypi.com/)
> - [Download Raspberry Pi Imager](https://www.raspberrypi.com/software/)
> - alternativ: [direkter Download Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/)

## Installation

### Raspberry Pi OS Lite auf SD-Karte kopieren

Am einfachsten kopieren Sie das benötigte Image (Raspberry Pi OS (other) → Raspberry Pi OS Lite (64-bit)) mit dem Raspberry Pi Imager (Link s. oben) auf eine Micro-SD-Karte. In der Regel genügt ein sehr kleines Medium (z. B. 4 GB). Stecken Sie die SD-Karte nach erfolgreichem Kopiervorgang in den Raspberry Pi.

### Erster Start

Booten Sie den Raspberry Pi, indem Sie ihn mit einer Stromquelle verbinden und melden sich erstmalig mit den Initial-Zugangsdaten (User: pi / PW: raspberry) an.

?> Berücksichtigen Sie bei Eingaben auf der Konsole die standardmäßig eingestellte [englische Tastaturbelegung](https://de.wikipedia.org/wiki/Tastaturbelegung#USA).\
Auf einer QWERTZ-Tastatur tippen Sie für das Standard-Passwort also: raspberr**z**\
Da später auf den Raspberry Pi per ssh zugegriffen wird, ist eine Änderung der Tastaturbelegung für die Konsole eigentlich nicht erforderlich. 

### Grundkonfiguration

Warten Sie ggf. die Update-Routine ab und starten Sie im Anschluss die initiale, grafisch gestützte Grundkonfiguration:

```bash
    sudo raspi-config
	### Auf einer QWERTZ-Tastatur tippen Sie: sudo raspißconfig
```

#### Mögliche Einstellungen in raspi-config

*   System Options
    *   Wireless LAN
		* Country
		* SSID
		* Passphrase
    *   Password → Passwort für User pi ändern
    *   Hostname → ggf. Netzwerknamen des Rasperry Pi einstellen
    *   Boot / Auto Login → Console Autologin wählen (sofern sie den Raspberry Pi als Digital-Signage-System verwenden möchten)
*   Display Options  
    *   Underscan → ggf. schwarze Display-Umrandung entfernen
    *   Screenblanking → mit "Yes" aktivieren
*   Interface Options  
    *   SSH → mit "Yes" aktivieren
*   Localisation Options
    *   Locale bzw. Default locale: z. B. de_DE.UTF-8
    *   Timezone: Europe/Berlin
*   Advanced Options
    *   Expand Filesystem

Anschließend beenden Sie raspi-config ("Finish") und beantworten die Frage nach einem Reboot mit "Ja".

### IP-Adresse herausfinden

Um nachfolgend den Raspberry Pi von der Ferne per SSH bedienen zu können, muss die IP-Adresse und ggf. auch die MAC-Adresse der aktiven Netzwerkkarte ermittelt werden:

```bash
    sudo ip address
```

## Update von Raspberry Pi OS Lite

```bash
# Paketlisten aktualisieren und installierte Pakete updaten
sudo apt update && sudo apt upgrade -y
```