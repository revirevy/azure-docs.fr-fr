---
title: Utiliser Shared Image Gallery pour créer un pool personnalisé - Azure Batch | Microsoft Docs
description: Créer un pool Batch avec Shared Image Gallery pour des images personnalisées vers des nœuds de calcul qui contiennent les logiciels et les données dont vous avez besoin pour votre application. Les images personnalisées sont un moyen efficace de configurer les nœuds de calcul pour exécuter vos charges de travail Batch.
services: batch
author: LauraBrenner
manager: evansma
ms.service: batch
ms.topic: article
ms.date: 08/28/2019
ms.author: labrenne
ms.openlocfilehash: 2cff6a0e48fc7bf58a642f509fcda6b114e002ef
ms.sourcegitcommit: 21e33a0f3fda25c91e7670666c601ae3d422fb9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/05/2020
ms.locfileid: "77022934"
---
# <a name="use-the-shared-image-gallery-to-create-a-custom-pool"></a>Utiliser Shared Image Gallery pour créer un pool personnalisé

Quand vous créez un pool Azure Batch à l’aide de Configuration de la machine virtuelle, vous spécifiez une image de machine virtuelle qui fournit le système d’exploitation pour chaque nœud de calcul dans le pool. Vous pouvez créer un pool de machines virtuelles avec une image Place de Marché Azure prise en charge ou créer une image personnalisée avec [Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md).

## <a name="benefits-of-the-shared-image-gallery"></a>Avantages de Shared Image Gallery

Lorsque vous utilisez Shared Image Gallery pour votre image personnalisée, vous contrôlez le type et la configuration du système d’exploitation ainsi que le type de disque de données. Votre image partagée peut inclure des applications et des données de référence qui deviennent disponibles sur tous les nœuds du pool Batch dès qu’ils sont approvisionnés.

Vous pouvez également avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Quand vous utilisez une version d’image pour créer une machine virtuelle, la version d’image permet de créer des disques pour la machine virtuelle.

L’utilisation d’une image partagée réduit le temps de préparation des nœuds de calcul du pool appelés à exécuter votre charge de travail Batch. Il est possible d’utiliser une image de la Place de Marché et d’installer des logiciels sur chaque nœud de calcul après l’approvisionnement, mais l’utilisation d’une image partagée est généralement plus efficace. En outre, vous pouvez spécifier plusieurs réplicas pour l’image partagée. Ainsi, vous gagnerez du temps lors de la création de pools avec de nombreuses machines virtuelles (plus de 600 machines virtuelles).

Le recours à une image partagée pour votre scénario peut offrir plusieurs avantages :

