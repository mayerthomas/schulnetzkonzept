# Raspberry Pi OS Lite

![logo raspberry pi os lite](../_media/logo_raspberry-pi-os.png "Provided by raspberry.com")

Raspberry Pi OS Lite ist ein sehr kleines, auf Debian basierendem System für den Raspberry Pi, einem preiswerten und stromsparenden Einplatinencomputer.

> **Hilfreiche Links**
> - [Offizielle Webseite des Raspberry Pi](https://www.raspberrypi.com/)
> - [Download Raspberry Pi Imager](https://www.raspberrypi.com/software/)
> - alternativ: [direkter Download Raspberry Pi OS Lite](https://www.raspberrypi.com/software/operating-systems/)

## Installation

### Raspberry Pi OS auf SD-Karte kopieren

Am einfachsten kopieren Sie das benötigte Image (Raspberry Pi OS (other) → Raspberry Pi OS (64-bit)) mit dem Raspberry Pi Imager (Link s. oben) auf eine Micro-SD-Karte. In der Regel genügt ein sehr kleines Medium (z. B. 4 GB). Stecken Sie die SD-Karte nach erfolgreichem Kopiervorgang in den Raspberry Pi.

### Erster Start

Booten Sie den Raspberry Pi, indem Sie ihn mit einer Stromquelle verbinden und melden sich erstmalig mit den Initial-Zugangsdaten (User: pi / PW: raspberry) an.

### Grundkonfiguration

Warten Sie ggf. die Update-Routine ab und starten Sie im Anschluss die initiale, grafisch gestützte Grundkonfiguration:

```bash
    sudo raspi-config
```

?> Sollte das Tastaturlayout noch nicht stimmen, orientieren Sie sich vorübergehend an der US-Tastaturbelegung: [https://de.wikipedia.org/wiki/Tastaturbelegung#USA](https://de.wikipedia.org/wiki/Tastaturbelegung#USA).

#### Mögliche Einstellungen in raspi-config

*   System Options
    *   Wireless LAN → ggf. hier Zugangsdaten für WLAN hinterlegen
    *   Password → Passwort für User pi ändern
    *   Hostname → ggf. Netzwerknamen des Rasperry Pi einstellen
    *   Boot / Auto Login → Console Autologin wählen
*   Display Options  
    *   Underscan → ggf. schwarze Display-Umrandung entfernen
    *   Screenblanking → mit "Yes" aktivieren
*   Interface Options  
    *   SSH → mit "Yes" aktivieren
*   Localisation Options
    *   Locale: de\_DE.UTF-8 aktivieren und als default locale festlegen
    *   Timezone: Europe/Berlin
    *   ggf. WLAN-Country festlegen
*   Advanced Options
    *   Expand Filesystem

Anschließend starten Sie das System mit "Reboot" neu.

### IP-Adresse herausfinden

Um nachfolgend den Raspberry Pi von der Ferne per SSH bedienen zu können bzw. um ggf. notwendige Firewalleinstellungen tätigen zu können, muss die IP-Adresse und ggf. auch die MAC-Adresse der aktiven Netzwerkkarte ermittelt werden:

```bash
    sudo ip address
```