# Plan d'adressage — Projet ATP

> Mission T1 (infrastructure virtuelle) et T4/T5/T7 (VPN) — MSPR Bloc 5, RNCP35584

## 1. Bridges Proxmox et sous-réseaux par site

| Bridge | Rôle | Site | Sous-réseau |
|---|---|---|---|
| `vmbr0` | WAN — accès Internet de l'hyperviseur | — | Réseau local (DHCP) |
| `vmbr1` | LAN postes | Londres (siège) | 10.1.10.0/24 |
| `vmbr2` | LAN Serveurs | Londres (siège) | 10.1.20.0/24 |
| `vmbr3` | LAN | Ponte-Vedra Beach (USA) | 10.2.10.0/24 |
| `vmbr4` | LAN | Sydney (Australie) | 10.3.10.0/24 |
| `vmbr5` | LAN | Monaco | 10.4.10.0/24 |
| `vmbr6` | LAN PRA | Monaco | 10.1.20.0/24 *(identique à Londres Serveurs — voir §3)* |
| `vmbr7` | LAN Datacenter | Datacenter Cloud Hébergeur | 10.5.10.0/24 |

## 2. Passerelles firewall par site

| Site | Interface | Adresse |
|---|---|---|
| Londres | LAN (postes) | 10.1.10.254/24 |
| Londres | LAN Serveurs | 10.1.20.254/24 |
| Ponte-Vedra Beach | LAN | 10.2.10.254/24 |
| Sydney | LAN | 10.3.10.254/24 |
| Monaco | LAN | 10.4.10.254/24 |
| Monaco | LAN PRA | 10.1.20.254/24 |
| Datacenter Cloud | LAN | 10.5.10.254/24 |

## 3. Pourquoi le LAN PRA de Monaco reprend le sous-réseau de Londres Serveurs

Conformément à la mission **T7** du cahier des charges, aucun paramètre réseau ne doit être modifié sur les serveurs ni les postes de travail en cas de déclenchement du PRA. Le sous-réseau `10.1.20.0/24` est donc **étiré entre Londres et Monaco** via un tunnel VPN de niveau 2 (L2), qui fait office de pont Ethernet entre les deux sites : les deux réseaux ne forment plus qu'un seul domaine de diffusion, malgré la distance géographique. C'est ce qui permet aux serveurs de secours de Monaco de prendre le relai avec exactement la même adresse IP que les serveurs de production de Londres.

## 4. Tableau des VPN

| VPN | Type | Extrémités | Réseau(x) concerné(s) | Objectif |
|---|---|---|---|---|
| VPN 1 | IPsec site-à-site | FW-Londres ↔ FW-PonteVedra | 10.1.0.0/16 ↔ 10.2.10.0/24 | Interconnexion sécurisée siège / filiale USA (T4) |
| VPN 2 | IPsec site-à-site | FW-Londres ↔ FW-Sydney | 10.1.0.0/16 ↔ 10.3.10.0/24 | Interconnexion sécurisée siège / filiale Australie (T4) |
| VPN 3 | IPsec site-à-site | FW-Londres ↔ FW-Monaco | 10.1.0.0/16 ↔ 10.4.10.0/24 | Interconnexion sécurisée siège / filiale Monaco (T4) |
| VPN 4 | L2 (pont Ethernet) | FW-Londres ↔ FW-Monaco | 10.1.20.0/24 (étiré, même sous-réseau des 2 côtés) | Bascule PRA sans reconfiguration réseau — RPO 36h / RTO 48h (T7) |
| VPN 5 | Road warrior (nomade) | Poste nomade ↔ FW-Londres | Plage VPN dédiée (ex. 10.9.0.0/24) → accès bureau virtuel LAN Serveurs | Connexion sécurisée des utilisateurs nomades (staff arbitral, juges-arbitres) (T5) |

*(Aucun VPN n'est requis vers le site Datacenter Cloud : la liaison passe par Internet en simulation, sans chiffrement supplémentaire, conformément au cahier des charges.)*

## 5. Simulation du réseau "Internet" (labo)

Dans la maquette labo, l'ensemble des firewalls communique entre eux sur `vmbr0`, adossé au réseau réel de l'hyperviseur (ex. `192.168.40.0/24`). En production réelle, chaque firewall serait plutôt raccordé à une IP publique fournie par son propre FAI.
