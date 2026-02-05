Aquest document descriu el fitxer YAML utilitzat per crear una **Application d’Argo CD** mitjançant Helm.
Aquest és el fitxer que s’introdueix a **Argo CD → New App → Edit as YAML**.

Aquesta guia serveix per ajudar les entitats que volen desplegar l'agent **Common**. A continuació es detallen els paràmetres de configuració:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  # Nom de l'aplicació que es desplega actualment a ArgoCD; aquest és el nom que es mostrarà.
  name: 'common01-deployer'                         # Nom de l'aplicació de desplegament a ArgoCD
  namespace: argocd                                 # Namespace del teu ArgoCD
spec:
  project: default
  source:
    repoURL: 'https://code.europa.eu/api/v4/projects/951/packages/helm/stable'
    path: '""'
    targetRevision: 2.4.2                           # Versió del paquet
    helm:
      values: |
        values:
          branch: v2.4.2                            # Branca del repositori amb els valors
        resourcePreset: default                     # Estableix a "low" per deshabilitar les sol·licituds de recursos
        agentList:                                  # Llista de tots els agents a desplegar
          authorities:
            - authority01
          consumers:
            - consumer01
          providers:
            - dataprovider01
        project: default                            # Projecte al qual s'adjunta el namespace
        namespaceTag: common01                      # Identificador del desplegament i part del FQDN
        domainSuffix: example.com                   # Darrera part del FQDN
        argocd:
          appname: common01                         # Nom de l'aplicació ArgoCD generada 
          namespace: argocd                         # Namespace del teu ArgoCD
        cluster:
          # FQDN (Nom de domini completament qualificat) del teu clúster de Kubernetes
          address: https://kubernetes.default.svc
          namespace: common01                       # On es desplegarà l'aplicació
          issuer: dev-prod                          # Emissor del certificat
          internalIssuer: dev-selfsigned            # Emissor de certificats autosignats
          kubeStateHost: kube-prometheus-stack-kube-state-metrics.devsecopstools.svc.cluster.local:8080    # Enllaç al servei kube-state-metrics
        secrets:
          secretEngine: example                     # Nom del motor de secrets KV que es crearà a OpenBao
          role: example-role                        # Nom del rol que es crearà a OpenBao
        kafka:
          ha: true                                  # true crea 3 rèpliques de cada component, false en crea 1 de cada
          topic:
            # Estableix aquest valor a true per fer que Kafka creï automàticament els tòpics necessaris
            autocreate: true
        mailpit:
          # Estableix aquest valor a true per activar Mailpit
          enabled: true
        monitoring:
          # Estableix aquest valor a true per habilitar les funcions de monitoratge
          enabled: true 
```
