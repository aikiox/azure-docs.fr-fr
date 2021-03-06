---
title: "Mise à niveau vers Azure SQL Database V12 à l’aide du portail Azure | Microsoft Docs"
description: "Explique comment effectuer une mise à niveau vers Azure SQL Database V12, notamment la mise à niveau des bases de données Web et Business et la mise à niveau d’un serveur V11 migrant ses bases de données directement dans un pool élastique à l’aide du Portail Azure."
services: sql-database
documentationcenter: 
author: stevestein
manager: jhubbard
editor: 
ms.assetid: 81a59ada-c459-4437-ad25-4346b75454ac
ms.service: sql-database
ms.custom: V11
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-management
ms.date: 08/08/2016
ms.author: sstein
translationtype: Human Translation
ms.sourcegitcommit: 145cdc5b686692b44d2c3593a128689a56812610
ms.openlocfilehash: bc7fcebaebe510e1ce00e6f55b4db7787e5ce53c


---
# <a name="upgrade-to-azure-sql-database-v12-using-the-azure-portal"></a>Mise à niveau vers Azure SQL Database V12 à l’aide du portail Azure
> [!div class="op_single_selector"]
> * [Portail Azure](sql-database-upgrade-server-portal.md)
> * [PowerShell](sql-database-upgrade-server-powershell.md)
> 
> 

SQL Database V12 est la version la plus récente. Il est donc recommandé d’effectuer une mise à niveau vers SQL Database V12. Pour une matrice de fonctionnalité comparant Azure SQL Database avec SQL Server, voir [Fonctionnalités Azure SQL Database](sql-database-features.md).

Cet article fournit des instructions sur la mise à niveau des serveurs et bases de données SQL Database V11 existants vers SQL Database V12.

Lors du processus de mise à niveau vers la version V12, vous allez mettre à niveau toutes les bases de données Web et Business vers un nouveau niveau de service. C’est pourquoi des instructions pour la mise à niveau des bases de données Web et Business sont incluses.

En outre, la migration vers un [pool élastique](sql-database-elastic-pool.md) peut être plus économique que la mise à niveau vers des niveaux de performances individuels (niveaux tarifaires) pour des bases de données autonomes. Les pools simplifient également la gestion des bases de données, car vous devez uniquement gérer les paramètres de performances du pool, et non gérer séparément les niveaux de performances des bases de données individuelles. Si vous disposez de bases de données sur plusieurs serveurs, envisagez de les déplacer dans le même serveur et de tirer parti de leur regroupement au sein d’un pool. Vous pouvez facilement [auto-migrer des bases de données à partir de serveurs V11 directement dans des pools élastiques à l’aide de PowerShell](sql-database-upgrade-server-powershell.md). Vous pouvez également utiliser le portail pour migrer des bases de données V11 dans un pool, mais vous devez déjà disposer d’un serveur V12 dans le portail pour créer un pool. Des instructions sont fournies plus loin dans cet article pour créer le pool après la mise à niveau du serveur si vous disposez de [bases de données qui peuvent retirer des avantages d’un pool](sql-database-elastic-pool-guidance.md).

Notez que vos bases de données resteront en ligne et continueront à fonctionner pendant toute la durée de l’opération de mise à niveau. Au moment précis de la transition vers le nouveau niveau de performances, une perte temporaire des connexions à la base de données peut parfois se produire pendant une très courte durée, généralement pendant environ 90 secondes, mais cette durée peut atteindre 5 minutes. Si votre application [gère les problèmes temporaires d’interruption de connexion](sql-database-connectivity-issues.md) , il suffit d’établir une protection contre la perte de connexion à la fin de la mise à niveau.

Il n’est pas possible d’annuler la mise à niveau vers SQL Database V12. Après une mise à niveau, le serveur ne peut pas être restauré vers la version V11.

Après la mise à niveau vers la version V12, les [recommandations du niveau de service](sql-database-service-tier-advisor.md) et les [considérations sur les performances du pool élastique](sql-database-elastic-pool-guidance.md) ne sont pas disponibles immédiatement. Elles le seront lorsque le service aura eu le temps d’évaluer vos charges de travail sur le nouveau serveur. L’historique des recommandations du serveur V11 ne s’applique pas aux serveurs V12 et n’est donc pas conservé.

