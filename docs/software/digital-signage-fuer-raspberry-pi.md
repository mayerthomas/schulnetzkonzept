# Digital Signage für Raspberry Pi

Mit einem Raspberry Pi und einem HDMI-Display lässt sich mit verhältnismäßig wenig Aufwand ein kostengünstiges und zuverlässiges Anzeigesystem (z. B. zur Anzeige des Vertretungsplans) umsetzen.

Die im Folgenden dargestellte Variante startet lediglich einen Browser mit einer gewünschten Webseite im Vollbildmodus, lädt diese in gewünschten Abständen neu und verdunkelt zu gewünschten Zeiten den Bildschirm. Das Konzept geht also davon aus, dass die anzuzeigenden Inhalte bereits in Form einer externen Webseite vorliegen.

## Notwendige Pakete installieren

```bash
# Paketlisten aktualisieren und installierte Pakete updaten
sudo apt update && sudo apt upgrade -y
    
# Notwendige Software installieren
sudo apt install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox xdotool chromium-browser -y
```

## Skript für automatisches Neuladen des Chrome-Browsers erstellen

Damit die gewünschte Webseite auch nach einem etwaigen Verlust der Netzwerkverbindung erneut geladen werden kann, ist nachfolgendes Skript notwendig. Diese führt mit Hilfe des Programms xdotool die Tastenkombination Shift+F5 auf dem geöffneten Browserfenster aus. Weiter unten wird ein Task eingerichtet, der diese Aktion alle 5 Minuten anstößt.

```bash
nano /home/pi/chromium-reload.sh
```

```bash
# Datei /home/pi/chromium-reload.sh

#!/bin/bash
WID=$(xdotool search --onlyvisible --class chromium|head -1)
xdotool windowactivate ${WID}
xdotool key Shift+F5
```

## Skript-Datei ausführbar machen

```bash
chmod 755 /home/pi/chromium-reload.sh
```

## Einstellungen für den Autostart des X-Servers
```bash
sudo nano /etc/xdg/openbox/autostart
```
```bash
# Datei /etc/xdg/openbox/autostart

# Am Dateiende einfügen:
# Bildschirmschoner und Bildschirmabschaltung deaktivieren
xset s off
xset s noblank
xset -dpms

# Beenden des X-Servers via Strg+Alt+Backspace
setxkbmap -option terminate:ctrl_alt_bksp

# Chromium im Kiosk-Modus mit gewünschter URL starten
chromium-browser --kiosk --touch-events=enabled --disable-pinch --noerrdialogs --disable-session-crashed-bubble --simulate-outdated-no-au='Tue, 31 Dec 2099 23:59:59 GMT' --disable-component-update --overscroll-history-navigation=0 --disable-features=Translate --app=https://de.wikipedia.org
```

!> Ersetzen Sie in obiger Datei die URL https://www.wikipedia.org durch die anzuzeigende Webseite.

## X-Server ohne Cursor nach Console-Autologin automatisch starten

```bash
nano /home/pi/.bash_profile
```

```bash
# Datei /home/pi/.bash\_profile

[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && startx -- -nocursor
```

## Cronjobs für automatische Bildschirmabschaltung, Neustart und Chromium-Reload-Skript einrichten

```bash
crontab -e
# ggf. Nano als Editor auswählen
```

```bash
# Datei crontab des users pi

#nach letzter Zeile hinzufügen:
    
# Display ausschalten täglich von Montag bis Freitag um 16:00 Uhr
0 16 * * 1-5 xset -display :0 dpms force off

# Neustart täglich von Montag bis Freitag um 06:30 Uhr - schaltet auch Display wieder ein
30 6 * * 1-5 sudo reboot

# Skript chromium-reload.sh alle 5 Minuten ausführen
*/5 * * * * sh /home/pi/chromium-reload.sh
```

## HDMI-Einstellungen

Sofern das Display nach dem Booten des Raspberry Pi eingeschaltet wird, kann es vorkommen, dass der HDMI-Ausgang des Mini-Computers nicht aktiviert ist. Außerdem wird beim im Cronjob festgelegten Abschalten des Displays der Bildschirm standardmäßig nicht abgeschaltet (HDMI-Blanking) sondern nur schwarz geschaltet, was unnötig Energie verschwendet. Sofern Sie die Einstellung für HDMI-Blanking setzen, müssen Sie sicherstellen, dass der eingesetzte Bildschirm sich wieder ordnungsgemäß aktiviert, sobald der Raspberry Pi darauf wieder etwas anzeigen will.

```bash
sudo nano /boot/config.txt
```

```bash
# Datei /boot/config.txt

# Kommentarzeichen (#) entfernen bei:
hdmi_force_hotplug=1
hdmi_drive=2
```

## Gerät neu starten

```bash
sudo reboot
```

## Einstellungen am Browser vornehmen

Manche Setups erfordern weitere Anpassungen am Browser (z. B. das Deaktivieren des Angebots von Übersetzungen). Um den Browser mit Menü bzw. Einstellungsmöglichkeiten zu starten, gehen Sie wie folgt vor:

- Am Raspberry Pi mit angeschlossener Tastatur und Tastenkombination Strg+Alt+Backspace den X-Server schließen
- Auf Konsole einen neuen X-Server mit Cursor starten: startx
- Mit Tastenkombination Alt+F4 Chromium im Kiosk-Modus schließen
- Rechtsklick auf Desktop → Web Browser