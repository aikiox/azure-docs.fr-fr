---
title: "Déplacer des données vers et à partir de Stockage Blob Azure | Microsoft Docs"
description: "Découvrez comment copier des données d’objets blob dans Azure Data Factory. Les exemples suivants indiquent comment copier des données vers et depuis Azure Blob Storage et Base de données SQL Azure."
keywords: "données d’objets blob, copie d’objet blob azure"
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: monicar
ms.assetid: bec8160f-5e07-47e4-8ee1-ebb14cfb805d
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/12/2016
ms.author: jingwang
translationtype: Human Translation
ms.sourcegitcommit: 701d82971b7da92fb0946cbfc7f708ad32501ef3
ms.openlocfilehash: 69b530c085cc99959e66e75f2c8ebb391c9d26f4


---
# <a name="move-data-to-and-from-azure-blob-using-azure-data-factory"></a>Déplacer des données vers et depuis un objet Blob Azure à l’aide d’Azure Data Factory
Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données vers et à partir d’un objet Blob Azure en les extrayant des données blob provenant d’un autre magasin de données. Cet article s’appuie sur l’article relatif aux [activités de déplacement des données](data-factory-data-movement-activities.md) qui présente une vue d’ensemble du déplacement des données avec l’activité de copie et les combinaisons de magasin de données prises en charge.

## <a name="supported-sources-and-sinks"></a>Sources et récepteurs pris en charge
Consultez le tableau [Banques de données prises en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) pour obtenir la liste des banques de données prises en charge en tant que sources et réceptrices pour l’activité de copie. Vous pouvez déplacer des données à partir de toute banque de données source prise en charge vers le stockage blob Azure ou à partir du stockage blob Azure vers toute banque de données réceptrice prise en charge.

L’activité de copie prend en charge la copie des données depuis/vers des comptes de stockage Azure à usage général et le stockage d’objets blob à chaud/froid. L’activité prend en charge la lecture à partir d’objets blob de blocs, d’annexe ou de page, mais l’écriture vers les objets blob de blocs uniquement.

## <a name="create-pipeline"></a>Création d’un pipeline
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données vers/depuis un stockage blob Azure à l’aide de différents outils/API.  

* Assistant de copie
* Portail Azure
* Visual Studio
* Azure PowerShell
* API .NET
* API REST

Consultez [Didacticiel de l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) pour obtenir des instructions détaillées pour créer un pipeline avec activité de copie de différentes manières.

## <a name="copy-data-wizard"></a>Assistant Copier des données
Le moyen le plus simple de créer un pipeline qui copie les données vers/depuis le stockage d’objets blob Azure (Azure Blob Storage) consiste à utiliser l’Assistant Copier des données. Consultez la page [Didacticiel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Data Factory Copy](data-factory-copy-data-wizard-tutorial.md) pour une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copier des données.

