### 1. Despliegue del Clúster K3s


```
# Instalar K3s inyectando variables para omitir Traefik y ServiceLB
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable traefik --disable servicelb" sh -

# Configurar permisos administrativos
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config

# Exportar la ruta del archivo de configuración
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
source ~/.bashrc

# Comprobar el estado del nodo
kubectl get nodes
```
### 2. Instalar nfs
```
# Añadir el repositorio oficial para el servidor NFS
helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner/
helm repo update

# Crear el espacio de nombres
kubectl create namespace nfs-server

# Instalar el servidor NFS anclado al disco de la máquina virtual (local-path)
helm -n nfs-server install nfs-server nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner \
  --set persistence.enabled=true \
  --set persistence.storageClass=local-path \
  --set persistence.size=10Gi

# Verificar las clases de almacenamiento (deben aparecer 'local-path' y 'nfs')
kubectl get storageclass
``` 
### 3. INGRESS-NGINX

```
# 1. Añadir el repositorio oficial de Ingress-Nginx 
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx 
helm repo update

# 2. Generar el archivo de configuración simplificado
cat <<EOF > ingress-nginx-values.yaml
controller:
  # Permite que Nginx escuche directamente en las IPs de la VM
  hostNetwork: true
  # Desactivamos el servicio tipo LoadBalancer (no disponible en este entorno)
  service:
    type: ClusterIP
  # Argumento CRÍTICO: Habilita el puente transparente para el Tier 2
  extraArgs:
    enable-ssl-passthrough: ""
EOF

# 3. Desplegar el controlador Ingress
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  -f ingress-nginx-normal-values.yaml

# 4. Verificar que el pod del controlador está en estado Running
kubectl get pods -n ingress-nginx
```

### 4. cert-manager (sense cluster-issuer)

```
# 1. Añadir el repositorio oficial de Jetstack (los creadores de Cert-Manager)
helm repo add jetstack https://charts.jetstack.io
helm repo update

# 2. Instalar Cert-Manager junto con sus recursos personalizados (CRDs)
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4 \
  --set installCRDs=true

# 3. Comprobar que los tres pods principales se inician correctamente
kubectl get pods -n cert-manager

# 4. crear es clouflare-secret
cat <<EOF > cloudflare-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token-secret
  namespace: cert-manager
type: Opaque
stringData:
  api-token: xxxxxx
EOF

kubectl apply -f cloudflare-secret.yaml
# Por seguridad, borramos el archivo con el token en texto plano
rm cloudflare-secret.yaml

# 5. Cream el cluster-issuer
cat <<EOF > cluster-issuer-cloudflare.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-cloudflare
spec:
  acme:
    email: tu-email@uib.es
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-cloudflare-account-key
    solvers:
    - dns01:
        cloudflare:
          apiTokenSecretRef:
            name: cloudflare-api-token-secret
            key: api-token
EOF

kubectl apply -f cluster-issuer-cloudflare.yaml
```


### 5. Argocd

``` 
# Añadir el repositorio oficial de ArgoCD
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
# Instalar ArgoCD 
helm install argocd argo/argo-cd \ --namespace argocd \ --create-namespace


cat <<EOF > argocd-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    # Vinculación con el ClusterIssuer de Cloudflare para DNS-01
    cert-manager.io/cluster-issuer: "letsencrypt-cloudflare"
    # Forzar redirección a HTTPS
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    # Instrucción crítica: Nginx debe hablar con ArgoCD por HTTPS internamente
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - argocd.simpl.datosmeteo.com
    # Nombre del secreto donde se guardará el certificado real
    secretName: argocd-secret-tls
  rules:
  - host: argocd.simpl.datosmeteo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              number: 443
EOF

# Aplicar el manifiesto
kubectl apply -f argocd-ingress.yaml

```

