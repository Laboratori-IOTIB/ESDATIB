Aquest document descriu el fitxer YAML utilitzat per crear una **Application d’Argo CD** mitjançant Helm.
Aquest és el fitxer que s’introdueix a **Argo CD → New App → Edit as YAML**.

Aquesta guia serveix per ajudar les entitats que volen desplegar l'agent **Consumer**. A continuació es detallen els paràmetres de configuració:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: 'consumer01-deployer'           # Nom de l'aplicació de desplegament en ArgoCD.
  namespace: argocd                     # Namespace on s'executa el teu ArgoCD.
spec:
  project: default
  source:
    repoURL: 'https://code.europa.eu/api/v4/projects/903/packages/helm/stable' # URL oficial del repositori de Helm.
    path: '""'
    targetRevision: v2.4.1                  # Versió del paquet a desplegar (assegureu-vos que coincideixi amb la versió desitjada).
    helm:
      values: |
        values:
          branch: v2.4.1                    # Branca del repositori amb els valors - per a versions publicades hauria de ser la branca de la release.
        project: default
        namespaceTag: 
          consumer: consumer01              # Identificador únic del desplegament i part del FQDN per a aquest agent Consumer.
          authority: authority01            # Identificador del desplegament de l'Authority (ha de coincidir amb l'entorn).
          common: common01                  # Identificador del desplegament dels components comuns (Common).
        domainSuffix: example.com           # Darrera part del FQDN (sufix del domini). Modifiqueu-ho pel vostre domini.
        resourcePreset: default             # Estableix a "low" per deshabilitar les sol·licituds de recursos (requests) si l'entorn és limitat.
        argocd:
          appname: consumer01               # Nom de l'aplicació ArgoCD que es generarà automàticament.
          namespace: argocd                 # Namespace del teu ArgoCD.
        cluster:
          address: https://kubernetes.default.svc # Adreça del clúster de destinació (normalment local).
          namespace: consumer01             # Namespace on es desplegarà l'aplicació final del Consumer.
          commonToolsNamespace: common01    # Namespace on està desplegada la pila principal de monitoratge.
          issuer: dev-prod                  # Emissor de certificats (Certificate Issuer), ex: le-staging o le-prod.
        secrets:
          secretEngine: example             # Nom del motor de secrets creat a OpenBao.
          role: example-role                # Rol creat a OpenBao per a l'accés.
        monitoring:
          enabled: true                     # Habilitar (true) o deshabilitar (false) el monitoratge.
    chart: consumer                         # Nom del chart de Helm a utilitzar.
  destination:
    server: 'https://kubernetes.default.svc' # Servidor de destinació (el mateix clúster on s'executa ArgoCD).
    namespace: consumer01                   # Namespace on es desplegarà el paquet helm inicial.
```
