---
title: "Qu’est-ce qu’Azure Relay ? | Microsoft Docs"
description: "Présentation d’Azure Relay"
services: service-bus-relay
documentationcenter: .net
author: banisadr
manager: timlt
editor: 
ms.assetid: 1e3e971d-2a24-4f96-a88a-ce3ea2b1a1cd
ms.service: service-bus-relay
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: get-started-article
ms.date: 11/16/2016
ms.author: babanisa,sethm
translationtype: Human Translation
ms.sourcegitcommit: 14a5c029eef714134079b960109e58d821d0a609
ms.openlocfilehash: 34b54d54b20d9aec69486c5ef98c97beee21bd94


---
# <a name="what-is-azure-relay"></a>Qu’est-ce qu’Azure Relay ?
Le service Azure Relay facilite les applications hybrides en offrant la possibilité d’exposer les services qui résident dans un réseau d’entreprise sur le cloud public en toute sécurité, sans avoir à ouvrir une connexion de pare-feu ni à exiger des modifications intrusives dans une infrastructure de réseau d’entreprise. Azure Relay prend en charge une grande variété de protocoles de transport et normes de services web.

Le service de relais prend en charge le trafic unidirectionnel standard, le trafic de demande/réponse et le trafic d’homologue à homologue. Il prend également en charge la distribution des événements sur Internet pour activer les scénarios de publication/abonnement et la communication par socket bidirectionnelle pour une efficacité accrue de point à point. 

Dans le modèle de transfert de données par relais, un service local se connecte au service de relais via un port sortant et crée un socket bidirectionnel pour la communication liée à des adresses de rendez-vous spécifiques. Le client peut ensuite communiquer avec le service local en envoyant le trafic vers le service de relais ciblant l’adresse de rendez-vous. Le service de relais « relaie » ensuite les données au service local via le socket bidirectionnel dédié à chaque client. Le client n’a pas besoin d’une connexion directe au service local ni de savoir où se trouve le service, et le service local n’a pas besoin d’ouvrir de ports entrants sur le pare-feu.

Les principaux éléments de fonctionnalité d’Azure Relay sont la communication bidirectionnelle, non mise en mémoire tampon à travers les limites du réseau avec limitation de type TCP, la découverte de point de terminaison, l’état de la connectivité et la sécurité des points de terminaison superposés. Les fonctionnalités d’Azure Relay diffèrent des technologies d’intégration au niveau du réseau telles que le réseau VPN, car le relais peut être limité à un point de terminaison d’application unique sur un ordinateur unique, alors que la technologie VPN est beaucoup plus intrusive, car elle repose sur la modification de l’environnement réseau.

Azure Relay comprend deux fonctionnalités :

1. [Connexions hybrides](#hybrid-connections) : utilise les sockets web standard ouverts permettant des scénarios multi-plateformes.
2. [Relais WCF](#wcf-relays) : utilise Windows Communication Foundation (WCF) pour activer les appels de procédure à distance. Le relais WCF est le relais hérité que de nombreux clients peuvent déjà utiliser avec leurs modèles de programmation WCF.

Les connexions hybrides et relais WCF permettent une connexion sécurisée aux actifs existants au sein d’un réseau d’entreprise. L’utilisation de l’un par rapport à l’autre dépend de vos besoins particuliers, comme décrit dans le tableau suivant :

|  | Relais WCF | les connexions hybrides |
| --- |:---:|:---:|
| **WCF** |x | |
| **.NET Core** | |x |
| **.NET Framework** |x |x |
| **JavaScript/NodeJS*** | |x |
| **Java*** | |x |
| **Protocole Open basé sur des normes** | |x |
| **Plusieurs modèles de programmation RPC** | |x |

*Par disponibilité générale

## <a name="hybrid-connections"></a>les connexions hybrides
La fonctionnalité de [connexions hybrides Azure Relay](relay-hybrid-connections-protocol.md) est une évolution de protocole ouvert sécurisé des fonctionnalités existantes du relais, qui peut être implémentée sur n’importe quelle plateforme et dans n’importe quel langage incluant une fonctionnalité WebSocket de base, ce qui comprend explicitement l’API WebSocket dans les navigateurs web courants. Les connexions hybrides sont basées sur HTTP et WebSockets.

## <a name="wcf-relays"></a>Relais WCF
Le relais WCF fonctionne pour l’ensemble de .NET Framework (NETFX) et pour WCF. Vous lancez la connexion entre votre service local et le service de relais à l’aide d’une suite de liaisons de « relais » WCF. En coulisses, les liaisons de relais se mappent à de nouveaux éléments de liaison de transport destinés à créer des composants de canal WCF qui s'intègrent à Service Bus dans le cloud.

## <a name="service-history"></a>Historique des services
Les connexions hybrides remplacent l’ancienne fonctionnalité appelée « BizTalk Services » qui a été créée sur le relais WCF Azure Service Bus. La nouvelle fonctionnalité de connexions hybrides vient compléter la fonction de relais WCF existante et ces deux possibilités de service cohabiteront dans le service de relais. Elles partagent une passerelle commune, mais ont des implémentations différentes.

## <a name="next-steps"></a>Étapes suivantes :
* [FAQ Relay](relay-faq.md)
* [Créer un espace de noms](relay-create-namespace-portal.md)
* [Prise en main de .NET](relay-hybrid-connections-dotnet-get-started.md)
* [Prise en main de Node](relay-hybrid-connections-node-get-started.md)




<!--HONumber=Jan17_HO1-->


