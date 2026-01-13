## Procés complet d’adhesió a l’espai de dades

El procés d’adhesió és necessari **per a cada agent** (CONSUMER o PROVIDER).  
Com que el procés està dividit en parts diferenciades, és possible realitzar la **primera part abans del desplegament o configuració del clúster**.

<p align="center">
  <img src="Imatges/Esquema_OnBoarding.png" alt="Diagrama OnBoarding ESDATIB" height="500">
</p>

El procés d’adhesió a l’ESDATIB es divideix en **dues parts principals**:

> ⚠️ **Atenció:** Aquest procés complet, amb ambdues parts, és necessari **per a cada agent**.  
> Si una entitat vol actuar com a **CONSUMER** i **PROVIDER**, haurà de realitzar el procés **dues vegades**, una per a cada agent.

---

### 1. Part sense necessitat de desplegament ni configuració

L’entitat ha de generar una sol·licitud d’adhesió que serà gestionada per l’administració de **governança de l’ESDATIB**.

**Emplenament del formulari**  
L’entitat ha de completar el formulari de sol·licitud disponible a:  
[https://authority.fe.authority01.ibtourismdataspace.org/onboarding/application/request](https://authority.fe.authority01.ibtourismdataspace.org/onboarding/application/request)

- Al camp **Participant_Type** apareixen quatre opcions disponibles.
- Actualment, només estan operatives les opcions **Consumer** i **Data Provider**.
- L’entitat ha de seleccionar **una d’aquestes opcions** segons el seu rol.

> ℹ️ Una entitat que vulgui **consumir i proveir dades** haurà de realitzar el procés **dues vegades**, una per a cada tipus de participant.  
> Aquest procés generarà **dos usuaris diferenciats**: un amb rol `_consumer` i un altre amb rol `_provider`.

<p align="center">
  <img src="Imatges/FormulariOnBoarding.jpeg" alt="Formulari OnBoarding ESDATIB" height="500">
</p>

Un cop completat el formulari i generades les credencials, l’usuari serà redirigit a un portal on podrà iniciar sessió amb les credencials acabades de crear:  
[https://authority.fe.authority01.ibtourismdataspace.org/onboarding/application/additional-request](https://authority.fe.authority01.ibtourismdataspace.org/onboarding/application/additional-request)

**Pujar documentació i enviar la sol·licitud**  
En aquest portal, l’entitat ha de pujar un fitxer `ID.pdf` i fer clic a **Submit Application Request**.

<p align="center">
  <img src="Imatges/UploadIdFile.png" alt="Pujar ID per OnBoarding ESDATIB" height="500">
</p>

Un cop realitzada la sol·licitud, aquesta estarà llesta per ser processada per la **Governança**.  
Els passos següents requereixen **desplegament dels agents al clúster**.

---

### 2. Part amb desplegament dels agents
> ⚠️ Proximament operatiu.
- En aquesta fase ja és necessari **desplegar els agents al clúster** per poder accedir a l’espai de dades i obtenir les credencials corresponents.  

Consulta la **[Guia de procés desplegament Agents](Desplegament/Readme.md)**.


