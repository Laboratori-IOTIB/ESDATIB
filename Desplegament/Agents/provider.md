
Aquest document descriu el fitxer YAML utilitzat per crear una **Application d’Argo CD** mitjançant Helm.
Aquest és el fitxer que s’introdueix a **Argo CD → New App → Edit as YAML**.

## YAML de l’Application (Argo CD)
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: 'dataprovider01-deployer'           # nom de l'aplicació desplegadora a Argo CD
  namespace: argocd                         # namespace on està desplegat Argo CD
spec:
  project: default
  source:
    repoURL: 'https://code.europa.eu/api/v4/projects/904/packages/helm/stable'
    path: '""'
    targetRevision: 2.4.1                   # versió del paquet Helm
    helm:
      values: |
        values:
          branch: v2.4.1                    # branca del repositori de valors (en versions alliberades ha de ser la branca de release)
        project: default
        namespaceTag:
          dataprovider: dataprovider01      # identificador del desplegament i part del FQDN del Data Provider
          authority: authority01            # identificador del desplegament i part del FQDN de l'autoritat
          common: common01                  # identificador del desplegament i part del FQDN dels components comuns
        domainSuffix: example.com           # part final del FQDN (domini base de l'entorn)
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
