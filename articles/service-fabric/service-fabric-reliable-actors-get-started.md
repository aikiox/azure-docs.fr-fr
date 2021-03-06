---
title: Prise en main de Service Fabric Reliable Actors | Microsoft Docs
description: "Ce didacticiel vous guide dans les étapes de création, de débogage et de déploiement d’un service simple basé sur acteur à l’aide de Service Fabric Reliable Actors."
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: 
ms.assetid: d4aebe72-1551-4062-b1eb-54d83297f139
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/04/2017
ms.author: vturecek
translationtype: Human Translation
ms.sourcegitcommit: 2304a9433bb382c0c7ddcf36733838ac769b9584
ms.openlocfilehash: 98e519df244e9ae934b8100ea9820a7f765b1ee3


---
# <a name="getting-started-with-reliable-actors"></a>Prise en main de Reliable Actors
> [!div class="op_single_selector"]
> * [C# sur Windows](service-fabric-reliable-actors-get-started.md)
> * [Java sur Linux](service-fabric-reliable-actors-get-started-java.md)
> 
> 

Cet article explique les notions de base d’Azure Service Fabric Reliable Actors et vous présente la création, le débogage et le déploiement d’une application Reliable Actor simple dans Visual Studio.

## <a name="installation-and-setup"></a>Installation et configuration
Avant de commencer, assurez-vous d’avoir configuré l’environnement de développement Service Fabric sur votre ordinateur.
Si vous devez le configurer, consultez les instructions détaillées sur la [configuration de l’environnement de développement](service-fabric-get-started.md).

## <a name="basic-concepts"></a>Concepts de base
Pour prendre en main Reliable Actors, il vous suffit de comprendre quelques concepts de base :

* **Service d’acteur**. Les entités Reliable Actors sont empaquetées dans des Reliable Services qui peuvent être déployés dans l’infrastructure Service Fabric. Les instances d’acteur sont activées dans une instance de service nommée.
* **Enregistrement d’acteur**. Comme avec les Reliable Services, un service Reliable Actor doit être enregistré avec le runtime Service Fabric. En outre, le type d’acteur doit être enregistré auprès du runtime de l’acteur.
* **Interface d’acteur**. L’interface d’acteur est utilisée pour définir une interface publique fortement typée d’un acteur. Dans la terminologie du modèle Reliable Actor, l’interface d’acteur définit les types de messages que l’acteur peut comprendre et traiter. L’interface d’acteur est utilisée par d’autres acteurs et applications clientes pour « envoyer » (de façon asynchrone) des messages à l’acteur. Reliable Actors peut implémenter plusieurs interfaces.
* **Classe ActorProxy**. La classe ActorProxy est utilisée par des applications clientes pour appeler les méthodes exposées par le biais de l’interface d’acteur. La classe ActorProxy fournit deux fonctionnalités importantes :
  
  * Résolution de noms : elle est en mesure de localiser l’acteur dans le cluster (en recherchant le nœud du cluster dans lequel il est hébergé).
  * Gestion des défaillances : elle peut retenter les appels de méthode et déterminer de nouveau l’emplacement de l’acteur après une défaillance qui, par exemple, nécessite le déplacement de l’acteur vers un autre nœud du cluster.

Les règles suivantes, qui se rapportent aux interfaces d’acteur, sont importantes :

* Les méthodes d'interface d'acteur ne peuvent pas être surchargées.
* Les méthodes d’interface d'acteur ne doivent pas avoir de paramètres de sortie, de paramètres de référence ou de paramètres facultatifs.
* Les interfaces génériques ne sont pas prises en charge.

## <a name="create-a-new-project-in-visual-studio"></a>Création d'un projet dans Visual Studio
Après avoir installé Service Fabric Tools pour Visual Studio, vous pouvez créer des types de projet. Les nouveaux types de projets se trouvent sous la catégorie **Cloud** de la boîte de dialogue **Nouveau projet**.

![Outils Service Fabric pour Visual Studio - nouveau projet][1]

Dans la boîte de dialogue suivante, vous pouvez choisir le type de projet que vous souhaitez créer.

![Modèles de projets Service Fabric][5]

Pour le projet HelloWorld, nous allons utiliser le service Service Fabric Reliable Actors.

Une fois que vous avez créé la solution, la structure suivante doit s’afficher :

![Structure d’un projet Service Fabric][2]

## <a name="reliable-actors-basic-building-blocks"></a>Blocs de construction de base des acteurs fiables
Une solution Reliable Actors standard est composée de trois projets :

* **Le projet d’application (MyActorApplication)**. Il s’agit du projet qui regroupe tous les services pour le déploiement. Il contient *ApplicationManifest.xml* et les scripts PowerShell pour gérer l’application.
* **Le projet d’interface (MyActor.Interfaces)**. Il s'agit du projet qui contient la définition d'interface de l'acteur. Dans le projet MyActor.Interfaces, vous pouvez définir les interfaces qui seront utilisées par les acteurs de la solution. Les interfaces de l’acteur peuvent être définies dans n’importe quel projet avec n’importe quel nom. Toutefois l’interface définit le contrat de l’acteur qui est partagé par l’implémentation de l’acteur et les clients appelant l’acteur, donc il est généralement justifié de le définir dans un assembly distinct de l’implémentation de l’acteur et qui peut être partagé par plusieurs autres projets.

```csharp
public interface IMyActor : IActor
{
    Task<string> HelloWorld();
}
```

* **Le projet de service de l’acteur (MyActor)**. Il s'agit du projet utilisé pour définir le service Service Fabric qui va héberger l'acteur. Il contient l’implémentation de l’acteur. Une implémentation de l’acteur est une classe qui dérive d’un type de base ( `Actor` ) et implémente les interfaces définies dans le projet MyActor.Interfaces. Une classe d’acteur doit également implémenter un constructeur qui accepte une `ActorService` instance et un `ActorId` et les transmet à la classe de base `Actor`. Cela permet l’injection d’une dépendance de constructeur pour des dépendances de plateforme.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<string> HelloWorld()
    {
        return Task.FromResult("Hello world!");
    }
}
```

Le service de l’acteur doit être enregistré avec un type de service dans le runtime Service Fabric. Afin que le service de l’acteur exécute vos instances d’acteur, le type d’acteur doit également être enregistré auprès du Service de l’acteur. La méthode d’inscription `ActorRuntime` effectue ce travail pour les acteurs.

```csharp
internal static class Program
{
    private static void Main()
    {
        try
        {
            ActorRuntime.RegisterActorAsync<MyActor>(
                (context, actorType) => new ActorService(context, actorType, () => new MyActor())).GetAwaiter().GetResult();

            Thread.Sleep(Timeout.Infinite);
        }
        catch (Exception e)
        {
            ActorEventSource.Current.ActorHostInitializationFailed(e.ToString());
            throw;
        }
    }
}

