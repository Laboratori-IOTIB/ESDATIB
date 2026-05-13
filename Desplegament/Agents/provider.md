
Aquest document descriu el fitxer YAML utilitzat per crear una **Aplicació d'Argo CD** mitjançant Helm.
Aquest és el fitxer que s’introdueix a **Argo CD → New App → Edit as YAML**.

## YAML de l’Application (Argo CD)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: 'dataprovider01-deployer'           # nom de l'aplicació desplegadora a Argo CD
  namespace: argocd                         # namespace on està desplegat Argo CD
spec:
  project: default
  source:
    repoURL: 'https://gitlab.com/api/v4/projects/80413413/packages/helm/stable'
    path: '""'
    targetRevision: 1.0.7-SNAPSHOT.29.6825468d                   # versió del paquet Helm
    helm:
      values: |
        values:
          branch: v3.0.2                    # branca del repositori de valors (en versions alliberades ha de ser la branca de release)
        project: default
        namespaceTag:
          dataprovider: dataprovider01      # identificador del desplegament i part del FQDN del Data Provider
          authority: authority01            # identificador del desplegament i part del FQDN de l'autoritat
          common: common01                  # identificador del desplegament i part del FQDN dels components comuns
        domainSuffix: example.com           # part final del FQDN (domini base de l'entorn)
        domainSuffixAuth: ibtourismdataspace.org # No modificar  
        resourcePreset: default             # establir a "low" per no definir requests de recursos
        argocd:
          appname: dataprovider01           # nom de l'aplicació generada a Argo CD
          namespace: argocd                 # namespace on està desplegat Argo CD
        cluster:
          address: https://kubernetes.default.svc
          namespace: dataprovider01         # namespace on es desplegarà l'aplicació
          commonToolsNamespace: common01    # namespace on està desplegat l'stack principal de monitoratge
          issuer: dev-prod                  # issuer dels certificats TLS
        secrets:
          role: example-role                # rol creat a OpenBao per accedir als secrets
          secretEngine: example             # nom del motor de secrets creat a OpenBao
        crossplane:
          enabled: true                     # indica si s'han de desplegar components d'infraestructura, només pot existir una instància de Crossplane per clúster
          kafka:
            username: user                  # nom d'usuari de Kafka
                                            # ha de seguir el patró: <namespace>_infrabe
                                            # exemple: dataprovider01_infrabe
            password: pass                  # contrasenya de Kafka, s'ha d'obtenir del secret common01-kafka-credentials d'OpenBao
          gitea:
            username: gitops_test           # nom d'usuari de Gitea
            password: pass                  # contrasenya de Gitea, la variable pot prendre qualsevol valor (configureu-la segons les vostres preferències)
                                            
        monitoring:
          enabled: true                     # indica si el monitoratge ha d'estar activat

    chart: data-provider
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: dataprovider01               # namespace on es desplegarà el paquet
```

Una vegada s'ha desplegat l'aplicació, hi haurà alguns pods que donaran error. Els pods que donaran error són:
tier2-proxy, tier2-gateway. Aquests s'arreglaran quan s'hagi acabat de fer l'onboarding. 



## Instal·lació de MinIO

En cas que es vulgui instal·lar el minio, es pot seguir aquests passos: 
> **⚠️IMPORTANT:** En l'script i les explicacions següents utilitzarem **`dataprovider01`** com a exemple. Recordeu **substituir `dataprovider01` pel nom real del vostre namespace** a tot el fitxer YAML i a les rutes de configuració.

### 1. Crear el fitxer .yaml on copiarem l'script:

```bash
nano minio-setup.yaml
```
### 2. Copiar l'script:

Un cop emplenats tots els camps amb la informació corresponent, es pot copiar tot l'script dins del fitxer .yaml creat anteriorment.

```yaml
# 1. L'EMMAGATZEMATGE (PVC - PersistentVolumeClaim)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: minio-pvc-dades
  namespace: dataprovider01      # Substituïu pel vostre namespace real
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi              # Modifiqueu la capacitat segons necessitat
---
# 2. EL DESPLEGAMENT (DEPLOYMENT)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minio-deployment
  namespace: dataprovider01      # Substituïu pel vostre namespace real
  labels:
    app: minio-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: minio-app
  template:
    metadata:
      labels:
        app: minio-app
    spec:
      containers:
      - name: minio
        image: minio/minio:latest
        imagePullPolicy: Always
        args:
        - server
        - /data
        - --console-address
        - :9001
        env:
        # Credencials d'accés: Aquests valors seran necessaris més endavant a OpenBao
        - name: MINIO_ROOT_USER
          value: el_teu_usuari   # Correspon a la variable 'access_key'
        - name: MINIO_ROOT_PASSWORD
          value: clau_segura_123 # Correspon a la variable 'secret_key'
        ports:
        - containerPort: 9000    # Port per a l'API de dades
          protocol: TCP
        - containerPort: 9001    # Port per a la interfície web (Consola)
          protocol: TCP
        volumeMounts:
        - name: storage
          mountPath: /data
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: minio-pvc-dades
---
# 3. EL SERVEI DE XARXA (SERVICE)
apiVersion: v1
kind: Service
metadata:
  name: minio-servei
  namespace: dataprovider01      # Substituïu pel vostre namespace real
spec:
  type: ClusterIP
  selector:
    app: minio-app
  ports:
    - name: api
      port: 9000
      targetPort: 9000
    - name: console
      port: 9001
      targetPort: 9001
---
# 4. CONFIGURACIÓ DE L'ACCÉS EXTERN (INGRESS)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minio-ingress
  namespace: dataprovider01      # Substituïu pel vostre namespace real
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-azure # Emissor de certificats
spec:
  ingressClassName: nginx
  rules:
    - host: minio.example.com    # Canvieu-ho pel domini base del vostre entorn
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: minio-servei
                port:
                  number: 9001   # Exposem el port 9001 per accedir a la consola web
  tls:
    - hosts:
        - minio.example.com      # Mateix domini que a l'apartat 'rules'
      secretName: minio-tls-cert
```

### 3. Executar l'script:

```bash
kubectl apply -f minio-setup.yaml
```
### 4. Canviar els valors de les variables.

Perquè els components puguin fer servir aquest servei de MinIO, cal configurar les credencials al motor de secrets. Aneu a la interfície d'OpenBao i navegueu fins a la ruta del vostre connector (per exemple: Common01 → dataprovider01-simpl-edc → Create new version).

Actualitzeu les tres variables següents perquè coincideixin amb la configuració que acabeu de desplegar al YAML:

    -fr_gxfs_s3_access_key: Introduïu el valor que hàgiu posat a la variable MINIO_ROOT_USER.

    -fr_gxfs_s3_secret_key: Introduïu el valor que hàgiu posat a la variable MINIO_ROOT_PASSWORD.

    -fr_gxfs_s3_endpoint: Introduïu l'adreça DNS interna del clúster (és recomanable no utilitzar IPs fixes en entorns Kubernetes). Seguint l'exemple d'aquesta guia, el format correcte és:
    http://minio-servei.dataprovider01.svc.cluster.local:9000
