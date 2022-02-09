# FOG-Project

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

### Debian als Basissystem

Für FOG empfiehlt sich Debian als Basissystem. Verwenden Sie zur Installation die [auf diesem Projekt hinterlegte Anleitung](/betriebssysteme/debian).

Dimensionierungsbeispiel der virtuellen Maschine für die Vorhaltung von 2 - 3 Images:

- 1 CPU
- 250 GB Festplattenspeicher
- 4 GB Arbeitsspeicher
- 1 Netzwerkkarte mit Zugriff auf das Netzwerk LAN_CLIENTS

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

Wenn die Installation an der Zeile "Press \[Enter\] key when database is updated/installed." angelangt ist, rufen Sie über den Browser die Weboberfläche von FOG auf: http://10.1.1.1/fog und klicken dort auf den Button "INSTALL/UPGRADE NOW".

Anschließend kehren Sie zur Konsole zurück und Drücken die Enter-Taste, um die Installation abzuschließen.

Nun können Sie sich mit Benutzernamen "fog" und Passwort "password" auf der Weboberfläche von FOG anmelden. Bitte ändern Sie umgehend das Passwort von FOG unter "User Management" → "List all users".

### Einstellungen im DHCP-Server (bei uns OPNsense)

Damit Clients beim Booten über das Netzwerk beim FOG-Server landen, müssen beim DHCP-Server folgende Einstellungen getätigt werden

- next-server: 10.1.1.1 (IP Ihres FOG-Servers)
- filename: undionly.kpxe
- UEFI 32 bit file name : ipxe32.efi
- UEFI 64 bit file name: ipxe.efi

## Wartung

### Update von FOG

Vor Updates empfiehlt sich das Anfertigen einer Sicherung, z. B. durch das Erstellen eines Snapshots auf dem Virtualisierungs-Host.

Als FOG-Administrator können Sie unter "FOG Configuration" in Erfahrung bringen, ob Updates bereitstehen. Wenn ein Update bereitsteht, gehen Sie wie folgt vor:

```bash
cd /root/fogproject
git pull
cd bin
./installfog.sh

# Beantworten Sie Fragen während des Update-Prozesses analog zu Ihren Antworten beim Installationsprozess (s. oben).
```

- Testen Sie Ihren aktualisierten FOG-Server.
- Nach erfolgreichem Test können Sie ggf. den zuvor erstellten Snapshot auf dem Virtualisierungs-Host entfernen.