- [ESDATIB – Espai de Dades Turístiques de les Illes Balears](#esdatib--espai-de-dades-turistiques-de-les-illes-balears)
- [Objectius principals](#objectius-principals)
- [Arquitectura i marc tecnològic](#arquitectura-i-marc-tecnològic)
  - [Agents](#agents)
- [Recursos e infraestructura](#recursos-e-infraestructura)
  - [Altres requisits necessaris per al desplegament dels mòduls en el clúster](#altres-requisits-necessaris-per-al-desplegament-dels-mòduls-en-el-clúster)
  - [Requisits del clúster](#requisits-del-clúster)
    - [Requisits Experimental](#requisits-experimental)
    - [Requisits Producció](#requisits-producció)
- [Procés d’adhesió a l’ESDATIB](#procés-dadhesió-a-lesdatib)

---
<p align="center">
  <img src="Imatges/LG_CH.png" alt="Diagrama de l’ESDATIB" height="70">
  &nbsp;&nbsp;&nbsp;
  <img src="Imatges/logo_fueib_uib.png" alt="Logos FEUIB i UIB" height="70">
</p>
<p align="center">
  <img src="Imatges/espai cotib.png" alt="Logo Espai COTIB" height="70">
  &nbsp;&nbsp;&nbsp;
  <img src="Imatges/logo-CTIG.png" alt="Logo CTIG" height="80">
</p>




# ESDATIB – Espai de Dades Turístiques de les Illes Balears

Web oficial: https://ibtourismdataspace.org/

L’**ESDATIB (Espai de Dades Turístiques de les Illes Balears)** és una iniciativa orientada a centralitzar, visualitzar i facilitar l’accés a dades turístiques rellevants de les Illes Balears. Està pensat per donar servei a la comunitat, al teixit empresarial i a les administracions públiques, promovent un ús eficient, segur i interoperable de les dades.

Aquest repositori inclou els components i la documentació necessaris perquè les entitats puguin integrar-se i adherir-se a l’ESDATIB.

---

## Objectius principals

- Centralitzar dades turístiques provinents de diferents fonts.
- Facilitar l’accés, la reutilització i la compartició de dades.
- Millorar la presa de decisions basada en dades.
- Impulsar la interoperabilitat i els espais de dades sectorials.
- Fomentar la innovació i la col·laboració públic-privada.

---

## Arquitectura i marc tecnològic

L’arquitectura base de l’ESDATIB es desenvolupa sobre **SIMPL OPEN**, una iniciativa europea que forma part del programa **SIMPL**.

🔗 Més informació sobre SIMPL:  
https://simpl-programme.ec.europa.eu/

SIMPL OPEN proporciona una infraestructura que assegura:
- Interoperabilitat entre sistemes i actors.
- Governança de dades.
- Sobirania i control dels participants.
- Compliment dels principis europeus d’espais de dades.

### Agents

SIMPL OPEN es fonamenta en una estructura d’agents, on cadascun pot desenvolupar funcions específiques dins de l’espai de dades. Actualment, s’identifiquen tres tipus d’agents principals:

- **CONSUMER**: Agent necessari per accedir i consumir dades dins de l’espai. Permet que les entitats o sistemes obtinguin informació de manera controlada.
- **PROVIDER**: Agent que ofereix i comparteix dades dins de l’espai. Garantitza que les dades siguin accessibles, actualitzades i documentades segons els estàndards de l’ESDATIB.
- **AUTHORITY**: Agent encarregat de la governança i supervisió de l’espai de dades. Gestiona permisos, polítiques d’accés i assegura el compliment de normes i regulacions.

En aquest cas, l’**agent de governança (AUTHORITY)** es desplegat per l’administració de l’ESDATIB.  
Qualsevol altra entitat que vulgui adherir-se necessitarà desplegar els agents corresponents a les accions que desitgi realitzar, sent necessari disposar dels agents **CONSUMER** i **PROVIDER** per poder accedir i pujar a les dades dins de l’espai.

A part dels agents principals, que permeten realitzar accions actives dins de l’espai, cada entitat necessita abans desplegar un **paquet comú**, anomenat **COMMON**.  
Aquest paquet és necessari **una sola vegada per entitat**, i no per cada agent, i conté les configuracions i recursos bàsics que permeten que els agents funcionin correctament dins de l’ESDATIB.

![Diagrama de l’ESDATIB](Imatges/Esquema.png)

---

## Recursos e infraestructura

L’arquitectura **SIMPL OPEN** ha estat desenvolupada amb la intenció de ser desplegada en un **cluster de Kubernetes**.  
Per aquest motiu, qualsevol entitat que vulgui adherir-se a l’ESDATIB i accedir a l’espai de dades haurà de disposar d’un **cluster de Kubernetes** operatiu on fer el desplegament.  

El **component COMMON** i els agents necessaris per a l’entitat (**CONSUMER**, **PROVIDER** o ambdós) hauran de ser desplegats en aquest cluster, amb els arxius de configuració ja preparats per utilitzar l’eina **ArgoCD**.  

Tot i això, el primer pas de sol·licitud d’adhesió a l’espai de dades **no requereix tenir habilitat el cluster ni haver desplegat cap agent**.  

Es recomana utilitzar un **servei cloud**, ja que facilita el desplegament i el manteniment. L’equip de l’ESDATIB ha utilitzat el servei de **Azure**, que és on podrà oferir més suport.

### Altres requisits necessaris per al desplegament dels mòduls en el clúster

- **DNS / Hostname**  
  Cada entitat necessita un **domini** que s’utilitzarà com a base per crear les adreces dels diferents serveis dels agents dins de l’espai de dades.

### Requisits del clúster

#### Requisits Experimental

Actualment, l’espai es troba en una fase primigènia i els requisits es mantenen segons l’estàndard experimental fixat pels desenvolupadors de SIMPL.

| Components desplegats       | Worker Nodes | Persistent Volumes (RWO) | CPU per node | RAM per node |
|-----------------------------|-------------|--------------------------|-------------|--------------|
| Common + Consumer           | 3           | 11 GB                    | 4           | 16 GB        |
| Common + Provider           | 3           | 11 GB                    | 4           | 16 GB        |
| Common + Provider + Consumer| 4           | 11 GB                    | 4           | 16 GB        |

#### Requisits Producció

*(Incloure aquí els requisits per a entorns de producció, per exemple: dimensions de cluster, seguretat, backups, alta disponibilitat, monitorització, etc.)*

---

## Procés d’adhesió a l’ESDATIB

El procés d’adhesió o **onboarding** compta amb **dues parts diferenciades**:

### 1. Part sense necessitat de desplegament ni configuració
- Sol·licitud d’integració a l’ESDATIB:  
  L’entitat genera una sol·licitud que serà gestionada per l’administració de governança de l’ESDATIB.  

### 2. Part amb desplegament dels agents
- En aquesta fase ja és necessari **desplegar els agents al cluster** per poder accedir als seus serveis i obtenir les credencials corresponents.

Consulta la **[Guia de procés d’adhesió](OnBoarding/README.md)**.