```

Si vous démarrez à partir d’un nouveau projet dans Visual Studio et que vous n’avez qu’une seule définition d’acteur, l’enregistrement est inclus par défaut dans le code généré par Visual Studio. Si vous définissez d’autres acteurs dans le service, vous devez ajouter l’enregistrement des acteurs à l’aide de ce qui suit :

```csharp
 ActorRuntime.RegisterActorAsync<MyOtherActor>();

```

> [!TIP]
> Le runtime Service Fabric Actors émet des [événements et compteurs de performances liés aux méthodes d’acteur](service-fabric-reliable-actors-diagnostics.md#actor-method-events-and-performance-counters). Ces événements sont utiles dans les diagnostics et la surveillance des performances.
> 
> 

## <a name="debugging"></a>Débogage
Service Fabric Tools pour Visual Studio prend en charge le débogage sur votre ordinateur local. Vous pouvez démarrer une session de débogage en appuyant sur la touche F5. Visual Studio génère (si nécessaire) des packages. En outre, il déploie l’application sur le cluster Service Fabric local et attache le débogueur.

Pendant le processus de déploiement, vous pouvez afficher la progression dans la fenêtre **Sortie** .

![Fenêtre de sortie de débogage Service Fabric][3]

## <a name="next-steps"></a>Étapes suivantes
* [Comment les Acteurs fiables utilisent la plateforme Service Fabric](service-fabric-reliable-actors-platform.md)
* [Gestion des états d’acteur](service-fabric-reliable-actors-state-management.md)
* [Cycle de vie des acteurs et Garbage Collection](service-fabric-reliable-actors-lifecycle.md)
* [Documentation de référence de l’API d’acteur](https://msdn.microsoft.com/library/azure/dn971626.aspx)
* [Exemple de code](https://github.com/Azure/servicefabric-samples)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject.PNG
[2]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-projectstructure.PNG
[3]: ./media/service-fabric-reliable-actors-get-started/debugging-output.PNG
[4]: ./media/service-fabric-reliable-actors-get-started/vs-context-menu.png
[5]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject1.PNG



<!--HONumber=Jan17_HO1-->


