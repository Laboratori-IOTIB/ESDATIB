# Rols definits i Usuaris associats

- **SD_CONSUMER**: usuari consumidor  
- **SD_PUBLISHER**: usuari publicant  
- **CATALOG_R**: gestor del catàleg  
- **ONBOARDER_M**: encarregat de desplegar el Tier 2 de l’agent  
- **T1UAR_M**: gestor de rols i usuaris en el Tier 1

### Usuaris predefinits i rols associats

| Usuari | Rol |
|--------|-----|
| a.w | ONBOARDER_M, T1UAR_M |
| t.w | T1UAR_M |
| m.b | CATALOG_R, SD_CONSUMER |
| j.r | CATALOG_R, SD_PUBLISHER |
| s.p | SERVICE_PROVIDER |

---

## Credencials dels Usuaris

Per gestionar les credencials dels usuaris que utilitza cada agent, s’utilitza el servei de Tier-1 anomenat **Keycloak**. Aquest es trobarà a la URL:

`https://participant.be.<agent>.<hostname>/auth`

Keycloak demana un **usuari** i una **contrasenya d'entrada**, que es poden obtenir al servei **OpenBao**, el Vault de secrets dels nostres agents.

---

# Serveis de l'Agent Proveïdor

Els serveis de l'agent **Provider** es poden classificar de la manera següent:

- **Creació de Descripcions**: Serveis relacionats amb la creació i publicació de descripcions de conjunts de dades al **catàleg federat de dades**.  
- **Consulta de Descripcions**: Serveis relacionats amb la consulta al catàleg federat de dades.  
- **Atributs d'identitat**: Serveis que gestionen els atributs d'identitat dins l'agent.  
- **Rols d'usuari**: Serveis que gestionen els rols dels usuaris dins l'agent.

### Detall de Funcionalitats

| Funcionalitat | URL | Rol requerit |
|---------------|-----|--------------|
| Creació de keypairs, CSR i pujada de certificats x.509 | `https://participant.fe.<agent>.<hostname>/participant-utility/agent-configuration` | ONBOARDER_M, T1UAR_M |
| Veure els atributs d'identitat | `https://participant.fe.<agent>.<hostname>/users-roles/identity-attributes-info` | T1UAR_M |
| Assignar atributs d'identitat a rols | `https://participant.fe.<agent>.<hostname>/users-roles/roles` | T1UAR_M |
| Creació de descripcions amb SD Tooling | `https://sd-ui.<agent>.<hostname>/` | CATALOG_R, SD_PUBLISHER |
| Consulta al catàleg federat de dades | `https://catalogue-ui.<agent>.<hostname>/` | CATALOG_R, SD_PUBLISHER |

---

# Serveis de l'Agent Consumidor

Els serveis de l'agent **Consumer** es poden classificar de la següent manera:

- **Consulta de Descripcions**: Serveis relacionats amb la consulta al catàleg federat de dades.  
- **Atributs d'identitat**: Serveis que gestionen els atributs d'identitat dins l'agent.  
- **Rols d'usuari**: Serveis que gestionen els rols dels usuaris dins l'agent.

### Detall de Funcionalitats

| Funcionalitat | URL | Rol requerit |
|---------------|-----|--------------|
| Creació de keypairs, CSR i pujada de certificats x.509 | `https://participant.fe.<agent>.<hostname>/participant-utility/agent-configuration` | ONBOARDER_M, T1UAR_M |
| Veure els atributs d'identitat | `https://participant.fe.<agent>.<hostname>/users-roles/identity-attributes-info` | T1UAR_M |
| Assignar atributs d'identitat a rols | `https://participant.fe.<agent>.<hostname>/users-roles/roles` | T1UAR_M |
| Consulta al catàleg federat de dades | `https://catalogue-ui.<agent>.<hostname>/` | CATALOG_R, SD_CONSUMER |

---

# Com canviar les contrasenyes dels usuaris de l'agent

## 1. Accedir a Keycloak

### 1.1 Trobar el pod de Keycloak via ArgoCD
- Entrar a **ArgoCD**
- Navegar dins l'agent
- Localitzar el pod de **Keycloak**

### 1.2 Obtenir la URL de Keycloak
- Fer clic al pod i anar a la pestanya **Summary**
- Al YAML, cercar la variable `KC_HOSTNAME_ADMIN` i copiar-ne el valor (primera URL)

### 1.3 Obtenir les credencials d'accés des d'OpenBao
- Entrar a **OpenBao**
- `Common01` → `<namespace-de-l'agent>-keycloak`
- L'usuari és la **key** i la contrasenya el **value**

## 2. Canviar la contrasenya

### 2.1 Iniciar sessió a Keycloak
- Obrir la URL obtinguda al pas 1.2
- Introduir usuari i contrasenya del pas 1.3

### 2.2 Seleccionar el realm "participant"
- Al menú desplegable de l'esquerra (on posa "Keycloak"), seleccionar **participant**

![Canvi de realm a participant](imatges/participant.png)

### 2.3 Anar a Users i triar l'usuari
- Al menú lateral, anar a **Users**
- Fer clic sobre l'usuari al qual es vol canviar la contrasenya

### 2.4 Restablir la contrasenya
- Anar a la pestanya **Credentials**
- Fer clic a **Reset password**
- Introduir la nova contrasenya i confirmar-la
- **Nota**: Veureu un camp "Temporary", aquest si es desactiva no es podrà tornar a canviar la contrasenya
- Fer clic a **Reset password**

![Reset de contrasenya d'usuari](imatges/user-credentials.png)
