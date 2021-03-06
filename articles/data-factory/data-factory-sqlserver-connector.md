---
title: "Déplacer des données vers et à partir de SQL Server | Microsoft Docs"
description: "Apprendre à déplacer des données vers/depuis une base de données SQL Server locale ou une machine virtuelle Azure à l’aide d’Azure Data Factory."
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: monicar
ms.assetid: 864ece28-93b5-4309-9873-b095bbe6fedd
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/01/2016
ms.author: jingwang
translationtype: Human Translation
ms.sourcegitcommit: 1b2514e1e6f39bb3ce9d8a46f4af01835284cdcc
ms.openlocfilehash: b473a9e5bc70b9f2b100032b45a0114f8745ab46


---
# <a name="move-data-to-and-from-sql-server-on-premises-or-on-iaas-azure-vm-using-azure-data-factory"></a>Déplacement des données vers et depuis SQL Server local ou sur IaaS (Machine virtuelle Azure) à l’aide d’Azure Data Factory
Cet article explique comment utiliser l’activité de copie pour déplacer des de/vers SQL Server vers un autre magasin de données. Cet article s'appuie sur l'article des [activités de déplacement des données](data-factory-data-movement-activities.md) qui présente une vue d'ensemble du déplacement des données, et les boutiques de données prises en charge comme sources et récepteurs.

## <a name="supported-versions"></a>Versions prises en charge
Ce connecteur SQL Server prend en charge la copie de données à partir de/vers les versions ci-dessous d’une instance hébergée en local ou dans Azure IaaS à l’aide de l’authentification SQL et de l’authentification Windows :

* SQL Server 2016
* SQL Server 2014
* SQL Server 2012
* SQL Server 2008 R2
* SQL Server 2008
* SQL Server 2005

## <a name="create-pipeline"></a>Création d’un pipeline
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données vers/depuis une base de données SQL Server à l’aide de différents outils/API.  

* Assistant de copie
* Portail Azure
* Visual Studio
* Azure PowerShell
* API .NET
* API REST

Consultez [Didacticiel de l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) pour obtenir des instructions détaillées pour créer un pipeline avec activité de copie de différentes manières.

## <a name="enabling-connectivity"></a>Activation de la connectivité
Les concepts et les étapes nécessaires pour la connexion avec SQL Server sont hébergés localement ou dans les machines virtuelles Iaas Azure (Infrastructure-as-a-Service) sont les mêmes. Dans les deux cas, vous devez utiliser la passerelle de gestion des données de connectivité.

Consultez l’article [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md) pour en savoir plus sur la passerelle de gestion des données et obtenir des instructions détaillées sur la configuration de la passerelle. La configuration d’une instance de passerelle est pré requise pour la connexion avec SQL Server.

Vous pouvez installer la passerelle sur la même machine locale ou l’instance de machine virtuelle cloud en tant que serveur SQL Server pour de meilleures performances. Nous recommandons de les installer sur des machines séparées ou les machines virtuelles Cloud. Placer la passerelle et SQL Server sur des ordinateurs distincts réduit les conflits de ressources.

## <a name="copy-data-wizard"></a>Assistant Copier des données
Le moyen le plus simple de créer un pipeline qui copie des données à partir d’une base de données SQL Server vers n’importe quel magasin de données récepteur pris en charge consiste à utiliser l’Assistant Copier des données. Consultez la page [Didacticiel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Data Factory Copy](data-factory-copy-data-wizard-tutorial.md) pour une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copier des données.

L’exemple suivant présente des exemples de définitions de JSON que vous pouvez utiliser pour créer un pipeline à l’aide du [portail Azure](data-factory-copy-activity-tutorial-using-azure-portal.md), de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou [d’Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Les exemples suivants indiquent comment copier des données vers et depuis SQL Server et un système Blob Storage Microsoft Azure. Toutefois, les données peuvent être copiées **directement** vers l’un des récepteurs indiqués [ici](data-factory-data-movement-activities.md#supported-data-stores-and-formats) , via l’activité de copie de Microsoft Azure Data Factory.     

## <a name="sample-copy-data-from-sql-server-to-azure-blob"></a>Exemple : copie de données depuis SQL Server à un objet Blob Azure
L’exemple suivant montre :

1. Service lié de type [OnPremisesSqlServer](data-factory-sqlserver-connector.md#sql-server-linked-service-properties).
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service).
3. Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [SqlServerTable](data-factory-sqlserver-connector.md#sql-server-dataset-type-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [SqlSource](data-factory-sqlserver-connector.md#sql-server-copy-activity-type-properties) et [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties).

L’exemple copie des données appartenant à une série horaire depuis une table SQL Server vers un objet blob Azure toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

Dans un premier temps, configurez la passerelle de gestion des données. Les instructions se trouvent dans l’article [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md) .

**Service SQL Server lié**

    {
      "Name": "SqlServerLinkedService",
      "properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
          "connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;",
          "gatewayName": "<gatewayname>"
        }
      }
    }

**Service lié Azure Blob Storage**

    {
      "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
          "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
      }
    }