### 6. Common01
```  
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: common01-deployer
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://code.europa.eu/api/v4/projects/951/packages/helm/stable'
    chart: common_components
    targetRevision: 3.0.0
    helm:
      values: |
        values:
          branch: v3.0.0
        resourcePreset: low
        agentList:
          authorities:
            - authority01
          consumers:
            - consumer01
          providers:
            - dataprovider01
        project: default
        namespaceTag: common01
        domainSuffix: simpl.datosmeteo.com
        argocd:
          appname: common01
          namespace: argocd
        cluster:
          address: https://kubernetes.default.svc
          namespace: common01
          issuer: letsencrypt-cloudflare
          internalIssuer: dev-selfsigned
          kubeStateHost: kube-prometheus-stack-kube-state-metrics.devsecopstools.svc.cluster.local:8080
        secrets:
          secretEngine: common01
          role: common01-role
        kafka:
          ha: false
          topic:
            autocreate: true
        mailpit:
          enabled: true
        # Desactivación de componentes con problemas de red/recursos
        monitoring:
          enabled: false
        notification:
          enabled: false
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: common01
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### 7. Data provider

```

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
    targetRevision: 1.0.4-SNAPSHOT.17.8130c477                   # versió del paquet Helm
    helm:
      values: |
        values:
          branch: v3.0.2                    # branca del repositori de valors (en versions alliberades ha de ser la branca de release)
        project: default
        namespaceTag:
          dataprovider: dataprovider01      # identificador del desplegament i part del FQDN del Data Provider
          authority: authority01            # identificador del desplegament i part del FQDN de l'autoritat
          common: common01                  # identificador del desplegament i part del FQDN dels components comuns
        domainSuffix: simpl.datosmeteo.com           # part final del FQDN (domini base de l'entorn)
        resourcePreset: low             # establir a "low" per no definir requests de recursos
        argocd:
          appname: dataprovider01           # nom de l'aplicació generada a Argo CD
          namespace: argocd                 # namespace on està desplegat Argo CD
        cluster:
          address: https://kubernetes.default.svc
          namespace: dataprovider01         # namespace on es desplegarà l'aplicació
          commonToolsNamespace: common01    # namespace on està desplegat l'stack principal de monitoratge
          issuer: letsencrypt-cloudflare                  # issuer dels certificats TLS
        secrets:
          role: common01-role                # rol creat a OpenBao per accedir als secrets
          secretEngine: common01             # nom del motor de secrets creat a OpenBao
        crossplane:
          enabled: false                     # indica si s'han de desplegar components d'infraestructura, només pot existir una instància de Crossplane per clúster
          kafka:
            username: dataprovider01_infrabe
            password: mOkTA4XGFZHWo0d4                  # contrasenya de Kafka, s'ha d'obtenir del secret common01-kafka-credentials d'OpenBao
          gitea:
            username: gitops_test           # nom d'usuari de Gitea
            password: pass                  # contrasenya de Gitea, la variable pot prendre qualsevol valor (configureu-la segons les vostres preferències)
                                           
        monitoring:
          enabled: false                     # indica si el monitoratge ha d'estar activat
 
    chart: data-provider
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: dataprovider01               # namespace on es desplegarà el paquet
```

### 8. canviar ses urls de authority per es domini corresponent i redirecció de port: 

```
# 1. Cambiar en el entorno del tier2-gateway (Aquí estaba el 'authority.be...')
kubectl get configmap tier2-gateway-env-configmap -n dataprovider01 -o yaml | sed 's/authority01.simpl.datosmeteo.com/authority01.esdatib.org/g' | kubectl apply -f -

# 2. Cambiar en la configuración Spring del tier2-gateway
kubectl get configmap tier2-gateway-spring-configmap -n dataprovider01 -o yaml | sed 's/authority01.simpl.datosmeteo.com/authority01.esdatib.org/g' | kubectl apply -f -

# 3. Cambiar en el Authentication Provider (Para asegurar que no quede ningún rastro)
kubectl get configmap authentication-provider-configmap -n dataprovider01 -o yaml | sed 's/authority01.simpl.datosmeteo.com/authority01.esdatib.org/g' | kubectl apply -f -

# Resetetjar es pods
kubectl rollout restart deployment tier2-gateway authentication-provider -n dataprovider01

# Confirmar
kubectl get configmap tier2-gateway-env-configmap -n dataprovider01 -o yaml
kubectl get configmap authentication-provider-configmap -n dataprovider01 -o yaml

-------#sd-creation-wizar -----------------------------------------------------
hem vist que amb es fors que es mostraran a continuació no es canvien les 2 urls del deployment:

for cm in $(kubectl get configmaps -n dataprovider01 -o name); do \
  kubectl get $cm -n dataprovider01 -o yaml | sed 's/authority01.simpl.datosmeteo.com/authority01.esdatib.org/g' | kubectl apply -f -; \
