---
title: "Réplication de machines virtuelles Hyper-V (sans VMM) dans Azure à l’aide d’Azure Site Recovery et du Portail Azure | Microsoft Docs"
description: "Explique comment déployer Azure Site Recovery pour orchestrer la réplication, le basculement et la récupération des machines virtuelles Hyper-V locales qui ne sont pas gérées par VMM dans Azure à l’aide du Portail Azure."
services: site-recovery
documentationcenter: 
author: rayne-wiselman
manager: jwhit
editor: 
ms.assetid: 1777e0eb-accb-42b5-a747-11272e131a52
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 11/23/2016
ms.author: raynew
translationtype: Human Translation
ms.sourcegitcommit: 1268d29b0d9c4368f62918758836a73c757c0c8d
ms.openlocfilehash: aeccda397ea3c311afddd88b3a8b9c6dab2461d7

---

# <a name="replicate-hyper-v-virtual-machines-without-vmm-to-azure-using-azure-site-recovery-with-the-azure-portal"></a>Répliquez des machines virtuelles Hyper-V (sans VMM) vers Azure avec Azure Site Recovery sur le Portail Azure.
> [!div class="op_single_selector"]
> * [Portail Azure](site-recovery-hyper-v-site-to-azure.md)
> * [Azure Classic](site-recovery-hyper-v-site-to-azure-classic.md)
> * [PowerShell - Resource Manager](site-recovery-deploy-with-powershell-resource-manager.md)
>
>

Bienvenue dans le service Azure Site Recovery !

Site Recovery est un service Azure qui participe à votre stratégie de continuité des activités et de récupération d’urgence. Il orchestre la réplication des machines virtuelles et des serveurs physiques locaux vers le cloud (Azure) ou un centre de données secondaire. Lorsque des pannes se produisent sur votre site principal, vous effectuez un basculement sur le site secondaire pour préserver la disponibilité des applications et des charges de travail. Vous restaurez votre site principal dès lors qu’il retrouve un fonctionnement normal. Pour en savoir plus, consultez [Qu’est-ce que Site Recovery ?](site-recovery-overview.md)

Cet article explique comment répliquer ou migrer des machines virtuelles Hyper-V locales dans Azure à l’aide d’Azure Site Recovery dans le Portail Azure. Dans ce scénario, les serveurs Hyper-V ne sont pas gérés dans des clouds VMM. Déployez la réplication pour le basculement de machines virtuelles vers Azure lorsque votre site principal n’est pas disponible et restaurez-les en local à partir d’Azure lorsque le site principal fonctionne à nouveau normalement. Suivez les étapes de cet article pour migrer des machines virtuelles vers Azure (sans restauration). Ensuite, après avoir exécuté un basculement test réussi, vous pouvez effectuer un basculement planifié pour terminer la migration.


Après avoir lu cet article, envoyez vos commentaires en bas ou posez vos questions techniques sur le [Forum Azure Recovery Services](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr).

## <a name="quick-reference"></a>Référence rapide

Pour un déploiement complet, nous vous recommandons vivement de suivre toutes les étapes de l’article. Si vous ne disposez pas de suffisamment de temps, voici un résumé rapide.
 
 **Zone** | **Détails**
 --- | --- 
 **Scénario de déploiement** | Réplication de machines virtuelles Hyper-V non gérées dans des clouds VMM sur Azure à l’aide du Portail Azure 
 **Configuration requise en local** | Au moins un serveur Hyper-V exécutant au moins Windows Server 2012 R2 avec activation des dernières mises à jour ou du rôle Hyper-V ou Microsoft Hyper-V Server 2012 R2 avec les dernières mises à jour.<br/><br/> Les hôtes Hyper-V doivent disposer d’un accès à Internet et doivent être en mesure d’accéder à des URL spécifiques directement ou par le biais d’un proxy. [Informations complètes](#on-premises-prerequisites). 
 **Limitations en local** | Les proxies basés sur HTTPS ne sont pas pris en charge.
 **Fournisseur/agent** | Le fournisseur Azure Site Recovery et l’agent Recovery Services sont installés sur l’hôte Hyper-V lors du déploiement. 
 **Conditions requises pour Azure** | Compte Azure<br/><br/> Coffre Recovery Services<br/><br/> Compte de stockage LRS ou GRS dans la région du coffre<br/><br/> Compte de stockage standard<br/><br/> Réseau virtuel Azure dans la région du coffre. [Informations complètes](#azure-prerequisites). 
 **Limitations relatives à Azure** | Si vous utilisez GRS, vous avez besoin d’un autre compte LRS pour la journalisation.<br/><br/> Les comptes de stockage créés dans le portail Azure ne peuvent pas passer d’un groupe de ressources à l’autre, que ce soit dans le même abonnement ou dans des abonnements différents. <br/><br/> Premium Storage n’est pas pris en charge.<br/><br/> Les réseaux Azure utilisés pour Site Recovery ne peuvent pas passer d’un groupe de ressources à l’autre, que ce soit dans le même abonnement ou dans des abonnements différents. 
 **Réplication de machines virtuelles** | Les machines virtuelles doivent respecter les [conditions préalables d’Azure](site-recovery-best-practices.md#azure-virtual-machine-requirements)<br/><br/> 
 **Limitations relatives à la réplication** | Vous ne pouvez pas répliquer de machines virtuelles Hyper-V exécutant Linux avec une adresse IP statique.<br/><br/> Vous pouvez exclure des disques spécifiques de la réplication, mais non le disque du système d’exploitation.
 **Étapes du déploiement** | **1)** Créez un coffre Recovery Services -> **2)** Créez un site Hyper-V incluant tous les hôtes Hyper-V -> **3)** Configurez les hôtes Hyper-V -> **4**) Préparez Azure (abonnement, stockage, réseau) -> **5)** Configurez les paramètres de réplication -> **6)** Activez la réplication -> **7)** Testez la réplication et le basculement. **8)** si vous effectuez une migration, exécutez un basculement planifié. 

## <a name="azure-deployment-models"></a>Modèles de déploiement Azure