**Jeu de données d’entrée de SQL Server**

L’exemple suppose que vous avez créé une table « MyTable » dans SQL Server et qu’elle contient une colonne appelée « timestampcolumn » pour les données de série chronologique. Vous pouvez interroger plusieurs tables au sein d’une même base de données en utilisant un jeu de données unique, mais une seule table doit être utilisée pour la propriété typeProperty tableName du jeu de données.

La définition de external sur true informe le service Data Factory que le jeu de données est externe à la Data Factory et non produite par une activité dans la Data Factory.

    {
      "name": "SqlServerInput",
      "properties": {
        "type": "SqlServerTable",
        "linkedServiceName": "SqlServerLinkedService",
        "typeProperties": {
          "tableName": "MyTable"
        },
        "external": true,
        "availability": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "externalData": {
            "retryInterval": "00:01:00",
            "retryTimeout": "00:10:00",
            "maximumRetry": 3
          }
        }
      }
    }

**Jeu de données de sortie Azure Blob**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois, le jour et l’heure de l’heure de début.

    {
      "name": "AzureBlobOutput",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
          "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
          "partitionedBy": [
            {
              "name": "Year",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "yyyy"
              }
            },
            {
              "name": "Month",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "MM"
              }
            },
            {
              "name": "Day",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "dd"
              }
            },
            {
              "name": "Hour",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "HH"
              }
            }
          ],
          "format": {
            "type": "TextFormat",
            "columnDelimiter": "\t",
            "rowDelimiter": "\n"
          }
        },
        "availability": {
          "frequency": "Hour",
          "interval": 1
        }
      }
    }

**Pipeline avec activité de copie**

Le pipeline contient une activité de copie qui est configurée pour utiliser ces jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **SqlSource** et le type **sink** est défini sur **BlobSink**. La requête SQL spécifiée pour la propriété **SqlReaderQuery** sélectionne les données de la dernière heure à copier.

    {  
        "name":"SamplePipeline",
        "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[  
          {
            "name": "SqlServertoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [
              {
                "name": " SqlServerInput"
              }
            ],
            "outputs": [
              {
                "name": "AzureBlobOutput"
              }
            ],
            "typeProperties": {
              "source": {
                "type": "SqlSource",
                "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
              },
              "sink": {
                "type": "BlobSink"
              }
            },
           "scheduler": {
              "frequency": "Hour",
              "interval": 1
            },
            "policy": {
              "concurrency": 1,
              "executionPriorityOrder": "OldestFirst",
              "retry": 0,
              "timeout": "01:00:00"
            }
          }
         ]
       }
    }


Dans cet exemple, **sqlReaderQuery** est spécifié pour SqlSource. L'activité de copie exécute cette requête dans la source de base de données SQL server pour obtenir les données. Vous pouvez également spécifier une procédure stockée en indiquant **sqlReaderStoredProcedureName** et **storedProcedureParameters** (si la procédure stockée accepte des paramètres). sqlReaderQuery peut faire référence à plusieurs tables de la base de données référencée par le jeu de données d’entrée. Cela n’est pas limité à la seule table définie en tant que typeProperty tableName du jeu de données.

Si vous ne spécifiez pas sqlReaderQuery ou sqlReaderStoredProcedureName, les colonnes définies dans la section Structure sont utilisées pour sélectionner une requête à exécuter dans l'Azure SQL Database. Si la définition du jeu de données ne possède pas de structure, toutes les colonnes de la table sont sélectionnées.

