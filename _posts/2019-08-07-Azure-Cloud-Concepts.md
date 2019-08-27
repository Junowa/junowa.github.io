---
layout: post
author: junowa
---

This post gathers Azure Cloud concepts description.

## Azure Availability Sets

### Concepts

Les availability-sets assurent la disponibilité des VMs dans les cas suivants:

- **événements de maintenance matérielle non planifiée**: une VM migre dynamiquement dans le DC Azure suite à une panne; il y a alors une interruption de courte période de la VM
- **temps d’arrêt inattendu**: la migration dynamique n'est pas possible, un autre type de migration (je n'ai pas trouvé les détails dans la doc) est utilisée, provoquant aussi une interruption de la VM.
- **événements de maintenance planifiés**, principalement l'application des mises à jour de l'infrastructure Azure. Il est possible d'anticiper ces événements.

Azure recommande de regrouper a minima 2 VMs dans un groupe de disponibilité au sein d’un centre de données afin d'assurer la disponibilité d’au moins une des machines virtuelles pendant un événement de maintenance planifié ou non, avec le niveau de 99,95 %.

Chaque machine virtuelle de votre groupe à haute disponibilité se voit attribuer un domaine de mise à jour et un domaine d’erreur.

**Domaine de mise à jour**

Azure déploie ces maintenances planifiées par domaine de mise à jour.

Un seul domaine ne peut être redémarré à la fois.Un domaine de mise à jour redémarré bénéficie de 30 minutes pour récupérer avant que la maintenance ne soit lancée sur un autre domaine de mise à jour.

**Domaine d'erreur**

Les domaines d’erreur définissent le groupe de machines virtuelles partageant une source d’alimentation et un commutateur réseau communs.
Par défaut, les VMs d'une groupe de disponibilité sont réparties dans 3 domaines d'erreur.

Les domaines d'erreur protége des pannes réseau et serveur physique.
Les disques managés sont quant à eux placés dans de domaine d'erreur de stockage alignés sur les domaines d'erreur (disques managés dans des clusters de stockage disctincts).

L'affectation d'une VM à un groupe de disponibilité se fait à sa création (impossible de rattacher une VM existante à un groupe de disponibilité).

### Utilisation d'un Load Balancer

Placer plusieurs machines virtuelles de la même couche dans le même équilibrage de charge et groupe à haute disponibilité permet de toujours avoir au moins une instance disponible pour le trafic.

Il n'est pas possible dans un même backend d'avoir 2 VM provenant des groupes de disponibilité différents.

Dans une availability-set, une VM est obligatoirement rattachée une adresse IP publique ou au backend pool d'un load balancer.

### Azure Availability Zones

Elle permettent d'augmenter la disponibilité, en répartissant les ressources dans une zone géographique distincte dans la même région.


source : https://docs.microsoft.com/fr-fr/azure/virtual-machines/windows/manage-availability


## Azure Storage Redundancy

The different redundancy options are:
- Locally redundant storage (LRS)
- Zone-redundant storage (ZRS): cover
- Geo-redundant storage (GRS)
- Read-access geo-redundant storage (RA-GRS)

