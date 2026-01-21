- [Desplegament dels Prerrequisits](#desplegament-aplicació-amb-argocd)
  - [Desplegament i configuració d'ingress nginx](#instal·lació-dargocd)
  - [Desplegament i configuració d'ArgoCD](#instruccions-per-desplegar-laplicació)
  - [Desplegament i configuració d'External DNS](#instruccions-per-desplegar-laplicació)
  - [Desplegament i configuració de CertificateManager](#instruccions-per-desplegar-laplicació)
  - [Desplegament i configuració de NFS CSI provisioner](#instruccions-per-desplegar-laplicació)
---

# Desplegament dels Prerrequisits
## Desplegament i configuració d'ingress nginx

Controlador d'ingrés basat en Nginx, gestiona el tràfic d’entrada HTTP/HTTPS cap a les aplicacions del clúster. Treballa conjuntament amb External DNS i Certificate Manager per exposar serveis de manera segura i automatitzada.

![Ingress](../Imatges/ingress.png)

### Instal·lació

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

kubectl create namespace ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx \
    --namespace ingress-nginx \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux


helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --version 4.10.0
```

L'eina tindrà el seu propi namespace. Comprovam que s'ha instal·lat correctament:

```bash
kubectl get pods -n ingress-nginx
```

Hauria de sortir:

```bash
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-qhjcq       0/1     Completed   0          4d20h
ingress-nginx-admission-patch-s2695        0/1     Completed   0          4d20h
ingress-nginx-controller-9cc49f96f-96xdq   1/1     Running     0          4d20h
```

A continuació comprovarem l'IP extern o port del controlador ingress

```bash
kubectl get svc -n ingress-nginx
```

Hauria de sortir:

```bash
NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.111.45.147   <none>        80:32691/TCP,443:32613/TCP   4d20h
ingress-nginx-controller-admission   ClusterIP   10.101.37.174   <none>        443/TCP                      4d20h
```

Podem veure que per defecte tipus de servei del controlador d'ingress és "NodePort". En el nostre cas vàrem canviar el tipus a "LoadBalancer".

Un Load Balancer (equilibrador de càrrega) és un component que:

1. Rep el trànsit entrant des d’Internet

2. El distribueix entre diversos pods o nodes

3. Proporciona un únic punt d’entrada a les aplicacions

4. Millora la disponibilitat, escalabilitat i tolerància a fallades

5. En Kubernetes, el Load Balancer és habitualment el punt d’accés públic cap al clúster.

En el cas d'Azure, aquest LoadBalancer rep una ip pública automàticament. Si a l'entorn on es desplega aquest LoadBalancer no hi ha aquesta assignació automàtica 
de la IP pública, s'haurà de fer manualment.

## Desplegament i configuració d'ArgoCD

### Instal·lació d’ArgoCD

*(En aquesta secció s’haurà d’incloure la guia d’instal·lació d’ArgoCD al clúster de Kubernetes)*

### Instruccions per desplegar aplicació

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

<!-- <p align="center">
  <img src="Imatges/ArgoCd_Deploy_YAML.jpeg" alt="Desplegament d’ArgoCD amb YAML" height="400">
</p> -->

![Ingress](../Imatges/ArgoCd_Deploy_YAML.jpeg)

## Desplegament i configuració d'External DNS

Aquesta secció descriu **pas a pas** com desplegar i configurar **External-DNS** en un clúster **AKS** quan el domini està **comprat i gestionat a Azure DNS** (per exemple, domini adquirit a través d’Azure App Service).

> ℹ️ **Nota prèvia**
>
> En cas que es vulgui no es desplegui en un AKS, recomenam seguir 

---

### Prerequisits

- `az` (Azure CLI) autenticat (`az login`)
- `kubectl` configurat contra el clúster AKS
- `helm` instal·lat
- Clúster AKS amb **Managed Identity** habilitada
- Zona DNS ja creada a Azure DNS

---

### Variables necessàries

Definiu primer les variables següents:

```bash
# DADES DEL CLÚSTER I DNS
CLUSTER_NAME=""          # Nom del clúster AKS
CLUSTER_RG=""            # Resource Group del clúster (NO el MC_)
DNS_ZONE_NAME=""         # Nom del domini (ex: example.com)
DNS_ZONE_RG=""           # Resource Group on està la zona DNS
```


1. Obtenir identificadors d'Azure
```bash
# Subscription ID
SUBSCRIPTION_ID=$(az account show --query id -o tsv)

# Tenant ID
TENANT_ID=$(az account show --query tenantId -o tsv)

# Client ID de la identitat Kubelet (IDENTITAT CORRECTA)
IDENTITY_CLIENT_ID=$(az aks show \
  -g $CLUSTER_RG \
  -n $CLUSTER_NAME \
  --query identityProfile.kubeletidentity.clientId \
  -o tsv)
```
2. Assignar permissos a la identidad del clúster
   
External-DNS necessita permisos sobre la zona DNS per poder crear i actualitzar registres.

```bash
az role assignment create \
  --assignee $IDENTITY_CLIENT_ID \
  --role "DNS Zone Contributor" \
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$DNS_ZONE_RG"
```
> ⚠️ **Atenció:**  
> Si la vostra subscripció **no permet assignar rols**, aquesta comanda retornarà error.  
> En aquest cas, un administrador d’Azure haurà de realitzar l’assignació manualment.


3. Crear el fitxer de configuració azure.json
Aquest fitxer permet que External-DNS s’autentiqui amb Azure mitjançant **Managed Identity**.
```bash
cat <<EOF > azure.json
{
  "tenantId": "$TENANT_ID",
  "subscriptionId": "$SUBSCRIPTION_ID",
  "resourceGroup": "$DNS_ZONE_RG",
  "useManagedIdentityExtension": true,
  "userAssignedIdentityID": "$IDENTITY_CLIENT_ID"
}
EOF
```
4. Crear el namespace d’External-DNS
```bash
kubectl create namespace external-dns \
  --dry-run=client -o yaml | kubectl apply -f -
```
5. Crear el secret amb la configuració d’Azure
Eliminar el secret anterior si existeix:
```bash
kubectl -n external-dns delete secret external-dns-azure-config \
  --ignore-not-found
```
Crear el nou secret:
```bash
kubectl -n external-dns create secret generic external-dns-azure-config \
  --from-file=azure.json
```
6. Afegir el repositori Helm de Bitnami
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
7. Instal·lar o actualitzar External-DNS
```bash
helm upgrade --install external-dns bitnami/external-dns \
  --namespace external-dns \
  --set provider=azure \
  --set azure.useManagedIdentityExtension=true \
  --set azure.resourceGroup=$DNS_ZONE_RG \
  --set azure.tenantId=$TENANT_ID \
  --set azure.subscriptionId=$SUBSCRIPTION_ID \
  --set txtOwnerId=$CLUSTER_NAME \
  --set domainFilters={$DNS_ZONE_NAME} \
  --set sources={ingress} \
  --set extraVolumeMounts[0].name=azure-config-file \
  --set extraVolumeMounts[0].mountPath=/etc/kubernetes/azure.json \
  --set extraVolumeMounts[0].subPath=azure.json \
  --set extraVolumes[0].name=azure-config-file \
  --set extraVolumes[0].secret.secretName=external-dns-azure-config
```
8. Verificació
Reiniciar el deployment per assegurar que es carrega tota la configuració:
```bash
kubectl -n external-dns rollout restart deployment external-dns
```
Esperar uns segons:
```bash
sleep 10
```
Consultar els logs: 
```bash
kubectl -n external-dns logs -f \
  -l app.kubernetes.io/name=external-dns
```

## Desplegament i configuració de CertificateManager
## Desplegament i configuració de NFS CSI provisioner

**Atenció:**  
Aquest desplegament s’ha de fer seguint el manual d’usuari de NFS Provider i està pensat per entorns **Azure**.

---

### Prerequisits

```md
- `kubectl` configurat contra el clúster
- `helm` instal·lat
- StorageClass per defecte funcional al clúster


1. Afegir el repositori Helm
```bash
helm repo add kvaps https://kvaps.github.io/charts
helm repo update
```
2. Instal·lar el NFS Server Provisioner
```bash
helm install nfs-server kvaps/nfs-server-provisioner \
  --namespace nfs-server \
  --create-namespace \
  --set persistence.enabled=true \
  --set persistence.storageClass=default \
  --set persistence.size=10Gi \
  --set storageClass.name=nfs
```
3. Compatibilitat amb Azure Storage
> ⚠️ **Atenció:**
> Azure **no és compatible** amb la StorageClass `csi-cinder-high-speed`.
> Per aquesta raó, es defineix manualment una StorageClass amb el driver real d’Azure.
```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-cinder-high-speed   # Nom requerit pels PVCs existents
provisioner: disk.csi.azure.com # Driver real d’Azure
parameters:
  skuName: Premium_LRS
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
EOF
```
4. Recrear PersistentVolumeClaims (PVC)
Després de canviar o crear la StorageClass, és necessari eliminar els PVCs
existents perquè Kubernetes els torni a crear amb la configuració correcta.
Eliminar els PVCs d’Elasticsearch:
```bash
kubectl delete pvc -n common01 -l app=elasticsearch-maste
```
Eliminar els PVCs de Logstash:
```bash
kubectl delete pvc -n common01 -l app=logstash-logstash-beats
```