Consultez la section [Sql Source](#sqlsource) et [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) pour obtenir la liste des propriétés prises en charge par SqlSource et BlobSink.

## <a name="sample-copy-data-from-azure-blob-to-sql-server"></a>Exemple : copie de données à partir d’un objet Blob Azure vers SQL Server
L’exemple suivant montre :

1. un service lié de type [OnPremisesSqlServer](data-factory-sqlserver-connector.md#sql-server-linked-service-properties);
2. un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service) ;
3. un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties).
4. un [jeu de données](data-factory-create-datasets.md) de sortie de type [SqlServerTable](data-factory-sqlserver-connector.md#sql-server-dataset-type-properties) ;
5. un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [BlobSource](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) et [SqlSink](data-factory-sqlserver-connector.md#sql-server-copy-activity-type-properties).

L'exemple copie des données de série horaire à partir d'un objet Blob Azure vers une table dans une base de données SQL Azure, toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

**Service SQL Server lié**

    {
      "Name": "SqlServerLinkedService",
      "properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
          "connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=False;User ID=<username>;Password=<password>;",
          "gatewayName": "<gatewayname>"
        }
      }
    }

**Service lié Azure Blob Storage**

    {
      "name": "StorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
          "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
      }
    }

**Jeu de données d'entrée d'objet Blob Azure**

Les données sont récupérées à partir d’un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le nom du chemin d'accès et du fichier de dossier pour l'objet blob sont évalués dynamiquement en fonction de l'heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois et le jour de l’heure de début et le nom de fichier utilise la partie heure de l’heure de début. Le paramètre « external » : « true » informe le service Data Factory que ce jeu de données est externe à la Data Factory et non produit par une activité dans la Data Factory.

    {
      "name": "AzureBlobInput",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
          "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
          "fileName": "{Hour}.csv",
          "partitionedBy": [
            {
              "name": "Year",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "yyyy"
              }
            },
            {
              "name": "Month",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "MM"
              }
            },
            {
              "name": "Day",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "dd"
              }
            },
            {
              "name": "Hour",
              "value": {
                "type": "DateTime",
                "date": "SliceStart",
                "format": "HH"
              }
            }
          ],
          "format": {
            "type": "TextFormat",
            "columnDelimiter": ",",
            "rowDelimiter": "\n"
          }
        },
        "external": true,
        "availability": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "externalData": {
            "retryInterval": "00:01:00",
            "retryTimeout": "00:10:00",
            "maximumRetry": 3
          }
        }
      }
    }

**Jeu de données de sortie de SQL Server**

L’exemple copie les données dans une table nommée « MyTable » dans SQL Server. Créez la table dans SQL Server avec le même nombre de colonnes que le fichier CSV d’objets Blob doit contenir. De nouvelles lignes sont ajoutées à la table toutes les heures.

    {
      "name": "SqlServerOutput",
      "properties": {
        "type": "SqlServerTable",
        "linkedServiceName": "SqlServerLinkedService",
        "typeProperties": {
          "tableName": "MyOutputTable"
        },
        "availability": {
          "frequency": "Hour",
          "interval": 1
        }
      }
    }

**Pipeline avec activité de copie**

Le pipeline contient une activité de copie qui est configurée pour utiliser ces jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **BlobSource** et le type **sink** est défini sur **SqlSink**.

    {  
        "name":"SamplePipeline",
        "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":[  
          {
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [
              {
                "name": "AzureBlobInput"
              }
            ],
            "outputs": [
              {
                "name": " SqlServerOutput "
              }
            ],
            "typeProperties": {
              "source": {
                "type": "BlobSource",
                "blobColumnSeparators": ","
              },
              "sink": {
                "type": "SqlSink"
              }
            },
           "scheduler": {
              "frequency": "Hour",
              "interval": 1
            },
            "policy": {
              "concurrency": 1,
              "executionPriorityOrder": "OldestFirst",
              "retry": 0,
              "timeout": "01:00:00"
            }
          }
          ]
       }
    }

## <a name="sql-server-linked-service-properties"></a>Propriétés du service lié SQL Server
Dans les exemples, vous avez utilisé un service lié de type **OnPremisesSqlServer** pour lier une base de données SQL Server locale à une fabrique de données. Le tableau suivant fournit la description des éléments JSON spécifiques au service lié SQL Server local.

Le tableau suivant fournit la description des éléments JSON spécifiques au service lié SQL Server.

| Propriété | Description | Requis |
| --- | --- | --- |
| type |Le type de propriété doit être défini sur **OnPremisesSqlServer**. |Oui |
| connectionString |Spécifiez les informations connectionString nécessaires pour connecter la base de données SQL Server locale à l’aide de l’authentification SQL ou de l’authentification Windows. |Oui |
| gatewayName |Nom de la passerelle que le service Data Factory doit utiliser pour se connecter à la base de données SQL Server locale. |Oui |
| username |Spécifiez le nom d’utilisateur si vous utilisez l’authentification Windows. Exemple : **domainname\\username**. |Non |
| password |Spécifiez le mot de passe du compte d’utilisateur que vous avez spécifié pour le nom d’utilisateur. |Non |

