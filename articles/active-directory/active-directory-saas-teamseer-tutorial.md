---
title: "Didacticiel : Intégration d’Azure Active Directory à TeamSeer | Microsoft Docs"
description: "Découvrez comment utiliser TeamSeer avec Azure Active Directory pour activer l’authentification unique, l’approvisionnement automatique et bien plus encore."
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 6ec4806f-fe0f-4ed7-8cfa-32d1c840433f
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/11/2016
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: d978744bfecddc43c0bf5e803f1fa6bc281ee79b


---
# <a name="tutorial-azure-active-directory-integration-with-teamseer"></a>Didacticiel : Intégration d’Azure AD à TeamSeer
L’objectif de ce didacticiel est de montrer comment intégrer Azure et TeamSeer.  
Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un abonnement Azure valide
* Un locataire TeamSeer

À l’issue de ce didacticiel, les utilisateurs d’Azure AD que vous avez affectés à TeamSeer pourront s’authentifier de manière unique dans l’application sur votre site d’entreprise TeamSeer (connexion initiée par le fournisseur du service) ou en s’aidant de la [Présentation du volet d’accès](active-directory-saas-access-panel-introduction.md).

Le scénario décrit dans ce didacticiel se compose des blocs de construction suivants :

1. Activation de l’intégration d’applications pour TeamSeer
2. Configuration de l'authentification unique
3. Configuration de l'approvisionnement des utilisateurs
4. Affectation d’utilisateurs

![Scénario](./media/active-directory-saas-teamseer-tutorial/IC789618.png "Scenario")

## <a name="enabling-the-application-integration-for-teamseer"></a>Activation de l’intégration d’applications pour TeamSeer
Cette section décrit l’activation de l’intégration d’applications pour TeamSeer.

### <a name="to-enable-the-application-integration-for-teamseer-perform-the-following-steps"></a>Pour activer l’intégration d’applications pour TeamSeer, procédez comme suit :
1. Dans le volet de navigation gauche du portail Azure Classic, cliquez sur **Active Directory**.
   
   ![Active Directory](./media/active-directory-saas-teamseer-tutorial/IC700993.png "Active Directory")
2. Dans la liste **Annuaire** , sélectionnez l'annuaire pour lequel vous voulez activer l'intégration d'annuaire.
3. Pour ouvrir la vue des applications, dans la vue d'annuaire, cliquez sur **Applications** dans le menu du haut.
   
   ![Applications](./media/active-directory-saas-teamseer-tutorial/IC700994.png "Applications")
4. Cliquez sur **Ajouter** en bas de la page.
   
   ![Ajouter une application](./media/active-directory-saas-teamseer-tutorial/IC749321.png "Add application")
5. Dans la boîte de dialogue **Que voulez-vous faire ?**, cliquez sur **Ajouter une application à partir de la galerie**.
   
   ![Ajouter une application à partir de la galerie](./media/active-directory-saas-teamseer-tutorial/IC749322.png "Add an application from gallerry")
6. Dans la **zone de recherche**, entrez **TeamSeer**.
   
   ![Galerie d’applications](./media/active-directory-saas-teamseer-tutorial/IC789619.png "Application Gallery")
7. Dans le volet des résultats, sélectionnez **TeamSeer**, puis cliquez sur **Terminer** pour ajouter l’application.
   
   ![TeamSeer](./media/active-directory-saas-teamseer-tutorial/IC789620.png "TeamSeer")

