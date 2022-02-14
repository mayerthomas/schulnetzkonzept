# FOG-Project

![logo fog-project](../_media/logo_fog-project.png "Provided by fogproject.org")

Mit der Open-Source-Software FOG können Sie über das Netzwerk selbst erstellte Images auf zahlreiche Computer gleichzeitig verteilen. 

> **Wichtige Funktion von FOG**
> - Inventarisierung und Gruppierung von Computersystemen
> - Verwaltung von Images und Verteilung dieser via PXE-Boot
> - Snapins: Verteilung/Ausführung von Dateien und Skripten (z. B. zum Softwaredeployment oder zur Änderung von Einstellungen)
> - Energieverwaltung von Computersystemen (z. B. Fernabschaltung zu definierten Zeiten)
> - uvm.

Die aktuellen Beschreibungen basieren auf FOG in der Version 1.5.

> **Hilfreiche Links**
> - [FOG Projektseite](https://fogproject.org/)
> - [FOG Forum](https://forums.fogproject.org/)
> - [FOG Wiki](https://wiki.fogproject.org/)

## Installation

### Notwendige Pakete installieren

```bash
# Notwendige Tools
apt install git sudo -y
```

### Installation von FOG

```bash
sudo -i
git clone https://github.com/FOGProject/fogproject.git /root/fogproject
cd /root/fogproject/bin
./installfog.sh
```

> Während des Installationsprozesses sind folgende Angaben zu tätigen:
> - Linux-Version: 2 (Debian bases)
> - Installationsmethode: N (normal Server)
> - Änderung der Default-Netzwerkkarte: N (default network interface)
> - Router-Adresse für den DHCP-Server: N (router address for the dhcp server)
> - DNS-Anfragen durch DHCP: N (DHCP to handle DNS)
> - FOG als DHCP-Server: N (FOG server for DHCP service)
> - Zusätzliche Sprachpakete: N (additional language packs)
> - HTTPS aktivieren: N (https und Zertifikat können über OPNsense realisiert werden)
> - Änderung des Hostnamens: N
> - Installation Fortsetzen: Y (wish to coninue)

Wenn die Installation an der Zeile "Press [Enter] key when database is updated/installed." angelangt ist, rufen Sie über den Browser die Weboberfläche von FOG (z. B. http://10.1.1.1/fog) auf und klicken dort auf den Button "INSTALL/UPGRADE NOW".

Anschließend kehren Sie zur Konsole zurück und Drücken die Enter-Taste, um die Installation abzuschließen.

Nun können Sie sich mit Benutzernamen "fog" und Passwort "password" auf der Weboberfläche von FOG anmelden. Bitte ändern Sie umgehend das Passwort von FOG unter "User Management" → "List all users".

### Einstellungen im DHCP-Server (z. B. von OPNsense)

Damit Clients beim Booten über das Netzwerk beim FOG-Server landen, müssen beim DHCP-Server folgende Einstellungen getätigt werden

- next-server: z. B. 10.1.1.1 (IP Ihres FOG-Servers)
- filename: undionly.kpxe
- UEFI 32 bit file name : ipxe32.efi
- UEFI 64 bit file name: ipxe.efi

## Update von FOG

Vor Updates empfiehlt sich das Anfertigen einer Sicherung, z. B. durch das Erstellen eines Snapshots auf dem Virtualisierungs-Host.

Als FOG-Administrator können Sie unter "FOG Configuration" in Erfahrung bringen, ob Updates bereitstehen. Wenn dem so ist, gehen Sie wie folgt vor:

```bash
cd /root/fogproject
git pull
cd bin
./installfog.sh

# Beantworten Sie Fragen während des Update-Prozesses analog zu Ihren Antworten beim Installationsprozess (s. oben).
```

## Software-Deployment für Windows-Clients

### Winget und FOG-Snapins
Mit Winget stellt Microsoft ein Tool zur Verfügung, mit dem sich zahlreiche Programme mit einem einzigen Befehl über die Kommandozeile installieren, updaten oder deinstallieren lassen. Winget ist ab Windows 11 vorinstalliert. Für Windows 10 muss das Programm [App-Installer](https://www.microsoft.com/en-us/p/app-installer/9nblggh4nns1) in seiner aktuellsten Version installiert werden, um Winget nutzen zu können. 

Mithilfe der FOG-Snapins können in .bat-Dateien hinterlegte Winget-Befehle an Windows-Clients im Netzwerk gesendet werden. Hierzu muss auf den Windows-Clients der FOG-Client installiert werden. Die Installationsdatei für den FOG-Client finden Sie auf der Webseite Ihrer FOG-Installation im Footer.

### Beispiele für Winget-Befehle

Starten Sie zum Ausführen von winget-Befehlen eine Kommandozeile mit Administrator-Berechtigungen.

```batch
:: Suche nach Programmen mit dem Begriff "firefox"
winget install firefox

:: Automatische Installation des Pakets mit der id "Mozilla.Firefox"
winget install -h --id Mozilla.Firefox

:: Automatische Deinstallation des Pakets mit der id "Mozilla.Firefox"
winget uninstall -h --id Mozilla.Firefox

:: Automatische Aktualisierung des Pakets mit der id "Mozilla.Firefox"
winget upgrade -h --id Mozilla.Firefox

:: Automatische Aktualisierung aller Pakete, welche über winget verfügbar sind
winget upgrade -h --all

:: Anzeige installierter Pakete
winget list

:: Anzeigen der allgemeinen Hilfe für Winget
winget --help

:: Anzeigen der Hilfe für das Teilprogramm install von Winget
winget install --help
```

?> Anstelle der Konsole kann für die Suche von Programmen auch die Webseite [winget.run](https://winget.run/) verwendet werden. Diese zeigt für gefundene Anwendungen entsprechende Installationsbefehle an.

### Beispiel für FOG-Snapin mit Winget-Befehlen

#### Datei Browser-Installation.bat erzeugen

```batch
:: Datei Browser-Installation.bat
winget install -h --id Mozilla.Firefox
winget install -h --id Opera.Opera
```

#### Snapin in FOG anlegen

- Snapins → Create New Snapin
- Snapin Name: z. B. Browser-Installation
- Snapin Type: Normal Snapin
- Snapin Template: Batch Script
- Snapin Run With: cmd.exe
- Snapin Run With Argument: /c
- Datei hochladen: z. B. Browser-Installation.bat
- Reboot after install: Haken entfernen
- Create new Snapin: Klick auf "Add"

#### Snapin über FOG auf Computer ausführen

- Hosts → List all Hosts → Host anklicken
- Basic Tasks → Advanced → Single Snapin
- Snapin im Dropdown auswählen → Klick auf "Task"

?> Snapins können auch auf mehreren Hosts gleichzeitig ausgeführt werden. Dazu legt man eine Gruppe mit den gewünschten Hosts an und führt das Snapin für die Gruppe aus.