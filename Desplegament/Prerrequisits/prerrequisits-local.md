# Desplegament dels Prerequisits

Aquest apartat es basa en la documentació oficial de [Simpl-Open](https://code.europa.eu/simpl/simpl-open/documentation).

Són els serveis que donen suport a la resta dels serveis associats als agents.

En aquesta guia es treballa amb una màquina virtual (VM) on es desplegarà el clúster. Es tracta d'una configuració d'un únic node, que és la més senzilla i amb menys requisits de recursos. Tanmateix, no inclou les característiques pròpies d'un clúster Kubernetes distribuït, com ara alta disponibilitat, escalabilitat horitzontal i tolerància a fallades. Aquesta VM disposa d'una única IP pública. Com que no tenim més IPs públiques disponibles, no necessitem cap balancejador de càrrega com podria ser MetalLB. A més, com que el servidor es troba dins la xarxa de la universitat i no tenim accés directe al gestor de DNS (els registres els gestionen els tècnics de la universitat), no desplegarem `external-dns`.

![Requeriments](../../Imatges/serveisrequerits.png)

Aquests tenen les següents funcionalitats:

1. **Ingress NGINX**: actua com a controlador d’ingrés del clúster, gestionant l’accés extern als serveis desplegats a Kubernetes mitjançant regles HTTP/HTTPS. Permet exposar els serveis de manera segura i centralitzada, i facilita la gestió del trànsit d’entrada.

2. **Argo CD**: eina de desplegament continu basada en el paradigma GitOps. S’encarrega de sincronitzar l’estat del clúster amb la configuració declarativa definida en repositoris Git, garantint desplegaments traçables, reproductibles i auditables. En el nostre cas, ArgoCD sincronitzarà els repositoris de [SIMPL-Open Europe](https://code.europa.eu/simpl), els quals contenen les versions de les aplicacions i serveis associats a cada agent, amb l'estat del clúster.

3. **NFS CSI Provisioner**: proveeix un Container Storage Interface (CSI) basat en NFS que permet la creació dinàmica de volums persistents, especialment útil per a volums amb accés compartit (ReadWriteMany).

4. **Certificate Manager**: gestiona de manera automàtica l’emissió, renovació i ús de certificats digitals (per exemple, TLS/SSL) dins del clúster, facilitant la comunicació segura entre serveis i amb l’exterior.

## Ordre del desplegament

- [1. Desplegament del Clúster K3s](#1-desplegament-del-clúster-k3s)
- [2. Instal·lar NFS](#2-instal·lar-nfs)
- [3. Ingress-NGINX](#3-ingress-nginx)
- [4. cert-manager (sense ClusterIssuer)](#4-cert-manager-sense-clusterissuer)
- [5. ArgoCD](#5-argocd)

---

## 1. Desplegament del Clúster K3s

```bash
# Instal·lar K3s injectant variables per ometre Traefik i ServiceLB
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable traefik --disable servicelb" sh -

# Configurar permisos administratius
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config

# Exportar la ruta de l'arxiu de configuració
echo "export KUBECONFIG=~/.kube/config" >> ~/.bashrc
source ~/.bashrc

# Comprovar l'estat del node
kubectl get nodes
```

---

## 2. Instal·lar NFS

```bash
# Afegir el repositori oficial per al servidor NFS
helm repo add nfs-ganesha-server-and-external-provisioner \
  https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner/
helm repo update

# Crear l'espai de noms
kubectl create namespace nfs-server

# Instal·lar el servidor NFS ancorat al disc de la màquina virtual (local-path)
helm -n nfs-server install nfs-server \
  nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner \
  --set persistence.enabled=true \
  --set persistence.storageClass=local-path \
  --set persistence.size=10Gi

# Verificar les classes d'emmagatzematge (han d'aparèixer 'local-path' i 'nfs')
kubectl get storageclass
```

---

## 3. Ingress-NGINX

```bash
# 1. Afegir el repositori oficial d'Ingress-Nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# 2. Generar l'arxiu de configuració simplificat
cat <<EOF > ingress-nginx-values.yaml
controller:
  # Permet que Nginx escolti directament a les IPs de la VM
  hostNetwork: true
  # Desactivem el servei de tipus LoadBalancer (no disponible en aquest entorn)
  service:
    type: ClusterIP
  # Argument CRÍTIC: habilita el pont transparent per al Tier 2
  extraArgs:
    enable-ssl-passthrough: ""
EOF

# 3. Desplegar el controlador Ingress
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  -f ingress-nginx-values.yaml

# 4. Verificar que el pod del controlador està en estat Running
kubectl get pods -n ingress-nginx
```

---

## 4. cert-manager (sense ClusterIssuer)

```bash
# 1. Afegir el repositori oficial de Jetstack (els creadors de cert-manager)
helm repo add jetstack https://charts.jetstack.io
helm repo update

# 2. Instal·lar cert-manager juntament amb els seus recursos personalitzats (CRDs)
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4 \
  --set installCRDs=true

# 3. Comprovar que els tres pods principals s'inicien correctament
kubectl get pods -n cert-manager

# 4. Crear el secret de Cloudflare
cat <<EOF > cloudflare-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: <NOM_DEL_SECRET_DEL_TOKEN>
  namespace: cert-manager
type: Opaque
stringData:
  api-token: <EL_TEU_TOKEN_DE_CLOUDFLARE>
EOF

kubectl apply -f cloudflare-secret.yaml
# Per seguretat, esborrem l'arxiu amb el token en text pla
rm cloudflare-secret.yaml

# 5. Crear el ClusterIssuer
cat <<EOF > cluster-issuer-cloudflare.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: <NOM_DEL_CLUSTER_ISSUER>
spec:
  acme:
    email: <EL_TEU_EMAIL>
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: <NOM_DEL_SECRET_DE_LA_CLAU_PRIVADA>
    solvers:
    - dns01:
        cloudflare:
          apiTokenSecretRef:
            name: <NOM_DEL_SECRET_DEL_TOKEN>
            key: api-token
EOF

kubectl apply -f cluster-issuer-cloudflare.yaml
```

---

## 5. ArgoCD

```bash
# Afegir el repositori oficial d'ArgoCD
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Instal·lar ArgoCD
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace
```

```bash
# Crear l'Ingress per a ArgoCD
cat <<EOF > argocd-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    # Vinculació amb el ClusterIssuer de Cloudflare per a DNS-01
    cert-manager.io/cluster-issuer: "<NOM_DEL_CLUSTER_ISSUER>"
    # Forçar redirecció a HTTPS
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    # Instrucció crítica: Nginx ha de comunicar-se amb ArgoCD per HTTPS internament
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - argocd.<EL_TEU_DOMINI>
    # Nom del secret on es desarà el certificat real
    secretName: argocd-secret-tls
  rules:
  - host: argocd.<EL_TEU_DOMINI>
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

# Aplicar el manifest
kubectl apply -f argocd-ingress.yaml
```
