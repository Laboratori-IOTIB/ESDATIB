- [Desplegament amb ArgoCD](#desplegament-aplicació-amb-argocd)
  - [Instal·lació d’ArgoCD](#instal·lació-dargocd)
  - [Instruccions per desplegar l’aplicació](#instruccions-per-desplegar-laplicació)
- [Configuració d’agents](#configuració-dagents)
  - [Configuració paquet COMMON](#configuració-paquet-common)
    - [Arxiu YAML del paquet COMMON](#arxiu-yaml-del-paquet-common)
    - [Passos posteriors al desplegament del COMMON](#passos-posteriors-al-desplegament-del-common)
  - [Configuració Agent PROVIDER](#configuració-agent-provider)
    - [Arxiu YAML del paquet PROVIDER](#arxiu-yaml-del-paquetprovider)
    - [Passos posteriors al desplegament del PROVIDER](#passos-posteriors-al-desplegament-del-provider)
  - [Configuració Agent CONSUMER](#configuració-agent-consumer)
    - [Arxiu YAML del paquet CONSUMER](#arxiu-yaml-del-paquetconsumer)
    - [Passos posteriors al desplegament del CONSUMER](#passos-posteriors-al-desplegament-del-consumer)

---

# Desplegament d’aplicacions amb ArgoCD

## Instal·lació d’ArgoCD

*(En aquesta secció s’haurà d’incloure la guia d’instal·lació d’ArgoCD al clúster de Kubernetes, si escau.)*

## Instruccions per desplegar aplicació

Amb **ArgoCD** instal·lat al **clúster de Kubernetes**, la instal·lació de paquets i aplicacions es realitza mitjançant un arxiu de configuració `.yaml`, disponible a la carpeta `Arxius_Deployment`.

> ⚠️ **Atenció:** Abans de realitzar el desplegament, cal revisar les seccions específiques de cada component.

És **imprescindible** desplegar primer el paquet **COMMON** abans que qualsevol altre component.  
Posteriorment, s’ha de desplegar l’agent corresponent i, un cop finalitzat aquest procés, si és necessari, es pot desplegar l’altre agent.

> ⚠️ El paquet **COMMON** s’ha de desplegar només una vegada per entitat.  
> Els agents **CONSUMER** i **PROVIDER** s’han de desplegar segons les necessitats de cada entitat.

En tots els casos, el desplegament amb **ArgoCD** segueix el mateix procediment:

1. Fer clic a **+ New App**
2. Seleccionar **Edit as YAML**
3. Copiar i enganxar l’arxiu de configuració `.yaml` corresponent
4. Fer clic a **Save** per iniciar el desplegament de l’aplicació

<p align="center">
  <img src="Imatges/ArgoCd_Deploy_YAML.jpeg" alt="Desplegament d’ArgoCD amb YAML" height="400">
</p>

---

# Configuració d’agents

## Configuració paquet COMMON

### Arxiu YAML del paquet COMMON

El paquet **COMMON** inclou tots els recursos bàsics necessaris per al correcte funcionament dels agents dins de l’espai de dades:

- Secrets generals i credencials compartides
- Configuració de xarxa i serveis comuns
- Rutes i ingressos necessaris per als agents

> ⚠️ Aquest paquet només s’ha de desplegar **una vegada per entitat**.

### Passos posteriors al desplegament del COMMON

*(Detallar aquí les verificacions, configuracions addicionals o accions necessàries després del desplegament del COMMON.)*

---

## Configuració Agent PROVIDER

### Arxiu YAML del paquet PROVIDER

L’agent **PROVIDER** permet oferir dades dins de l’ESDATIB. La seva configuració inclou:

- Volums persistents per a les dades compartides
- Serveis i ingressos per accedir als recursos
- Configuració de permisos i seguretat
- Integració amb el paquet **COMMON**

### Passos posteriors al desplegament del PROVIDER

*(Detallar aquí les accions necessàries per validar i activar l’agent PROVIDER.)*

---

## Configuració Agent CONSUMER

### Arxiu YAML del paquet CONSUMER

L’agent **CONSUMER** permet accedir i consumir dades dins de l’ESDATIB. La seva configuració inclou:

- Serveis i rutes per connectar amb els agents **PROVIDER**
- Permisos i credencials d’accés a les dades
- Configuració de *logging* i monitorització
- Integració amb el paquet **COMMON**

### Passos posteriors al desplegament del CONSUMER

*(Detallar aquí les accions necessàries per validar l’accés i el consum de dades.)*

---
