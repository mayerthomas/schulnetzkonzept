# VLANs

## Physisches Netzwerk

Im Schulnetzkonzept wird eine physische Netzwerkkarten verwendet. Es empfiehlt sich eine Glasfaser-Karte mit einer Geschwindigkeit von 10 Gbit/s. 

**Mögliche Einstellungen für einen Proxmox-Virtualisierunghost:**
> VM-Host → Network → Create: Linux Bridge
> * Name: z. B. vmbr0
> * IPv4/CIDR: 10.1.254.10/24
> * Gateway (IPv4): 10.1.254.1 (OPNsense-Interface LAN_MANAGEMENT)
> * Haken bei "Autostart"
> * Haken bei "VLAN aware"
> * Bridge ports: <Name der verwendeten Netzwerkkarte>

Die Verwaltungsoberfläche von Proxmox kann über die an die Linux-Bridge vergebebe IP erreicht werden.

## Virtuelles Netzwerk

Aus Sicherheits- und Performancegründen wird das Schulnetz in mehrere Netze aufgeteilt. Die Realisierung im Schulnetzkonzept erfolgt über VLAN-fähige Switche.

**Mögliche VLAN-Einteilung:**

<!-- tabs:start -->

#### **LAN_MANAGEMENT**

- VLAN: 10
- Subnetz: 10.1.254.0/24

| VLAN                  | Netzwerkteilnehmer                                          | IP/IP-Bereich                   |
| --------------------- | ----------------------------------------------------------- | ------------------------------- |
|10 u VA                | OPNsense-Interface LAN_MANAGEMENT                           | 10.1.254.1                      | 
|10 u + 11 + 12 + 13 SP | Management-Interface des Virtualisierungshosts              | 10.1.254.10                     | 
|10 u SP                | NAS zur Datensicherung                                      | 10.1.254.20                     | 
|10 u SP                | WLAN-Controller als Hardware                                | 10.1.254.30                     | 
|10 u VA                | alternativ: WLAN-Controller als virtuelle Maschine          | 10.1.254.30                     | 
|10 u + 12 + 13 SP      | Access-Points zur Nutzung in LAN_SCHULGERAETE und LAN_BYOD  | 10.1.254.50 - 10.1.254.99       | 
|10 u SP                | DHCP-Bereich (OPNsense: Hardware-Clients)                   | 10.1.1.254.200 - 10.1.1.254.254 | 
|10 u VA                | DHCP-Bereich (OPNsense: virtuelle Clients)                  | 10.1.1.254.200 - 10.1.1.254.254 | 

#### **LAN_SERVER**

- VLAN: 11
- Subnetz: 10.1.100.0/24

| VLAN                  | Netzwerkteilnehmer                                          | IP/IP-Bereich                   |
| --------------------- | ----------------------------------------------------------- | ------------------------------- |
| 11 VA                 | OPNsense-Interface LAN_SERVER                               | 10.1.100.1                      | 
| -                     | OPNsense virtuelle IPs für HAProxy                          | 10.1.100.2 - 10.1.100.3         | 
| 11 VA                 | Samba-Server                                                | 10.1.100.7                      | 
| 11 VA                 | Nextcloud-Server                                            | 10.1.100.8                      | 
| 11 VA                 | Collabora-Server für Nextcloud                              | 10.1.100.9                      | 
| 11 u SP               | OPNsense-DHCP-Bereich (OPNsense: Hardware-Clients)          | 10.1.100.200 - 10.1.100.254     | 
| 11 VA                 | OPNsense-DHCP-Bereich (OPNsense: virtuelle Clients)         | 10.1.100.200 - 10.1.100.254     | 

#### **LAN_SCHULGERAETE**

- VLAN: 12
- Subnetz: 10.1.0.0/20

| VLAN                  | Netzwerkteilnehmer                                          | IP/IP-Bereich                   |
| --------------------- | ----------------------------------------------------------- | ------------------------------- |
| 12 VA                 | OPNsense-Interface LAN_SCHULGERAETE                         | 10.1.0.1                        | 
| 12 VA                 | FOG-Project-Server                                          | 10.1.1.1                        | 
| 12 u SP               | OPNsense-DHCP-Bereich (kabelgebundene Clients)              | 10.1.11.1 - 10.1.15.254         | 
| 12 VA                 | OPNsense-DHCP-Bereich (virtuelle Clients)                   | 10.1.11.1 - 10.1.15.254         | 
| 12 s. Access-Points   | OPNsense-DHCP-Bereich (WLAN-Clients)                        | 10.1.11.1 - 10.1.15.254         | 

#### **LAN_BYOD**

- VLAN: 13
- Subnetz: 10.1.16.0/20

| VLAN                  | Netzwerkteilnehmer                                          | IP/IP-Bereich                   |
| --------------------- | ----------------------------------------------------------- | ------------------------------- |
| 13 VA	                | OPNsense-Interface LAN_BYOD                                 | 10.1.16.1                       | 
| s. Access-Points      | DHCP-Bereich (WLAN-Clients)                                 | 10.1.27.1 - 10.1.31.254         | 

#### **WAN_1**

- VLAN: 21
- Subnetz: 192.168.1.0/24

| VLAN                  | Netzwerkteilnehmer                                          | IP/IP-Bereich                   |
| --------------------- | ----------------------------------------------------------- | ------------------------------- |
| 21 VA	                | OPNsense-Interface WAN_1                                    | 192.168.1.1                     | 
| 21 u SP               | Router 1. Internetanschluss (z. B. FritzBox)                | 192.168.1.254                   | 

#### **WAN_2**
?>Die Verwendung einer 2. WAN-Schnittstelle ist optional. Sie ermöglicht bei Ausfall eines Internetanschlusses einen Failover-Betrieb.
- VLAN: 22
- Subnetz: 192.168.2.0/24

| VLAN                  | Netzwerkteilnehmer                                          | IP/IP-Bereich                   |
| --------------------- | ----------------------------------------------------------- | ------------------------------- |
| 22 VA	                | OPNsense-Interface WAN_2                                    | 192.168.2.1                     | 
| 22 u SP               | Router 2. Internetanschluss (z. B. FritzBox)                | 192.168.2.254                   | 

<!-- tabs:end -->

**Erklärung zu den Abkürzungen in der VLAN-Tabelle:**
- u: untagged
- VA: Virtueller Adapter
- SP: Switch-Port

