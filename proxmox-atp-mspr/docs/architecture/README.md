# Architecture (T1)

Ce dossier contient les livrables écrits de la mission **T1 — Infrastructure virtuelle Proxmox**.

## Contenu attendu

- `schema-architecture.png` : schéma réseau complet (sites, VLANs, firewalls, VPN)
- `plan-adressage.md` : tableau des plages IP et VLANs par site
- `procedure-installation.md` : procédure pas-à-pas de montage de la maquette

## Rappel du besoin (cahier des charges)

Mettre en place un hyperviseur unique hébergeant l'ensemble des VM du projet, configuré pour
simuler les réseaux UK / US / Monaco / Australie + un site "datacenter" pour l'hébergeur cloud
(mission T3). Découpage de chaque site en un ou plusieurs VLANs.
