---
title: Forum Aux Questions (FAQ) sur Event Hubs | Microsoft Docs
description: FAQ sur Event Hubs.
services: event-hubs
documentationcenter: na
author: sethmanheim
manager: timlt
editor: 
ms.assetid: bfa10984-eb22-4671-861a-f377a90d9372
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/01/2016
ms.author: sethm
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: eb8a1f5b46ed5bfbdc61789ffc48a79927d9c19d


---
# <a name="event-hubs-faq"></a>FAQ sur les hubs d'événements
Event Hubs permet l’admission , la persistance et le traitement à grande échelle de données d’événements provenant de sources de données à débit élevé et/ou de millions d’appareils. Combinés aux files d’attente et aux rubriques Service Bus, Event Hubs offre des déploiements persistants de commande et de contrôle des scénarios [Internet des objets (IoT)](https://azure.microsoft.com/services/iot-hub/) .

Cet article traite des informations de tarification et répond à certaines questions fréquemment posées sur Event Hubs :

## <a name="pricing-information"></a>Informations sur la tarification
Pour des informations complètes sur la tarification des hubs d’événements, consultez la rubrique [Tarification des hubs d’événements](https://azure.microsoft.com/pricing/details/event-hubs/).

## <a name="how-are-event-hubs-ingress-events-calculated"></a>Comment les événements d'entrée des hubs d'événements sont-ils calculés ?
Chaque événement envoyé à un hub d'événements est considéré comme un message facturable. Un *événement d’entrée* est défini comme une unité de données inférieure ou égale à 64 Ko. Tout événement dont la taille est inférieure ou égale à 64 Ko est considéré comme un événement facturable. Si l’événement est supérieur à 64 Ko, le nombre d’événements facturables est calculé en fonction de la taille de l’événement, par multiples de 64 Ko. Par exemple, un événement de 8 Ko envoyé au hub d'événements est facturé comme un événement, mais un message de 96 Ko envoyé au hub d'événements est facturé comme deux événements.

Les événements utilisés à partir d'un hub d'événements, ainsi que les opérations de gestion et les appels de contrôle comme les points de contrôle, ne sont pas comptabilisés comme des événements d'entrée facturables, mais s’ajoutent à l'allocation d'unité de débit.

## <a name="what-are-event-hubs-throughput-units"></a>Que sont les unités de débit des hubs d'événements ?
Vous devez sélectionner explicitement les unités de débit Event Hubs via le portail Azure ou les modèles Resource Manager Event Hubs. Les unités de débit s’appliquent à tous les Event Hubs dans un espace de noms Event Hubs, et chaque unité de débit fournit à l’espace de noms les fonctionnalités suivantes :

* Jusqu’à 1 Mo par seconde d'événements d'entrée (événements envoyés à un hub d'événements), mais pas plus de 1 000 événements d'entrée, opérations de gestion ou appels d’API de contrôle par seconde.
* Jusqu'à 2 Mo par seconde d'événements de sortie (événements utilisés à partir d'un hub d'événements).
* Jusqu'à 84 Go de stockage d'événements (suffisant pour la période de rétention de 24 heures par défaut).

Les unités de débit des hubs d'événements sont facturées par heure, en fonction du nombre maximal d'unités sélectionnées pendant l'heure donnée.

## <a name="how-are-event-hubs-throughput-unit-limits-enforced"></a>Comment les limites des unités de débit des hubs d'événements sont-elles appliquées ?
Si le débit d'entrée total ou le taux d'entrée total de tous les hubs d'événements d’un espace de noms dépasse les allocations d'unité de débit cumulées, les expéditeurs sont limités et reçoivent des erreurs indiquant que le quota d'entrée a été dépassé.

Si le débit de sortie total ou le taux de sortie total de tous les hubs d'événements d’un espace de noms dépasse les allocations d'unité de débit cumulées, les récepteurs sont limités et reçoivent des erreurs indiquant que le quota de sortie a été dépassé. Les quotas d'entrée et de sortie sont appliqués séparément afin qu'aucun expéditeur n’entraîne un ralentissement de la consommation d'événements, ni qu’un récepteur n’empêche l’envoi d'événements à un hub d'événements.

Notez que la sélection des unités de débit est indépendante du nombre de partitions des hubs d'événements. Bien que chaque partition offre un débit maximal de 1 Mo par seconde en entrée (avec un maximum de 1 000 événements par seconde) et 2 Mo par seconde en sortie, il n'existe aucun coût fixe pour les partitions elles-mêmes. Les frais s’appliquent aux unités de débit agrégées sur tous les Event Hubs d’un espace de noms Event Hubs. Avec ce modèle, vous pouvez créer suffisamment de partitions pour prendre en charge la charge maximale anticipée de leurs systèmes, sans encourir de frais d'unité de débit jusqu'à ce que la charge des événements sur le système nécessite réellement des valeurs de débit plus élevées, sans avoir à modifier la structure et l'architecture de vos systèmes à mesure que la charge sur le système augmente.

## <a name="is-there-a-limit-on-the-number-of-throughput-units-that-can-be-selected"></a>Y a-t-il une limite au nombre d'unités de débit qui peuvent être sélectionnées ?
Par défaut, il existe un quota de 20 unités de débit par espace de noms. Vous pouvez demander un quota d'unités de débit supérieur en soumettant un ticket de support. Au-delà de la limite de 20 unités de débit, des regroupements 20 et 100 unités de débit sont disponibles. Notez que l'utilisation de plus de 20 unités de débit empêche toute modification du nombre d'unités de débit sans soumission d’un ticket de support.

## <a name="is-there-a-charge-for-retaining-event-hubs-events-for-more-than-24-hours"></a>Existe-t-il des frais pour la rétention de plus de 24 heures des événements de hubs d'événements ?
Le niveau Standard des hubs d’événements permet de conserver les messages au-delà de 24 heures, avec un maximum de 30 jours. Si la taille du nombre total d’événements stockés dépasse l’allocation de stockage pour le nombre d’unités de débit sélectionnées (84 Go par unité de débit), la taille excédentaire est facturée au tarif de stockage Blob Azure publié. L'allocation de stockage dans chaque unité de débit couvre tous les frais de stockage pour les périodes de rétention de 24 heures (par défaut), même si l'unité de débit est utilisée pour l'allocation d'entrée maximale.

## <a name="what-is-the-maximum-retention-period"></a>Quelle est la période de rétention maximale ?
Le niveau Standard des hubs d'événements prend actuellement en charge une période de rétention maximale de 7 jours. Notez que les hubs d'événements ne sont pas destinés à servir de magasin de données permanent. Les périodes de rétention supérieures à 24 heures sont destinées aux scénarios dans lesquels il est utile de pouvoir reproduire un flux d'événements sur les mêmes systèmes ; par exemple, pour tester ou vérifier un nouveau modèle d’apprentissage de machine sur des données existantes.

## <a name="how-is-the-event-hubs-storage-size-calculated-and-charged"></a>Comment la taille de stockage des hubs d'événements est-elle calculée et facturée ?
La taille totale de tous les événements stockés, y compris de toute surcharge interne pour les en-têtes d'événements ou les structures de stockage sur disque dans tous les hubs d'événements, est mesurée tout au long de la journée. À la fin de la journée, la taille maximale de stockage est calculée. L'allocation de stockage quotidienne est calculée en fonction du nombre minimal d'unités de débit qui ont été sélectionnées pendant la journée (chaque unité de débit fournit une allocation de 84 Go). Si la taille totale dépasse l'allocation de stockage quotidienne calculée, l'excédent de stockage est facturé selon les taux de stockage Azure Blob (au taux **Stockage localement redondant** ).

## <a name="can-i-use-a-single-amqp-connection-to-send-and-receive-from-event-hubs-and-service-bus-queuestopics"></a>Puis-je utiliser une connexion AMQP unique pour l’envoi et la réception à partir de Event Hubs et de files d’attente/rubriques Service Bus ?
Oui, à condition que la totalité des Event Hubs, files d’attente et rubriques se trouve dans le même espace de noms. Par conséquent, vous pouvez implémenter une connectivité bidirectionnelle et répartie vers de nombreux appareils, avec des latences inférieures à la seconde, de façon économique et hautement évolutive.

## <a name="do-brokered-connection-charges-apply-to-event-hubs"></a>Des frais de connexion répartie s'appliquent-ils aux hubs d'événements ?
Pour les expéditeurs, des frais de connexion s'appliquent uniquement lorsque le protocole AMQP est utilisé. Il n'y a aucun frais de connexion pour l'envoi d'événements à l'aide de HTTP, quel que soit le nombre de systèmes ou de périphériques d’envoi. Si vous prévoyez d'utiliser AMQP (par exemple, pour améliorer le flux d'événements ou activer la communication bidirectionnelle sur des scénarios de commande et de contrôle IoT), reportez-vous à la page [Informations de tarification Service Bus](https://azure.microsoft.com/pricing/details/service-bus/) pour savoir ce qui constitue une connexion répartie et comment elle est mesurée.

## <a name="what-is-the-difference-between-event-hubs-basic-and-standard-tiers"></a>Quelle est la différence entre les niveaux De Base et Standard pour les hubs d’événements ?
Le niveau Standard de Event Hubs fournit des fonctionnalités au-delà de ce qui est disponible au niveau De base, ainsi que dans certains systèmes concurrents. Ces fonctionnalités incluent des périodes de rétention de plus de 24 heures et la possibilité d’utiliser une connexion AMQP unique pour envoyer des commandes à un grand nombre d’appareils avec des latences de moins d’une seconde, ainsi que pour envoyer la télémétrie de ces appareils vers Event Hubs. Pour consulter la liste des fonctionnalités, voir [Tarification Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

## <a name="geographic-availability"></a>Disponibilité géographique
La fonctionnalité Hubs d’événements est disponible dans les régions suivantes :

| Zone géographique | Régions |
| --- | --- |
| États-Unis |Centre des États-Unis, est des États-Unis, est des États-Unis 2, centre sud des États-Unis, ouest des États-Unis |
| Europe |Europe du Nord, Europe de l’Ouest |
| Asie-Pacifique |Asie orientale, Asie du Sud-Est |
| Japon |Est du Japon, ouest du Japon |
| Brésil |Sud du Brésil |
| Australie |Est de l’Australie, sud-est de l’Australie |

## <a name="support-and-sla"></a>Prise en charge et contrats SLA
Un support technique pour les hubs d'événements est disponible via les [forums de la communauté](https://social.msdn.microsoft.com/forums/azure/home). La gestion de la facturation et des abonnements est fournie gratuitement.

Pour en savoir plus sur notre contrat SLA, consultez la section [Contrats de niveau de Service](https://azure.microsoft.com/support/legal/sla/) .

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les hubs d’événements, consultez les articles suivants :

* [Vue d’ensemble des hubs d’événements][Vue d’ensemble des hubs d’événements].
* Un [exemple d'application complet qui utilise des concentrateurs d’événements][exemple d'application complet qui utilise des concentrateurs d’événements].

[Vue d’ensemble des hubs d’événements]: event-hubs-overview.md
[exemple d'application complet qui utilise des concentrateurs d’événements]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097



<!--HONumber=Nov16_HO3-->


