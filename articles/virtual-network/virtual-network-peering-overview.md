---
title: "Homologation de réseaux virtuels Azure | Microsoft Docs"
description: "En savoir plus sur l’homologation de réseaux virtuels dans Azure."
services: virtual-network
documentationcenter: na
author: NarayanAnnamalai
manager: jefco
editor: tysonn
ms.assetid: eb0ba07d-5fee-4db0-b1cb-a569b7060d2a
ms.service: virtual-network
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: narayan;anavin
ms.openlocfilehash: 7d3e6a34b5851a5a35a530b18efc3db3e2249274
ms.sourcegitcommit: 3df3fcec9ac9e56a3f5282f6c65e5a9bc1b5ba22
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2017
---
# <a name="virtual-network-peering"></a>Homologation de réseaux virtuels

L’homologation de réseaux virtuels vous permet de connecter deux [réseaux virtuels](virtual-networks-overview.md) Azure en toute transparence. Une fois homologués, les réseaux virtuels apparaissent comme un seul réseau à des fins de connectivité. Le trafic entre les machines virtuelles des réseaux virtuels homologués est acheminé via l’infrastructure principale de Microsoft de façon assez semblable au trafic entre des machines virtuelles d’un même réseau virtuel via des adresses IP *privées* seulement. 

Voici quelques-uns des avantages de l’homologation de réseaux virtuels :