Vous pouvez chiffrer les informations d’identification à l’aide de l’applet de commande **New-AzureRmDataFactoryEncryptValue** et les utiliser dans la chaîne de connexion comme indiqué dans l’exemple suivant (propriété **EncryptedCredential**) :  

    "connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",

### <a name="samples"></a>Exemples
**JSON pour utilisation de l’authentification SQL**

    {
        "name": "MyOnPremisesSQLDB",
        "properties":
        {
            "type": "OnPremisesSqlLinkedService",
            "typeProperties": {
                "connectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=False;User ID=<username>;Password=<password>;",
                "gatewayName": "<gateway name>"
            }
        }
    }

**JSON pour utilisation de l’authentification Windows**

Si le nom d’utilisateur et le mot de passe sont spécifiés, la passerelle les utilise pour identifier le compte utilisateur spécifié pour connecter la base de données SQL Server locale. Dans le cas contraire, la passerelle se connecte directement à SQL Server avec le contexte de sécurité de la passerelle (son compte de démarrage).

    {
         "Name": " MyOnPremisesSQLDB",
         "Properties":
         {
             "type": "OnPremisesSqlLinkedService",
             "typeProperties": {
                 "ConnectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=True;",
                 "username": "<domain\\username>",
                 "password": "<password>",
                 "gatewayName": "<gateway name>"
            }
         }
    }


Pour plus d’informations sur la définition des informations d’identification pour une source de données SQL Server, consultez [Configuration des informations d’identification et de la sécurité](data-factory-data-management-gateway.md#encrypting-credentials).

## <a name="sql-server-dataset-type-properties"></a>Propriétés de type du jeu de données SQL Server
Dans les exemples, vous avez utilisé un jeu de données de type **SqlServerTable** pour représenter une table dans une base de données SQL Server.  

Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d’un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Server, objet Blob Azure, table Azure, etc.).

La section typeProperties est différente pour chaque type de jeu de données et fournit des informations sur l'emplacement des données dans le magasin de données. La section **typeProperties** pour le jeu de données de type **SqlServerTable** a les propriétés suivantes :

| Propriété | Description | Requis |
| --- | --- | --- |
| TableName |Nom de la table ou de la vue dans l’instance de base de données SQL Server à laquelle le service lié fait référence. |Oui |

## <a name="sql-server-copy-activity-type-properties"></a>Propriétés de type de l’activité de copie SQL Server
Si vous déplacez des données à partir d’une base de données SQL Server, vous définissez le type de source dans l’activité de copie sur **SqlSource**. De même, si vous déplacez des données vers une base de données SQL Server, vous définissez le type de récepteur dans l’activité de copie sur **SqlSink**. Cette section fournit une liste de propriétés prises en charge par SqlSource et SqlSink.

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d'entrée et de sortie et les différentes stratégies sont disponibles pour tous les types d'activités.

> [!NOTE]
> L'activité de copie accepte uniquement une entrée et produit une seule sortie.
>
>

En revanche, les propriétés disponibles dans la section typeProperties de l'activité varient pour chaque type d'activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

### <a name="sqlsource"></a>SqlSource
Lorsqu’une source dans une activité de copie est de type **SqlSource**, les propriétés suivantes sont disponibles dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Requis |
| --- | --- | --- | --- |
| SqlReaderQuery |Utilise la requête personnalisée pour lire des données. |Chaîne de requête SQL. Par exemple : select * from MyTable. Peut faire référence à plusieurs tables de la base de données référencée par le jeu de données d’entrée. S’il n’est pas spécifié, l’instruction SQL est exécutée : select from MyTable. |Non |
| sqlReaderStoredProcedureName |Nom de la procédure stockée qui lit les données de la table source. |Nom de la procédure stockée. |Non |
| storedProcedureParameters |Paramètres de la procédure stockée. |Paires nom/valeur. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. |Non |

Si **sqlReaderQuery** est spécifié pour la SqlSource, l'activité de copie exécute cette requête dans la source Azure SQL Database pour obtenir les données.

Vous pouvez également spécifier une procédure stockée en indiquant **sqlReaderStoredProcedureName** et **storedProcedureParameters** (si la procédure stockée accepte des paramètres).