## <a name="configuring-single-sign-on"></a>Configuration de l'authentification unique
Cette section explique comment permettre aux utilisateurs de s’authentifier sur TeamSeer avec leur compte Azure AD en utilisant la fédération basée sur le protocole SAML.  
Dans le cadre de cette procédure, vous devez créer un fichier de certificat codé en base 64.  
Si cette procédure ne vous est pas familière, consultez [Comment convertir un certificat binaire en fichier texte](http://youtu.be/PlgrzUZ-Y1o).

### <a name="to-configure-single-sign-on-perform-the-following-steps"></a>Pour configurer l’authentification unique, procédez comme suit :
1. Sur la page d’intégration d’applications **TeamSeer** du portail Azure Classic, cliquez sur **Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configurer l’authentification unique**.
   
   ![Configurer l’authentification unique](./media/active-directory-saas-teamseer-tutorial/IC789621.png "Configure Single Sign-On")
2. Dans la page **Comment voulez-vous que les utilisateurs se connectent à TeamSeer ?**, sélectionnez **Authentification unique avec Microsoft Azure AD**, puis cliquez sur **Suivant**.
   
   ![Configurer l’authentification unique](./media/active-directory-saas-teamseer-tutorial/IC789628.png "Configure Single Sign-On")
3. Dans la zone de texte **URL de connexion à TeamSeer** de la page **Configurer l’URL de l’application**, tapez votre URL au format « *http://www.teamseer.com/companyid* », puis cliquez sur **Suivant**.
   
   ![Configurer l’URL de l’application](./media/active-directory-saas-teamseer-tutorial/IC789629.png "Configure App URL")
4. Dans la page **Configurer l’authentification unique sur TeamSeer**, cliquez sur **Télécharger le certificat**, puis enregistrez le fichier de certificat sur votre ordinateur.
   
   ![Configurer l’authentification unique](./media/active-directory-saas-teamseer-tutorial/IC789630.png "Configure Single Sign-On")
5. Dans une autre fenêtre de navigateur web, connectez-vous au site de votre entreprise TeamSeer en tant qu’administrateur.
6. Accédez à **HR Admin**.
   
   ![HR Admin](./media/active-directory-saas-teamseer-tutorial/IC789634.png "HR Admin")
7. Cliquez sur **Setup**.
   
   ![Setup](./media/active-directory-saas-teamseer-tutorial/IC789635.png "Setup")
8. Cliquez sur **Set up SAML provider details**.
   
   ![SAML Settings](./media/active-directory-saas-teamseer-tutorial/IC789636.png "SAML Settings")
9. Dans la section des détails sur le fournisseur SAML, procédez comme suit :
   
   ![SAML Settings](./media/active-directory-saas-teamseer-tutorial/IC789637.png "SAML Settings")
   
   1. Dans la page **Configurer l’authentification unique sur TeamSeer** du portail Azure Classic, copiez la valeur de **URL du service d’authentification unique**, puis collez-la dans la zone de texte **URL**.
   2. Créez un fichier **codé en base 64** à partir du certificat téléchargé.  
      
      > [!TIP]
      > Pour plus d’informations, consultez [Comment convertir un certificat binaire en fichier texte](http://youtu.be/PlgrzUZ-Y1o)
      > 
      > 
   3. Ouvrez votre certificat codé en base 64 dans le Bloc-notes, copiez son contenu dans le Presse-papiers, puis collez-le dans la zone de texte **IdP Public Certificate** .
10. Pour configurer le fournisseur SAML, procédez comme suit :
    
    ![SAML Settings](./media/active-directory-saas-teamseer-tutorial/IC789638.png "SAML Settings")
    
    1. Dans la zone de test **Tester l’adresse de messagerie**, entrez l’adresse de messagerie de l’utilisateur de test.
    2. Dans la zone de texte **Émetteur** , entrez l’URL de l’émetteur du fournisseur du service.
    3. Cliquez sur **Save**.
11. Dans le portail Azure Classic, sélectionnez la confirmation de la configuration de l’authentification unique, puis cliquez sur **Terminer** pour fermer la boîte de dialogue **Configurer l’authentification unique**.
    
    ![Configurer l’authentification unique](./media/active-directory-saas-teamseer-tutorial/IC789639.png "Configure Single Sign-On")

## <a name="configuring-user-provisioning"></a>Configuration de l'approvisionnement des utilisateurs
Pour se connecter à TeamSeer, les utilisateurs d’Azure AD doivent être approvisionnés dans TeamSeer.  
Dans le cas de TeamSeer, l’approvisionnement est une tâche manuelle.

### <a name="to-provision-a-user-accounts-perform-the-following-steps"></a>Pour approvisionner un compte d’utilisateur, procédez comme suit :
1. Connectez-vous au site d’entreprise **TeamSeer** en tant qu’administrateur.
2. Procédez comme suit :
   
   ![HR Admin](./media/active-directory-saas-teamseer-tutorial/IC789640.png "HR Admin")
   
   1. Accédez à **HR Admin \> Users**.
   2. Cliquez sur **Run the New User wizard**.
3. Dans la section **User Details** , procédez comme suit :
   
   ![User Details](./media/active-directory-saas-teamseer-tutorial/IC789641.png "User Details")
   
   1. Indiquez le prénom, le nom et le nom d’utilisateur (adresse de messagerie) du compte AAD valide que vous souhaitez approvisionner, dans les zones de texte **First Name**, **Surname** et **User name (Email address)**.
   2. Cliquez sur **Next**.
4. Suivez les instructions à l’écran pour ajouter un nouvel utilisateur, puis cliquez sur **Finish**.

> [!NOTE]
> Vous pouvez utiliser n’importe quel outil ou API de création de compte d’utilisateur, fourni par TeamSeer, pour approvisionner des comptes utilisateur AAD.
> 
> 

## <a name="assigning-users"></a>Affectation d’utilisateurs
Pour tester votre configuration, vous devez autoriser les utilisateurs d’Azure AD concernés à accéder à votre application.

### <a name="to-assign-users-to-teamseer-perform-the-following-steps"></a>Pour affecter des utilisateurs à TeamSeer, procédez comme suit :
1. Dans le portail Azure Classic, créez un compte de test.
2. Dans la page d’intégration d’application **TeamSeer**, cliquez sur **Affecter des utilisateurs**.
   
   ![Affecter des utilisateurs](./media/active-directory-saas-teamseer-tutorial/IC789642.png "Assign Users")
3. Sélectionnez votre utilisateur de test, cliquez sur **Affecter**, puis sur **Oui** pour confirmer votre affectation.
   
   ![Oui](./media/active-directory-saas-teamseer-tutorial/IC767830.png "Yes")

Si vous souhaitez tester vos paramètres d’authentification unique, ouvrez le volet d’accès. Pour plus d'informations sur le panneau d'accès, consultez [Présentation du panneau d’accès](active-directory-saas-access-panel-introduction.md).




<!--HONumber=Nov16_HO3-->


