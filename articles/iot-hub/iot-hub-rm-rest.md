---
title: "Création d’un IoT Hub Azure à l’aide de l’API REST du fournisseur de ressources | Microsoft Docs"
description: "Utilisation de l’API REST du fournisseur de ressources pour créer un IoT Hub."
services: iot-hub
documentationcenter: .net
author: dominicbetts
manager: timlt
editor: 
ms.assetid: 52814ee5-bc10-4abe-9eb2-f8973096c2d8
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/16/2016
ms.author: dobett
translationtype: Human Translation
ms.sourcegitcommit: 2abfeebeac222f4371b0945e1aeb6fcf8e51595d
ms.openlocfilehash: 826e359ebeaf9af4df3c3b1559549a57e8228f72


---
# <a name="create-an-iot-hub-using-the-resource-provider-rest-api-net"></a>Création d’un IoT Hub à l’aide de l’API REST du fournisseur de ressources (.NET)
[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## <a name="introduction"></a>Introduction
Vous pouvez utiliser [l’API REST du fournisseur de ressources IoT Hub][lnk-rest-api] pour créer et gérer des IoT Hubs Azure par programme. Ce didacticiel vous montre comment utiliser l’API REST du fournisseur de ressources IoT Hub pour créer un IoT Hub à partir d’un programme C#.

> [!NOTE]
> Azure dispose de deux modèles de déploiement pour créer et utiliser des ressources : [Azure Resource Manager et classique](../azure-resource-manager/resource-manager-deployment-model.md).  Cet article traite de l’utilisation du modèle de déploiement Azure Resource Manager.
> 
> 

Pour réaliser ce didacticiel, vous avez besoin des éléments suivants :

* Microsoft Visual Studio 2015.
* Un compte Azure actif. <br/>Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit][lnk-free-trial] en quelques minutes.
* [Azure PowerShell 1.0][lnk-powershell-install] ou version ultérieure.

