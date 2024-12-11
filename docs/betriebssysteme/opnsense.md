# OPNsense

![logo opnsense](../_media/logo_opnsense.png "Provided by opnsense.org")

> **Die Dokumentation zu OPNsense ist noch nicht vollständig umgezogen.**
> - [Zur bisherigen Seite](https://old.schulnetzkonzept.de/opnsense/)


OPNsense ist eine Firewall-Distribution auf der Basis des Betriebssystems FreeBSD. Im Gegensatz zu den anderen vorgestellten Software-Komponenten ist OPNsense also ein eigenständiges Betriebssystem.

Im Schulnetzkonzept kommt OPNsense aber nicht nur die Aufgabe der Firewall zu. Vielmehr ist es die Kommunikationszentrale mit u. a. folgenden Aufgaben:

*   Firewall und NAT
*   DNS-Server
*   URL-Filter mit DNS-Blocklisten
*   DHCP-Server
*   NTP-Server
*   Reverse-Proxy
*   Zertifikatsmanagement
*   VPN-Server

> **Hilfreiche Links**
> - [OPNsense Downloadseite](https://opnsense.org/download/)
> - [OPNsense Forum](https://forum.opnsense.org/)

## Installation

### Virtuelle Maschine erzeugen
1. Offizielles stabiles amd64-DVD-Image von der OPNsense-Downloadseite herunterladen
2. Heruntergeladene bz2-Datei entpacken und ISO-Datei auf auf dem Virtualisierungs-Host ablegen.
3. Auf dem Virtualisierungsserver eine neue virtuelle Maschine erzeugen und sich dabei an nachfolgendem Dimensionierungsbeispiel orientieren:
   *   OS: FreeBSD x64
   *   2 CPU
   *   100 GB Festplattenspeicher
   *   8 GB Arbeitsspeicher
   *   4 Netzwerkkarten mit Zugriff auf das interne LAN-Netzwerk mit den unter [VLANs beschriebenen VLAN-Einstellugen](virtualisierung/vlans?id=virtuelles-netzwerk).
   *   1 Netzwerkkarte mit Zugriff auf das externe WAN-Netzwerk
4. In den Einstellungen zur virtuellen Maschine beim CD-Laufwerk das OPNsense-Image einbinden und das CD-Laufwerk als primäres Bootmedium festlegen.
5. Die virtuelle Maschine einschalten und das Konsolenfenster öffnen. Nun sollte der Installationsassistent von OPNsense zu sehen sein.

### Grundinstallation von OPNsense
Folgende Schritte sind bei der Installation zu durchlaufen:
*   Anmeldung mit Benutzernamen "installer" und Passwort "opnsense"
*   Start der Installation mit "OK, let's go"
*   Keymap mit "Change keymap" umstellen auf "de.kbd"
*   Akzeptieren der Umgebungseinstellungen mit "Accept these settings"
*   Geführte Installation einleiten mit "Guided installation"
*   Auswahl der Festplatte, auf der OPNsense installiert werden soll
*   Installationsmodus festlegen auf "GPT/UEFI mode"
*   Vorgeschlagene Swap-Partitionsgröße mit "Yes" bestätigen
*   root-Passwort festlegen
*   Neustart des Systems
*   Nach dem Neustart müssen in der Regel noch die virtuellen Netzwerkkarten richtig zugewiesen werden. Die Netzwerkkarten heißen im System je nach verwendeter (virtueller) Hardware z. B. em0, em1, ... oder igb0, igb1, ... oder hn0, hn1, ...

### Erster Start von OPNsense nach Installation
*   Melden Sie sich zunächst an der OPNsense-Konsole als root an und wählen Sie die Option "1) Assign Interfaces".
*   VLANs werden hier nicht benötigt, da diese bereits über den Virtualisierungshost den virtuellen Netzwerkkarten zugewiesen wurden. *   Suchen Sie im Virtualisierungshost nach den MAC-Adressen der 4 virtuellen OPNsense-Netzwerkkarten und ordnen Sie anschließend die richtigen Netzwerkkarten-Namen zu. 
*   Die weitere Konfiguration erfolgt überwiegend über die Web-Schnittstelle von OPNsense. Diese erreichen Sie nun innerhalb des Schulnetzes (intern) mit der auf der OPNsense-Konsole angezeigten LAN-IP-Adresse (i. d. R.: https://192.168.1.1). Die Zugangsdaten hierfür sind:
    *   Benutzername: root
    *   Passwort: <haben Sie während des Installationsprozesses selbst vergeben>

### Erste Konfiguration über die Web-Schnittstelle
Rufen Sie die Web-Schnittstelle auf und melden Sie sich an.

Beim ersten Start der Web-Oberfläche führt Sie ein Setup-Wizard durch wichtige Bestandteile der Erstkonfiguration. Hierbei sind folgende Angaben empfehlenswert:

> * General Information
>   * Hostnamen: z. B. opnsense
>   * Domain: z. B. schulnetz.intra
>   * Haken bei "Allow DNS servers to be overriden by DHCP/PPP on WAN", sofern sie den DNS-Server Ihres Internet-Providers verwenden wollen.
> * Unbound DNS
>   * Haken bei "Enable Resolver"
> * Time Server Information
>   * Timezone: Europe/Berlin
> * Configure WAN Interface
>   * IPv4 Configuration Type: DHCP, sofern sie von Ihrem Internet-Provider eine IP zugewiesen bekommen
>   * Haken bei "Block RFC1918 Private Networks"
>   * Haken bei "Block bogon networks"
> * Configure LAN Interface
>   * LAN IP Address: z. B. 10.1.0.1
>   * Subnet Mask: z. B. 20

Nach Klick auf "Reload" werden die Einstellungen neu geladen. Sofern sie die LAN-IP-Adresse geändert haben, müssen Sie im Browser die neue IP eingeben, um die Web-Oberfläche wieder erreichen zu können.

Sie können diesen Setup-Wizard zu einem späteren Zeitpunkt erneut unter System/Setup Wizard aufrufen.

!> **Hinweis:** Sofern Sie für die WAN-Einstellungen nicht DHCP verwenden, sollten Sie unter Interfaces → WAN das IPv4-Upstream-Gateway nicht auf dem Wert "Auto" belassen, sondern explizit ein Gateway angeben. Mit der Einstellung "Auto" kann es an diversen Stellen zu unvorhersehbarem Verhalten kommen.

### Installation des VM-Gast-Plugins
Zur Besseren Verwaltung der OPNsense-VM vom Virtualisierungshost aus, empfiehlt sich, das passende VM-Gast-Plugin zu installieren:

> System/Firmware/Plugins/
> * Für Qemu-Virtualisierungshost (z. B. Proxmox): os-qemu-gust-agent → Install
> * Für VMWare-Virtualisierungshost: os-vmware → Install
> * Für VirtualBox-Virtualisierungshost: os-virtualbox → Install

### SSH-Zugriff aktivieren
Sofern Sie mit einem SSH-Client auf die Konsole (z. B. via Putty) oder das Dateisystem (z. B. via WinSCP) von OPNSense zugreifen möchten, müssen Sie den SSH-Server aktivieren:

> System / Settings / Administration / Secure Shell
> * Secure Shell Server: Haken bei: Enable Secure Shell
> * Sofern sie kein Zertifikat zur Authentifizierung verwenden:
>   * Root Login: Haken bei: Permit root user login
>   * Authentication Method: Haken bei: Permit password login

## Konfiguration der Netzwerkschnittstellen und grundlegende Firewall-Regeln
Nutzen Sie zur Konfiguration der Netzwerkschnittstellen die unter [VLANs beschriebene VLAN-Einteilung und IP-Konfiguration](virtualisierung/vlans?id=virtuelles-netzwerk).

### Interfaces zuweisen 
Die Zuweisung der Interfaces zu den virtuellen Netzwerkschnittstellen erfolgt über Interfaces / Assignments.

### Interfaces konfigurieren
Die Konfiguration der Interfaces erfolgt über Interfaces / <[Interfacename]>.

> Folgende Einstellungen sind je **LAN-Interface** zu tätigen:
> * Basic Configuration
>   * Haken bei "Enable interface"
>   * Description: z. B.: LAN_MANAGEMENT
> * Generic Configuration
>   * IPv4 Configuration Type: Static IPv4
>   * IPv6 Configuration Type: None
> * Static IPv4 configuration
>   * IPv4 address: z. B. 10.1.254.1/24
>   * IPv4 gateway rules: Disabled

> Folgende Einstellungen sind je **WAN-Interface** zu tätigen:
> * Basic Configuration
>   * Haken bei "Enable interface"
>   * Description: z. B.: WAN_1
> * Generic Configuration
>   * Haken bei "Block private networks"
>   * Haken bei "Block bogon networks"
>   * IPv4 Configuration Type: Static IPv4
>   * IPv6 Configuration Type: None
> * Static IPv4 configuration
>   * IPv4 address: z. B. 10.1.254.1/24
>   * IPv4 gateway rules: Disabled