Azure dispose de deux [modèles de déploiement](../azure-resource-manager/resource-manager-deployment-model.md) pour créer et utiliser des ressources : le modèle Azure Resource Manager et le modèle classique. De plus, Azure propose deux portails : le [portail Azure Classic](https://manage.windowsazure.com/), qui prend en charge le modèle de déploiement classique, et le [portail Azure](https://ms.portal.azure.com/), qui gère les deux modèles de déploiement.

 Cet article explique comment effectuer un déploiement dans le portail Azure, qui offre une expérience de déploiement plus rationalisée. Le portail classique peut être utilisé pour gérer les coffres existants. Il est impossible de créer des coffres à l’aide du portail classique.

## <a name="site-recovery-in-your-business"></a>Site Recovery dans votre entreprise

Les organisations ont besoin d’une stratégie BCDR qui détermine la façon dont les applications et les données demeurent opérationnelles et disponibles pendant les temps d’arrêt prévus et imprévus, et qui précise comment rétablir au plus vite des conditions de travail normales. Voici ce que Site Recovery peut faire :
 
 - Protection hors site pour les applications professionnelles s’exécutant sur des machines virtuelles Hyper-V
 - Fourniture d’un emplacement unique pour configurer, gérer et surveiller la réplication, le basculement et la récupération
 - Basculement simple vers Azure et restauration depuis Azure sur des serveurs hôtes Hyper-V dans votre site local
 - Plans de récupération incluant plusieurs machines virtuelles et permettant ainsi le basculement simultané de charges de travail d’applications hiérarchisées





## <a name="scenario-architecture"></a>Architecture du scénario
Voici les composants du scénario :

* **Hôte ou cluster Hyper-V**: serveurs hôtes ou clusters Hyper-V locaux. Les hôtes Hyper-V exécutant des machines virtuelles à protéger sont rassemblés dans des sites Hyper-V logiques lors du déploiement de Site Recovery.
* **Fournisseur Azure Site Recovery et agent Azure Recovery Services**: lors du déploiement, vous installez le fournisseur Azure Site Recovery et l’agent Microsoft Azure Recovery Services sur les serveurs hôtes Hyper-V. Le fournisseur communique avec Azure Site Recovery sur le port HTTPS 443 pour répliquer l’orchestration. L’agent hébergé sur le serveur hôte Hyper-V réplique les données vers le stockage Azure via le port HTTPS 443, par défaut.
* **Azure**: il vous faut un abonnement Azure, un compte de stockage Azure pour stocker des données répliquées, et un réseau virtuel Azure, afin que les machines virtuelles Azure puissent se connecter à un réseau après un basculement.

![Architecture de site Hyper-V](./media/site-recovery-hyper-v-site-to-azure/architecture.png)

## <a name="azure-prerequisites"></a>Conditions préalables pour Azure


| **Configuration requise** | **Détails** |
| --- | --- |
| **Compte Azure** | Compte [Microsoft Azure](http://azure.microsoft.com/). Vous pouvez commencer par une version d’ [essai gratuit](https://azure.microsoft.com/pricing/free-trial/). [En savoir plus](https://azure.microsoft.com/pricing/details/site-recovery/) sur la tarification Site Recovery. |
| **Stockage Azure** | Compte de stockage standard. Vous pouvez utiliser un compte de stockage LRS ou GRS. Nous vous recommandons d’utiliser un compte GRS, afin que les données soient résilientes si une panne se produit au niveau régional, ou si la région principale ne peut pas être récupérée. [En savoir plus](../storage/storage-redundancy.md).<br/><br/> Ce compte doit se trouver dans la même région que le coffre Recovery Services.<br/><br/> Premium Storage n’est pas pris en charge.<br/><br/> Les données répliquées sont stockées dans Azure Storage et les machines virtuelles Azure sont créées au moment du basculement.<br/><br/> [Découvrez plus d’informations](../storage/storage-introduction.md) sur Azure Storage. |
| **Réseau Azure** |Vous aurez besoin d’un réseau virtuel Azure auquel les machines virtuelles Azure se connecteront au moment du basculement. Le réseau virtuel Azure doit se trouver dans la même région que le coffre Recovery Services. |

## <a name="on-premises-prerequisites"></a>Conditions préalables locales
Voici les éléments dont vous aurez besoin en local :

| **Configuration requise** | **Détails** |
| --- | --- |
| **Hyper-V** |Un ou plusieurs serveurs locaux exécutant **Windows Server 2012 R2** avec les dernières mises à jour et le rôle Hyper-V activé ou **Microsoft Hyper-V Server 2012 R2**.<br/><br/>Le serveur Hyper-V doit contenir une ou plusieurs machines virtuelles.<br/><br/>Les serveurs Hyper-V doivent être connectés à Internet, directement ou via un proxy.<br/><br/>Les correctifs mentionnés dans le document [KB2961977](https://support.microsoft.com/en-us/kb/2961977 "KB2961977") doivent être installés sur les serveurs Hyper-V. |
| **Fournisseur et agent** |Pendant le déploiement d’Azure Site Recovery, vous devez installer le fournisseur Azure Site Recovery. Dans le cadre de l’installation du fournisseur, l’agent Azure Recovery Services est installé sur chaque serveur Hyper-V exécutant les machines virtuelles que vous souhaitez protéger. Tous les serveurs Hyper-V d’un coffre Site Recovery doivent disposer des mêmes versions du fournisseur et de l’agent.<br/><br/>Le fournisseur devra se connecter à Azure Site Recovery via Internet. Le trafic peut être envoyé directement ou via un proxy. Notez que HTTPS basé sur proxy n’est pas pris en charge. Le serveur proxy doit autoriser l’accès aux éléments suivants :  <br/><br/> ``*.accesscontrol.windows.net``<br/><br/> ``*.backup.windowsazure.com``<br/><br/> ``*.hypervrecoverymanager.windowsazure.com`` <br/><br/> ``*store.core.windows.net``<br/><br/> ``*.blob.core.windows.net``<br/><br/> ``https://www.msftncsi.com/ncsi.txt``<br/><br/> ``time.windows.com``<br/><br/> ``time.nist.gov``<br/><br/> Si vous avez des règles de pare-feu fondées sur l’adresse IP sur le serveur, vérifiez que ces règles autorisent la communication vers Azure.<br/><br/> Autorisez les [plages d’adresses IP de centres de données Azure](https://www.microsoft.com/download/confirmation.aspx?id=41653) et le port HTTPS (443).<br/><br/> Autorisez les plages d’adresses IP relatives à la région de votre abonnement Azure et à la région des États-Unis de l’Ouest. |

## <a name="virtual-machine-prerequisites"></a>Configuration requise pour les machines virtuelles
| **Configuration requise** | **Détails** |
| --- | --- |
| **Machines virtuelles protégées** |Avant de basculer une machine virtuelle, vous devez vous assurer que le nom affecté à la machine virtuelle Azure est conforme à la [configuration requise pour Azure](site-recovery-best-practices.md#azure-virtual-machine-requirements). Le cas échéant, vous pouvez modifier le nom une fois la réplication activée pour la machine virtuelle.<br/><br/> Sur les machines protégées, la capacité d’un disque ne doit pas dépasser 1023 Go. Une machine virtuelle peut comporter jusqu’à 64 disques (jusqu’à 64 To).<br/><br/> Les clusters invités de disques partagés ne sont pas pris en charge.<br/><br/> Si la machine virtuelle source propose l’association de cartes réseau, elle est convertie en une carte réseau unique après le basculement vers Azure.<br/><br/>La protection des machines virtuelles exécutant Linux avec une adresse IP statique n’est pas prise en charge. |

## <a name="prepare-for-deployment"></a>Préparation du déploiement
Pour préparer un déploiement, vous devez :

1. [configurer un réseau Azure](#set-up-an-azure-network) dans lequel les machines virtuelles Azure sont placées lorsqu’elles sont créées après un basculement.
2. [configurer un compte de stockage Azure](#set-up-an-azure-storage-account) pour les données répliquées.
3. [préparer les hôtes Hyper-V](#prepare-the-hyper-v-hosts) pour assurer leur accès aux URL requises.

### <a name="set-up-an-azure-network"></a>Configurer un réseau Azure

Configurez un réseau Azure afin que les machines virtuelles Azure créées après le basculement puissent être connectées à un réseau.

* Ce réseau doit se trouver dans la même région que celui dans lequel vous allez déployer le coffre Recovery Services.
* Selon le modèle de ressource que vous souhaitez utiliser pour les machines virtuelles Azure ayant fait l’objet d’un basculement, configurez le réseau Azure en mode [Azure Resource Manager](../virtual-network/virtual-networks-create-vnet-arm-pportal.md) ou [Classic](../virtual-network/virtual-networks-create-vnet-classic-pportal.md).
* Nous vous recommandons de configurer un réseau avant de commencer. Dans le cas contraire, vous devrez le configurer lors du déploiement de Site Recovery.

> [!NOTE]
> La [migration des réseaux](../azure-resource-manager/resource-group-move-resources.md) entre les groupes de ressources d’un même abonnement ou de plusieurs abonnements n’est pas prise en charge pour les réseaux utilisés pour le déploiement de Site Recovery.
>
>

### <a name="set-up-an-azure-storage-account"></a>Configurer un compte de stockage Azure

- Vous avez besoin d’un compte de stockage Azure standard pour stocker les données répliquées sur Azure.
- Selon le modèle de ressource que vous souhaitez utiliser pour les machines virtuelles Azure ayant fait l’objet d’un basculement, configurez un compte en mode [Azure Resource Manager](../storage/storage-create-storage-account.md) ou [Classic](../storage/storage-create-storage-account-classic-portal.md).
- Nous vous recommandons de configurer un compte de stockage avant de commencer. Dans le cas contraire, vous devrez le configurer lors du déploiement de Site Recovery. Ce compte doit se trouver dans la même région que le coffre Recovery Services.
- Vous ne pouvez pas déplacer de comptes de stockage utilisés par Site Recovery entre des groupes de ressources au sein du même abonnement, ou entre différents abonnements.


### <a name="prepare-the-hyper-v-hosts"></a>préparer les hôtes Hyper-V
* Assurez-vous que les hôtes Hyper-V respectent la [configuration requise](#on-premises-prerequisites).

### <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services
1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **Nouveau** > **Gestion** > **Sauvegarde et récupération de sites (OMS)**. Vous pouvez également sélectionner **Parcourir** > **Coffres **Recovery Services > **Ajouter**.

    ![Nouveau coffre](./media/site-recovery-hyper-v-site-to-azure/new-vault3.png)
3. Dans **Nom** , spécifiez un nom convivial permettant d’identifier le coffre. Si vous avez plusieurs abonnements, sélectionnez-en un.
4. [Créez un groupe de ressources](../azure-resource-manager/resource-group-template-deploy-portal.md) ou sélectionnez un groupe existant et spécifiez une région Azure. Les machines seront répliquées dans cette région. Pour découvrir les régions prises en charge, référez-vous à la disponibilité géographique de la page [Détails des prix d'Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/)
5. Si vous souhaitez accéder rapidement au coffre depuis le tableau de bord, cliquez sur **Épingler au tableau de bord**, puis sur **Créer un coffre**.

    ![Nouveau coffre](./media/site-recovery-hyper-v-site-to-azure/new-vault-settings.png)

Le nouveau coffre s’affiche dans la zone **Tableau de bord** > **Toutes les ressources**, et dans le panneau **Coffres Recovery Services** principal.

## <a name="get-started"></a>Prise en main
Site Recovery propose une expérience de prise en main, qui vous permet d’effectuer le déploiement aussi rapidement que possible. Ainsi, la fonction Prise en main vérifie la configuration requise et vous guide à travers les étapes de déploiement de Site Recovery, dans l’ordre adéquat.

Grâce à cette fonction, vous pouvez sélectionner le type de machine que vous souhaitez répliquer, voire l’endroit auquel vous souhaitez effectuer la réplication. Vous pouvez configurer des serveurs locaux, des comptes Azure Storage et des réseaux. Il est également possible de créer des stratégies de réplication et d’effectuer la planification de la capacité. Une fois que vous avez configuré votre infrastructure, vous activez la réplication pour les machines virtuelles. Vous pouvez exécuter le basculement de machines spécifiques, ou créer des plans de récupération pour effectuer le basculement de plusieurs machines.

Lancez la fonction Prise en main en sélectionnant le mode de déploiement de Site Recovery. Le flux de la fonction Prise en main change légèrement selon vos exigences en matière de réplication.

## <a name="step-1-choose-your-protection-goals"></a>Étape 1 : sélectionner vos objectifs en matière de protection
Sélectionnez les éléments à répliquer et l’emplacement de la réplication.

1. Dans le volet **Coffres Recovery Services**, choisissez votre coffre et cliquez sur **Paramètres**.
2. Dans **Paramètres** > **Prise en main**, cliquez sur **Site Recovery** > **Étape 1 : Préparer l’infrastructure** > **Objectif de protection**.

    ![Sélectionner des objectifs](./media/site-recovery-hyper-v-site-to-azure/choose-goals.png)
3. Dans la zone **Protection goal (Objectif de la protection)**, sélectionnez **To Azure (Vers Azure)**, puis **Yes, with Hyper-V (Oui, avec Hyper-V)**. Sélectionnez **Non** pour confirmer que vous n’utilisez pas VMM. Cliquez ensuite sur **OK**.

    ![Sélectionner des objectifs](./media/site-recovery-hyper-v-site-to-azure/choose-goals2.png)

## <a name="step-2-set-up-the-source-environment"></a>Étape 2 : configurer l’environnement source
Configurez le site Hyper-V, installez le fournisseur Azure Site Recovery et l’agent Azure Recovery Services sur des hôtes Hyper-V et inscrivez les hôtes dans le coffre.

1. Cliquez sur **Étape 2 : Préparer l’infrastructure** > **Source**. Pour ajouter un nouveau site Hyper-V en tant que conteneur pour vos hôtes ou clusters Hyper-V, cliquez sur **+Site Hyper-V**.

    ![Configurer la source](./media/site-recovery-hyper-v-site-to-azure/set-source1.png)
2. Dans le panneau **Créer un site Hyper-V** , spécifiez un nom pour le site. Cliquez ensuite sur **OK**. Sélectionnez le site que vous venez de créer.

    ![Configurer la source](./media/site-recovery-hyper-v-site-to-azure/set-source2.png)
3. Cliquez sur **+Serveur Hyper-V** pour ajouter un serveur au site.
4. Sous **Ajouter un serveur** > **Type de serveur**, vérifiez que **Serveur Hyper-V** est affiché. Assurez-vous que le serveur Hyper-V à ajouter est conforme à la [configuration requise](#on-premises-prerequisites) et est en mesure d’accéder aux URL spécifiées.
5. Téléchargez le fichier d’installation du fournisseur Azure Site Recovery. Vous allez exécuter ce fichier pour installer le fournisseur et l’agent Recovery Services sur chaque hôte Hyper-V.
6. Téléchargez la clé d’inscription. Vous en aurez besoin lorsque vous exécuterez le programme d’installation. Une fois générée, la clé est valide pendant 5 jours.

    ![Configurer la source](./media/site-recovery-hyper-v-site-to-azure/set-source3.png)
7. Exécutez le fichier d’installation du fournisseur sur chaque hôte que vous avez ajouté au site Hyper-V. Si vous effectuez l’installation sur un cluster Hyper-V, exécutez le programme d’installation sur chaque nœud du cluster. L’installation et l’inscription de chaque nœud de cluster Hyper-V permettent de s’assurer que les machines virtuelles restent protégées même si elles migrent entre les nœuds.

### <a name="install-the-provider-and-agent"></a>Installer le fournisseur et l’agent
1. Exécutez le fichier de configuration du fournisseur.
2. Dans **Microsoft Update** , vous pouvez choisir des mises à jour, pour que celles du fournisseur soient installées conformément à votre stratégie Microsoft Update.
3. Dans le champ **Installation**, acceptez ou modifiez l’emplacement d’installation du fournisseur par défaut et cliquez sur **Installer**.
4. Sur la page **Paramètres du coffre**, cliquez sur **Parcourir** pour sélectionner le fichier de clé du coffre que vous avez téléchargé. Spécifiez l’abonnement Azure Site Recovery, le nom du coffre et le site Hyper-V auquel appartient le serveur Hyper-V.

    ![Enregistrement du serveur](./media/site-recovery-hyper-v-site-to-azure/provider3.png)

5. Dans **Paramètres de proxy**, indiquez comment le fournisseur à installer sur le serveur doit se connecter à Azure Site Recovery par le biais d’Internet.

* Si vous voulez que le fournisseur se connecte directement, sélectionnez **Se connecter directement sans proxy**.
* Si vous voulez vous connecter avec le proxy actuellement défini sur le serveur, sélectionnez **Se connecter avec des paramètres de proxy existants**.
* Si votre proxy existant nécessite une authentification, ou si vous voulez utiliser un proxy personnalisé pour la connexion du fournisseur, sélectionnez **Se connecter avec des paramètres de proxy personnalisés**.
* Si vous utilisez un proxy personnalisé, vous devez spécifier l’adresse, le port et les données d’identification
* Si vous utilisez un proxy, vérifiez que les URL décrites dans les [conditions préalables](#on-premises-prerequisites) y sont autorisées.

    ![Internet](./media/site-recovery-hyper-v-site-to-azure/provider7.PNG)

6. Une fois l’installation terminée, cliquez sur **Inscrire** pour inscrire le serveur dans le coffre.

![Emplacement d’installation](./media/site-recovery-hyper-v-site-to-azure/provider2.png)

7. Une fois l’inscription terminée, les métadonnées du serveur Hyper-V sont récupérées par Azure Site Recovery et le serveur s’affiche dans le panneau **Paramètres** > **Site Recovery Infrastructure** > **Hyper-V Hosts** (Hôtes Hyper-V).

### <a name="command-line-installation"></a>Installation à partir de la ligne de commande
L’agent et le fournisseur Azure Site Recovery peuvent également être installés à l’aide de la ligne de commande suivante. Cette méthode peut être utilisée pour installer le fournisseur sur un module Server Core pour Windows Server 2012 R2.

1. Téléchargez le fichier d’installation du fournisseur et la clé d’inscription dans un dossier, par exemple C:\ASR.
2. À partir d’une invite de commandes avec élévation de privilèges, exécutez les commandes suivantes pour extraire le programme d’installation du fournisseur :

            C:\Windows\System32> CD C:\ASR
            C:\ASR> AzureSiteRecoveryProvider.exe /x:. /q
3. Exécutez cette commande pour installer les composants :

            C:\ASR> setupdr.exe /i
4. Ensuite, exécutez les commandes suivantes pour inscrire le serveur dans le coffre :

            CD C:\Program Files\Microsoft Azure Site Recovery Provider\
            C:\Program Files\Microsoft Azure Site Recovery Provider\> DRConfigurator.exe /r  /Friendlyname <friendly name of the server> /Credentials <path of the credentials file>

<br/>
Où :

* **/Credentials** : paramètre obligatoire, qui spécifie l’emplacement auquel le fichier de clé d’inscription se trouve  
* **/FriendlyName** : paramètre obligatoire, qui correspond au nom du serveur hôte Hyper-V qui s’affiche sur le portail Microsoft Azure Site Recovery
* **/proxyAddress** : paramètre facultatif qui spécifie l’adresse du serveur proxy
* **/proxyport** : paramètre facultatif qui spécifie le port du serveur proxy
* **/proxyUsername** : paramètre facultatif qui spécifie le nom d’utilisateur proxy (si le proxy nécessite une authentification)
* **/proxyPassword** : paramètre facultatif qui spécifie le mot de passe pour l'authentification auprès du serveur proxy (si le proxy nécessite une authentification).

## <a name="step-3-set-up-the-target-environment"></a>Étape 3 : configurer l’environnement cible

Spécifiez le compte Azure Storage à utiliser pour la réplication, ainsi que le réseau Azure auquel les machines virtuelles Azure se connecteront après le basculement.

1. Cliquez sur **Préparer l’infrastructure** > **Cible**, puis sélectionnez l’abonnement et le groupe de ressources dans lesquels vous voulez créer les machines virtuelles basculées. Choisissez le modèle de déploiement que vous souhaitez utiliser dans Azure (classique ou gestion des ressources) pour les machines virtuelles basculées.

3. Site Recovery vérifie que vous disposez d’un ou de plusieurs réseaux et comptes Azure Storage compatibles.

    ![Storage](./media/site-recovery-vmware-to-azure/enable-rep3.png))


4. Si vous n’avez pas créé de compte de stockage et que vous souhaitez le faire à l’aide de Resource Manager, cliquez sur **+Compte de stockage** pour effectuer l’opération en ligne. Dans le panneau **Créer un compte de stockage** , saisissez le nom, le type, l’abonnement associé et l’emplacement du compte de stockage. Ce compte doit se trouver au même emplacement que le coffre Recovery Services.

    ![Storage](./media/site-recovery-hyper-v-site-to-azure/gs-createstorage.png)


Si vous souhaitez créer un compte de stockage en suivant le modèle Classic, vous pouvez utiliser le [Portail Azure](../storage/storage-create-storage-account-classic-portal.md).


Si vous n’avez pas créé de réseau Azure et que vous souhaitez le faire avec Azure Resource Manager, cliquez sur **+Réseau** afin d’effectuer l’opération en ligne. Dans le panneau **Créer un réseau virtuel** , spécifiez le nom, la plage d’adresses, les informations sur le sous-réseau associé, l’abonnement et l’emplacement du réseau virtuel. Ce réseau doit se trouver au même emplacement que le coffre Recovery Services.

   ![Réseau](./media/site-recovery-hyper-v-site-to-azure/gs-createnetwork.png)

Si vous souhaitez créer un réseau en suivant le modèle classique, vous pouvez utiliser le [portail Azure](../virtual-network/virtual-networks-create-vnet-classic-pportal.md).

## <a name="step-4-set-up-replication-settings"></a>Étape 4 : configurer les paramètres de réplication
1. Pour créer une stratégie de réplication, cliquez sur **Préparer l’infrastructure** > **Paramètres de réplication** > **+Créer et associer**.

    ![Réseau](./media/site-recovery-hyper-v-site-to-azure/gs-replication.png)
2. Dans la zone **Créer et associer une stratégie** , indiquez le nom de la stratégie.
3. Dans le champ **Copier la fréquence** , spécifiez la fréquence à laquelle répliquer les données delta après la réplication initiale (toutes les 30 secondes ou toutes les 5 ou 15 minutes).
4. Dans **Rétention des points de récupération**, spécifiez la durée de la fenêtre de rétention pour chaque point de récupération (en heures). Les machines protégées peuvent être récupérées à tout moment pendant cette fenêtre temporelle.
5. Dans le champ **Fréquence des captures instantanées de cohérence d’application** , spécifiez la fréquence de création des points de récupération contenant des instantanés cohérents au niveau des applications (entre 1 et 12 heures). Hyper-V utilise deux types d’instantanés : un instantané standard qui fournit un instantané incrémentiel de la machine virtuelle complète et un instantané cohérent avec l'application qui prend un instantané des données d'application d'une machine virtuelle. Les instantanés cohérents avec l'application utilisent le service VSS (Volume Shadow Copy Service) pour s'assurer que les applications sont dans un état cohérent lors de la prise des instantanés. Notez que si vous activez les instantanés cohérents avec l'application, cela affectera les performances des applications exécutées sur les machines virtuelles sources. Assurez-vous que la valeur définie est inférieure au nombre de points de récupération supplémentaires que vous configurez.
6. Dans la zone **Heure de début de la réplication initiale**, indiquez à quel moment démarrer la réplication initiale. La réplication se produit via votre bande passante Internet. Il est donc préférable de prévoir son exécution en dehors des heures de bureau. Cliquez ensuite sur **OK**.

    ![Stratégie de réplication](./media/site-recovery-hyper-v-site-to-azure/gs-replication2.png)

Lorsque vous créez une stratégie, elle est automatiquement associée au site Hyper-V. Cliquez sur **OK**. Vous pouvez associer un site Hyper-V (et les machines virtuelles qu’il contient) à plusieurs stratégies de réplication dans **Paramètres** > **Réplication** > nom de la stratégie > **Associate Hyper-V Site** (Associer le site Hyper-V).

## <a name="step-5-capacity-planning"></a>Étape 5 : planifier la capacité
Votre infrastructure de base est désormais configurée. Vous pouvez donc réfléchir à la planification de la capacité et déterminer si des ressources supplémentaires sont nécessaires.

Site Recovery propose une fonctionnalité, Capacity Planner, qui vous permet d’allouer les bonnes ressources pour l’environnement source, les composants de récupération de sites, la mise en réseau et le stockage. Vous pouvez exécuter Capacity Planner en mode rapide, afin d’obtenir une estimation basée sur le nombre moyen de machines virtuelles et de disques ainsi que sur l’espace de stockage disponible, ou en mode détaillé. Dans ce mode, vous saisissez des chiffres au niveau des charges de travail. Avant de commencer, vous devez :

* collecter les informations relatives à votre environnement de réplication, notamment les machines virtuelles, le nombre de disques par machine virtuelle et le stockage par disque ;
* déterminer le taux de modification (l’évolution) quotidienne des données répliquées. Pour cela, vous pouvez utiliser la fonction [Capacity Planner pour réplica Hyper-V](https://www.microsoft.com/download/details.aspx?id=39057) .

1. Cliquez sur **Télécharger** pour télécharger l’outil, puis exécutez-le. [Lisez l’article](site-recovery-capacity-planner.md) relatif à cet outil.
2. Quand vous avez terminé, sélectionnez **Oui** in **Have you run the Capacity Planner (Avez-vous exécuté Capacity Planner ?)**(Avez-vous exécuté Capacity Planner ?)

   ![Planification de la capacité](./media/site-recovery-hyper-v-site-to-azure/gs-capacity-planning.png)

### <a name="network-bandwidth-considerations"></a>Remarques relatives à la bande passante réseau
Vous pouvez utiliser l’outil Capacity Planner pour calculer la bande passante requise par la réplication (réplication initiale, puis delta). Vous disposez de plusieurs options pour déterminer la quantité de bande passante utilisée pour la réplication :

* **Limite de bande passante** : le trafic Hyper-V qui est répliqué vers Azure passe par un hôte Hyper-V spécifique. Vous pouvez limiter la bande passante sur le serveur hôte.
* **Ajustement de la bande passante**: vous pouvez influencer la bande passante utilisée pour la réplication à l’aide de quelques clés de Registre.

#### <a name="throttle-bandwidth"></a>Limiter la bande passante
1. Ouvrez le composant logiciel enfichable MMC de Microsoft Azure Backup sur le serveur hôte Hyper-V. Par défaut, un raccourci vers Microsoft Azure Backup est créé sur le Bureau. Vous pouvez également le trouver ici : C:\Program Files\Microsoft Azure Recovery Services Agent\bin\wabadmin.
2. Dans le composant logiciel enfichable, cliquez sur **Modifier les propriétés**.
3. Dans l’onglet **Limitation**, sélectionnez **Activer la limitation de la bande passante sur Internet pour les opérations de sauvegarde** et définissez les limites relatives aux heures de travail et aux heures non travaillées. Les plages valides vont de 512 Kbits/s à 102 Mbits/s par seconde.

    ![Limiter la bande passante](./media/site-recovery-hyper-v-site-to-azure/throttle2.png)

Vous pouvez également utiliser l’applet de commande [Set-OBMachineSetting](https://technet.microsoft.com/library/hh770409.aspx) pour définir la limitation. Voici un exemple :

    $mon = [System.DayOfWeek]::Monday
    $tue = [System.DayOfWeek]::Tuesday
    Set-OBMachineSetting -WorkDay $mon, $tue -StartWorkHour "9:00:00" -EndWorkHour "18:00:00" -WorkHourBandwidth  (512*1024) -NonWorkHourBandwidth (2048*1024)

**Set-OBMachineSetting -NoThrottle** indique qu’aucune limitation n’est requise.

#### <a name="influence-network-bandwidth"></a>Influer sur la bande passante réseau
1. Dans le registre, accédez à **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Replication**.
   * Pour influer sur le trafic de la bande passante sur un disque de réplication, modifiez la valeur du paramètre **UploadThreadsPerVM**, ou créez la clé si elle n’existe pas.
   * Pour influer sur la bande passante utilisée pour le trafic lié à la restauration automatique à partir d’Azure, modifiez la valeur du paramètre **DownloadThreadsPerVM**.
2. La valeur par défaut est 4. Dans un réseau « surutilisé », ces clés de Registre doivent être modifiées par rapport aux valeurs par défaut. La valeur maximale est de 32. Surveillez le trafic pour optimiser la valeur.

## <a name="step-6-enable-replication"></a>Étape 6 : activer la réplication
À présent, activez la réplication comme suit :

1. Cliquez sur **Étape 2 : Répliquer l’application** > **Source**. Après avoir activé la réplication pour la première fois, cliquez sur l’option **+Répliquer** dans le coffre pour activer la réplication des machines supplémentaires.

    ![Activer la réplication](./media/site-recovery-hyper-v-site-to-azure/enable-replication.png)
2. Dans le panneau **Source**, sélectionnez le site Hyper-V. Cliquez ensuite sur **OK**.
3. Sous **Cible** , sélectionnez l’abonnement de coffre et le modèle de basculement que vous souhaitez utiliser dans Azure (gestion des ressources ou classique) après le basculement.
4. Sélectionnez le compte de stockage que vous souhaitez utiliser. Si vous souhaitez utiliser un compte de stockage différent de ceux dont vous disposez, vous pouvez en [créer un](#set-up-an-azure-storage-account). Pour créer un compte de stockage en mode Resource Manager, cliquez sur **Créer**. Si vous souhaitez créer un compte de stockage en suivant le modèle Classic, vous pouvez utiliser le [Portail Azure](../storage/storage-create-storage-account-classic-portal.md). Cliquez ensuite sur **OK**.
5. Sélectionnez le sous-réseau et le réseau Azure auxquels les machines virtuelles Azure se connectent lorsqu’elles sont démarrées après le basculement. Sélectionnez **Effectuez maintenant la configuration pour les machines sélectionnées** pour appliquer le paramètre réseau à l’ensemble des machines que vous sélectionnez à des fins de protection. Sélectionnez **Configurer ultérieurement** pour sélectionner le réseau Azure pour chaque machine. Si vous souhaitez utiliser un réseau différent de ceux dont vous disposez, [créez-le](#set-up-an-azure-network). Pour créer un réseau en mode Resource Manager, cliquez sur **Créer**.Pour créer un réseau en mode Classic, faites-le dans le [portail Azure](../virtual-network/virtual-networks-create-vnet-classic-pportal.md). Sélectionnez un sous-réseau, le cas échéant. Cliquez ensuite sur **OK**.
   ![Activer la réplication](./media/site-recovery-hyper-v-site-to-azure/enable-replication11.png) 

6. Dans **Machines virtuelles** > **Sélectionner les machines virtuelles** , cliquez sur chaque machine à répliquer. Vous pouvez uniquement sélectionner les machines pour lesquelles la réplication peut être activée. Cliquez ensuite sur **OK**.

    ![Activer la réplication](./media/site-recovery-hyper-v-site-to-azure/enable-replication5-for-exclude-disk.png)
7. Dans **Propriétés** > **Configurer les propriétés**, choisissez le système d’exploitation des machines virtuelles sélectionnées, ainsi que le disque du système d’exploitation. Par défaut, tous les disques de la machine virtuelle sont sélectionnés pour la réplication. Si vous le souhaitez, vous pouvez exclure des disques de la réplication afin de réduire la consommation de bande passante liée à la réplication de données inutiles dans Azure. Par exemple, vous pouvez ne pas répliquer les disques comportant des données temporaires, ou des données actualisées à chaque redémarrage d’une machine ou d’une application (par exemple, pagefile.sys ou tempdb dans Microsoft SQL Server). Vous pouvez exclure un disque de la réplication en le désélectionnant. Vérifiez que le nom de la machine virtuelle Azure (nom de la cible) est conforme à la [configuration requise pour les machines virtuelles Azure](site-recovery-best-practices.md#azure-virtual-machine-requirements) , et modifiez-le si nécessaire. Cliquez ensuite sur **OK**. Vous pouvez opter pour une définition ultérieure des propriétés.

    ![Activer la réplication](./media/site-recovery-hyper-v-site-to-azure/enable-replication6-with-exclude-disk.png)

     > [!NOTE]
     > 
     > * Seuls les disques de base peuvent être exclus de la réplication. Vous ne pouvez pas exclure le disque du système d’exploitation, et il est déconseillé d’exclure les disques dynamiques. ASR ne peut pas identifier quel disque du disque dur virtuel est un disque de base ou un disque dynamique au sein de la machine virtuelle invitée.  Si tous les disques de volume dynamique dépendants ne sont pas exclus, un disque dynamique protégé apparaîtra comme un disque défectueux sur la machine virtuelle de basculement, et les données de ce disque ne seront pas accessibles.  
    > * Une fois la réplication activée, vous ne pouvez pas ajouter ni supprimer de disques pour la réplication. Si vous voulez ajouter ou exclure un disque, vous devez désactiver la protection de la machine virtuelle, puis la réactiver.
    > * Si vous excluez un disque requis pour le bon fonctionnement d’une application, après le basculement vers Azure, vous devez le créer manuellement dans Azure afin que l’application répliquée puisse s’exécuter. Sinon, vous pouvez intégrer Azure 
    > * Automation dans un plan de récupération pour créer le disque pendant le basculement de la machine.
    > * Les disques que vous créez manuellement dans Azure ne seront pas restaurés automatiquement. Par exemple, si vous basculez trois disques et que vous en créez deux directement dans une machine virtuelle Azure, seuls les trois disques qui ont été basculés seront restaurés automatiquement à partir d’Azure sur Hyper-V. Vous ne pouvez pas inclure de disques créés manuellement dans le processus de restauration automatique ou de réplication inverse d’Hyper-V vers Azure.
    >
    >       

8. Dans **Paramètres de réplication** > **Configurer les paramètres de réplication**, sélectionnez la stratégie de réplication que vous souhaitez appliquer aux machines virtuelles protégées. Cliquez ensuite sur **OK**. Vous pouvez modifier la stratégie de réplication dans **Paramètres** > **Stratégies de réplication** > nom de la stratégie > **Modifier les paramètres**. Les modifications que vous appliquez seront utilisées pour les nouvelles machines et les machines dont la réplication est déjà en cours.

   ![Activer la réplication](./media/site-recovery-hyper-v-site-to-azure/enable-replication7.png)

Vous pouvez suivre la progression du travail **Activer la protection** dans **Paramètres** > **Travaux** > **Travaux Site Recovery**. Une fois le travail **Finaliser la protection** exécuté, la machine est prête pour le basculement.

### <a name="view-and-manage-vm-properties"></a>Afficher et gérer les propriétés des machines virtuelles
Nous vous recommandons de vérifier les propriétés de la machine source.

1. Cliquez sur **Paramètres** > **Éléments protégés** > **Éléments répliqués** et sélectionnez la machine.

    ![Activer la réplication](./media/site-recovery-hyper-v-site-to-azure/test-failover1.png)
2. Dans **Propriétés** , vous pouvez afficher les informations sur la réplication et le basculement de la machine virtuelle.

    ![Activer la réplication](./media/site-recovery-hyper-v-site-to-azure/test-failover2.png)
3. Dans **Calcul et réseau** > **Propriétés de calcul**, vous pouvez spécifier la taille de la cible et le nom de la machine virtuelle Azure. Si besoin, modifiez ce nom afin de respecter les exigences d’Azure. Vous pouvez également afficher et modifier les informations sur le réseau cible, le sous-réseau et l’adresse IP qui seront affectés à la machine virtuelle Azure. Notez les points suivants :

   * Vous pouvez définir l’adresse IP cible. Si vous ne fournissez pas d’adresse IP, la machine ayant basculé utilisera le service DHCP. Si vous définissez une adresse qui n’est pas disponible au moment du basculement, ce dernier échoue. Vous pouvez utiliser la même adresse IP cible pour le test de basculement si cette adresse est disponible sur le réseau de test de basculement.
   * Le nombre de cartes réseau est déterminé par la taille spécifiée pour la machine virtuelle cible, comme suit :

     * Si le nombre de cartes réseau sur la machine source est inférieur ou égal au nombre de cartes autorisé pour la taille de la machine cible, la cible présente le même nombre de cartes que la source.
     * Si le nombre de cartes de la machine virtuelle source dépasse la valeur de taille cible autorisée, la taille cible maximale est utilisée.
     * Par exemple, si une machine source présente deux cartes réseau et que la taille de la machine cible en accepte quatre, la machine cible présentera deux cartes. Si la machine source inclut deux cartes, mais que la taille cible prise en charge accepte une seule carte, la machine cible présentera une seule carte.     
     * Si la machine virtuelle possède plusieurs cartes réseau, elles se connectent toutes au même réseau.
     * Si la machine virtuelle possède plusieurs cartes réseau, la première qui s’affiche dans la liste devient la carte réseau *par défaut* dans la machine virtuelle Azure.

     ![Activer la réplication](./media/site-recovery-hyper-v-site-to-azure/test-failover4.png)

4. Les disques de données et du système d’exploitation de la machine virtuelle qui seront répliqués s’affichent dans **Disques** .

## <a name="step-7-test-the-deployment"></a>Étape 7 : Tester le déploiement
Pour tester le déploiement, vous pouvez exécuter un test de basculement pour une seule machine virtuelle, ou un plan de récupération qui contient une ou plusieurs machines virtuelles.

### <a name="prepare-for-test-failover"></a>Préparer un test de basculement
* Pour exécuter un test de basculement, nous vous recommandons de créer un réseau Azure isolé de votre réseau de production Azure (comportement par défaut quand vous créez un réseau dans Azure). [En savoir plus](site-recovery-failover.md#run-a-test-failover) .
* Pour obtenir les meilleures performances possibles lorsque vous effectuez un basculement vers Azure, assurez-vous que vous avez installé l’agent Azure sur l’ordinateur protégé. Cet agent permet de démarrer le système plus rapidement et facilite le dépannage. Installez l’agent [Linux](https://github.com/Azure/WALinuxAgent) ou [Windows](http://go.microsoft.com/fwlink/?LinkID=394789).
* Pour tester entièrement votre déploiement, vous aurez besoin d’une infrastructure pour permettre à la machine répliquée de fonctionner comme prévu. Si vous souhaitez tester Active Directory et DNS, vous pouvez créer une machine virtuelle jouant le rôle de contrôleur de domaine avec DNS, puis la répliquer sur Azure, via Azure Site Recovery. Pour en savoir plus, lisez [Considérations en matière de test de basculement pour Active Directory](site-recovery-active-directory.md#test-failover-considerations).
* Si vous avez exclu des disques de la réplication, vous devrez peut-être créer ces disques manuellement dans Azure après le basculement afin que l’application s’exécute comme prévu.
* Si vous souhaitez exécuter un basculement non planifié au lieu d’un test de basculement, notez les éléments suivants :

  * qu’il est préférable d’arrêter les machines principales avant d’exécuter un basculement non planifié lorsque c’est possible. Vous êtes ainsi sûr que les machines source et les réplicas ne fonctionnent pas en même temps.
  * Lorsque vous effectuez un basculement non planifié, la réplication des données depuis les machines principales s’arrête et les différences dans les données ne sont pas transférées après qu’un basculement non planifié a commencé. En outre, si vous exécutez un basculement non planifié sur un plan de récupération, il sera exécuté jusqu’à la fin, même si une erreur se produit.

### <a name="prepare-to-connect-to-azure-vms-after-failover"></a>Préparer la connexion aux machines virtuelles Azure après le basculement
Si vous souhaitez vous connecter à des machines virtuelles Azure à l’aide de RDP après le basculement, assurez-vous que vous effectuez les opérations suivantes :

**Sur la machine locale, avant le basculement**:

* Pour permettre l’accès par Internet, activez la fonction RDP, vérifiez que les règles TCP et UDP sont ajoutées pour **Public** et assurez-vous que RDP est autorisé dans le champ **Pare-feu Windows** -> **Applications et fonctionnalités autorisées** et ce, pour tous les profils.
* Pour permettre l’accès via une connexion site à site, activez RDP sur la machine, en vérifiant que ce dernier est autorisé dans le champ **Pare-feu Windows** -> **Applications et fonctionnalités autorisées** pour les réseaux de types **Domaine** et **Privé**.
* Installez [l’agent Azure VM](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409) sur la machine locale.
* Vérifiez que la stratégie SAN du système d’exploitation est définie sur la valeur OnlineAll. [En savoir plus](https://support.microsoft.com/kb/3031135)
* Désactivez le service IPSec avant d’exécuter le basculement.

**Sur la machine virtuelle Azure, après le basculement**:

* Ajoutez une adresse IP publique à la carte réseau associée à la machine virtuelle Azure pour autoriser RDP.
* Assurez-vous qu’aucune de vos stratégies de domaine ne vous empêche de vous connecter à une machine virtuelle avec une adresse publique.
* Essayez de vous connecter. Si vous ne pouvez pas vous connecter, vérifiez que la machine virtuelle est en cours d’exécution. Pour accéder à d’autres conseils de dépannage, lisez [cet article](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx).

Si vous souhaitez accéder à une machine virtuelle Azure exécutant Linux après le basculement à l’aide d’un client Secure Shell (ssh), procédez comme suit :

**Sur la machine locale, avant le basculement**:

* Assurez-vous que le service Secure Shell, sur la machine virtuelle Azure, est défini pour démarrer automatiquement au démarrage du système.
* Vérifiez que les règles de pare-feu autorisent une connexion SSH à ce dernier.

**Sur la machine virtuelle Azure, après le basculement**:

* Les règles des groupes de sécurité réseau figurant sur la machine virtuelle basculée et le sous-réseau Azure auquel elle est connectée permettent d’autoriser les connexions entrantes avec le port SSH.
* Un point de terminaison public doit être créé pour autoriser les connexions entrantes sur le port SSH (port TCP 22 par défaut).
* Si la machine virtuelle est accessible via une connexion VPN (Express Route ou VPN de site à site), vous pouvez utiliser le client pour vous connecter directement à la machine virtuelle via SSH.

### <a name="run-a-test-failover"></a>Exécution d’un test de basculement
Pour exécuter le test de basculement, procédez comme suit :

1. Pour effectuer le basculement d’une seule machine virtuelle, dans **Paramètres** > **Éléments répliqués**, cliquez sur la machine virtuelle, puis sur **+Test de basculement**.

    ![Test de basculement](./media/site-recovery-hyper-v-site-to-azure/run-failover1.png)
2. Pour effectuer le basculement d’un plan de récupération, dans **Paramètres** > **Plans de récupération**, cliquez avec le bouton droit sur le plan et sélectionnez **Test de basculement**. Pour créer un plan de récupération, suivez [ces instructions](site-recovery-create-recovery-plans.md).
3. Dans le champ **Test de basculement** , sélectionnez le réseau Azure auquel les machines virtuelles Azure seront connectées après le basculement.

    ![Test de basculement](./media/site-recovery-hyper-v-site-to-azure/run-failover2.png)
4. Cliquez sur **OK** pour commencer le basculement. Vous pouvez suivre la progression du basculement en cliquant sur la machine virtuelle pour ouvrir ses propriétés, ou en sélectionnant la tâche **Test de basculement** dans **Paramètres** > **Travaux Site Recovery**.
5. Lorsque le basculement atteint la phase **Terminer le test** , procédez comme suit :

   1. Examinez la machine virtuelle de réplication dans le portail Microsoft Azure. Vérifiez que la machine virtuelle démarre correctement.
   2. Si vous êtes autorisé à accéder aux machines virtuelles à partir de votre réseau local, vous pouvez initier une connexion Bureau à distance à la machine virtuelle.
   3. Si vous avez exclu des disques de la réplication, vous devrez peut-être créer ces disques manuellement dans Azure après le basculement afin que l’application s’exécute comme prévu.
   4. Cliquez sur **Terminer le test** pour finir l’opération.
   5. Cliquez sur **Notes** pour consigner et enregistrer les éventuelles observations associées au test de basculement.
   6. Cliquez sur **Le test de basculement est terminé**. Nettoyez l’environnement de test pour mettre automatiquement hors tension et supprimer la machine virtuelle de test.
   7. À cette étape, les éléments ou machines virtuelles créés automatiquement par Site Recovery lors du test de basculement sont supprimés. Toutefois, les éléments supplémentaires que vous avez créés pour le test de basculement ne sont pas supprimés.

      > [!NOTE]
      > Si un test de basculement s’étend sur plus de deux semaines, le système le force à se terminer.
      >
      >
6. Une fois le basculement terminé, vous devez également voir la machine Azure de réplication apparaître dans le portail Azure > **Machines virtuelles**. Vous devrez peut-être vous assurer que la machine virtuelle présente la taille appropriée, qu’elle est bien connectée au réseau approprié et qu’elle s’exécute.
7. Si vous avez [préparé les connexions après le basculement](#prepare-to-connect-to-Azure-VMs-after-failover) , vous devez être à même de vous connecter à la machine virtuelle Azure.

## <a name="failover"></a>Basculement
Une fois que vous avez terminé la réplication initiale de vos ordinateurs, vous pouvez appeler des basculements en cas de besoin. Site Recovery prend en charge divers types de basculement : test de basculement, basculement planifié et basculement non planifié.
[En savoir plus](site-recovery-failover.md) sur les différents types de basculement et obtenir une description détaillée de quand et comment les effectuer.

> [!NOTE]
> Si votre intention est de migrer des machines virtuelles vers Azure, nous vous recommandons fortement d’utiliser une [opération de basculement planifié](site-recovery-failover.md#run-a-planned-failover-primary-to-secondary). Une fois que l’application migrée est validée dans Azure à l’aide du test de basculement, procédez comme indiqué sous [Terminer la migration](#Complete-migration-of-your-virtual-machines-to-Azure) pour finaliser la migration de vos machines virtuelles. Il est inutile d’effectuer une validation ou une suppression. La fonction Terminer la migration termine la migration, supprime la protection de la machine virtuelle et arrête la facturation d’Azure Site Recovery pour l’ordinateur.
>
>

### <a name="run-an-unplanned-failover"></a>Exécuter un basculement non planifié
Ce type de basculement doit être choisi lorsqu’un site principal devient inaccessible en raison d’un problème inattendu, tel qu’une attaque de virus ou une panne d’alimentation. Cette procédure explique comment exécuter un test de basculement non planifié pour un plan de récupération. Vous pouvez également exécuter le basculement d’une machine virtuelle unique, via l’onglet Machines virtuelles. Avant de commencer, vérifiez que toutes les machines virtuelles dont vous voulez effectuer le basculement ont terminé la réplication initiale.

1. Sélectionnez **Plans de récupération > nom_planrécupération**.
2. Dans le panneau Plan de récupération, cliquez sur **Basculement non planifié**.
3. Dans la page **Basculement non planifié**, choisissez les emplacements source et cible.
4. Sélectionnez **Arrêter les machines virtuelles et synchroniser les dernières données** pour indiquer que le logiciel Site Recovery doit essayer d’arrêter les machines virtuelles protégées et de synchroniser les données, afin que la dernière version de ces dernières fasse l’objet d’un basculement.
5. Après le basculement, les machines virtuelles présentent un état de validation en attente.  Cliquez sur **Valider** pour valider le basculement.

[En savoir plus](site-recovery-failover.md#run-an-unplanned-failover)

## <a name="complete-migration-of-your-virtual-machines-to-azure"></a>Terminer la migration de vos machines virtuelles vers Azure
> [!NOTE]
> Les étapes suivantes s’appliquent uniquement si vous effectuez la migration des machines virtuelles vers Azure.
>
>

1. Effectuez le basculement planifié comme indiqué [ici](site-recovery-failover.md).
2. Dans **Paramètres > Éléments répliqués**, cliquez sur la machine virtuelle et sélectionnez **Terminer la migration**.

    ![completemigration](./media/site-recovery-hyper-v-site-to-azure/migrate.png)
3. Cliquez sur **OK** pour terminer la migration. Vous pouvez suivre la progression du basculement en cliquant sur la machine virtuelle pour ouvrir ses propriétés ou en sélectionnant le travail Terminer la migration dans **Paramètres > Travaux Site Recovery**.

## <a name="monitor-your-deployment"></a>Surveiller votre déploiement
Voici comment vous pouvez surveiller l’intégrité, l’état et les paramètres de configuration de votre déploiement Site Recovery :

1. Cliquez sur le nom du coffre pour accéder au tableau de bord **Essentials** . Ce tableau affiche l’état de la réplication, les tâches, les plans de récupération, l’intégrité du serveur et les événements Site Recovery.  Vous pouvez personnaliser Essentials de manière afficher les vignettes et les dispositions qui sont les plus utiles, y compris l’état des autres coffres Site Recovery et Backup.

    ![Essentials](./media/site-recovery-hyper-v-site-to-azure/essentials.png)
2. Dans la mosaïque **Intégrité** , vous pouvez surveiller le fonctionnement des serveurs du site qui rencontrent un problème, ainsi que les événements signalés par Site Recovery au cours des dernières 24 heures.
3. Vous pouvez gérer et surveiller la réplication dans les vignettes **Éléments répliqués**, **Plans de récupération** et **Travaux Site Recovery**. Vous pouvez accéder au détail des travaux dans **Paramètres** -> **Travaux** -> **Travaux Site Recovery**.



<!--HONumber=Dec16_HO2-->