* Le trafic réseau entre les réseaux virtuels homologués est privé. Le trafic entre les réseaux virtuels reste sur le réseau principal de Microsoft. Aucun chiffrement et aucune connexion Internet publique, ni passerelle ne sont nécessaires pour que les réseau virtuels communiquent.
* Connexion à latence faible et haut débit entre les ressources de différents réseaux virtuels.
* La possibilité pour les ressources d’un réseau virtuel de communiquer avec les ressources d’un autre réseau virtuel, une fois que les réseaux virtuels sont homologués.
* La possibilité de transférer des données dans des abonnements Azure, des modèles de déploiement et dans les régions Azure (préversion).
* Possibilité d’homologuer deux réseaux virtuels créés via le modèle de déploiement Azure Resource Manager ou d’homologuer un réseau virtuel créé via Resource Manager pour un réseau virtuel créé via le modèle de déploiement classique. Pour en savoir plus sur les modèles de déploiement Azure, consultez l’article [Déploiement Azure Resource Manager et déploiement classique : comprendre les modèles de déploiement et l’état de vos ressources](../azure-resource-manager/resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
* Aucune interruption des ressources dans un réseau virtuel lors de la création de l’homologation ou après que l’homologation est créée.

## <a name="requirements-constraints"></a>Exigences et contraintes

* L’appairage de réseaux virtuels au sein d’une même région est généralement possible. L’appairage de réseaux virtuels appartenant à des régions différentes est actuellement en préversion dans les régions États-Unis Centre-Ouest, Centre du Canada et Ouest des États-Unis 2. Avant de procéder à l’homologation de réseaux virtuels dans des régions différentes, vous devez d’abord [enregistrer votre abonnement](virtual-network-create-peering.md#register) pour la préversion. Toute tentative d’homologation entre des réseaux virtuels se trouvant dans des régions différentes échouera si vous n’avez pas suivi la procédure d’enregistrement pour la préversion.
    > [!WARNING]
    > Les homologations de réseaux virtuels créées entre les régions peuvent ne pas avoir le même niveau de disponibilité et de fiabilité que les homologations d’une version publique. Les appairages de réseaux virtuels peuvent avoir des fonctionnalités limitées et peuvent ne pas être disponibles dans toutes les régions Azure. Pour les notifications les plus récentes sur la disponibilité et l’état de cette fonctionnalité, consultez la page relative aux [mises à jour du réseau virtuel Azure](https://azure.microsoft.com/updates/?product=virtual-network).

* Les réseaux virtuels homologués doivent avoir des espaces d’adressage IP qui ne se chevauchent pas.
* Il n’est pas possible d’ajouter des plages d’adresses à l’espace d’adressage d’un réseau virtuel, ni d’en supprimer une fois que celui-ci est homologué avec un autre réseau virtuel. Si vous avez besoin d’ajouter des plages d’adresses à l’espace d’adressage d’un réseau virtuel homologué, vous devez supprimer l’homologation, ajouter l’espace d’adressage, puis ajouter à nouveau l’homologation.
* L’homologation se fait entre deux réseaux virtuels. Il n’y a aucune relation transitive entre homologations. Par exemple, si virtualNetworkA est homologué avec virtualNetworkB, lequel est homologué avec virtualNetworkC, virtualNetworkA n’est *pas* homologué avec virtualNetworkC.
* Vous pouvez homologuer des réseaux virtuels qui existent dans deux abonnements différents sous réserve qu’un utilisateur privilégié (voir la section [specific permissions](create-peering-different-deployment-models-subscriptions.md#permissions) [Autorisations spécifiques]) de chacun de ces abonnements autorise l’homologation, et que les abonnements soient associés au même locataire Azure Active Directory. Vous pouvez utiliser une [passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json#V2V) pour vous connecter aux réseaux virtuels des abonnements associés à différents locataires Active Directory.
* Deux réseaux virtuels peuvent être homologués s’ils sont créés via le modèle de déploiement Resource Manager ou si l’un est créé via le modèle de déploiement Resource Manager et l’autre via le modèle de déploiement classique. Toutefois, les réseaux virtuels créés via le modèle de déploiement classique ne peuvent pas être homologués l’un envers l’autre. Vous pouvez utiliser une [passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json#V2V) pour connecter des réseaux virtuels créés via le modèle de déploiement classique.
* Si la communication entre les machines virtuelles de réseaux virtuels homologués ne présente pas de restrictions de bande passante supplémentaires, une bande passante réseau maximale qui dépend de la taille de la machine virtuelle s’applique toujours. Pour en savoir plus sur la bande passante réseau maximale pour différentes tailles de machine virtuelle, lisez les articles [Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) sur les tailles de machine virtuelle.

     ![Homologation de réseaux virtuels de base](./media/virtual-networks-peering-overview/figure03.png)

## <a name="connectivity"></a>Connectivité

Une fois les réseaux virtuels homologués, les ressources de l’un peuvent se connecter directement à celles du réseau virtuel homologué.

La latence du réseau entre des machines virtuelles de réseaux virtuels homologués dans la même région est la même que celle d’un seul réseau virtuel. Le débit du réseau repose sur la bande passante autorisée pour la machine virtuelle proportionnellement à sa taille. Aucune restriction de bande passante supplémentaire n’est appliquée au sein de l’homologation.

Le trafic entre les machines virtuelles dans des réseaux virtuels homologués est acheminé directement via l’infrastructure principale de Microsoft et non via une passerelle ou une connexion Internet publique.

Les machines virtuelles d’un réseau virtuel peuvent accéder à l’équilibreur de charge interne du réseau virtuel homologué dans la même région. La prise en charge d’équilibreur de charge interne ne s’étend pas aux réseaux virtuels homologués à l’échelle mondiale (préversion). La version de disponibilité générale de l’homologation de réseaux virtuels mondiale prendra en charge l’équilibreur de charge interne.

Des groupes de sécurité réseau peuvent être appliqués dans l’un ou l’autre des réseaux virtuels pour bloquer l’accès à d’autres réseaux virtuels ou à des sous-réseaux si besoin est.
Lors de la configuration de l’homologation de réseaux virtuels, vous pouvez ouvrir ou fermer les règles du groupe de sécurité réseau entre les réseaux virtuels. Si vous ouvrez totalement la connectivité entre les réseaux virtuels homologués (ce qui est l’option par défaut), vous pouvez alors appliquer des groupes de sécurité réseau à des sous-réseaux ou des machines virtuelles spécifiques pour bloquer ou refuser certains accès. Pour en savoir plus sur les groupes de sécurité réseau, consultez [Filtrer le trafic réseau avec les groupes de sécurité réseau](virtual-networks-nsg.md).

## <a name="service-chaining"></a>Chaînage de services

Vous pouvez configurer des itinéraires définis par l’utilisateur qui pointent vers des machines virtuelles de réseaux virtuels homologués en tant qu’adresse IP du « tronçon suivant » pour activer le chaînage de services. Le chaînage de services vous permet de diriger le trafic d’un réseau virtuel vers une appliance virtuelle d’un réseau virtuel homologué via les itinéraires définis.

Vous pouvez également déployer efficacement des environnements de type hub-and-spoke où le nœud peut héberger des composants d’infrastructure tels qu’une appliance virtuelle réseau. Tous les réseaux virtuels spoke peuvent ensuite être homologués avec le réseau virtuel hub. Le trafic peut passer par des appliances virtuelles réseau exécutées sur le réseau virtuel hub. En résumé, l’homologation de réseaux virtuels permet de définir l’adresse IP du tronçon suivant dans l’itinéraire défini par l’utilisateur sur l’adresse IP d’une machine virtuelle du réseau virtuel homologué. Pour en savoir plus sur les itinéraires définis par l’utilisateur, consultez [Itinéraires définis par l’utilisateur et transfert IP](virtual-networks-udr-overview.md). Pour découvrir comment créer une topologie de réseau de type hub et spoke, consultez [Implement a hub-spoke network topology in Azure](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json#virtual network-peering) (Implémenter une topologie de réseau de type hub et spoke dans Azure).

## <a name="gateways-and-on-premises-connectivity"></a>Passerelles et connectivité locale

Chaque réseau virtuel, qu’il soit homologué ou non avec un autre réseau virtuel, peut posséder sa propre passerelle et l’utiliser pour se connecter à un réseau local. Vous pouvez également configurer des [connexions de réseau virtuel à réseau virtuel](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md) à l’aide de passerelles, même si les réseaux virtuels sont homologués.

Lorsque les deux options d’interconnexion de réseaux virtuels sont configurées, le trafic entre les réseaux virtuels transite via la configuration d’homologation (c’est-à-dire via le réseau principal Azure).

Lorsque des réseaux virtuels sont homologués dans la même région, vous pouvez également configurer la passerelle du réseau virtuel homologué en tant que point de transit vers un réseau local. Dans ce cas, le réseau virtuel qui utilise une passerelle distante ne peut pas posséder sa propre passerelle. Un réseau virtuel ne peut posséder qu’une seule passerelle. Il peut s’agir d’une passerelle locale ou distante (dans le réseau virtuel homologué), comme l’illustre l’image suivante :

![Transit d’homologation de réseaux virtuels](./media/virtual-networks-peering-overview/figure04.png)

Le transit de la passerelle n’est pas pris en charge dans la relation d’homologation entre des réseaux virtuels créés via des modèles de déploiement différents ou des régions différentes. Les deux réseaux virtuels dans la relation d’homologation doivent avoir été créés via Resource Manager et doivent être dans la même région pour qu’un transit de la passerelle fonctionne. Les réseaux virtuels homologués à l’échelle mondiale ne prennent actuellement pas en charge le transit de passerelle.

Lorsque des réseaux virtuels qui partagent une même connexion Azure ExpressRoute sont homologués, le trafic entre eux transite via la relation d’homologation (c’est-à-dire via le réseau principal Azure). Vous pouvez toujours utiliser des passerelles locales dans chaque réseau virtuel pour vous connecter au circuit local. Vous pouvez également utiliser une passerelle partagée et configurer le transit pour la connectivité locale.

## <a name="permissions"></a>Autorisations

L’homologation de réseaux virtuels est une opération nécessitant des privilèges. Il s’agit d’une fonction distincte sous l’espace de noms VirtualNetworks. Un utilisateur peut se voir accorder des droits spécifiques pour autoriser l’homologation. Un utilisateur qui dispose d’un accès en lecture-écriture au réseau virtuel hérite automatiquement de ces droits.

Un utilisateur administrateur ou disposant des privilèges d’homologation peut lancer une opération d’homologation sur un autre réseau virtuel. Le niveau d’autorisation minimum nécessaire est Contributeur de réseaux. S’il existe une demande d’homologation correspondante de l’autre côté et si les autres exigences sont remplies, l’homologation est établie.

Par exemple, si vous homologuez des réseaux virtuels nommés myVirtualNetworkA et myVirtualNetworkB, votre compte doit avoir le rôle ou les autorisations minimum suivants pour chaque réseau virtuel :

|Réseau virtuel|Modèle de déploiement|Rôle|Autorisations|
|---|---|---|---|
|myVirtualNetworkA|Gestionnaire de ressources|[Collaborateur de réseau](../active-directory/role-based-access-built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write|
| |Classique|[Collaborateur de réseau classique](../active-directory/role-based-access-built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|N/A|
|myVirtualNetworkB|Gestionnaire de ressources|[Collaborateur de réseau](../active-directory/role-based-access-built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/peer|
||Classique|[Collaborateur de réseau classique](../active-directory/role-based-access-built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|Microsoft.ClassicNetwork/virtualNetworks/peer|

## <a name="monitor"></a>Surveiller

Lors de l’homologation de deux réseaux virtuels créés via le Gestionnaire de ressources, une homologation doit être configurée pour chaque réseau virtuel dans l’homologation.
Vous ne pouvez pas surveiller le statut de votre connexion d’homologation. Les valeurs possibles pour l’état de l’homologation sont les suivantes :

* **Initiée** : Lorsque vous créez l’homologation au deuxième réseau virtuel à partir du premier réseau virtuel, l’état d’homologation est Initiée.

* **Connectée** : Lorsque vous créez l’homologation à partir du deuxième réseau virtuel au premier réseau virtuel, l’état d’homologation est Connectée. Si vous affichez l’état d’homologation pour le premier réseau virtuel, vous voyez que son état est passé de Initiée à Connectée. L’homologation n’est pas correctement établie tant que l’état d’homologation pour les deux homologations de réseau virtuel est Connectée.

* **Déconnecté** : Si un de vos liens d’homologation est supprimé après qu’une connexion a été établie, l’état d’homologation est Déconnectée.

## <a name="troubleshoot"></a>Résolution des problèmes

Pour résoudre les problèmes liés aux flux de trafic sur votre connexion d’homologation, vous pouvez [vérifier les routages effectifs](virtual-network-routes-troubleshoot-portal.md).

Vous pouvez aussi résoudre les problèmes de connectivité à une machine virtuelle d’un réseau virtuel homologué à l’aide de la [vérification de la connectivité](../network-watcher/network-watcher-connectivity-portal.md) de Network Watcher. La vérification de la connectivité affiche le routage directement depuis l’interface réseau de la machine virtuelle source jusqu’à l’interface réseau de la machine virtuelle de destination.

## <a name="limits"></a>limites

Le nombre d’homologations autorisées pour un même réseau virtuel est limité. Pour plus d’informations, consultez [Abonnement Azure et limites, quotas et contraintes de service](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

## <a name="pricing"></a>Tarification

Un coût nominal s’applique pour le trafic entrant et sortant qui utilise une connexion d’homologation de réseau virtuel. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/virtual-network).

## <a name="next-steps"></a>Étapes suivantes

* Suivez un didacticiel d’homologation de réseaux virtuels. Une homologation est générée entre des réseaux virtuels créés via des modèles de déploiement identiques ou différents qui existent dans des abonnements identiques ou différents. Suivez un didacticiel pour l’un des scénarios suivants :

    |Modèle de déploiement Azure  | Abonnement  |
    |---------|---------|
    |Les deux modèles Resource Manager |[Identique](virtual-network-create-peering.md)|
    | |[Différent](create-peering-different-subscriptions.md)|
    |Un modèle Resource Manager, un modèle classique     |[Identique](create-peering-different-deployment-models.md)|
    | |[Différent](create-peering-different-deployment-models-subscriptions.md)|

* Découvrez comment créer une [topologie de réseau Hub and Spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json#virtual network-peering)
* En savoir plus sur tous les [paramètres d’homologation de réseaux virtuels et sur la façon de les modifier](virtual-network-manage-peering.md).