done

for dp in $(kubectl get deployments -n dataprovider01 -o name); do \
  kubectl get $dp -n dataprovider01 -o yaml | sed 's/authority01.simpl.datosmeteo.com/authority01.esdatib.org/g' | kubectl apply -f -; \
done
kubectl rollout restart deployment -n dataprovider01


```

No basta, hem d'anar nes deployer de simpl-creation-wizard i canviar es domini que surten a les 2 urls referents a authority (vendràn amb es domini del provider) per el domini de l'autoritat de l'espai de dades que es vol conectar el provider. 

A més hem de **desactivar la sincronització automàtica** de l'aplicació "dataprovider", cosa que es pot fer des de l'argocd.
Después de solucionar el problema per fer l'onboarding correctement hi ha un altre problema si el proveidor no té obert el port 19194 (pensam que si el té obert anirà bé). El problema ve quan el consumer vol començar la negociació d'un contracte amb un asset publicat pel nostre proveidor. Si el té tancat, com és el nostre cas, el consumer no podrà establir la negociació perquè no arribarà la solicitu (la cual ha d'arribar pel port mencionat). Per solucionar aquest problema afegirem el següent ingress: 

```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tier2
  namespace: dataprovider01
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  rules:
  - host: tls.participant.dataprovider01.simpl.datosmeteo.com
    http:
      paths:
      - backend:
          service:
            name: tier2-gateway
            port:
              number: 443
        path: /
        pathType: ImplementationSpecific
EOF
``` 

### Punts clau de la configuració de l'Ingress

- **Múltiplex de ports (443 → 19194)**: El punt més crític a nivell d'infraestructura. Com que el firewall de la UIB només permet el trànsit pel port **443**, l'Ingress actua com un pont: rep la petició pel port estàndard de navegació (HTTPS) i la redirigeix internament al port **19194** de l'EDC, saltant-se així la restricció del firewall.
    
- **`pathType: ImplementationSpecific`**: Necessari quan fem servir reescritures de rutes i expressions regulars complexes en Nginx, ja que ens dóna la flexibilitat que els tipus `Prefix` o `Exact` no permeten.
    

---


### 9. consumer

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: consumer01-deployer
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://gitlab.com/api/v4/projects/80314429/packages/helm/stable
    targetRevision: v1.0.13-SNAPSHOT.latest
    chart: consumer
    helm:
      # Hemos eliminado la línea "values:" extra de aquí abajo
      values: |
        branch: v3.0.1
        project: default
        namespaceTag:
          consumer: consumer01
          authority: authority01
          common: common01
        domainSuffix: simpl.datosmeteo.com
        resourcePreset: low
        argocd:
          appname: consumer01
          namespace: argocd
        cluster:
          address: https://kubernetes.default.svc
          namespace: consumer01
          commonToolsNamespace: common01
          issuer: letsencrypt-cloudflare
        secrets:
          secretEngine: common01
          role: common01-role
        monitoring:
          enabled: false
  destination:
    server: https://kubernetes.default.svc
    namespace: consumer01
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true

```


```

for cm in $(kubectl get configmaps -n consumer01 -o name); do 
  kubectl get $cm -n consumer01 -o yaml | sed 's/authority01.simpl.datosmeteo.com/authority01.esdatib.org/g' | kubectl apply -f -; 
done

for dp in $(kubectl get deployments -n consumer01 -o name); do 
  kubectl get $dp -n consumer01 -o yaml | sed 's/authority01.simpl.datosmeteo.com/authority01.esdatib.org/g' | kubectl apply -f -; 
done
```

Per tal de poder veure el catàleg i començar els contractes amb el proveidor de l'asset en concret, s'ha d'anar la url del consumer ".../users-roles" i assignar a "Consumer_P" els rols "CATALOG_R" i "SD_CONSUMER".



Pq puguin pijar el botó "obtenir dades" i negociar el contracte

```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tier2
  namespace: consumer01
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
spec:
  rules:
  - host: tls.participant.consumer01.simpl.datosmeteo.com
    http:
      paths:
      - backend:
          service:
            name: tier2-gateway
            port:
              number: 443
        path: /
        pathType: ImplementationSpecific
EOF
```

