---
title: "Gestion d’un espace de travail Machine Learning | Microsoft Docs"
description: "Gestion de l&quot;accès aux espaces de travail Azure Machine Learning, et déploiement et gestion des services web d&quot;API ML"
services: machine-learning
documentationcenter: 
author: garyericson
manager: jhubbard
editor: cgronlun
ms.assetid: daf3d413-7a77-4beb-9a7a-6b4bdf717719
ms.service: machine-learning
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/05/2016
ms.author: garye
translationtype: Human Translation
ms.sourcegitcommit: c5479dd817e8929ff5e9129d4643d49758e7ab16
ms.openlocfilehash: 32e74821309507fed4a20b005dbfbba12ffb0703


---
# <a name="manage-an-azure-machine-learning-workspace"></a>Gestion d'un espace de travail Azure Machine Learning
> [!NOTE]
> Les procédures décrites dans cet article s’appliquent aux services web Azure Machine Learning classiques. Pour plus d’informations sur la gestion des services web dans le portail des services web Azure Machine Learning, voir [Gérer un service web à l’aide du portail des services web Azure Machine Learning](machine-learning-manage-new-webservice.md).
> 
> 

À l’aide du portail Azure Classic, vous pouvez gérer vos espaces de travail Machine Learning pour :

* surveiller l’utilisation de l’espace de travail
* configurer l’espace de travail pour autoriser ou refuser l’accès
* Gérer les services web créés dans l’espace de travail
* supprimer l’espace de travail

[!INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]

De plus, l’onglet du tableau de bord affiche l’utilisation de l’espace de travail, ainsi que des informations sur ce dernier en un clin d’œil.  

> [!TIP]
> Dans Azure Machine Learning Studio, dans l’onglet **SERVICES WEB**, vous pouvez ajouter, mettre à jour ou supprimer un service web Machine Learning.
> 
> 

Pour gérer un espace de travail :

