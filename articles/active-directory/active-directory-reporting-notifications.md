---
title: Notifications de rapports Azure Active Directory
description: Comment utiliser les notifications de rapports Azure Active Directory pour signaler les connexions suspectes.
services: active-directory
documentationcenter: 
author: dhanyahk
manager: femila
editor: 
ms.assetid: ae6d4b0e-5931-4cb3-98bf-9be91b381c92
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/07/2016
ms.author: dhanyahk
translationtype: Human Translation
ms.sourcegitcommit: dcda8b30adde930ab373a087d6955b900365c4cc
ms.openlocfilehash: 90ba006f25c27e1dc24c85fdc4ef1d5e2a8d0e3d


---
# <a name="azure-active-directory-reporting-notifications"></a>Notifications de rapports Azure Active Directory
## <a name="what-reports-generate-email-notifications"></a>Quels rapports génèrent les notifications par courrier électronique
À ce stade, seul le rapport Activité de connexion anormale déclenche des notifications par courrier électronique.

## <a name="what-is-an-irregular-sign-in"></a>Qu'est-ce qu'une « connexion anormale » ?
Les connexions irrégulières sont celles qui ont été identifiées par nos algorithmes d’apprentissage automatique, sur la base d'une condition de « déplacement impossible » associée à un emplacement et un périphérique de connexion anormaux. Cela peut signifier qu'un pirate a essayé de se connecter à l'aide de ce compte.

## <a name="who-receives-the-email-notifications"></a>Qui reçoit les notifications par courrier électronique ?
Le courrier électronique est envoyé à tous les administrateurs généraux titulaires d'une licence Active Directory Premium. Par mesure de précaution, ce courrier électronique est également envoyé sur leur adresse de messagerie de secours. Les administrateurs doivent inclure aad-alerts-noreply@mail.windowsazure.com dans leur liste d'expéditeurs approuvés pour ne pas rater le courrier électronique.

## <a name="how-often-are-these-emails-sent"></a>Quelle est la fréquence d’envoi de ces courriers électroniques ?
Le courrier électronique est envoyé si 10 nouvelles activités de connexion anormale se produisent au cours des 30 derniers jours, ou depuis que le dernier courrier électronique a été envoyé, selon la valeur qui est inférieure.

## <a name="how-do-i-access-the-report-mentioned-in-the-email"></a>Comment puis-je accéder au rapport mentionné dans le courrier électronique ?
Lorsque vous cliquerez sur le lien, vous serez redirigé vers la page du rapport du portail Azure Classic. Pour accéder au rapport, vous devez être à la fois :

* Un administrateur ou un coadministrateur de votre abonnement Azure
* Un administrateur général du répertoire, titulaire d’une licence Active Directory Premium. Pour plus d’informations, consultez la page [Éditions d’Azure Active Directory](active-directory-editions.md).

## <a name="can-i-turn-off-these-emails"></a>Puis-je désactiver ces courriers électroniques ?
Oui, pour désactiver les notifications liées à des connexions anormales dans le portail Azure Classic, cliquez sur **Configurer**, puis sélectionnez **Désactivé** sous la rubrique **Notifications**.

## <a name="whats-next"></a>Étapes suivantes
* Curieux de savoir quels rapports de sécurité, d'audit et d'activité sont disponibles ? Découvrez [Rapports de sécurité, d'audit et d'activité d'Azure AD](active-directory-view-access-usage-reports.md)
* [Prise en main d’Azure Active Directory Premium (AD)](active-directory-get-started-premium.md)
* [Ajout d’une marque de société aux pages de connexion et du volet d’accès](active-directory-add-company-branding.md)




<!--HONumber=Dec16_HO4-->


