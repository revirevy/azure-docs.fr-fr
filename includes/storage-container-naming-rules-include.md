---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: tamram
ms.openlocfilehash: aeca3550b5fc58694779e7e54ce6ca547ba30e17
ms.sourcegitcommit: 3e98da33c41a7bbd724f644ce7dedee169eb5028
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/18/2019
ms.locfileid: "67177678"
---
Chaque objet blob du stockage Azure doit résider dans un conteneur. Le conteneur fait partie du nom d'objet blob. Par exemple, `mycontainer` est le nom du conteneur dans ces exemples d’URI d’objet blob :

    https://storagesample.blob.core.windows.net/mycontainer/blob1.txt
    https://storagesample.blob.core.windows.net/mycontainer/photos/myphoto.jpg

Un conteneur doit posséder un nom DNS valide, conforme aux règles de nommage suivantes :

1. Les noms de conteneur doivent commencer par une lettre ou un chiffre, et peuvent comporter uniquement des lettres, des chiffres et des tirets (-).
2. Chaque tiret (-) doit être immédiatement précédé et suivi d’une lettre ou d’un chiffre ; les tirets consécutifs sont interdits.
3. Toutes les lettres du conteneur doivent être minuscules.
4. Les noms de conteneurs doivent comporter entre 3 et 63 caractères.

> [!IMPORTANT]
> Notez que le nom d'un conteneur doit toujours être en minuscules. Si vous incluez une majuscule dans un nom de conteneur ou si vous ne respectez pas les règles d'affectation de noms, vous pouvez recevoir une erreur 400 (demande incorrecte). 
> 
> 