Si vous ne spécifiez pas sqlReaderQuery ou sqlReaderStoredProcedureName, les colonnes définies dans la section structure du code JSON du jeu de données sont utilisées pour créer une requête à exécuter sur Azure SQL Database. Si la définition du jeu de données ne possède pas de structure, toutes les colonnes de la table sont sélectionnées.

> [!NOTE]
> Quand vous utilisez **sqlReaderStoredProcedureName**, vous devez toujours spécifier une valeur pour la propriété **tableName** du code JSON du jeu de données. Cependant, il n’existe aucune validation effectuée pour cette table.
>
>

### <a name="sqlsink"></a>SqlSink
**SqlSink** prend en charge les propriétés suivantes :

| Propriété | Description | Valeurs autorisées | Requis |
| --- | --- | --- | --- |
| writeBatchTimeout |Temps d’attente pour que l’opération d’insertion de lot soit terminée avant d’expirer. |intervalle de temps<br/><br/>  Exemple : « 00:30:00 » (30 minutes). |Non |
| writeBatchSize |Insère des données dans la table SQL lorsque la taille du tampon atteint writeBatchSize |Nombre entier (nombre de lignes) |Non (valeur par défaut : 10000) |
| sqlWriterCleanupScript |Spécifiez une requête pour exécuter l'activité de copie de sorte que les données d'un segment spécifique sont nettoyées. Consultez la [section sur la répétition](#repeatability-during-copy) pour plus de détails. |Une instruction de requête. |Non |
| sliceIdentifierColumnName |Spécifiez le nom de la colonne que l’activité de copie doit remplir avec l’identificateur de segment généré automatiquement, et qui est utilisée pour nettoyer les données d’un segment spécifique lors de la réexécution. Consultez la [section sur la répétition](#repeatability-during-copy) pour plus de détails. |Nom d’une colonne avec le type de données binary(32). |Non |
| sqlWriterStoredProcedureName |Nom de la procédure stockée qui met à jour/insère les données dans la table cible. |Nom de la procédure stockée. |Non |
| storedProcedureParameters |Paramètres de la procédure stockée. |Paires nom/valeur. Les noms et la casse des paramètres doivent correspondre aux noms et à la casse des paramètres de la procédure stockée. |Non |
| sqlWriterTableType |Spécifiez le nom du type de table à utiliser dans la procédure stockée. L’activité de copie place les données déplacées disponibles dans une table temporaire avec ce type de table. Le code de procédure stockée peut ensuite fusionner les données copiées avec les données existantes. |Nom de type de table. |Non |

## <a name="troubleshooting-connection-issues"></a>Résolution des problèmes de connexion
1. Configurez votre serveur SQL Server pour qu’il accepte les connexions à distance. Démarrez **SQL Server Management Studio**, cliquez avec le bouton droit de la souris sur **Serveur**, puis cliquez sur **Propriétés**. Sélectionnez **Connexions** dans la liste, puis cochez **Autoriser les accès distants à ce serveur**.

    ![Activation des connexions à distance](.\\media\\data-factory-sqlserver-connector\\AllowRemoteConnections.png)

    Vous trouverez la procédure détaillée à la page [Configurer l’option de configuration du serveur remote access](https://msdn.microsoft.com/library/ms191464.aspx) .
2. Lancez le **Gestionnaire de configuration SQL Server**. Développez **Configuration du réseau SQL Server** pour l’instance souhaitée, puis sélectionnez **Protocoles pour MSSQLSERVER**. Les protocoles doivent s’afficher dans le volet droit. Activez TCP/IP en cliquant avec le bouton droit sur **TCP/IP** et en cliquant sur **Activer**.

    ![Activation de TCP/IP](.\\media\\data-factory-sqlserver-connector\\EnableTCPProptocol.png)

    Consultez la page [Activer ou désactiver un protocole réseau de serveur](https://msdn.microsoft.com/library/ms191294.aspx) pour obtenir des détails et découvrir d’autres façons d’activer le protocole TCP/IP.
3. Dans la même fenêtre, double-cliquez sur **TCP/IP** pour lancer la fenêtre des **propriétés de TCP/IP**.
4. Allez sous l’onglet **Adresses IP** . Faites défiler l’écran vers le bas jusqu’à la section **IPAll** . Notez le **Port TCP** (**1433**, par défaut).
5. Créez une **règle de Pare-feu Windows** sur l’ordinateur pour autoriser le trafic à entrer par ce port.  
6. **Vérifiez la connexion**: servez-vous de SQL Server Management Studio sur un autre ordinateur pour vous connecter à SQL Server en utilisant un nom qualifié complet. Par exemple : « <machine>.<domain>.corp.<company>.com, 1433 ».

   > [!IMPORTANT]

   > Pour plus d’informations, consultez [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md).
   >
   > Consultez [Résolution des problèmes de passerelle](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) pour obtenir des conseils sur la résolution des problèmes de connexion/passerelle.
   >
   >


## <a name="identity-columns-in-the-target-database"></a>Colonnes d’identité dans la base de données cible
Cette section fournit un exemple qui copie des données d’une table source sans colonne d’identité vers une table de destination avec une colonne d’identité.

**Table source :**

    create table dbo.SourceTbl
    (
           name varchar(100),
           age int
    )

**Table de destination :**

    create table dbo.TargetTbl
    (
           id int identity(1,1),
           name varchar(100),
           age int
    )


Notez que la table cible possède une colonne d’identité.

**Définition du jeu de données JSON source**

    {
        "name": "SampleSource",
        "properties": {
            "published": false,
            "type": " SqlServerTable",
            "linkedServiceName": "TestIdentitySQL",
            "typeProperties": {
                "tableName": "SourceTbl"
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            },
            "external": true,
            "policy": {}
        }
    }

**Définition de jeu de données JSON de destination**

    {
        "name": "SampleTarget",
        "properties": {
            "structure": [
                { "name": "name" },
                { "name": "age" }
            ],
            "published": false,
            "type": "AzureSqlTable",
            "linkedServiceName": "TestIdentitySQLSource",
            "typeProperties": {
                "tableName": "TargetTbl"
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            },
            "external": false,
            "policy": {}
        }
    }


Notez que vos tables source et cible ont des schémas différents (la cible possède une colonne supplémentaire avec identité). Dans ce scénario, vous devez spécifier la propriété **structure** dans la définition du jeu de données cible, qui n’inclut pas la colonne d’identité.

Ensuite, vous mappez les colonnes à partir du jeu de données source vers les colonnes du jeu de données de destination. Consultez la section [Exemples de mappages de colonnes](#column-mapping-samples) pour obtenir un exemple.

[!INCLUDE [data-factory-type-repeatability-for-sql-sources](../../includes/data-factory-type-repeatability-for-sql-sources.md)]

[!INCLUDE [data-factory-sql-invoke-stored-procedure](../../includes/data-factory-sql-invoke-stored-procedure.md)]

[!INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

### <a name="type-mapping-for-sql-server--azure-sql"></a>Mappage de type pour SQL Server et Azure SQL
Comme mentionné dans l’article consacré aux [activités de déplacement des données](data-factory-data-movement-activities.md) , l’activité de copie convertit automatiquement des types source en types récepteur à l’aide de l’approche en 2 étapes suivante :

1. Conversion de types natifs source en types .NET
2. Conversion de types .NET en types récepteur natifs

Lors du déplacement de données vers et à partir de SQL Azure, SQL Server, Sybase, les mappages suivants sont utilisés à partir du type SQL en type .NET et vice versa.

Le mappage est identique au mappage du type de données SQL Server pour ADO.NET.

| Type de moteur de base de données SQL Server | Type de .NET Framework |
| --- | --- |
| bigint |Int64 |
| binaire |Byte[] |
| bit |Boolean |
| char |String, Char[] |
| date |DateTime |
| DateTime |DateTime |
| datetime2 |DateTime |
| Datetimeoffset |Datetimeoffset |
| Décimal |Décimal |
| Attribut FILESTREAM (varbinary(max)) |Byte[] |
| Float |Double |
| image |Byte[] |
| int |Int32 |
| money |Décimal |
| nchar |String, Char[] |
| ntext |String, Char[] |
| numérique |Décimal |
| nvarchar |String, Char[] |
| real |Single |
| rowversion |Byte[] |
| smalldatetime |DateTime |
| smallint |Int16 |
| smallmoney |Décimal |
| sql_variant |Objet * |
| texte |String, Char[] |
| time |intervalle de temps |
| timestamp |Byte[] |
| tinyint |Byte |
| uniqueidentifier |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| xml |xml |

[!INCLUDE [data-factory-type-conversion-sample](../../includes/data-factory-type-conversion-sample.md)]

[!INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.



<!--HONumber=Nov16_HO3-->


