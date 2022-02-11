# VMs

## Proxmox als Virtualisierungshost

![logo proxmox](../_media/logo_proxmox.png "Provided by proxmox.com")

Für die Umsetzung des Schulnetzkonzepts werden virtuelle Maschinen auf der Basis von Proxmox eingesetzt. Für deren Betrieb benötigen Sie einen entsprechend ausgestatteten und für die Virtualisierung geeigneten Server.

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
> - [SSH-Consolen-Client Putty](https://www.putty.org/)
> - [SCP-Dateitransfer-Client WinSCP](https://winscp.net)

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
> - Betriebssystem: [Ubuntu](betriebssysteme/ubuntu)
> - Dimensionierung der VM
>   - mind. 4 CPUs
>   - 40 GB Festplattenspeicher
>   - mind. 8 GB RAM
>   - 1 Netzwerkkarte
>   - Internetanbindung mit 1 Gbit/s duplex
> - Installierte Software
>   - [Jitis Meet](software/jitis-meet)
> - **Wichtiger Hinweis:** Um die für Jitsi notwendige Bandbreite (ca. 2 MBit/s je Teilnehmer) sicherstellen zu können, empfiehlt sich die datenschutzkonforme Installation der erforderlichen Komponenten auf einer virtuellen Maschine bei einem gut angebundenen deutschen Hoster und nicht im eigenen Schulnetz.