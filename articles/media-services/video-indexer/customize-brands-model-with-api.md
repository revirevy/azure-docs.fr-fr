---
title: Azure Video Indexer permet de personnaliser le modèle de marques
titleSuffix: Azure Media Services
description: Cet article explique comment utiliser Azure Video Indexer pour personnaliser un modèle de marques.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 01/14/2020
ms.author: anzaman
ms.openlocfilehash: 81ba4cc7be5f9361d21aaea2ba78d0fd6f0f8c95
ms.sourcegitcommit: 7221918fbe5385ceccf39dff9dd5a3817a0bd807
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/21/2020
ms.locfileid: "76289915"
---
# <a name="customize-a-brands-model-with-the-video-indexer-api"></a>Personnaliser un modèle de marques avec l’API Video Indexer

Video Indexer prend en charge la détection de marques dans les messages vocaux et visuels lors de l’indexation et de la réindexation de contenu vidéo et audio. La fonctionnalité de détection de marques identifie les noms de produits, services et entreprises suggérés par la base de données de marques Bing. Par exemple, si le nom de Microsoft est mentionné dans du contenu vidéo ou audio, ou s’il apparaît sous forme de texte visuel dans une vidéo, Video Indexer le détecte et l’interprète comme un nom de marque dans le contenu. Un modèle de marques personnalisé permet d’exclure certaines marques de la détection et d’inclure les marques devant faire partie de votre modèle qui peuvent ne pas se trouver dans la base de données de marques de Bing.

Pour un aperçu détaillé, consultez [Vue d’ensemble](customize-brands-model-overview.md).

Vous pouvez utiliser les API Video Indexer pour créer, utiliser et modifier des modèles personnalisés de marques détectées dans une vidéo, comme décrit dans cette rubrique. Vous pouvez également utiliser le site web Video Indexer, comme décrit dans [Personnaliser un modèle de marques avec le site web Video Indexer](customize-brands-model-with-api.md).

## <a name="create-a-brand"></a>Créer une marque