## <a name="prepare-to-upgrade"></a>Préparation de la mise à niveau
* **Mise à niveau de toutes les bases de données Web et Business** : consultez la section [Mise à niveau de toutes les bases de données Web et Business](sql-database-upgrade-server-portal.md#upgrade-all-web-and-business-databases) ci-dessous ou [Surveiller et gérer un pool élastique (PowerShell)](sql-database-elastic-pool-manage-powershell.md).
* **Vérifier et suspendre la géoréplicationréplication**: si votre base de données SQL Azure est configurée pour la géoréplication, vous devez documenter la configuration actuelle et [arrêter la géoréplication](sql-database-geo-replication-portal.md#remove-secondary-database). Une fois la mise à niveau terminée, reconfigurez votre base de données pour la géoréplication.
* **Ouvrez ces ports si vous avez des clients sur une machine virtuelle Azure**: si votre programme client se connecte à SQL Database V12 pendant que votre client s’exécute sur une machine virtuelle Azure, vous devez ouvrir les plages de ports 11000-11999 et 14000-14999 sur la machine virtuelle. Pour plus d'informations, consultez [Ports pour SQL Database V12](sql-database-develop-direct-route-ports-adonet-v12.md).

## <a name="start-the-upgrade"></a>Lancer la mise à niveau
1. Dans le [portail Azure](https://portal.azure.com/), accédez au serveur que vous souhaitez mettre à niveau en sélectionnant **PARCOURIR** > **Serveurs SQL**, puis en sélectionnant le serveur v2.0 que vous souhaitez mettre à niveau.
2. Sélectionnez **Dernière mise à jour de la base de données SQL**, puis **Mettre à niveau ce serveur**.
   
      ![mettre à niveau le serveur][1]
3. La mise à niveau d’un serveur vers la dernière mise à jour de la base de données SQL est permanente et irréversible. Pour confirmer la mise à niveau, tapez le nom de votre serveur et cliquez sur **OK**.

## <a name="upgrade-all-web-and-business-databases"></a>Mise à niveau de toutes les bases de données Web et Business
Si votre serveur dispose d’une base de données Web ou Business, vous devez la mettre à niveau. Au cours du processus de mise à niveau vers la version SQL Database V12, vous allez également mettre à jour toutes les bases de données Web et Business vers un nouveau niveau de service.    

Pour vous aider lors de la mise à niveau, le service Base de données SQL recommande un niveau de service et de performances approprié (niveau tarifaire) pour chaque base de données. En analysant l’historique d’utilisation de votre base de données, le service recommande le niveau qui convient le mieux pour l’exécution des charges de travail de votre base de données existante.

1. Dans le panneau **Mettre à niveau ce serveur** , sélectionnez chaque base de données pour l’examiner, puis sélectionnez le niveau tarifaire recommandé pour la mise à niveau. Vous pouvez également parcourir les niveaux tarifaires disponibles et sélectionner celui qui convient le mieux à votre environnement.

     ![bases de données][2]


1. Lorsque vous cliquez sur un niveau suggéré, le panneau **Choisir votre niveau de tarification** s’affiche, dans lequel vous pouvez sélectionner un niveau, puis cliquer sur le bouton **Sélectionner** pour le modifier. Sélectionnez un nouveau niveau pour chaque base de données Web ou Business.
   
    Il est important de noter que votre base de données SQL n’est pas verrouillée sur un niveau de performances ou de service spécifique. De cette façon, si les exigences de votre base de données changent, vous pouvez facilement passer à un autre niveau de service ou de performances disponible. En fait, les bases de données SQL De base, Standard et Premium sont facturées à l’heure et peuvent être augmentées ou réduites 4 fois par période de 24 heures.
   
    ![de films][6]

Lorsque toutes les bases de données du serveur sont éligibles, vous pouvez commencer la mise à niveau.

## <a name="confirm-the-upgrade"></a>Confirmer la mise à niveau
1. Lorsque toutes les bases de données sur le serveur sont mises à niveau, vous devez **SAISIR LE NOM DU SERVEUR** pour vérifier que vous souhaitez effectuer la mise à niveau, avant de cliquer sur **OK**.
   
    ![vérifier la mise à niveau][3]
2. La mise à niveau démarre. Sa progression est affichée dans une zone de notification. Le processus de mise à jour est lancé. Selon les détails de votre mise à niveau de bases de données vers V12, la procédure peut prendre un certain temps. Pendant cette période, toutes les bases de données sur le serveur restent en ligne, mais les actions de gestion du serveur et des bases de données sont restreintes.
   
    ![mise à niveau en cours][4]
   
    Au moment précis de la transition vers le nouveau niveau de performances, une perte temporaire des connexions à la base de données peut parfois se produire pendant quelques secondes. Si une application gère les problèmes temporaires d’interruption de connexion (logique de nouvelle tentative), il suffit d’établir une protection contre la perte de connexion à la fin de la mise à jour.
3. Après l’opération de mise à niveau, le panneau **Dernière mise à jour** affichera **Activé**.
   
    ![V12 activé][5]  

## <a name="move-your-databases-into-an-elastic-pool"></a>Déplacer des bases de données dans un pool élastique
Dans le [portail Azure](https://portal.azure.com/) , accédez au serveur V12 et cliquez sur **Ajouter un pool**.

- ou -

Si vous voyez un message indiquant **Cliquez ici pour afficher le pool élastique recommandé pour votre serveur**, cliquez dessus pour créer facilement un pool optimisé pour les bases de données de votre serveur. Pour en savoir plus, consultez la page [Considérations sur les prix et performances pour un pool élastique](sql-database-elastic-pool-guidance.md).

![Ajouter un pool à un serveur][7]

Suivez les instructions de l’article [Créer un pool élastique](sql-database-elastic-pool.md) pour terminer de créer votre pool.

## <a name="monitor-databases-after-upgrading-to-sql-database-v12"></a>Surveillance des bases de données après la mise à niveau vers SQL Database V12
> [!IMPORTANT]
> Effectuez la mise à niveau vers la dernière version de SQL Server Management Studio (SSMS) pour tirer parti des nouvelles fonctionnalités de la version 12. [Téléchargez SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx).
> 
> 

Après la mise à niveau, surveillez activement la base de données pour vous assurer que les applications s’exécutent au niveau de performances souhaité, puis optimisez les paramètres en fonction des besoins.

Outre les bases de données individuelles, vous pouvez surveiller les pools élastiques [Surveiller, gérer et dimensionner un pool élastique avec le Portail Azure](sql-database-elastic-pool-manage-portal.md) ou avec [PowerShell](sql-database-elastic-pool-manage-powershell.md).

**Données sur la consommation de ressources** : pour les bases de données De base, Standard et Premium, des données sur la consommation des ressources sont disponibles par le biais de la vue de gestion dynamique (DMV)[sys.dm_ db_ resource_stats](http://msdn.microsoft.com/library/azure/dn800981.aspx) dans la base de données utilisateur. Cette vue offre presque en temps réel les informations de consommation des ressources avec un niveau de granularité de 15 secondes pour l’heure précédente de l’opération. La consommation de pourcentage DTU pour un intervalle est calculée comme la consommation de pourcentage maximal des dimensions UC, E/S et journal. Voici une requête pour calculer la moyenne de pourcentage DTU sur la dernière heure :

    SELECT end_time
         , (SELECT Max(v)
             FROM (VALUES (avg_cpu_percent)
                         , (avg_data_io_percent)
                         , (avg_log_write_percent)
           ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.dm_db_resource_stats
    ORDER BY end_time DESC;

Informations de surveillance supplémentaires :

* [Guide des performances Azure SQL Database pour les bases de données autonomes](http://msdn.microsoft.com/library/azure/dn369873.aspx).
* [Considérations sur les prix et performances pour un pool élastique](sql-database-elastic-pool-guidance.md).
* [Analyse d’une base de données SQL Azure à l’aide de vues de gestion dynamique](sql-database-monitoring-with-dmvs.md)

**Alertes :** configurez les « alertes » dans le portail Azure pour vous avertir quand la consommation DTU d’une base de données mise à niveau approche d’un niveau élevé. Les alertes de la base de données peuvent être configurées dans le portail Azure pour diverses mesures de performances comme DTU, UC, E/S et journal. Accédez à votre base de données et sélectionnez **Règles d’alerte** dans le panneau **Paramètres**.

Par exemple, vous pouvez configurer une alerte par courrier électronique sur « Pourcentage DTU » si la valeur moyenne du pourcentage DTU est supérieure à 75 % pendant les 5 dernières minutes. Reportez-vous à [Réception de notifications d’alerte](../monitoring-and-diagnostics/insights-receive-alert-notifications.md) pour en savoir plus sur la configuration des notifications d’alerte.

## <a name="next-steps"></a>Étapes suivantes
* [Rechercher des recommandations de pool et créer un pool](sql-database-elastic-pool-create-portal.md).
* [Modifier les niveaux de service et de performances de votre base de données](sql-database-scale-up.md).

## <a name="related-links"></a>Liens connexes
* [Fonctionnalités SQL Database](sql-database-features.md)
* [Planifier et préparer la mise à niveau vers la version V12 de la base de données SQL](sql-database-v12-plan-prepare-upgrade.md)

<!--Image references-->
[1]: ./media/sql-database-upgrade-server-portal/latest-sql-database-update.png
[2]: ./media/sql-database-upgrade-server-portal/upgrade-server2.png
[3]: ./media/sql-database-upgrade-server-portal/upgrade-server3.png
[4]: ./media/sql-database-upgrade-server-portal/online-during-upgrade.png
[5]: ./media/sql-database-upgrade-server-portal/enabled.png
[6]: ./media/sql-database-upgrade-server-portal/recommendations.png
[7]: ./media/sql-database-upgrade-server-portal/new-elastic-pool.png



<!--HONumber=Dec16_HO2-->


