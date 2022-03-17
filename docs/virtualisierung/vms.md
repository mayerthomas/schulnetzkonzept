# VMs

## Proxmox als Virtualisierungshost

![logo proxmox](../_media/logo_proxmox.png "Provided by proxmox.com")

Für die Umsetzung des Schulnetzkonzepts werden virtuelle Maschinen mit Proxmox als Virtualisierungshost eingesetzt. Hierzu benötigt man einen entsprechend ausgestatteten und für die Virtualisierung geeigneten Server.

> An meiner Schule mit ca. 700 Schülerinnen und Schülern und 65 Lehrkräften ist ein Server mit folgenden Eckdaten im Einsatz:
> - Prozessor: 2 x Intel Xeon E5 (2,2 GHz, 15 MB, 6 Kerne)
> - Speicher: 4 x 16 GB DDR3
> - Festplatten: 5 x 600 GB, 10k SAS
> - Controller: SAS Raid Controller 6G, 1 GB Speicher
> - Netzwerkkarten: 2 x Gigabit

Proxmox kann in einer kostenfreien No-Subscription-Version genutzt werden und bietet in einer benutzerfreundlichen Webschnittstelle neben den üblichen Virtualisierungsmöglichkeiten auch das erstellen von Backups und Replikationen.

Selbstverständlich lässt sich das Schulnetzkonzept auch mit anderen Virtualisierungslösungen umsetzten.

> **Hilfreiche Links**
> - [Proxmox Downloadseite](https://www.proxmox.com/de/downloads)
> - [Installationsanleitung Proxmox](https://pve.proxmox.com/wiki/Installation)
> - [Proxmox-Wiki](https://pve.proxmox.com/wiki/Main_Page)

## Virtuelle Maschinen im Schulnetzkonzept

Folgende virtuelle Maschinen werden an unserer Schule mit ca. 700 Schülerinnen und Schülern und 65 Lehrkräften im Rahmen des Schulnetzkonzepts eingesetzt:

> ### VM für die Netzwerkverwaltung (Router, Firewall, DNS, DHCP, ...)
> - Betriebssystem: [OPNsense](betriebssysteme/opnsense)
> - Dimensionierung der VM
>   - 2 CPUs
>   - 100 GB Festplattenspeicher
>   - 8 GB RAM
>   - 4 Netzwerkkarten

> ### VM für die Benutzerverwaltung
> - Betriebssystem: [Debian](betriebssysteme/debian)
> - Dimensionierung der VM
>   - 2 CPUs
>   - 40 GB Festplattenspeicher
>   - 4 GB RAM
>   - 1 Netzwerkkarte
> - Installierte Software
>   - [Samba](software/samba)
>   - [LDAP-Account-Manager](software/ldap-account-manager)
>   - [Self-Service-Password](software/self-service-password)

> ### VM für das Image- und Softwaredeployment
> - Betriebssystem: [Debian](betriebssysteme/debian)
> - Dimensionierung der VM
>   - 2 CPUs
>   - 250 GB Festplattenspeicher
>   - 8 GB RAM
>   - 1 Netzwerkkarte
> - Installierte Software
>   - [FOG Project](software/fog-project)

> ### VM für die Nextcloud
> - Betriebssystem: [Debian](betriebssysteme/debian)
> - Dimensionierung der VM
>   - 2 CPUs
>   - 300 GB Festplattenspeicher
>   - 8 GB RAM
>   - 1 Netzwerkkarte
> - Installierte Software
>   - [Nextcloud](software/nextcloud)

> ### VM für Collabora für die Nextcloud
> - Betriebssystem: [Debian](betriebssysteme/debian)
> - Dimensionierung der VM
>   - 2 CPUs
>   - 40 GB Festplattenspeicher
>   - 8 GB RAM
>   - 1 Netzwerkkarte
> - Installierte Software
>   - [Collabora](software/collabora)

> ### VM für Jitsi Meet
> - Betriebssystem: [Debian](betriebssysteme/debian)
> - Dimensionierung der VM
>   - mind. 4 CPUs
>   - 40 GB Festplattenspeicher
>   - mind. 8 GB RAM
>   - 1 Netzwerkkarte
>   - Internetanbindung mit 1 Gbit/s duplex
> - Installierte Software
>   - [Jitis Meet](software/jitis-meet)
> - **Wichtiger Hinweis:** Je Videokonferenzteilnehmer ist eine Bandbreite von  ca. 2 MBit/s erforderlich. Sofern Sie die für mehrere gleichzeitige Teilnehmer erforderlichen Übertragungsraten am Schulstandort nicht gewährleisten können, empfiehlt sich die datenschutzkonforme Installation von Jitsi Meet bei einem entsprechend angebundenen externen Hoster.