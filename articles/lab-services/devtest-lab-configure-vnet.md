---
title: Configuration d’un réseau virtuel dans Azure DevTest Labs | Microsoft Docs
description: Découvrez comment configurer un réseau virtuel et un sous-réseau existants, puis les utiliser dans une machine virtuelle avec Azure DevTest Labs
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 6cda99c2-b87e-4047-90a0-5df10d8e9e14
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/05/2019
ms.author: spelluru
ms.openlocfilehash: 6cf3d2f82c98a3caab47ff48a600316747932b72
ms.sourcegitcommit: 88ae4396fec7ea56011f896a7c7c79af867c90a1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/06/2019
ms.locfileid: "70390047"
---
# <a name="configure-a-virtual-network-in-azure-devtest-labs"></a>Configuration d’un réseau virtuel dans Azure DevTest Labs
Comme expliqué dans l’article [Ajouter une machine virtuelle à un laboratoire](devtest-lab-add-vm.md), quand vous créez une machine virtuelle dans un laboratoire, vous pouvez spécifier un réseau virtuel configuré. Par exemple, vous devez être en mesure d’accéder aux ressources de votre réseau d’entreprise à partir de vos machines virtuelles à l’aide du réseau virtuel configuré avec ExpressRoute ou un VPN de site à site.

Cet article explique comment ajouter votre réseau virtuel existant aux paramètres de réseau virtuel d’un lab afin qu’il puisse être sélectionné lors de la création de machines virtuelles.

> [!NOTE]
> Pour plus d’informations sur les coûts associés au service Réseau virtuel Azure, consultez [Tarification de Réseau virtuel Azure](../virtual-network/virtual-networks-overview.md#pricing).

## <a name="configure-a-virtual-network-for-a-lab-using-the-azure-portal"></a>Configurer un réseau virtuel pour un laboratoire en utilisant le portail Azure
Les étapes suivantes vous montrent comment ajouter un réseau virtuel (et sous-réseau) existant à un laboratoire pour qu’il puisse être utilisé lors de la création d’une machine virtuelle dans le même laboratoire. 

1. Connectez-vous au [Portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
1. Sélectionnez le laboratoire souhaité dans la liste des laboratoires. 
1. Dans le volet principal du lab, sélectionnez **Configuration et stratégies**.

    ![Accéder à la configuration et aux stratégies du lab](./media/devtest-lab-configure-vnet/policies-menu.png)
1. Dans la section **RESSOURCES EXTERNES**, sélectionnez **Réseaux virtuels**. La liste des réseaux virtuels configurés pour le lab actuel s’affiche ainsi que le réseau virtuel par défaut créé pour votre lab. 
1. Sélectionnez **Ajouter**.
   
    ![Ajout d’un réseau virtuel existant à votre labo](./media/devtest-lab-configure-vnet/lab-settings-vnet-add.png)
1. Dans le volet **Réseau virtuel**, sélectionnez **[Sélectionner un réseau virtuel]** .
   
    ![Sélection d’un réseau virtuel existant](./media/devtest-lab-configure-vnet/lab-settings-vnets-vnet1.png)
1. Dans le volet **Choisir un réseau virtuel**, sélectionnez le réseau virtuel souhaité. Une liste s’affiche et répertorie tous les réseaux virtuels figurant sous la même région de l’abonnement que le lab.
1. Après avoir sélectionné un réseau virtuel, vous revenez au volet **Réseau virtuel**. Sélectionnez le sous-réseau dans la liste figurant en bas.

    ![Liste des sous-réseaux](./media/devtest-lab-configure-vnet/lab-settings-vnets-vnet2.png)
    
    Le volet Sous-réseau lab s’affiche.

    ![Volet Sous-réseau lab](./media/devtest-lab-configure-vnet/lab-subnet.png)
     
   - Spécifiez un **nom de sous-réseau de laboratoire**.
   - Pour permettre l’utilisation d’un sous-réseau dans le laboratoire de création de machines virtuelles, sélectionnez **Utiliser dans la création de machines virtuelles**.
   - Pour activer une [adresse IP publique partagée](devtest-lab-shared-ip.md), sélectionnez **Enable shared public IP** (Activer l’adresse IP publique partagée).
   - Pour autoriser les adresses IP publiques dans un sous-réseau, sélectionnez **Allow public IP creation** (Autoriser la création d’adresses IP publiques).
   - Dans le champ **Nombre maximal de machines virtuelles par utilisateur**, spécifiez le nombre maximal de machines virtuelles par utilisateur pour chaque sous-réseau. Si vous souhaitez un nombre illimité de machines virtuelles, laissez ce champ vide.
1. Cliquez sur **OK** pour fermer le volet Sous-réseau lab.
1. Sélectionnez **Enregistrer** pour fermer le volet Réseau virtuel.

Maintenant que le réseau virtuel est configuré, il peut être sélectionné lors de la création d'une machine virtuelle. Pour savoir comment créer une machine virtuelle et spécifier un réseau virtuel, consultez l’article [Ajouter une machine virtuelle à un laboratoire](devtest-lab-add-vm.md). 

La [documentation sur le réseau virtuel](https://docs.microsoft.com/azure/virtual-network) d’Azure fournit plus d’informations sur l’utilisation des réseaux virtuels, notamment sur la configuration et la gestion d’un réseau virtuel et sur sa connexion à votre réseau local.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez ajouté le réseau virtuel souhaité à votre laboratoire, l’étape suivante consiste à [ajouter une machine virtuelle à votre laboratoire](devtest-lab-add-vm.md).

