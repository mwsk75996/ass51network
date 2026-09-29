# Assignment 51 – OSPF med flere subnets på Juniper SRX

Dette repository indeholder de komplette Junos-konfigurationer til de tre SRX-routere i assignment 51:

- `r2-config.txt` – konfiguration til R2
- `r4-config.txt` – konfiguration til R4
- `r5-config.txt` – konfiguration til R5

Topologien er vist i [`ass51.svg`](ass51.svg) (HLD tilpasset vores setup; Pers oprindelige diagram ligger i `topology.jpg`).

## Hvad går konfigurationen ud på?

R2, R4 og R5 er forbundet i en trekant (Lan5, Lan6, Lan10) med OSPF i area `0.0.0.0`, som i assignment 49/50. Nyt i 51 er, at hver router også har et USERLAN og en loopback:

- Loopback (`lo0`) er med i OSPF som interface i area 0.
- USERLAN'et ligger på `ge-0/0/4` og **ikke** som OSPF-interface. I stedet eksporteres det med en `export`-policy og `route-filter <prefix> exact`, så kun netop det subnet annonceres.
- `ge-0/0/4` er i security-zonen `lab`, så intra-zone-policyen tillader trafik mellem PC'erne.

## Adresser (LLD)

| Router | Interface | Netværk | Adresse |
|---|---|---|---|
| R2 | ge-0/0/1 | Lan10 10.10.12.0/28 | 10.10.12.2/28 |
| R2 | ge-0/0/2 | Lan5 10.10.10.0/28 | 10.10.10.1/28 |
| R2 | ge-0/0/4 | USERLAN1 192.168.13.0/24 | 192.168.13.1/24 (PC5 = .5) |
| R2 | lo0 | – | 192.168.100.1/32 |
| R4 | ge-0/0/2 | Lan5 10.10.10.0/28 | 10.10.10.2/28 |
| R4 | ge-0/0/3 | Lan6 10.10.11.0/28 | 10.10.11.2/28 |
| R4 | ge-0/0/4 | USERLAN2 192.168.14.0/24 | 192.168.14.1/24 (PC10 = .5) |
| R4 | lo0 | – | 192.168.100.2/32 |
| R5 | ge-0/0/2 | Lan10 10.10.12.0/28 | 10.10.12.1/28 |
| R5 | ge-0/0/3 | Lan6 10.10.11.0/28 | 10.10.11.1/28 |
| R5 | ge-0/0/4 | USERLAN3 192.168.15.0/24 | 192.168.15.1/24 (PC11 = .5) |
| R5 | lo0 | – | 192.168.100.3/32 |

PC'erne bruger routerens `.1` som default gateway.

## Indlæsning

Fra Junos configuration mode:

```text
load override terminal
```

Indsæt hele filen, afslut med `Ctrl+D`, og kontrollér med `show | compare` og `commit check`. Brug gerne `commit confirmed 5`.

> **Bemærk:** `load override` erstatter hele routerens konfiguration. Filerne indeholder de krypterede root-passwordhashes fra labrouterne og bør kun bruges på de tilsigtede enheder.

## Verifikation

```text
show ospf neighbor
show route terse
show route protocol ospf
```

USERLAN'erne fra de andre routere står som OSPF external-ruter (`OSPF` med preference 150), fordi de er importeret via `export`. Lan- og loopback-ruter er interne (preference 10).

## Status

Konfigurationerne er skrevet men **endnu ikke indlæst på routerne eller testet**. HLD, `show route terse`-output fra R4 og ping-beviser mangler.
