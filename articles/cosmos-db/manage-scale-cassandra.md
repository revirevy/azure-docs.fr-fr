---
title: Mise à l’échelle élastique avec l’API Cassandra dans Azure Cosmos DB
description: En savoir plus sur les options disponibles pour mettre à l’échelle un compte d’API Cassandra Azure Cosmos DB et leurs avantages et inconvénients
author: TheovanKraay
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 01/13/2020
ms.author: thvankra
ms.openlocfilehash: e2967a6d12fba2d81dad9de31e7476a027a39d1c
ms.sourcegitcommit: 64def2a06d4004343ec3396e7c600af6af5b12bb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/19/2020
ms.locfileid: "77468828"
---
# <a name="elastically-scale-an-azure-cosmos-db-cassandra-api-account"></a>Mettre à l’échelle de manière élastique un compte d’API Cassandra Azure Cosmos DB

Il existe diverses options pour explorer la nature élastique de l’API d’Azure Cosmos DB pour Cassandra. Pour comprendre comment mettre à l’échelle efficacement dans Azure Cosmos DB, il est important de comprendre comment approvisionner la quantité appropriée d’unités de requête (RU/s) pour tenir compte des exigences de performances de votre système. Pour en savoir plus sur les unités de requête, consultez l’article relatif aux [unités de requête](request-units.md). 

![Les opérations de base de données consomment des unités de requête](./media/request-units/request-units.png)

## <a name="handling-rate-limiting-429-errors"></a>Gestion de la limitation du débit (erreurs 429)

Azure Cosmos DB renvoie des erreurs de débit limité (429) si les clients consomment plus de ressources (RU/s) que la quantité que vous avez approvisionnée. L’API Cassandra dans Azure Cosmos DB traduit ces exceptions en erreurs surchargées sur le protocole natif Cassandra. 