Les exemples suivants présentent des exemples de définitions de JSON que vous pouvez utiliser pour créer un pipeline à l’aide [du Portail Azure](data-factory-copy-activity-tutorial-using-azure-portal.md), [de Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou [d’Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Ils indiquent comment copier des données vers et depuis le stockage d’objets blob Azure et la base de données SQL Azure. Toutefois, les données peuvent être copiées **directement** à partir de n’importe quelle source, vers tout récepteur pris en charge. Pour plus d’informations, consultez la section « Banques de données et formats pris en charge » de l’article [Déplacer des données à l’aide de l’activité de copie](data-factory-data-movement-activities.md).

## <a name="sample-copy-data-from-azure-blob-to-azure-sql"></a>Exemple : copie de données à partir d'un objet Blob Azure vers SQL Azure
L’exemple suivant montre :

1. Un service lié de type [AzureSqlDatabase](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties).
2. Un service lié de type [AzureStorage](#azure-storage-linked-service-properties).
3. Un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureBlob](#azure-blob-dataset-type-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureSqlTable](data-factory-azure-sql-connector.md#azure-sql-dataset-type-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [BlobSource](#azure-blob-copy-activity-type-properties) et [SqlSink](data-factory-azure-sql-connector.md#azure-sql-copy-activity-type-properties).

L’exemple copie des données de série horaire à partir d’un objet blob Azure vers une table SQL Azure, toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

**Service lié SQL Azure :**

```json
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Service lié Azure Storage :**

```json
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory prend en charge deux types de service lié Azure Storage : **AzureStorage** et **AzureStorageSas**. Pour le premier, vous spécifiez la chaîne de connexion qui inclut la clé de compte, et pour le second, vous spécifiez l'Uri de signature d’accès partagé (SAP). Pour plus d’informations, consultez la section [Services liés](#linked-services) .  

**Jeu de données d'entrée d'objet Blob Azure :**

Les données sont récupérées à partir d'un nouvel objet Blob toutes les heures (fréquence : heure, intervalle : 1). Le nom du chemin d'accès et du fichier de dossier pour l'objet blob sont évalués dynamiquement en fonction de l'heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois et le jour de début et le nom de fichier utilise l’heure de début. Le paramètre « external » : « true » informe Data Factory que la table est externe à la Data Factory et non produite par une activité dans la Data Factory.

```json
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/",
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
```
**Jeu de données de sortie SQL Azure :**

L'exemple copie les données dans une table nommée « MyTable » dans une base de données SQL Azure. Créez la table dans votre base de données SQL Azure avec le même nombre de colonnes que le fichier CSV d'objet Blob doit contenir. De nouvelles lignes sont ajoutées à la table toutes les heures.

```json
{
  "name": "AzureSqlOutput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
**Pipeline avec une activité de copie :**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **BlobSource** et le type **sink** est défini sur **SqlSink**.

```json
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
            "name": "AzureSqlOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
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
```
## <a name="sample-copy-data-from-azure-sql-to-azure-blob"></a>Exemple : copie de données à partir de SQL Azure vers un objet Blob Azure
L’exemple suivant montre :

1. Un service lié de type [AzureSqlDatabase](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties).
2. Un service lié de type [AzureStorage](#azure-storage-linked-service-properties).
3. Un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureSqlTable](data-factory-azure-sql-connector.md#azure-sql-dataset-type-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](#azure-blob-dataset-type-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [SqlSource](data-factory-azure-sql-connector.md#azure-sql-copy-activity-type-properties) et [BlobSink](#azure-blob-copy-activity-type-properties).

L’exemple copie des données de série horaire à partir d’une table SQL Azure vers un objet blob Azure, toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

**Service lié SQL Azure :**

```json
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Service lié Azure Storage :**

```json
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
Azure Data Factory prend en charge deux types de service lié Azure Storage : **AzureStorage** et **AzureStorageSas**. Pour le premier, vous spécifiez la chaîne de connexion qui inclut la clé de compte, et pour le second, vous spécifiez l'Uri de signature d’accès partagé (SAP). Pour plus d’informations, consultez la section [Services liés](#linked-services) .  

**Jeu de données d'entrée SQL Azure :**

L'exemple suppose que vous avez créé une table « MyTable » dans SQL Azure et qu'elle contient une colonne appelée « timestampcolumn » pour les données de série chronologique.

La définition de external sur true informe le service Data Factory que la table est externe à la Data Factory et non produite par une activité dans la Data Factory.

```json
{
  "name": "AzureSqlInput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
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
```

**Jeu de données de sortie Azure Blob :**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois, le jour et l’heure de l’heure de début.

```json
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}/",
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
```

**Pipeline avec l'activité de copie :**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **SqlSource** et le type **sink** est défini sur **BlobSink**. La requête SQL spécifiée pour la propriété **SqlReaderQuery** sélectionne les données de la dernière heure à copier.

```json
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[  
              {
                "name": "AzureSQLtoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                  {
                    "name": "AzureSQLInput"
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
```
## <a name="linked-services"></a>Services liés
Dans les exemples, vous avez utilisé un service lié de type **AzureStorage** pour lier un compte de stockage Azure à une fabrique de données. Le tableau suivant fournit la description des éléments JSON spécifiques au service lié Azure Storage.

Il existe deux types de services liés que vous pouvez utiliser pour lier un stockage d'objets blob Azure à une fabrique de données Azure. Il s’agit des services liés **AzureStorage** et **AzureStorageSas**. Le service lié Azure Storage fournit à la fabrique de données un accès global à Azure Storage. Tandis que le service lié Azure de stockage SAP (signature d'accès partagé) fournit à la fabrique de données un accès restreint/limité dans le temps à Azure Storage. Il n'existe aucune différence entre ces deux services liés. Choisissez le service lié qui répond à vos besoins. Les sections suivantes expliquent plus en détail ces deux services liés.

[!INCLUDE [data-factory-azure-storage-linked-services](../../includes/data-factory-azure-storage-linked-services.md)]

## <a name="azure-blob-dataset-type-properties"></a>Propriétés de type du jeu de données d’objet Blob Azure
Dans les exemples, vous avez utilisé un jeu de données de type **AzureBlob** pour représenter un conteneur d’objets blob et un dossier dans un stockage blob Azure.

Pour obtenir une liste complète des sections et propriétés JSON disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l'emplacement, le format, etc. des données dans le magasin de données. La section typeProperties pour le jeu de données de type **AzureBlob** a les propriétés suivantes :

| Propriété | Description | Requis |
| --- | --- | --- |
| folderPath |Chemin d'accès au conteneur et au dossier dans le stockage des objets Blobs. Exemple : monconteneurblob\mondossierblob\ |Oui |
| fileName |Le nom de l’objet Blob. fileName est facultatif et sensible à la casse.<br/><br/>Si vous spécifiez un nom de fichier, l’activité (y compris la copie) fonctionne sur l’objet Blob spécifique.<br/><br/>Lorsque fileName n’est pas spécifié, la copie inclut tous les objets Blob dans le paramètre folderPath du jeu de données d’entrée.<br/><br/>Lorsque fileName n'est pas spécifié pour un jeu de données de sortie, le nom du fichier généré aura ce format dans l'exemple suivant : Data.<Guid>.txt (par exemple : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |Non |
| partitionedBy |partitionedBy est une propriété facultative. Vous pouvez l'utiliser pour spécifier un folderPath dynamique et le nom de fichier pour les données de série chronologique. Par exemple, folderPath peut être paramétré pour toutes les heures de données. Consultez [Utilisation de la section propriété partitionedBy](#using-partitionedBy-property) pour obtenir plus d’informations et des exemples. |Non |
| format |Les types de formats suivants sont pris en charge : **TextFormat**, **AvroFormat**, **JsonFormat**, **OrcFormat**, **ParquetFormat**. Définissez la propriété **type** située sous Format sur l’une de ces valeurs. Pour plus d’informations, consultez les sections [Définition de TextFormat](#specifying-textformat), [Définition d’AvroFormat](#specifying-avroformat), [Définition de JsonFormat](#specifying-jsonformat), [Définition d’OrcFormat](#specifying-orcformat) et [Définition de ParquetFormat](#specifying-parquetformat). Si vous souhaitez copier des fichiers en l’état entre des magasins de fichiers (copie binaire), vous pouvez ignorer la section Format dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| compression |Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate** et **BZip2** ; niveaux pris en charge : **Optimal** et **Fastest** (le plus rapide). Pour plus d’informations, consultez la section [Prise en charge de la compression](#compression-support) . Pour l’instant, les paramètres de compression ne sont pas pris en charge pour les données au format **AvroFormat**, **OrcFormat** ou **ParquetFormat**. Pour ces formats, Data Factory utilise le codec de compression se trouvant dans les métadonnées pour lire les données. Toutefois, lors de l’écriture dans un fichier dans ces formats, Data Factory choisit le code de la compression par défaut pour ce format. Par exemple, SNAPPY pour ParquetFormat et ZLIB pour OrcFormat. |Non |

### <a name="using-partitionedby-property"></a>Utilisation de la propriété partitionedBy
Comme mentionné dans la section précédente, vous pouvez spécifier des valeurs folderPath et filename dynamiques pour les données de série chronologique avec la section **partitionedBy** , les macros Data Factory et les variables système : SliceStart et SliceEnd, qui indiquent les heures de début et de fin pour un segment spécifique de données.

Pour obtenir plus d’informations sur les jeux de données de série chronologique, la planification et les segments, consultez les articles [Création de jeux de données](data-factory-create-datasets.md) et [Planification et exécution](data-factory-scheduling-and-execution.md).

#### <a name="sample-1"></a>Exemple 1

```json
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

Dans cet exemple, {Slice} est remplacé par la valeur de la variable système Data Factory SliceStart au format (AAAAMMJJHH) spécifié. SliceStart fait référence à l'heure de début du segment. folderPath est différent pour chaque segment. Par exemple : wikidatagateway/wikisampledataout/2014100103 ou wikidatagateway/wikisampledataout/2014100104.

#### <a name="sample-2"></a>Exemple 2

```json
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
 [
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```

Dans cet exemple, l'année, le mois, le jour et l'heure de SliceStart sont extraits dans des variables distinctes qui sont utilisées par les propriétés folderPath et fileName.

[!INCLUDE [data-factory-file-format](../../includes/data-factory-file-format.md)]

[!INCLUDE [data-factory-compression](../../includes/data-factory-compression.md)]

## <a name="azure-blob-copy-activity-type-properties"></a>Propriétés de type de l’activité de copie d’objet Blob Azure
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les jeux de données d’entrée et de sortie et les stratégies sont disponibles pour tous les types d’activités.

En revanche, les propriétés disponibles dans la section typeProperties de l'activité varient pour chaque type d'activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs

Si vous déplacez des données à partir d’un stockage blob Azure, vous définissez le type de source dans l’activité de copie sur **BlobSource**. De même, si vous déplacez des données vers un stockage blob Azure, vous définissez le type de récepteur dans l’activité de copie sur **BlobSink**. Cette section fournit une liste de propriétés prises en charge par BlobSource et BlobSink.

**BlobSource** prend en charge les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Requis |
| --- | --- | --- | --- |
| recursive |Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. |True (valeur par défaut), False |Non |

**BlobSink** prend en charge les propriétés suivantes dans la section **typeProperties** :

| Propriété | Description | Valeurs autorisées | Requis |
| --- | --- | --- | --- |
| copyBehavior |Cette propriété définit le comportement de copie lorsque la source est BlobSource ou FileSystem. |**PreserveHierarchy :** conserve la hiérarchie des fichiers dans le dossier cible. Le chemin d’accès relatif du fichier source vers le dossier source est identique au chemin d’accès relatif du fichier cible vers le dossier cible.<br/><br/>**FlattenHierarchy**: tous les fichiers du dossier source figurent dans le premier niveau du dossier cible. Le nom des fichiers cibles est généré automatiquement. <br/><br/>**MergeFiles : (par défaut)** fusionne tous les fichiers du dossier source dans un même fichier. Si le nom de fichier/d’objet blob est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, le nom de fichier est généré automatiquement. |Non |

**BlobSource** prend également en charge ces deux propriétés pour la compatibilité descendante.

* **treatEmptyAsNull**: spécifie s’il faut traiter une chaîne null ou vide en tant que valeur null.
* **skipHeaderLineCount** : spécifie le nombre de lignes à ignorer. Applicable uniquement quand le jeu de données d’entrée utilise TextFormat.

De même, **BlobSink** prend en charge la propriété suivante pour la compatibilité descendante.

* **blobWriterAddHeader**: spécifie s’il faut ajouter un en-tête de définitions de colonne lors de l’écriture dans un jeu de données de sortie.

Les jeux de données prennent désormais en charge les propriétés suivantes qui implémentent la même fonctionnalité : **treatEmptyAsNull**, **skipLineCount**, **firstRowAsHeader**.

Le tableau suivant fournit des recommandations sur l’utilisation de nouvelles propriétés de jeu de données pour remplacer ces propriétés de BlobSink et BlobSource.

| Propriété de l’activité de copie | Propriété du jeu de données |
|:--- |:--- |
| skipHeaderLineCount sur BlobSource |skipLineCount et firstRowAsHeader. Les lignes sont d’abord ignorées, puis la première ligne est lue en tant qu’en-tête. |
| treatEmptyAsNull sur BlobSource |treatEmptyAsNull sur le jeu de données d’entrée |
| blobWriterAddHeader sur BlobSink |firstRowAsHeader sur le jeu de données de sortie |

Consultez la section [Définition de TextFormat](#specifying-textformat) pour plus d’informations sur ces propriétés.    

### <a name="recursive-and-copybehavior-examples"></a>exemples de valeurs recursive et copyBehavior
Cette section décrit le comportement résultant de l’opération de copie pour différentes combinaisons de valeurs recursive et copyBehavior.

| recursive | copyBehavior | Comportement résultant |
| --- | --- | --- |
| true |preserveHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit :  <br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré de la même manière que la source<br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5. |
| true |flattenHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit :  <br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier5 |
| true |mergeFiles |Pour un dossier source nommé Dossier1 et structuré comme suit :  <br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 + Fichier3 + Fichier4 + Fichier5 est fusionné dans un fichier avec le nom de fichier généré automatiquement |
| false |preserveHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit :  <br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré comme suit<br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/><br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |flattenHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré comme suit<br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/><br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |mergeFiles |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré comme suit<br/><br/>Dossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 est fusionné dans un fichier avec le nom de fichier généré automatiquement. nom généré automatiquement pour Fichier1<br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |

[!INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

[!INCLUDE [data-factory-type-conversion-sample](../../includes/data-factory-type-conversion-sample.md)]

[!INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.



<!--HONumber=Nov16_HO3-->