[!INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## <a name="prepare-your-visual-studio-project"></a>Préparer votre projet Visual Studio
1. Dans Visual Studio, créez un projet Windows Visual C# à l'aide du modèle de projet d' **application de console** . Nommez ce projet **CreateIoTHubREST**.
2. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur votre projet, puis cliquez sur **GÉRER LES PACKAGES NUGET**.
3. Dans le gestionnaire de packages NuGet, activez **Inclure la version préliminaire** et recherchez **Microsoft.Azure.Management.ResourceManager**. Cliquez sur **Installer**. Sous **Réviser les modifications**, cliquez sur **OK**, puis sur **J’accepte** pour accepter les licences.
4. Dans le gestionnaire de packages NuGet, recherchez **Microsoft.IdentityModel.Clients.ActiveDirectory**.  Cliquez sur **Installer**, dans **Réviser les modifications**, cliquez sur **OK**, puis cliquez sur **J’accepte** pour accepter la licence.
5. Dans Program.cs, remplacez les instructions **using** existantes par les instructions suivantes :
   
    ```
    using System;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Text;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.ResourceManager.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json;
    using Microsoft.Rest;
    using System.Linq;
    using System.Threading;
    ```
6. Dans Program.cs, ajoutez les variables statiques suivantes en remplaçant les valeurs des espaces réservés. Vous avez noté les éléments **ApplicationId**, **SubscriptionId**, **TenantId** et **Password** précédemment dans ce didacticiel. **Nom du groupe de ressources** est le nom du groupe de ressources que vous utiliserez pour créer le hub IoT ; il peut s’agit d’un groupe de ressources existant ou nouveau. **Nom de l’IoT Hub** est le nom de l’IoT Hub que vous allez créer, par exemple **MonHubIoT** (ce nom doit être globalement unique et inclure votre nom ou vos initiales). **Nom du déploiement** est le nom du déploiement, par exemple **Déploiement_01**.
   
    ```
    static string applicationId = "{Your ApplicationId}";
    static string subscriptionId = "{Your SubscriptionId}";
    static string tenantId = "{Your TenantId}";
    static string password = "{Your application Password}";
   
    static string rgName = "{Resource group name}";
    static string iotHubName = "{IoT Hub name including your initials}";
    ```

[!INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## <a name="use-the-resource-provider-rest-api-to-create-an-iot-hub"></a>Utilisation de l’API REST du fournisseur de ressources pour créer un IoT Hub
Utilisez [l’API REST du fournisseur de ressources IoT Hub][lnk-rest-api] pour créer un IoT Hub dans votre groupe de ressources. Vous pouvez également utiliser l’API REST du fournisseur de ressources pour apporter des modifications à un IoT Hub existant.

1. Ajoutez la méthode suivante au fichier Program.cs :
   
    ```
    static void CreateIoTHub(string token)
    {
   
    }
    ```
2. Ajoutez le code suivant à la méthode **CreateIoTHub** pour créer un objet **HttpClient** avec un jeton d’authentification dans les en-têtes :
   
    ```
    HttpClient client = new HttpClient();
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
    ```
3. Ajoutez le code suivant à la méthode **CreateIoTHub** pour décrire le IoT Hub à créer et générer une représentation JSON. Pour obtenir la liste des emplacements qui prennent en charge IoT Hub, consultez [Statut Azure][lnk-status] :
   
    ```
    var description = new
    {
      name = iotHubName,
      location = "East US",
      sku = new
      {
        name = "S1",
        tier = "Standard",
        capacity = 1
      }
    };
   
    var json = JsonConvert.SerializeObject(description, Formatting.Indented);
    ```
4. Ajoutez le code suivant à la méthode **CreateIoTHub** pour envoyer la demande REST à Azure, vérifiez la réponse et récupérez l’URL que vous pouvez utiliser pour surveiller l'état de la tâche de déploiement :
   
    ```
    var content = new StringContent(JsonConvert.SerializeObject(description), Encoding.UTF8, "application/json");
    var requestUri = string.Format("https://management.azure.com/subscriptions/{0}/resourcegroups/{1}/providers/Microsoft.devices/IotHubs/{2}?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var result = client.PutAsync(requestUri, content).Result;
   
    if (!result.IsSuccessStatusCode)
    {
      Console.WriteLine("Failed {0}", result.Content.ReadAsStringAsync().Result);
      return;
    }
   
    var asyncStatusUri = result.Headers.GetValues("Azure-AsyncOperation").First();
    ```
5. Ajoutez le code suivant à la fin de la méthode **CreateIoTHub** pour utiliser l’adresse **asyncStatusUri** récupérée à l’étape précédente pour attendre la fin de la tâche de déploiement :
   
    ```
    string body;
    do
    {
      Thread.Sleep(10000);
      HttpResponseMessage deploymentstatus = client.GetAsync(asyncStatusUri).Result;
      body = deploymentstatus.Content.ReadAsStringAsync().Result;
    } while (body == "{\"status\":\"Running\"}");
    ```
6. Ajoutez le code suivant à la fin de la méthode **CreateIoTHub** pour récupérer les clés de l’IoT Hub que vous avez créé et les imprimer sur la console :
   
    ```
    var listKeysUri = string.Format("https://management.azure.com/subscriptions/{0}/resourceGroups/{1}/providers/Microsoft.Devices/IotHubs/{2}/IoTHubKeys/listkeys?api-version=2016-02-03", subscriptionId, rgName, iotHubName);
    var keysresults = client.PostAsync(listKeysUri, null).Result;
   
    Console.WriteLine("Keys: {0}", keysresults.Content.ReadAsStringAsync().Result);
    ```

## <a name="complete-and-run-the-application"></a>Terminer et exécuter l’application
Vous pouvez maintenant terminer l’application en appelant la méthode **CreateIoTHub** avant sa génération et son exécution.

1. Ajoutez le code suivant à la fin de la méthode **Main** :
   
    ```
    CreateIoTHub(token.AccessToken);
    Console.ReadLine();
    ```
2. Cliquez sur **Build**, puis sur **Générer la solution**. Corrigez les éventuelles erreurs.
3. Cliquez sur **Déboguer**, puis **Démarrer le débogage** pour exécuter l’application. Le déploiement peut prendre plusieurs minutes.
4. Vous pouvez vérifier que votre application a ajouté le nouvel IoT Hub en accédant au [portail Azure][lnk-azure-portal] et en affichant votre liste de ressources, ou en utilisant l’applet de commande PowerShell **Get-AzureRmResource**.

> [!NOTE]
> Cet exemple d’application ajoute un IoT Hub S1 Standard qui vous est facturé. Lorsque vous avez terminé, vous pouvez supprimer le IoT Hub par le biais du [portail Azure][lnk-azure-portal] ou à l’aide de l’applet de commande PowerShell **Remove-AzureRmResource**.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez déployé un IoT Hub à l’aide de l’API REST du fournisseur de ressources, vous pouvez aller encore plus loin :

* Découvrez les capacités de [l’API REST du fournisseur de ressources IoT Hub][lnk-rest-api].
* Pour plus d’informations sur les capacités d’Azure Resource Manager, voir [Vue d’ensemble d’Azure Resource Manager][lnk-azure-rm-overview].

Pour en savoir plus sur le développement pour IoT Hub, consultez les pages suivantes :

* [Présentation du Kit de développement logiciel (SDK) C][lnk-c-sdk]
* [Kits de développement logiciel (SDK) Azure IoT][lnk-sdks]

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Simulation d’un appareil avec le Kit de développement logiciel (SDK) de la passerelle IoT][lnk-gateway]

<!-- Links -->
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-azure-portal]: https://portal.azure.com/
[lnk-status]: https://azure.microsoft.com/status/
[lnk-powershell-install]: /powershell/azureps-cmdlets-docs
[lnk-rest-api]: https://msdn.microsoft.com/library/mt589014.aspx
[lnk-azure-rm-overview]: ../azure-resource-manager/resource-group-overview.md

[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-gateway]: iot-hub-linux-gateway-sdk-simulated-device.md



<!--HONumber=Dec16_HO1-->


