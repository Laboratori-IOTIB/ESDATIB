
- [Desplegament amb ArgoCD](#desplegament-aplicació-amb-argocd)
- [Configuració Agents](#configuració-agents)
  - [Configuració paquet COMMON](#configuració-paquet-common)
  - [Configuració Agent PROVIDER](#configuració-agent-provider)
  - [Configuració Agent CONSUMER](#configuració-agent-consumer)

---

# Desplegament aplicació amb ArgoCD

Aquest document descriu com desplegar els components de l’ESDATIB dins d’un **cluster de Kubernetes** utilitzant **ArgoCD**.  
El desplegament automatitzat permet instal·lar els recursos bàsics i els agents de manera controlada, assegurant la correcta integració amb l’espai de dades.

> ⚠️ El paquet **COMMON** s’ha de desplegar només una vegada per entitat.  
> Els agents **CONSUMER** i **PROVIDER** s’han de desplegar segons les necessitats de cada entitat.

---

# Configuració Agents

### Configuració paquet COMMON

El paquet **COMMON** inclou tots els recursos bàsics necessaris per al correcte funcionament dels agents dins de l’espai de dades:  

- Secrets generals i credencials compartides  
- Configuració de xarxa i serveis comuns  
- Rutes i ingressos necessaris per als agents  

> ⚠️ Aquest paquet només cal desplegar-lo **una vegada per entitat**.

---

### Configuració Agent PROVIDER

L’agent **PROVIDER** permet oferir dades dins de l’ESDATIB. La configuració inclou:  

- Volums persistents per a les dades que es comparteixen  
- Serveis i ingressos per accedir als recursos  
- Configuració de permisos i seguretat  
- Integració amb el paquet COMMON  

---

### Configuració Agent CONSUMER

L’agent **CONSUMER** permet accedir i consumir dades dins de l’ESDATIB. La configuració inclou:  

- Serveis i rutes per connectar amb els PROVIDERs  
- Permisos i credencials per accedir a les dades  
- Configuració de logging i monitorització  
- Integració amb el paquet COMMON  

---
