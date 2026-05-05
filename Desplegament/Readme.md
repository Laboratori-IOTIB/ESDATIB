Aquest document conté una guia sobre l'ordre en què s'ha de depesplegar l'arquitectura. S'explica com desplegar els prerrequisits i els agents (Common, Consumer i Provider).

---
# Ordre del desplegament

1. Prerrequisits Mínims
   
El desplegament d’ESDATIB s’ha realitzat en un clúster AKS a Azure, on es pot oferir més suport. Per seguir la guia en aquest entorn:

[Prerrequisits Mínims Azure](Prerrequisits/Azure/prerrequisits.md)


Tot i així, també es presenta una guia per establir un clúster de Kubernetes amb K3s fora d’Azure:

[Prerrequisits Mínims Local](Prerrequisits/Local/prerrequisits.md)

Un cop desplegats els prerrequisits, el procés de desplegament continua de manera pràcticament idèntica en tots els entorns.

3. [Common Components Agent](Agents/common.md)
4. [Provider Agent](Agents/provider.md)
5. [Consumer Agent](Agents/consumer.md) 



