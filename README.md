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

SIMPL OPEN es fonamenta en una estructura d’agents, on cadascun pot desenvolupar funcions específiques dins de l’espai de dades. Actualment, s’identifiquen tres tipus d’agents principals:

- **CONSUMER**: Agent necessari per accedir i consumir dades dins de l’espai. Permet que les entitats o sistemes obtinguin informació de manera controlada.
- **PROVIDER**: Agent que ofereix i comparteix dades dins de l’espai. Garantitza que les dades siguin accessibles, actualitzades i documentades segons els estàndards de l’ESDATIB.
- **AUTHORITY**: Agent encarregat de la governança i supervisió de l’espai de dades. Gestiona permisos, polítiques d’accés i assegura el compliment de normes i regulacions.

En aquest cas, l’**agent de governança (AUTHORITY)** es desplegat per l’administració de l’ESDATIB.  
Qualsevol altra entitat que vulgui adherir-se necessitarà desplegar els agents corresponents a les accions que desitgi realitzar, sent necessari disposar dels agents **CONSUMER** i **PROVIDER** per poder pujar i accedir a les dades dins de l’espai.

A part dels agents principals, que permeten realitzar accions actives dins de l’espai, cada entitat necessita abans desplegar un **paquet comú**, anomenat **COMMON**.  
Aquest paquet és necessari **una sola vegada per entitat**, i no per cada agent, i conté les configuracions i recursos bàsics que permeten que els agents funcionin correctament dins de l’ESDATIB.
![Diagrama de l’ESDATIB](Imatges/Esquema.png)

---

## Recursos e Infrastructura








