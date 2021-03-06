---
title: "Connexion de Configuration Manager à Log Analytics | Microsoft Docs"
description: "Cet article décrit comment connecter Configuration Manager à Log Analytics et commencer à analyser des données."
services: log-analytics
documentationcenter: 
author: bandersmsft
manager: carmonm
editor: 
ms.assetid: f2298bd7-18d7-4371-b24a-7f9f15f06d66
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/02/2017
ms.author: banders
translationtype: Human Translation
ms.sourcegitcommit: 820a9463c0e58054cf70324b680c5af8fdcacade
ms.openlocfilehash: 18479c3680aa46113e59d25b6e034830f78a1859


---
# <a name="connect-configuration-manager-to-log-analytics"></a>Connexion de Configuration Manager à Log Analytics
Vous pouvez connecter System Center Configuration Manager à Log Analytics dans OMS pour synchroniser les données de regroupement d’appareils. Cela rend les données de votre déploiement de Configuration Manager disponibles dans OMS.

Plusieurs étapes sont nécessaires pour connecter Configuration Manager à OMS. Voici un aperçu rapide de l’ensemble du processus :

1. Dans le portail de gestion Azure, inscrivez Configuration Manager en tant qu’application web et/ou application API web, et assurez-vous de disposer de l’ID et de la clé secrète client résultant de l’inscription à partir d’Azure Active Directory. Pour plus d’informations sur cette étape, voir [Utiliser le portail pour créer une application et un principal du service Active Directory pouvant accéder aux ressources](../azure-resource-manager/resource-group-create-service-principal-portal.md).
2. Dans le portail de gestion Azure, [Configuration Manager (l’application web inscrite) l’autorisation d’accéder à OMS](#provide-configuration-manager-with-permissions-to-oms).
3. Dans Configuration Manager, [ajoutez une connexion à l’aide de l’Assistant Ajout de connexion OMS](#add-an-oms-connection-to-configuration-manager).
4. Dans Configuration Manager, vous pouvez [mettre à jour les propriétés de connexion](#update-oms-connection-properties) en cas d’expiration ou de perte du mot de passe ou de la clé secrète du client.
5. À l’aide des informations du portail OMS, [téléchargez et installez Microsoft Monitoring Agent](#download-and-install-the-agent) sur l’ordinateur exécutant le rôle de système de site de point de connexion de service de Configuration Manager. L’agent envoie les données de Configuration Manager à OMS.
6. Dans OMS, [importez les regroupements de Configuration Manager](#import-collections) en tant que groupes d’ordinateurs.
7. Dans OMS, affichez les données de Configuration Manager en tant que [groupes d’ordinateurs](log-analytics-computer-groups.md).

Pour plus d’informations sur la connexion de Configuration Manager à OMS, voir [Synchroniser des données de Configuration Manager sur Microsoft Operations Management Suite](https://technet.microsoft.com/library/mt757374.aspx).

## <a name="provide-configuration-manager-with-permissions-to-oms"></a>Accorder à Configuration Manager les autorisations d’accès à OMS
La procédure suivante décrit comment accorder au portail de gestion Azure les autorisations d’accès à OMS. Il s’agit plus précisément d’attribuer le *rôle Collaborateur* aux utilisateurs dans le groupe de ressources. Cela permet ensuite au portail de gestion Azure de connecter Configuration Manager à OMS.

> [!NOTE]
> Vous devez spécifier pour Configuration Manager des autorisations d’accès à OMS. Autrement, vous recevez un message d’erreur lorsque vous utilisez l’Assistant Configuration dans Configuration Manager.
>
>

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis cliquez sur **Parcourir** > **Log Analytics (OMS)** pour ouvrir le panneau Log Analytics (OMS).  
2. Dans le panneau **Log Analytics (OMS)**, cliquez sur **Ajouter** pour ouvrir le panneau **space de travail OMS**.  
   ![Panneau OMS](./media/log-analytics-sccm/sccm-azure01.png)
3. Dans le panneau **Espace de travail OMS**, entrez les informations suivantes, puis cliquez sur **OK**.

   * **Espace de travail OMS**
   * **Abonnement**
   * **Groupe de ressources**
   * **Emplacement**
   * **Niveau tarifaire**  
     ![Panneau OMS](./media/log-analytics-sccm/sccm-azure02.png)  

     > [!NOTE]
     > L’exemple ci-dessus crée un groupe de ressources. Le groupe de ressources est utilisé uniquement pour accorder à Configuration Manager les autorisations nécessaires pour accéder à l’espace de travail OMS dans cet exemple.
     >
     >
4. Cliquez sur **Parcourir** > **Groupes de ressources** pour ouvrir le panneau **Groupes de ressources**.
5. Dans le panneau **Groupes de ressources**, cliquez sur le groupe de ressources que vous avez créé ci-dessus pour ouvrir le panneau de paramètres &lt;Nom du groupe de ressources&gt;.  
   ![Panneau de paramètres de groupe de ressources](./media/log-analytics-sccm/sccm-azure03.png)
6. Dans le panneau de paramètres &lt;Nom du groupe de ressources&gt;, cliquez sur Contrôle d’accès (IAM) pour ouvrir le panneau d’utilisateurs &lt;Nom du groupe de ressources&gt;.  
   ![Panneau d’utilisateurs de groupe de ressources](./media/log-analytics-sccm/sccm-azure04.png)  
7. Dans le panneau d’utilisateurs &lt;Nom du groupe de ressources&gt;, cliquez sur **Ajouter** pour ouvrir le panneau **Ajouter un accès**.
8. Dans le panneau **Ajouter un accès**, cliquez sur **Sélectionner un rôle**, puis sélectionnez le rôle **Collaborateur**.  
   ![Sélectionner un rôle](./media/log-analytics-sccm/sccm-azure05.png)  
9. Cliquez sur **Ajouter des utilisateurs**, sélectionnez l’utilisateur de Configuration Manager, cliquez sur **Sélectionner**, puis sur **OK**.  
   ![Ajouter des utilisateurs](./media/log-analytics-sccm/sccm-azure06.png)  

## <a name="add-an-oms-connection-to-configuration-manager"></a>Ajouter une connexion OMS à Configuration Manager
Pour que vous puissiez ajouter une connexion OMS, il faut que votre environnement Configuration Manager ait un [point de connexion de service](https://technet.microsoft.com/library/mt627781.aspx) configuré pour le mode en ligne.

1. Dans l’espace de travail **Administration** de Configuration Manager, sélectionnez **Connecteur OMS**. Cette opération ouvre l’**Assistant Ajout de connexion OMS**. Sélectionnez **Suivant**.
2. Dans l’écran **Général**, vérifiez que vous avez effectué les actions suivantes et que vous disposez des détails de chaque élément, puis sélectionnez **Suivant**.

   1. Dans le portail de gestion Azure, vous avez inscrit Configuration Manager en tant qu’application web et/ou application API web, et vous disposez de l’[ID client résultant de l’inscription](../active-directory/active-directory-integrating-applications.md).
   2. Dans le portail de gestion Azure, vous avez créé une clé secrète d’application pour l’application inscrite dans Azure Active Directory.  
   3. Dans le portail de gestion Azure, vous avez accordé à l’application web inscrite l’autorisation d’accéder à OMS.  
      ![Page générale de l’Assistant Connexion à OMS](./media/log-analytics-sccm/sccm-console-general01.png)
3. Dans l’écran **Azure Active Directory**, configurez vos paramètres de connexion à OMS en complétant les champs **Client**, **ID client** et **Clé secrète du client**, puis sélectionnez **Suivant**.  
   ![Page Azure Active Directory de l’Assistant Connexion à la OMS](./media/log-analytics-sccm/sccm-wizard-tenant-filled03.png)
4. Si vous avez correctement accompli toutes les autres procédures, les informations de l’écran **Configuration de la connexion OMS** s’affichent automatiquement dans cette page. Les informations des paramètres de connexion doivent s’afficher pour **Abonnement Azure**, **Groupe de ressources Azure** et **Espace de travail Operations Management Suite**.  
   ![Page Connexion à OMS de l’Assistant Connexion à OMS](./media/log-analytics-sccm/sccm-wizard-configure04.png)
5. L’Assistant se connecte au Service OMS en utilisant les informations que vous avez entrées. Sélectionnez les regroupements d’appareils à synchroniser avec OMS, puis cliquez sur **Ajouter**.  
   ![Sélectionner les regroupements](./media/log-analytics-sccm/sccm-wizard-add-collections05.png)
6. Vérifiez vos paramètres de connexion dans l’écran **Résumé**, puis sélectionnez **Suivant**. L’écran **Progression** affiche l’état de la connexion, puis doit afficher **Terminé**.

> [!NOTE]
> Vous devez connecter OMS au site de niveau supérieur de votre hiérarchie. Si vous connectez OMS à un site principal autonome, puis ajoutez un site d’administration centrale à votre environnement, vous devrez supprimer et recréer la connexion OMS au sein de la nouvelle hiérarchie.
>
>

Après avoir lié Configuration Manager à OMS, vous pouvez ajouter ou supprimer des regroupements, et afficher les propriétés de la connexion OMS.

## <a name="update-oms-connection-properties"></a>Mettre à jour les propriétés de la connexion OMS
En cas d’expiration ou de perte de mot de passe ou de clé secrète du client, vous devez mettre à jour manuellement les propriétés de connexion OMS.

1. Dans Configuration Manager, accédez à **Services cloud**, puis sélectionnez **Connecteur OMS** pour ouvrir la page **Propriétés de connexion OMS**.
2. Dans cette page, cliquez sur l’onglet **Azure Active Directory** pour afficher vos informations **Client**, **ID client** et **Expiration de clé secrète client**. **Vérifiez** votre **Clé secrète client** pour voir si elle a expiré.

## <a name="download-and-install-the-agent"></a>Téléchargement et installation de l’agent
1. Dans le portail OMS, sélectionnez [Télécharger le fichier de configuration de l’agent sur OMS](log-analytics-windows-agents.md#download-the-agent-setup-file-from-oms).
2. Utilisez l’une des méthodes suivantes pour installer et configurer l’agent sur l’ordinateur exécutant le rôle de système de site de point de connexion de service de Configuration Manager :
   * [Installer l’agent à l’aide du programme d’installation](log-analytics-windows-agents.md#install-the-agent-using-setup)
   * [Installer l’agent à l’aide de la ligne de commande](log-analytics-windows-agents.md#install-the-agent-using-the-command-line)
   * [Installer l’agent à l’aide de DSC dans Azure Automation](log-analytics-windows-agents.md#install-the-agent-using-dsc-in-azure-automation)

## <a name="import-collections"></a>Importer des regroupements
Après avoir ajouté une connexion OMS à Configuration Manager et installé l’agent sur l’ordinateur exécutant le rôle de système de site de point de connexion de service de Configuration Manager, l’étape suivante consiste à importer des regroupements de Configuration Manager dans OMS en tant que groupes d’ordinateurs.

Une fois que l’importation activée, les informations d’appartenance au regroupement sont récupérées toutes les 3 heures pour tenir à jour les appartenances au regroupement. Vous pouvez choisir de désactiver l’importation à tout moment.

1. Dans le portail OMS, cliquez sur **Paramètres**.
2. Cliquez sur l’onglet **Groupes d’ordinateurs**, puis sur l’onglet **SCCM**.
3. Sélectionnez **Importer les appartenances aux regroupements Configuration Manager**, puis cliquez sur **Enregistrer**.  
   ![Groupes d’ordinateurs - Onglet SCCM](./media/log-analytics-sccm/sccm-computer-groups01.png)

## <a name="view-data-from-configuration-manager"></a>Afficher les données de Configuration Manager
Après l’ajout d’une connexion OMS à Configuration Manager et l’installation de l’agent sur l’ordinateur exécutant le rôle de système de site de point de connexion de service de Configuration Manager, les données de l’agent sont envoyées à OMS. Dans OMS, vos regroupements de Configuration Manager apparaissent sous la forme de [groupes d’ordinateurs](log-analytics-computer-groups.md). Vous pouvez afficher les groupes à partir de la page **Configuration Manager**, sous **Groupes d’ordinateurs** dans **Paramètres**.

Une fois les regroupements importés, vous pouvez voir combien d’ordinateurs avec des appartenances à des regroupements ont été détectés. Vous pouvez également voir le nombre de regroupements importés.

![Groupes d’ordinateurs - Onglet SCCM](./media/log-analytics-sccm/sccm-computer-groups02.png)

Lorsque vous cliquez sur l’un deux, la fenêtre Recherche s’ouvre, affichant tous les groupes importés ou tous les ordinateurs appartenant à chaque groupe. Dans [Recherche de journal](log-analytics-log-searches.md), vous pouvez démarrer une analyse approfondie des données de Configuration Manager.

## <a name="next-steps"></a>Étapes suivantes
* Utilisez [Recherche de journal](log-analytics-log-searches.md) pour afficher des informations détaillées sur vos données Configuration Manager.



<!--HONumber=Jan17_HO1-->