Si votre système n’est pas sensible à la latence, il peut suffire de gérer la limitation de débit à l’aide de nouvelles tentatives. Consultez l’[exemple de code Java](https://github.com/Azure-Samples/azure-cosmos-cassandra-java-retry-sample) pour savoir comment gérer la limitation de débit en toute transparence à l’aide de l’[extension Azure Cosmos DB](https://github.com/Azure/azure-cosmos-cassandra-extensions) pour la [stratégie de nouvelle tentative Cassandra](https://docs.datastax.com/en/developer/java-driver/4.4/manual/core/retries/) dans Java. Vous pouvez également utiliser l’[extension Spark](https://mvnrepository.com/artifact/com.microsoft.azure.cosmosdb/azure-cosmos-cassandra-spark-helper) pour gérer la limitation du débit.

## <a name="manage-scaling"></a>Gérer la mise à l’échelle

Si vous avez besoin de réduire la latence, il existe un éventail d’options pour gérer la mise à l’échelle et approvisionner le débit (RUs) dans l’API Cassandra :

* [Manuellement à l’aide du Portail Azure](#use-azure-portal)
* [Par programmation à l’aide des fonctionnalités du plan de contrôle](#use-control-plane)
* [Par programmation à l’aide de commandes CQL avec un Kit de développement logiciel (SDK) spécifique](#use-cql-queries)
* [Dynamiquement à l’aide d’Autopilot](#use-autopilot)

Les sections suivantes expliquent les avantages et les inconvénients de chaque approche. Vous pouvez ensuite décider de la meilleure stratégie pour équilibrer les besoins de mise à l’échelle de votre système, le coût global et les besoins d’efficacité de votre solution.

## <a id="use-azure-portal"></a>Utiliser le Portail Azure

Vous pouvez mettre à l’échelle les ressources dans un compte d’API Cassandra Azure Cosmos DB à l’aide du Portail Azure. Pour plus d’informations, consultez l’article [Approvisionner le débit sur les conteneurs et les bases de données](set-throughput.md). Cet article explique les avantages relatifs du paramétrage du débit au niveau de la [base de données](set-throughput.md#set-throughput-on-a-database) ou du [conteneur](set-throughput.md#set-throughput-on-a-container) dans le Portail Azure. Les termes « base de données » et « conteneur » mentionnés dans ces articles sont mappés respectivement à « keyspace » et « table » pour l’API Cassandra.

L’avantage de cette méthode est qu’il s’agit d’une méthode clé en main simple pour gérer la capacité de débit sur la base de données. Toutefois, l’inconvénient est que, dans de nombreux cas, votre approche de mise à l’échelle peut exiger que certains niveaux d’automatisation soient à la fois rentables et élevés. Les sections suivantes expliquent les scénarios et les méthodes appropriés.

## <a id="use-control-plane"></a>Utiliser le plan de contrôle

L’API d’Azure Cosmos DB pour Cassandra offre la possibilité d’ajuster le débit par programmation à l’aide de nos diverses fonctionnalités de plan de contrôle. Pour obtenir des conseils et des exemples, consultez les articles relatifs à [Azure Resource Manager](manage-cassandra-with-resource-manager.md), [PowerShell](powershell-samples-cassandra.md) et [Azure CLI](cli-samples-cassandra.md).

L’avantage de cette méthode est que vous pouvez automatiser l’augmentation ou la diminution des ressources en fonction d’une minuterie pour tenir compte des pics d’activité ou des périodes de faible activité. Jetez un coup d’œil à notre exemple [ici](https://github.com/Azure-Samples/azure-cosmos-throughput-scheduler) pour savoir comment y parvenir à l’aide d’Azure Functions et de PowerShell.

L’inconvénient de cette approche est que vous ne pouvez pas répondre à des besoins de mise à l’échelle fluctuants imprévisibles en temps réel. Au lieu de cela, vous devrez peut-être tirer parti du contexte de l’application dans votre système, au niveau du client ou du Kit de développement logiciel (SDK), ou à l’aide d’[Autopilot](provision-throughput-autopilot.md).

## <a id="use-cql-queries"></a>Utiliser des requêtes CQL avec un Kit de développement logiciel (SDK) spécifique

Vous pouvez mettre le système à l’échelle dynamiquement avec du code en exécutant les [commandes CQL ALTER](cassandra-support.md#keyspace-and-table-options) pour la base de données ou le conteneur donné.

L’avantage de cette approche est qu’elle vous permet de répondre aux besoins de mise à l’échelle de façon dynamique et de manière personnalisée et adaptée à votre application. Avec cette approche, vous pouvez toujours tirer parti des frais et tarifs de RU/s Standard. Si les besoins de mise à l’échelle de votre système sont prévisibles pour la plupart (environ 70 % ou plus), l’utilisation du Kit de développement logiciel (SDK) avec CQL peut être une méthode de mise à l’échelle automatique plus économique que l’utilisation d’Autopilot. L’inconvénient de cette approche est qu’elle peut être assez complexe d’implémenter de nouvelles tentatives alors que la limitation du débit peut augmenter la latence.

## <a id="use-autopilot"></a>Utiliser Autopilot

En plus de la façon manuelle ou par programmation d’approvisionner le débit, vous pouvez également configurer des conteneurs Azure Cosmos en mode Autopilot. Le mode Autopilot s’adapte automatiquement et instantanément à vos besoins en matière de consommation dans les limites RU spécifiées, sans compromettre les SLA. Pour en savoir plus, consultez l’article [Créer des conteneurs et des bases de données Azure Cosmos en mode Autopilot](provision-throughput-autopilot.md).

L’avantage de cette approche est qu’il s’agit du moyen le plus simple de gérer les besoins de mise à l’échelle dans votre système. Elle garantit de ne pas appliquer de limitation du débit **dans les plages RU configurées**. L’inconvénient est que, si les besoins en matière de mise à l’échelle dans votre système sont prévisibles, Autopilot peut être un moyen plus onéreux de gérer vos besoins de mise à l’échelle que d’utiliser les approches du plan de contrôle ou du Kit de développement logiciel (SDK) mentionnées ci-dessus.

## <a name="next-steps"></a>Étapes suivantes

- Bien démarrer avec la [création d’un compte d’API Cassandra, d’une base de données et d’une table](create-cassandra-api-account-java.md) à l’aide d’une application Java