1. Connectez-vous au [portail Azure Classic](https://manage.windowsazure.com/) à l’aide de votre compte Microsoft Azure (utilisez le compte associé à l’abonnement Azure).
2. Dans le volet des services Microsoft Azure, cliquez sur **MACHINE LEARNING**.
3. Cliquez sur l’espace de travail que vous souhaitez gérer.

La page de l’espace de travail comporte trois onglets :

* **TABLEAU DE BORD** : vous permet d’afficher l’utilisation et les informations de l’espace de travail
* **CONFIGURATION** : vous permet de gérer l’accès à l’espace de travail
* **SERVICES WEB** : vous permet de gérer les services web publiés à partir de cet espace de travail

## <a name="to-monitor-how-the-workspace-is-being-used"></a>Pour surveiller l’utilisation de l’espace de travail
Cliquez sur l’onglet **TABLEAU DE BORD** .

Dans le tableau de bord, vous pouvez afficher l’utilisation globale de votre espace de travail et obtenir un aperçu rapide des informations de l’espace de travail.

* Le graphique **CALCUL** affiche les ressources de calcul utilisées par l’espace de travail. Vous pouvez modifier l’affichage pour afficher les valeurs absolues ou relatives et vous pouvez modifier la plage de temps affichée dans le graphique.
* **Présentation de l’utilisation** affiche le stockage Azure utilisé par l’espace de travail.
* **Aperçu rapide** fournit un résumé des informations de l’espace de travail ainsi que des liens utiles.

> [!NOTE]
> Le lien **Se connecter à ML Studio** ouvre Machine Learning Studio via le compte Microsoft auquel vous êtes actuellement connecté. Le compte Microsoft que vous utilisez pour vous connecter au portail Azure Classic afin de créer l’espace de travail n’est pas automatiquement autorisé à ouvrir cet espace de travail. Pour ouvrir un espace de travail, vous devez être connecté au compte Microsoft qui a été défini comme propriétaire de l’espace de travail ou recevoir une invitation du propriétaire à rejoindre l’espace de travail.
> 
> 

## <a name="to-grant-or-suspend-access-for-users"></a>Pour octroyer ou suspendre un accès pour les utilisateurs
Cliquez sur l’onglet **Configurer** .

Sous l’onglet Configuration, vous pouvez :

* suspendre l’accès à l’espace de travail Machine Learning en cliquant sur REFUSER. Les utilisateurs ne seront plus en mesure d’ouvrir l’espace de travail de Machine Learning Studio. Pour restaurer l’accès, cliquez sur AUTORISER.

Pour gérer des comptes supplémentaires qui ont accès à l’espace de travail dans Machine Learning Studio, cliquez sur **Se connecter à ML Studio** sous l’onglet **TABLEAU DE BORD** (voir la remarque précédente concernant **Se connecter à ML Studio**). Cela ouvre l’espace de travail de Machine Learning Studio. À ce stade, cliquez sur l’onglet **PARAMÈTRES**, puis sur **UTILISATEURS**. Vous pouvez cliquer sur **INVITER PLUS D’UTILISATEURS** pour octroyer à des utilisateurs un accès à l’espace de travail, ou sélectionner un utilisateur et cliquer sur **SUPPRIMER**.

## <a name="to-manage-web-services-in-this-workspace"></a>Pour gérer des services Web dans cet espace de travail
Cliquez sur l’onglet **SERVICES WEB** .

Cela affiche une liste des services Web publiés à partir de cet espace de travail.
Pour gérer un service web, cliquez sur le nom dans la liste pour ouvrir la page du service web.

Un service web peut avoir un ou plusieurs points de terminaison.

* Vous pouvez définir des points de terminaison supplémentaires, en plus du point de terminaison « Par défaut ». Pour ajouter le point de terminaison, cliquez sur **Gérer les points de terminaison** en bas du tableau de bord pour ouvrir le portail des services web Azure Machine Learning.
* Pour supprimer un point de terminaison (vous ne pouvez pas supprimer le point de terminaison « Par défaut »), cliquez sur la case à cocher au début de la ligne du point de terminaison, puis sur **SUPPRIMER**. Cela supprime le point de terminaison du service web.
  
  > [!NOTE]
  > Si une application utilise le point de terminaison de service Web lors de la suppression de ce dernier, elle obtiendra une erreur la prochaine fois qu’elle essaiera d’accéder au service.
  > 
  > 

Cliquez sur le nom d’un point de terminaison de service web pour l’ouvrir. 

Le tableau de bord vous permet de voir l’utilisation globale de votre service web sur une période donnée. Vous pouvez sélectionner la période à afficher dans le menu déroulant Période situé dans le coin supérieur droit des graphiques d’utilisation. Le tableau de bord affiche les informations suivantes :

* **Requests Over Time** (Requêtes sur une période donnée) affiche un graphique détaillé du nombre de requêtes au cours de la période sélectionnée. Ce graphique vous permet d’identifier les pics d’utilisation.
* **Request-Response Requests** (Demandes de requête-réponse) affiche le nombre total d’appels de requête-réponse que le service a reçu au cours de la période sélectionnée et combien d’entre eux ont échoué.
* **Average Request-Response Compute Time** (Temps de calcul moyen d’une requête-réponse) affiche le temps moyen nécessaire pour exécuter les demandes reçues.
* **Batch Requests** (Requêtes de lots) affiche le nombre total de requêtes de traitement par lots que le service a reçu au cours de la période sélectionnée et combien d’entre elles ont échoué.
* **Average Job Latency** (Latence de travail moyenne) affiche le temps moyen nécessaire pour exécuter les requêtes reçues.
* **Errors** (Erreurs) affiche le nombre agrégé d’erreurs qui se sont produites lors des appels au service web.
* **Services Costs** (Coûts de service) affiche les frais du plan de facturation associé au service.

Dans la page Configurer, vous pouvez mettre à jour les propriétés suivantes :

* **Description** pour entrer une description du service web. Le champ Description est obligatoire.
* **Journalisation** pour activer ou désactiver la journalisation des erreurs sur le point de terminaison. Pour plus d’informations sur la journalisation, voir [Activation de la journalisation pour les services web Machine Learning](machine-learning-web-services-logging.md).
* **Activer l’exemple de données** pour fournir des exemples de données que vous pouvez utiliser pour tester votre service de demande-réponse. Si vous avez créé le service web dans Machine Learning Studio, les exemples de données proviennent des données utilisées dans votre modèle. Si vous avez créé le service par programme, les données proviennent des exemples de données que vous avez fournis dans le cadre du package JSON.




<!--HONumber=Jan17_HO1-->


