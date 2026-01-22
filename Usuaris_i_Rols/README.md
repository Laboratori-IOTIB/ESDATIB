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

 https://participant.be.<agent>.ibtourismdataspace.org/auth

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
| Creació de keypairs, CSR i pujada de certificats x.509 | `https://participant.fe.dataprovider01.ibtourismdataspace.org/participant-utility/agent-configuration` | ONBOARDER_M, T1UAR_M |
| Veure els atributs d'identitat | `https://participant.fe.dataprovider01.ibtourismdataspace.org/users-roles/identity-attributes-info` | T1UAR_M |
| Assignar atributs d'identitat a rols | `https://participant.fe.dataprovider01.ibtourismdataspace.org/users-roles/roles` | T1UAR_M |
| Creació de descripcions amb SD Tooling | `https://sd-ui.dataprovider01.ibtourismdataspace.org/` | CATALOG_R, SD_PUBLISHER |
| Consulta al catàleg federat de dades | `https://catalogue-ui.dataprovider01.ibtourismdataspace.org/` | CATALOG_R, SD_PUBLISHER |

---

# Serveis de l'Agent Consumidor

Els serveis de l'agent **Consumer** es poden classificar de la següent manera:

- **Consulta de Descripcions**: Serveis relacionats amb la consulta al catàleg federat de dades.  
- **Atributs d'identitat**: Serveis que gestionen els atributs d'identitat dins l'agent.  
- **Rols d'usuari**: Serveis que gestionen els rols dels usuaris dins l'agent.

### Detall de Funcionalitats

| Funcionalitat | URL | Rol requerit |
|---------------|-----|--------------|
| Creació de keypairs, CSR i pujada de certificats x.509 | `https://participant.fe.consumer01.ibtourismdataspace.org/participant-utility/agent-configuration` | ONBOARDER_M, T1UAR_M |
| Veure els atributs d'identitat | `https://participant.fe.consumer01.ibtourismdataspace.org/users-roles/identity-attributes-info` | T1UAR_M |
| Assignar atributs d'identitat a rols | `https://participant.fe.consumer01.ibtourismdataspace.org/users-roles/roles` | T1UAR_M |
| Consulta al catàleg federat de dades | `https://catalogue-ui.consumer01.ibtourismdataspace.org/` | CATALOG_R, SD_CONSUMER |