Le [créer une marque](https://api-portal.videoindexer.ai/docs/services/operations/operations/Create-Brand) API crée une marque personnalisée et l’ajoute au modèle de marques personnalisées pour le compte spécifié. 

> [!NOTE]
> Si vous affectez la valeur true à **activée** (dans le corps), la recherche est placée dans la *inclure* la liste afin que Video Indexer la détecte. La définition du paramètre **enabled** sur false place la marque dans la liste *Exclude* afin que Video Indexer ne la détecte pas.

D’autres paramètres que vous pouvez définir dans le corps :

* La valeur **referenceUrl** peut correspondre à n’importe quel site web de référence de la marque, comme un lien vers sa page Wikipédia.
* La valeur **tags** est une liste d’étiquettes pour la marque. Elle apparaît dans le champ *Category* (Catégorie) de la marque sur le site web Video Indexer. Par exemple, la marque « Azure » peut être étiquetée ou classée dans la catégorie « Cloud ».

### <a name="response"></a>response

La réponse fournit des informations sur la marque que vous venez de créer au format de l’exemple ci-dessous.

```json
{
  "referenceUrl": "https://en.wikipedia.org/wiki/Example",
  "id": 97974,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "created": "2018-04-25T14:59:52.7433333",
  "lastModified": "2018-04-25T14:59:52.7433333",
  "enabled": true,
  "description": "This is an example",
  "tags": [
    "Tag1",
    "Tag2"
  ]
}
```

## <a name="delete-a-brand"></a>Supprimer une marque

L’ [supprimer une marque](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Brand?) API supprime une marque du modèle de marques personnalisées pour le compte spécifié. Le compte est spécifié dans le paramètre **accountId**. Une fois le paramètre appelé, la marque disparaît des listes *Include* ou *Exclude*.

### <a name="response"></a>response

Aucun contenu n’est retourné en cas de suppression effective de la marque.

## <a name="get-a-specific-brand"></a>Obtenir une marque spécifique

Les[obtiennent une marque](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Brand?) API vous permet de rechercher les détails d’une marque dans le modèle de marques personnalisées pour le compte spécifié à l’aide de l’identifiant de marque.

### <a name="response"></a>response

La réponse fournit des informations sur la marque que vous avez recherchée (à l’aide de l’ID de marque) au format de l’exemple ci-dessous.

```json
{
  "referenceUrl": "https://en.wikipedia.org/wiki/Example",
  "id": 128846,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "created": "2018-01-06T13:51:38.3666667",
  "lastModified": "2018-01-11T13:51:38.3666667",
  "enabled": true,
  "description": "This is an example",
  "tags": [
    "Tag1",
    "Tag2"
  ]
}
```

> [!NOTE]
> Si **enabled** est défini sur **true**, cela signifie que la marque se trouve dans la liste *Include* et que Video Indexer la détectera, et si **enabled** est défini sur false, cela signifie que la marque figure dans la liste *Exclude* et que Video Indexer ne la détectera pas.

## <a name="update-a-specific-brand"></a>Mettre à jour une marque spécifique

Les[mettent à jour une marque](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Brand?) API vous permet de rechercher les détails d’une marque dans le modèle de marques personnalisées pour le compte spécifié à l’aide de l’identifiant de marque.

### <a name="response"></a>response

La réponse fournit les informations actualisées sur la marque mise à jour au format de l’exemple ci-dessous.

```json
{
  "referenceUrl": null,
  "id": 97974,
  "name": "Example",
  "accountId": "SampleAccountId",
  "lastModifierUserName": "SampleUserName",
  "Created": "2018-04-25T14:59:52.7433333",
  "lastModified": "2018-04-25T15:37:50.67",
  "enabled": false,
  "description": "This is an update example",
  "tags": [
    "Tag1",
    "NewTag2"
  ]
}
```

## <a name="get-all-of-the-brands"></a>Obtenir toutes les marques

Les [obtiennent toutes les marques](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Brands?) API retourne toutes les marques dans le modèle de marques personnalisées pour le compte spécifié, que la marque soit ou non dans la liste de marques à *inclure* ou à *exclure*.

### <a name="response"></a>response

La réponse renvoie une liste de toutes les marques de votre compte et les détails qui les concernent, au format de l’exemple ci-dessous.

```json
[
    {
        "ReferenceUrl": null,
        "id": 97974,
        "name": "Example",
        "accountId": "AccountId",
        "lastModifierUserName": "UserName",
        "Created": "2018-04-25T14:59:52.7433333",
        "LastModified": "2018-04-25T14:59:52.7433333",
        "enabled": true,
        "description": "This is an example",
        "tags": ["Tag1", "Tag2"]
    },
    {
        "ReferenceUrl": null,
        "id": 97975,
        "name": "Example2",
        "accountId": "AccountId",
        "lastModifierUserName": "UserName",
        "Created": "2018-04-26T14:59:52.7433333",
        "LastModified": "2018-04-26T14:59:52.7433333",
        "enabled": false,
        "description": "This is another example",
        "tags": ["Tag1", "Tag2"]
    },
]
```

> [!NOTE]
> La marque *Example* se trouve dans la liste *Include*, indiquant à Video Indexer de la détecter, et de la marque *Example2* figure dans la liste *Exclude*, indiquant à Video Indexer de ne pas la détecter.

## <a name="get-brands-model-settings"></a>Obtenir les paramètres du modèle de marques

Les [ obtiennent des paramètres de marques](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Brands) API retourne les paramètres du modèle de marques dans le compte spécifié. Les paramètres du modèle de marques indiquent si la détection depuis la base de données de marques de Bing est activée ou non. Si les marques de Bing ne sont pas activées, Video Indexer détectera uniquement les marques définies dans le modèle de marques personnalisé du compte spécifié.

### <a name="response"></a>response

La réponse indique si les marques de Bing sont activées au format de l’exemple ci-dessous.

```json
{
  "state": true,
  "useBuiltIn": true
}
```

> [!NOTE]
> Si **useBuiltIn** est défini sur true, les marques de Bing sont activées. Si *useBuiltin* est défini sur false, les marques de Bing sont désactivées. La valeur **state** peut être ignorée, car elle est désormais déconseillée.

## <a name="update-brands-model-settings"></a>Mettre à jour les paramètres du modèle de marques

La [mises à jour de marques](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Brands-Model-Settings?) met à jour les paramètres de modèle de marques dans le compte spécifié. Les paramètres du modèle de marques indiquent si la détection depuis la base de données de marques de Bing est activée ou non. Si les marques de Bing ne sont pas activées, Video Indexer détectera uniquement les marques définies dans le modèle de marques personnalisé du compte spécifié.

L’indicateur **useBuiltIn** défini sur true indique que les marques Bing sont activées. Si *useBuiltin* est défini sur false, les marques de Bing sont désactivées.

### <a name="response"></a>response

Aucun contenu n’est retourné en cas de mise à jour effective des paramètres du modèle de marques.

## <a name="next-steps"></a>Étapes suivantes

[Personnaliser le modèle de marques à l’aide du site web](customize-brands-model-with-website.md)
