---
title: Ajouter des fonctions CRUD à une application Angular avec l’API d’Azure Cosmos DB pour MongoDB
description: Il s’agit de la partie 6 de cette série de didacticiels sur la création d’une application MongoDB avec Angular et Node sur Azure Cosmos DB à l’aide des mêmes API que celles utilisées pour MongoDB
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18
ms.reviewer: sngun
ms.openlocfilehash: 0c39ffe40a490ee23ac65f892c46fba2578bce74
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/25/2019
ms.locfileid: "75441101"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---add-crud-functions-to-the-app"></a>Créer une application Angular avec l’API d’Azure Cosmos DB pour MongoDB - Ajouter des fonctions CRUD à l’application

Ce tutoriel en plusieurs parties montre comment créer une application écrite en Node.js avec Express et Angular, puis comment la connecter à votre [compte Cosmos configuré avec l’API de Cosmos DB pour MongoDB](mongodb-introduction.md). La partie 6 de ce didacticiel est basée sur la [partie 5](tutorial-develop-mongodb-nodejs-part5.md) et aborde les tâches suivantes :

> [!div class="checklist"]
> * Créer des fonctions Publier, Placer et Supprimer pour le service de héros
> * Exécuter l’application

> [!VIDEO https://www.youtube.com/embed/Y5mdAlFGZjc]

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer cette partie du didacticiel, assurez-vous d’avoir effectué les étapes de la [partie 5](tutorial-develop-mongodb-nodejs-part5.md) du didacticiel.

> [!TIP]
> Ce didacticiel vous guide à travers les étapes pour générer l’application pas à pas. Si vous souhaitez télécharger le projet terminé, vous pouvez obtenir l’application complète à partir du [référentiel cosmosdb-angular](https://github.com/Azure-Samples/angular-cosmosdb) sur GitHub.

## <a name="add-a-post-function-to-the-hero-service"></a>Ajouter une fonction Publier au service de héros

1. Dans Visual Studio Code, ouvrez les fichiers **routes.js** et **hero.service.js** côte à côte en appuyant sur le bouton **Scinder l’éditeur**![Bouton Scinder l’éditeur dans Visual Studio](./media/tutorial-develop-mongodb-nodejs-part6/split-editor-button.png).

    Vous pouvez voir que la ligne 7 du fichier routes.js appelle la fonction `getHeroes` de la ligne 5 du fichier **hero.service.js**.  Nous devons recréer cette association pour les fonctions Publier, Placer et Supprimer. 

    ![Fichiers routes.js et hero.service.js dans Visual Studio Code](./media/tutorial-develop-mongodb-nodejs-part6/routes-heroservicejs.png)
    
    Commençons par coder le service de héros. 

2. Copiez le code suivant dans le fichier **hero.service.js** après la fonction `getHeroes` et avant `module.exports`. Ce code :  
   * Utilise le modèle de héros pour publier un nouveau héros.
   * Vérifie les réponses pour voir s’il existe une erreur et retourne une valeur d’état 500.

   ```javascript
   function postHero(req, res) {
     const originalHero = { uid: req.body.uid, name: req.body.name, saying: req.body.saying };
     const hero = new Hero(originalHero);
     hero.save(error => {
       if (checkServerError(res, error)) return;
       res.status(201).json(hero);
       console.log('Hero created successfully!');
     });
   }

   function checkServerError(res, error) {
     if (error) {
       res.status(500).send(error);
       return error;
     }
   }
   ```

3. Dans le fichier **hero.service.js**, mettez à jour le `module.exports` pour inclure la nouvelle fonction `postHero`. 

    ```javascript
    module.exports = {
      getHeroes,
      postHero
    };
    ```

4. Dans le fichier **routes.js**, ajoutez un routeur pour la fonction `post` après le routeur `get`. Ce routeur publie un héros à la fois. En structurant ainsi le fichier du routeur, tous les points de terminaison API disponibles s’affichent clairement et le fichier **hero.service.js** fait tout le travail.

    ```javascript
    router.post('/hero', (req, res) => {
      heroService.postHero(req, res);
    });
    ```

5. Vérifiez que tout fonctionne en exécutant l’application. Dans Visual Studio Code, enregistrez les modifications, sélectionnez le bouton **Déboguer**![Icône de débogage dans Visual Studio Code](./media/tutorial-develop-mongodb-nodejs-part6/debug-button.png) sur le côté gauche, puis le bouton **Démarrer le débogage**![Icône Démarrer le débogage dans Visual Studio Code](./media/tutorial-develop-mongodb-nodejs-part6/start-debugging-button.png).

6. Retournez sur votre navigateur Internet et ouvrez l’onglet Réseau d’outils développeur (en appuyant sur F12 sur la plupart des ordinateurs). Accédez à [http://localhost:3000](http://localhost:3000) pour voir les appels effectués sur le réseau.

    ![Onglet Mise en réseau dans Chrome qui affiche l’activité du réseau](./media/tutorial-develop-mongodb-nodejs-part6/add-new-hero.png)

7. Ajoutez un nouveau héros en sélectionnant le bouton **Ajouter un nouveau héros**. Entrez l’ID « 999 », le nom « Fred » et le message « Bonjour », puis sélectionnez **Enregistrer**. Dans l’onglet Mise en réseau, vous devez voir que vous avez envoyé une demande PUBLIER pour un nouveau héros. 

    ![Onglet Mise en réseau dans Chrome qui affiche l’activité du réseau des fonctions Obtenir et Publier](./media/tutorial-develop-mongodb-nodejs-part6/post-new-hero.png)

    Revenons en arrière pour ajouter les fonctions Placer et Supprimer à l’application.

## <a name="add-the-put-and-delete-functions"></a>Ajoutez les fonctions Placer et Supprimer

1. Dans le fichier **routes.js**, ajoutez les routeurs `put` et `delete` après le routeur de la fonction Publier.

    ```javascript
    router.put('/hero/:uid', (req, res) => {
      heroService.putHero(req, res);
    });

    router.delete('/hero/:uid', (req, res) => {
      heroService.deleteHero(req, res);
    });
    ```

2. Copiez le code suivant dans le fichier **hero.service.js** après la fonction `checkServerError`. Ce code :
   * Crée les fonctions `put` et `delete`
   * Vérifie si le héros a été trouvé
   * Gère les erreurs 

   ```javascript
   function putHero(req, res) {
     const originalHero = {
       uid: parseInt(req.params.uid, 10),
       name: req.body.name,
       saying: req.body.saying
     };
     Hero.findOne({ uid: originalHero.uid }, (error, hero) => {
       if (checkServerError(res, error)) return;
       if (!checkFound(res, hero)) return;

      hero.name = originalHero.name;
       hero.saying = originalHero.saying;
       hero.save(error => {
         if (checkServerError(res, error)) return;
         res.status(200).json(hero);
         console.log('Hero updated successfully!');
       });
     });
   }

   function deleteHero(req, res) {
     const uid = parseInt(req.params.uid, 10);
     Hero.findOneAndRemove({ uid: uid })
       .then(hero => {
         if (!checkFound(res, hero)) return;
         res.status(200).json(hero);
         console.log('Hero deleted successfully!');
       })
       .catch(error => {
         if (checkServerError(res, error)) return;
       });
   }

   function checkFound(res, hero) {
     if (!hero) {
       res.status(404).send('Hero not found.');
       return;
     }
     return hero;
   }
   ```

3. Dans le fichier **hero.service.js**, exportez les nouveaux modules :

   ```javascript
    module.exports = {
      getHeroes,
      postHero,
      putHero,
      deleteHero
    };
    ```

4. Maintenant que nous avons mis à jour le code, sélectionnez le bouton **Redémarrer**![Bouton Redémarrer dans Visual Studio Code](./media/tutorial-develop-mongodb-nodejs-part6/restart-debugger-button.png) dans Visual Studio Code.

5. Actualisez la page de votre navigateur Internet et sélectionnez le bouton **Ajouter un nouveau héros**. Ajoutez un nouveau héros avec l’ID « 9 », le nom « Starlord », et le message « Salut ». Sélectionnez le bouton **Enregistrer** pour enregistrer le nouveau héros.

6. Sélectionnez maintenant le héros **Starlord** et changez le message « Salut » en « Au revoir », puis sélectionnez le bouton **Enregistrer**. 

    Vous pouvez maintenant sélectionner l’ID dans l’onglet réseau pour afficher la charge utile. Vous pouvez voir dans la charge utile que le message indique maintenant « Au revoir ».

    ![Application de héros et onglet Mise en réseau affichant la charge utile](./media/tutorial-develop-mongodb-nodejs-part6/put-hero-function.png) 

    Vous pouvez également supprimer un héros dans l’interface utilisateur et voir le temps nécessaire à la suppression. Essayez en sélectionnant le bouton « Supprimer » pour le héros nommé « Fred ».

    ![Application de héros et onglet Mise en réseau affichant le temps pour effectuer les fonctions](./media/tutorial-develop-mongodb-nodejs-part6/times.png) 

    Si vous actualisez la page, l’onglet réseau affiche le temps nécessaire pour obtenir les héros. Les durées sont courtes, mais un grand nombre dépend de l’emplacement de vos données dans le monde et de votre capacité à les répliquer dans une zone géographique proche de vos utilisateurs. Vous trouverez plus d’informations sur la géo-réplication prochainement, dans le didacticiel suivant.

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du didacticiel, vous avez :

> [!div class="checklist"]
> * Ajouté les fonctions Publier, Placer et Supprimer à l’application 

Revenez prochainement pour consulter une vidéo de la série.