* **Vous pouvez utiliser les mêmes images dans toutes les régions.** Vous pouvez créer des réplicas d’images partagées dans différentes régions afin que tous vos pools utilisent la même image.
* **Configurer le système d’exploitation.** Vous pouvez personnaliser la configuration du disque de système d’exploitation de l’image.
* **Préinstaller des applications.** La préinstallation des applications sur le disque du système d’exploitation est plus efficace et moins sujet aux erreurs que l’installation d’applications après l’approvisionnement des nœuds de calcul à l’aide d’un début de tâche.
* **Copier de grandes quantités de données en une seule fois.** Intégrez les données statiques à l’image partagée managée en les copiant sur les disques de données d’une image managée. Cette opération ne doit être effectuée qu’une seule fois et rend les données accessibles à chaque nœud du pool.
* **Augmentez la taille des pools.** Avec Shared Image Gallery, vous pouvez créer des pools plus grands avec vos images personnalisées ainsi qu’avec d’autres réplicas d’image partagée.
* **Performances supérieures à celles de l'image personnalisée.** À l’aide d’images partagées, le pool atteint jusqu’à 25 % plus rapidement l’état stable et la latence d’inactivité de la machine virtuelle est inférieure à 30 %.
* **Gestion des versions et regroupement d’images pour une gestion simplifiée.** La définition du regroupement d’images contient des informations sur la raison pour laquelle l’image a été créée, le système d’exploitation concerné et l’utilisation de l’image. Le regroupement d’images simplifie la gestion des images. Pour plus d’informations, consultez les [Définitions d’images](../virtual-machines/windows/shared-image-galleries.md#image-definitions).

## <a name="prerequisites"></a>Conditions préalables requises

* **Un compte Azure Batch.** Pour créer un compte Batch, consultez les démarrages rapides Batch à l’aide du [portail Azure](quick-create-portal.md) ou de l’[interface de ligne de commande Azure](quick-create-cli.md).

* **Une image Shared Image Gallery**. Pour créer une image partagée, vous devez disposer d'une image managée ou en créer une. L’image doit être créée à partir d’instantanés du disque de système d’exploitation de la machine virtuelle et, éventuellement, ses disques de données associés. Pour plus d'informations, consultez [Préparer une image managée](#prepare-a-managed-image).

> [!NOTE]
> Votre image partagée doit se trouver dans le même abonnement que le compte Batch. Votre image partagée peut se trouver dans différentes régions, à condition qu’elle ait des réplicas dans la même région que votre compte Batch.

## <a name="prepare-a-managed-image"></a>Préparer une image managée

Dans Azure, vous pouvez préparer une image managée à partir de ce qui suit :

* Captures instantanées du système d'exploitation et des disques de données d'une machine virtuelle Azure
* Machine virtuelle Azure généralisée dotée de disques managés
* Disque dur virtuel local généralisé chargé dans le cloud

Pour mettre à l’échelle des pools Batch de manière fiable avec une image personnalisée, nous vous recommandons de créer une image managée *uniquement* à l’aide de la première méthode: en utilisant des captures instantanées des disques de la machine virtuelle. Consultez les étapes suivantes pour préparer une machine virtuelle, prendre un instantané et créer une image à partir de l’instantané.

### <a name="prepare-a-vm"></a>Préparer une machine virtuelle

Si vous créez une machine virtuelle pour l'image, utilisez une image propriétaire de la Place de Marché Azure prise en charge par Batch comme image de base pour votre image managée. Seules les images propriétaires peuvent être utilisées comme image de base. Pour obtenir la liste complète des références d'image de la Place de marché Azure prises en charge par Azure Batch, consultez l'opération [Lister les références SKU d'agent de nœud](/java/api/com.microsoft.azure.batch.protocol.accounts.listnodeagentskus).

> [!NOTE]
> Vous ne pouvez pas, comme image de base, utiliser une image de fournisseurs tiers qui comporte des conditions de licence et d’achat supplémentaires. Pour plus d’informations sur ces images de la Place de marché, consultez les recommandations émises pour les machines virtuelles [Linux](../virtual-machines/linux/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms
) ou [Windows](../virtual-machines/windows/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms
).

* Assurez-vous que la machine virtuelle est créée avec un disque managé. Il s’agit du paramètre de stockage par défaut quand vous créez une machine virtuelle.
* N’installez pas d’extensions Azure, comme l’extension de script personnalisé, sur la machine virtuelle. Si l’image contient une extension préinstallée, Azure peut rencontrer des problèmes lors du déploiement du pool Batch.
* Lorsque vous utilisez des disques de données attachés, vous devez monter et formater les disques à partir d'une machine virtuelle pour les utiliser.
* Vérifiez que l’image du système d’exploitation de base que vous fournissez utilise le lecteur temporaire par défaut. L’agent de nœud Batch s’attend actuellement à ce que le lecteur temporaire par défaut soit utilisé.
* Une fois que la machine virtuelle s’exécute, connectez-la via le protocole RDP (pour Windows) ou SSH (pour Linux). Le cas échéant, installez les logiciels nécessaires ou copiez les données souhaitées.  

### <a name="create-a-vm-snapshot"></a>Créer un instantané de la machine virtuelle

Une capture instantanée est une copie complète en lecture seule d’un disque dur virtuel. Pour créer un instantané des disques de système d’exploitation ou de données d’une machine virtuelle, vous pouvez utiliser les outils en ligne de commande ou le portail Azure. Pour les étapes et les options permettant de créer un instantané, consultez les conseils pour les machines virtuelles [Linux](../virtual-machines/linux/snapshot-copy-managed-disk.md) ou [Windows](../virtual-machines/windows/snapshot-copy-managed-disk.md).

### <a name="create-an-image-from-one-or-more-snapshots"></a>Créer une image à partir d’un ou plusieurs instantanés

Pour créer une image managée à partir d’un instantané, utilisez les outils en ligne de commande Azure comme la commande [az image création](/cli/azure/image). Créez une image en spécifiant l'instantané d'un disque du système d'exploitation et éventuellement un ou plusieurs instantanés de disque de données.

### <a name="create-a-shared-image-gallery"></a>Créer une galerie Shared Image Gallery

Après avoir créé votre image managée, vous devez créer une galerie Shared Image Gallery pour rendre votre image personnalisée disponible. Pour apprendre à créer une galerie Shared Image Gallery pour vos images, consultez [Créer une galerie Shared Image Gallery avec l'interface de ligne de commande Azure](../virtual-machines/linux/shared-images.md) ou [Créer une galerie Shared Image Gallery à l'aide du portail Azure](../virtual-machines/linux/shared-images-portal.md).

## <a name="create-a-pool-from-a-shared-image-using-the-azure-cli"></a>Créer un pool à partir d’une image partagée à l’aide de l’interface de ligne de commande Azure

Pour créer un pool à partir de votre image partagée à l’aide de l’interface de ligne de commande Azure, utilisez la commande `az batch pool create`. Spécifiez l’ID de l’image partagée dans le champ `--image`. Vérifiez que le type de système d’exploitation et la référence SKU correspondent aux versions spécifiées par `--node-agent-sku-id`

```azurecli
az batch pool create \
    --id mypool --vm-size Standard_A1_v2 \
    --target-dedicated-nodes 2 \
    --image "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}" \
    --node-agent-sku-id "batch.node.ubuntu 16.04"
```

## <a name="create-a-pool-from-a-shared-image-using-c"></a>Créer un pool à partir d’une image partagée à l’aide de C#

Vous pouvez également créer un pool à partir d’une image partagée à l'aide du kit de développement logiciel (SDK) C#.

```csharp
private static VirtualMachineConfiguration CreateVirtualMachineConfiguration(ImageReference imageReference)
{
    return new VirtualMachineConfiguration(
        imageReference: imageReference,
        nodeAgentSkuId: "batch.node.windows amd64");
}

private static ImageReference CreateImageReference()
{
    return new ImageReference(
        virtualMachineImageId: "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}");
}

private static void CreateBatchPool(BatchClient batchClient, VirtualMachineConfiguration vmConfiguration)
{
    try
    {
        CloudPool pool = batchClient.PoolOperations.CreatePool(
            poolId: PoolId,
            targetDedicatedComputeNodes: PoolNodeCount,
            virtualMachineSize: PoolVMSize,
            virtualMachineConfiguration: vmConfiguration);

        pool.Commit();
    }
    ...
}
```

## <a name="create-a-pool-from-a-shared-image-using-the-azure-portal"></a>Créer un pool à partir d’une image partagée à l’aide du portail Azure

Effectuez les étapes suivantes pour créer un pool à partir d’une image partagée dans le portail Azure.

1. Ouvrez le [portail Azure](https://portal.azure.com).
1. Accédez à **Comptes Batch** et sélectionnez votre compte.
1. Sélectionnez **Pools**, puis **Ajouter** pour créer un pool.
1. Dans la section **Type d’image**, sélectionnez **Galerie d’images partagées**.
1. Complétez les sections restantes avec les informations relatives à votre image managée.
1. Sélectionnez **OK**.

![Créez un pool à partir d’une image partagée avec le portail.](media/batch-sig-images/create-custom-pool.png)

## <a name="considerations-for-large-pools"></a>Considérations relatives aux grands pools

Si vous envisagez de créer un pool avec des centaines ou des milliers de machines virtuelles ou plus à l’aide d’une image partagée, suivez les instructions ci-dessous.

* **Nombre de réplicas de la galerie Shared Image Gallery.**  Pour chaque pool avec un maximum de 600 instances, nous vous recommandons de garder au moins un réplica. Par exemple, si vous créez un pool de 3 000 machines virtuelles, vous devez conserver au moins 5 réplicas de votre image. Nous suggérons toujours de conserver plus de réplicas que le minimum exigé pour de meilleures performances.

* **Délai d'expiration du redimensionnement.** Si votre pool contient un nombre de nœuds fixe (pas de mise à l'échelle automatique), augmentez la propriété `resizeTimeout` du pool en fonction de la taille de celui-ci. Le délai d’attente de redimensionnement recommandé est d’au moins 15 minutes pour 1 000 machines. Par exemple, le délai d’expiration de redimensionnement recommandé pour un pool de 2 000 machines virtuelles est d’au moins 30 minutes.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir une présentation détaillée de Batch, consultez [Développer des solutions de calcul parallèles à grande échelle avec Batch](batch-api-basics.md).
