# Pràctica AdGuard Home — Bloc 4 · Tallafocs

**Mòdul:** 0378 Seguretat i alta disponibilitat  
**Centre:** Institut El Calamot  
**Curs:** 2025-2026  
**Alumne:** Carlos

---

## Índex

1. [Objectiu](#1-objectiu)
2. [Requisits previs](#2-requisits-previs)
3. [Desplegament amb Docker Compose](#3-desplegament-amb-docker-compose)
4. [Configuració inicial (Wizard)](#4-configuració-inicial-wizard)
5. [Activació de llistes de filtratge](#5-activació-de-llistes-de-filtratge)
6. [Regles de filtratge personalitzades](#6-regles-de-filtratge-personalitzades)
7. [Configuració DNS de l'equip](#7-configuració-dns-de-lequip)
8. [Demostració de bloqueig](#8-demostració-de-bloqueig)
9. [Panell de control](#9-panell-de-control)
10. [Explicació de decisions](#10-explicació-de-decisions)
11. [Vídeo de demostració (Configuració DNS)](#11-vídeo-de-demostració-configuració-dns)

---

## 1. Objectiu

Desplegar **AdGuard Home** com a servidor DNS local mitjançant Docker Compose, configurar-lo com a DNS principal de la màquina, activar llistes de filtratge per bloquejar dominis de publicitat i rastreig, i documentar tot el procés amb captures de pantalla.

---

## 2. Requisits previs

- Sistema operatiu: **Debian 13 (Trixie)**
- Docker Engine i Docker Compose v5.1.3 instal·lats
- L'usuari ha de pertànyer al grup `docker` per a la persistència de dades gestionada per Docker

---

## 3. Desplegament amb Docker Compose

### Fitxer `docker-compose.yml`

```yaml
services:
  adguardhome:
    image: adguard/adguardhome
    container_name: adguardhome
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "3000:3000/tcp"
    volumes:
      - adguard_work:/opt/adguardhome/work
      - adguard_config:/opt/adguardhome/conf

volumes:
  adguard_work:
  adguard_config:
```

### Explicació dels paràmetres

| Paràmetre | Descripció |
|---|---|
| `image: adguard/adguardhome` | Imatge oficial d'AdGuard Home |
| `ports` | Exposició dels ports 53 (DNS) i 3000 (Administració) |
| `volumes` | **Volúmens nombrats** gestionats per Docker per garantir la persistència real de la configuració i les dades |

---

## 4. Configuració inicial (Wizard)

Hem completat l'assistent a `http://localhost:3000` configurant:
- Escolta en totes les interfícies (`0.0.0.0`)
- Creació d'usuari administrador (`admin`)
- Contrasenya de gestió (`alumnat.`)

---

## 5. Activació de llistes de filtratge

S'han activat les llistes de bloqueig DNS des del menú *Filtros*:
- **AdGuard DNS filter** (~163K regles)
- **AdAway Default Blocklist** (~6.5K regles)

![Llistes de bloqueig](captures/02_llistes_bloqueig.png)

---

## 6. Regles de filtratge personalitzades

Hem afegit una regla manual per demostrar el bloqueig:
```
||doubleclick.net^
```

![Regla personalitzada](captures/03_regla_personalitzada.png)

---

## 7. Configuració DNS de l'equip

Hem configurat la connexió de xarxa per utilitzar el nostre propi servidor:
```bash
nmcli con mod WIFI_MODS ipv4.dns '127.0.0.1' ipv4.ignore-auto-dns yes
nmcli con up WIFI_MODS
```

Verificació al sistema:
```bash
$ cat /etc/resolv.conf
nameserver 127.0.0.1
```

---

## 8. Demostració de bloqueig

### Prova amb `dig`
```bash
$ dig @127.0.0.1 doubleclick.net
...
doubleclick.net.	10	IN	A	0.0.0.0
```

### Prova al navegador
Al navegar a `doubleclick.net`, el navegador mostra un error de connexió ja que la IP es resol a `0.0.0.0`.

![Bloqueig al navegador](captures/04_bloqueig_navegador.png)

---

## 9. Panell de control

El dashboard mostra el resum de l'activitat DNS i el percentatge de bloqueig aplicat.

![Dashboard](captures/01_dashboard.png)

---

## 10. Explicació de decisions

- **Volúmens nombrats**: S'utilitzen volúmens gestionats per Docker per assegurar que la configuració persisteixi encara que s'elimini el directori de treball.
- **DNS Upstream**: S'ha configurat Quad9 via DoH per a major seguretat.
- **Capa 7**: El filtratge es realitza a nivell de domini, actuant com un tallafocs d'aplicació.

---

## 11. Vídeo de demostració (Configuració DNS)

A causa de l'extensió de la pàgina de configuració, s'ha generat un vídeo que documenta el **scroll complet per la "Configuración DNS"** per mostrar tots els paràmetres de xarxa configurats.

![Vídeo demostratiu](captures/video_configuracio.webp)

---

## Estructura del repositori

```
Entrega_AdGuard/
├── docker-compose.yml
├── README.md
└── captures/
    ├── 01_dashboard.png
    ├── 02_llistes_bloqueig.png
    ├── 03_regla_personalitzada.png
    ├── 04_bloqueig_navegador.png
    ├── 05_configuracio_dns.png
    └── video_configuracio.webp
```
