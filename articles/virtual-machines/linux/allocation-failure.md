---
title: "Résolution des problèmes d’allocation de machines virtuelles Linux | Microsoft Docs"
description: "Résoudre les problèmes d’allocation pendant la création, le redémarrage ou le redimensionnement d’une machine virtuelle Linux dans Azure"
services: virtual-machines-linux, azure-resource-manager
documentationcenter: 
author: JiangChen79
manager: felixwu
editor: 
tags: top-support-issue,azure-resourece-manager,azure-service-management
ms.assetid: 1ef41144-6dd6-4a56-b180-9d8b3d05eae7
ms.service: virtual-machines-linux
ms.workload: na
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: troubleshooting
ms.date: 11/03/2016
ms.author: cjiang
ms.openlocfilehash: 8cf421afe9b41af050152dc965d42e7809b5f2d9
ms.sourcegitcommit: 3df3fcec9ac9e56a3f5282f6c65e5a9bc1b5ba22
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2017
---
# <a name="troubleshoot-allocation-failures-when-you-create-restart-or-resize-linux-vms-in-azure"></a>Résoudre les problèmes d’allocation pendant la création, le redémarrage ou le redimensionnement de machines virtuelles Linux dans Azure
Quand vous créez une machine virtuelle, redémarrez des machines virtuelles ayant été arrêtées (désallouées) ou redimensionnez une machine virtuelle, Microsoft Azure alloue des ressources de calcul à votre abonnement. Vous pouvez parfois recevoir des erreurs lorsque vous effectuez ces opérations, avant même d’avoir atteint les limites de votre abonnement Azure. Cet article explique les causes de certains échecs d’allocation courants et propose des solutions possibles. Les informations qu’il contient peuvent également vous être utiles dans le cadre de la planification du déploiement de vos services. Vous pouvez également [résoudre les problèmes d’allocation pendant la création, le redémarrage ou le redimensionnement de machines virtuelles Linux dans Azure](../windows/allocation-failure.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

[!INCLUDE [virtual-machines-common-allocation-failure](../../../includes/virtual-machines-common-allocation-failure.md)]

