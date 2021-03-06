---
title: Comment intégrer le schéma d’alerte courant à Logic Apps
description: Découvrez comment créer une application logique qui s’appuie sur le schéma d’alerte courant pour gérer toutes vos alertes.
ms.topic: conceptual
ms.subservice: alerts
ms.date: 05/27/2019
ms.openlocfilehash: 9042ed8ddbb698192e638fa7538f74561574c262
ms.sourcegitcommit: 747a20b40b12755faa0a69f0c373bd79349f39e3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/27/2020
ms.locfileid: "77668228"
---
# <a name="how-to-integrate-the-common-alert-schema-with-logic-apps"></a>Comment intégrer le schéma d’alerte courant à Logic Apps

Cet article explique comment créer une application logique qui s’appuie sur le schéma d’alerte courant pour gérer toutes vos alertes.

## <a name="overview"></a>Vue d’ensemble

Le [schéma d'alerte courant](https://aka.ms/commonAlertSchemaDocs) fournit un schéma JSON standardisé et extensible pour tous vos différents types d'alerte. Le schéma d'alerte courant est plus utile lorsqu'il est exploité par programmation, via des webhooks, des runbooks et des applications logiques. Dans cet article, nous montrons comment une simple application logique peut être créée pour gérer toutes vos alertes. Les mêmes principes peuvent être appliqués à d'autres méthodes programmatiques. L'application logique décrite dans cet article crée des variables bien définies pour les champs [« essentiels »](alerts-common-schema-definitions.md#essentials), et comment gérer une logique spécifique de type [alerte](alerts-common-schema-definitions.md#alert-context).


## <a name="prerequisites"></a>Prérequis 

Pour suivre cet article, le lecteur doit être familiarisé avec les concepts suivants : 
* Configuration des règles d’alerte ([métrique](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-metric), [journal](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-log), [journal d’activité](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log))
* Configuration de [groupes d’actions](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups)
* Activation du [schéma d’alerte courant](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-common-schema#how-do-i-enable-the-common-alert-schema) depuis des groupes d’actions

## <a name="create-a-logic-app-leveraging-the-common-alert-schema"></a>Créer une application logique reposant sur le schéma d’alerte courant

1. Appliquez les [procédures suivantes pour créer votre application logique](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups-logic-app). 

1.  Sélectionnez le déclencheur : **Lors de la réception d’une requête HTTP**.

    ![Déclencheurs d’application logique](media/action-groups-logic-app/logic-app-triggers.png "Déclencheurs d’application logique")

1.  Sélectionnez **Modifier** pour changer de déclencheur de requête HTTP.

    ![Déclencheurs de requêtes HTTP](media/action-groups-logic-app/http-request-trigger-shape.png "Déclencheurs de requêtes HTTP")


1.  Copiez et collez le schéma suivant :

    ```json
        {
            "type": "object",
            "properties": {
                "schemaId": {
                    "type": "string"
                },
                "data": {
                    "type": "object",
                    "properties": {
                        "essentials": {
                            "type": "object",
                            "properties": {
                                "alertId": {
                                    "type": "string"
                                },
                                "alertRule": {
                                    "type": "string"
                                },
                                "severity": {
                                    "type": "string"
                                },
                                "signalType": {
                                    "type": "string"
                                },
                                "monitorCondition": {
                                    "type": "string"
                                },
                                "monitoringService": {
                                    "type": "string"
                                },
                                "alertTargetIDs": {
                                    "type": "array",
                                    "items": {
                                        "type": "string"
                                    }
                                },
                                "originAlertId": {
                                    "type": "string"
                                },
                                "firedDateTime": {
                                    "type": "string"
                                },
                                "resolvedDateTime": {
                                    "type": "string"
                                },
                                "description": {
                                    "type": "string"
                                },
                                "essentialsVersion": {
                                    "type": "string"
                                },
                                "alertContextVersion": {
                                    "type": "string"
                                }
                            }
                        },
                        "alertContext": {
                            "type": "object",
                            "properties": {}
                        }
                    }
                }
            }
        }
    ```

1. Sélectionnez **+** **Nouvelle étape**, puis **Ajouter une action**.

    ![Ajouter une action](media/action-groups-logic-app/add-action.png "Ajouter une action")

1. À ce stade, vous pouvez ajouter une variété de connecteurs (Microsoft Teams, Slack, Salesforce, etc.) en fonction des exigences de votre entreprise. Vous pouvez utiliser les « champs essentiels » prêts à l’emploi. 

    ![Champs essentiels](media/alerts-common-schema-integrations/logic-app-essential-fields.png "Champs essentiels")
    
    Vous pouvez également créer une logique conditionnelle basée sur le type d’alerte en utilisant l’option 'Expression'.

    ![Expression d’application logique](media/alerts-common-schema-integrations/logic-app-expressions.png "Expression d’application logique")
    
     Le [champ 'monitoringService'](alerts-common-schema-definitions.md#alert-context) permet d’identifier de manière unique le type d’alerte à partir duquel vous pouvez créer la logique conditionnelle.

    
    Par exemple, l’extrait de code ci-dessous vérifie si l'alerte est une alerte de journal basée sur Application Insights et, le cas échéant, imprime les résultats de la recherche. Sinon, il imprime « NA » (non disponible).

    ```text
      if(equals(triggerBody()?['data']?['essentials']?['monitoringService'],'Application Insights'),triggerBody()?['data']?['alertContext']?['SearchResults'],'NA')
    ```
    
     En savoir plus sur [l’écriture d’expressions d’application logique](https://docs.microsoft.com/azure/logic-apps/workflow-definition-language-functions-reference#logical-comparison-functions).

    


## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur les groupes d’actions](../../azure-monitor/platform/action-groups.md).
* [Découvrez-en plus sur le schéma d’alerte commun](https://aka.ms/commonAlertSchemaDocs).

