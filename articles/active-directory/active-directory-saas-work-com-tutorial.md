---
title: "Didacticiel : Intégration d’Azure Active Directory à Work.com | Microsoft Docs"
description: "Découvrez comment utiliser Work.com avec Azure AD pour activer l’authentification unique, l’approvisionnement automatisé et bien plus encore."
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 98e6739e-eb24-46bd-9dd3-20b489839076
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/02/2016
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: f454e7e218764e00cc19ca67b0edade213834b75
ms.openlocfilehash: 69d94659f4eff72e1c449fd915616d81fd4712de


---
# <a name="tutorial-azure-active-directory-integration-with-workcom"></a>Didacticiel : Intégration d’Azure AD à Work.com
L’objectif de ce didacticiel est de montrer comment intégrer Azure et Work.com.  
Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un abonnement Azure valide
* Un abonnement Work.com pour lequel l’authentification unique est activée

À l’issue de ce didacticiel, les utilisateurs d’AAD auxquels vous avez affecté un accès à Work.com pourront s’authentifier de manière unique dans l’application sur votre site d’entreprise Work.com (connexion initiée par le fournisseur de services) ou à l’aide de la [Présentation du panneau d’accès](active-directory-saas-access-panel-introduction.md).

Le scénario décrit dans ce didacticiel se compose des blocs de construction suivants :

1. Activation de l’intégration d’applications pour Work.com
2. Configuration de l'authentification unique
3. Configuration de l'approvisionnement des utilisateurs
4. Affectation d’utilisateurs

![Scénario](./media/active-directory-saas-work-com-tutorial/IC794105.png "Scenario")

## <a name="enabling-the-application-integration-for-workcom"></a>Activation de l’intégration d’applications pour Work.com
Cette section décrit l’activation de l’intégration d’applications pour Work.com.

### <a name="to-enable-the-application-integration-for-workcom-perform-the-following-steps"></a>Pour activer l’intégration d’applications pour Work.com, procédez comme suit :
1. Dans le volet de navigation gauche du portail Azure Classic, cliquez sur **Active Directory**.
   
    ![Active Directory](./media/active-directory-saas-work-com-tutorial/IC700993.png "Active Directory")

2. Dans la liste **Annuaire** , sélectionnez l'annuaire pour lequel vous voulez activer l'intégration d'annuaire.

3. Pour ouvrir la vue des applications, dans la vue d'annuaire, cliquez sur **Applications** dans le menu du haut.
   
    ![Applications](./media/active-directory-saas-work-com-tutorial/IC700994.png "Applications")

4. Cliquez sur **Ajouter** en bas de la page.
   
    ![Ajouter une application](./media/active-directory-saas-work-com-tutorial/IC749321.png "Add application")

5. Dans la boîte de dialogue **Que voulez-vous faire ?**, cliquez sur **Ajouter une application à partir de la galerie**.
   
    ![Ajouter une application à partir de la galerie](./media/active-directory-saas-work-com-tutorial/IC749322.png "Add an application from gallerry")

6. Dans la **zone de recherche**, tapez **Work.com**.
   
    ![Galerie d’applications](./media/active-directory-saas-work-com-tutorial/IC794106.png "Application Gallery")

7. Dans le volet de résultats, sélectionnez **Work.com**, puis cliquez sur **Terminer** pour ajouter l’application.
   
    ![Work.com](./media/active-directory-saas-work-com-tutorial/IC794107.png "Work.com")

## <a name="configuring-single-sign-on"></a>Configuration de l'authentification unique
Cette section explique comment permettre aux utilisateurs de s’authentifier sur Work.com avec leur compte Azure AD en utilisant la fédération basée sur le protocole SAML.  
Dans le cadre de cette procédure, vous devez charger un certificat vers Work.com.

> [!NOTE]
> Pour configurer l’authentification unique, vous devez encore configurer un nom de domaine personnalisé Work.com. Vous devez définir au moins un nom de domaine, le tester, puis le déployer dans l’ensemble de votre entreprise.
> 
> 

### <a name="to-configure-single-sign-on-perform-the-following-steps"></a>Pour configurer l’authentification unique, procédez comme suit :
1. Connectez-vous à votre locataire Work.com en tant qu’administrateur.
2. Accédez à **Setup**.
   
    ![Paramétrage](./media/active-directory-saas-work-com-tutorial/IC794108.png "Setup")

3. Dans le volet de navigation gauche, dans la section **Administer**, cliquez sur **Domain Management** pour développer la section associée, puis cliquez sur **My Domain** pour ouvrir la page **My Domain**. 
   
    ![My Domain](./media/active-directory-saas-work-com-tutorial/IC767825.png "My Domain")

4. Pour vérifier que votre domaine a été configuré correctement, assurez-vous qu’il figure dans **Step 4 Deployed to Users**, et passez en revue **My Domain Settings**.
   
    ![Domaine déployé pour l’utilisateur](./media/active-directory-saas-work-com-tutorial/IC784377.png "Doman Deployed to User")

5. Dans une autre fenêtre de navigateur web, connectez-vous à votre portail Azure Classic.

6. Dans la page d’intégration d’application **Work.com**, cliquez sur **Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configurer l’authentification unique**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-work-com-tutorial/IC794109.png "Configure Single Sign-On")

7. Dans la page **Comment voulez-vous que les utilisateurs se connectent à Work.com ?**, sélectionnez **Authentification unique Microsoft Azure AD**, puis cliquez sur **Suivant**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-work-com-tutorial/IC794110.png "Configure Single Sign-On")

8. Dans la page **Configurer l’URL de l’application**, dans la zone de texte **URL de connexion à Work.com**, tapez l’URL utilisée par vos utilisateurs pour se connecter à votre application Work.com (par ex. « *http://company.my.salesforce.com* »), puis cliquez sur **Suivant** : 
   
    ![Configurer l’URL de l’application](./media/active-directory-saas-work-com-tutorial/IC794111.png "Configure App URL")

9. Dans la page **Configurer l’authentification unique sur Work.com**, cliquez sur **Télécharger le certificat**, puis enregistrez le fichier de certificat en local sur votre ordinateur.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-work-com-tutorial/IC794112.png "Configure Single Sign-On")

10. Connectez-vous à votre locataire Work.com.

11. Accédez à **Setup**.
    
    ![Paramétrage](./media/active-directory-saas-work-com-tutorial/IC794108.png "Setup")

12. Développez le menu **Security Controls**, puis cliquez sur **Single Sign-On Settings**.
    
    ![Single Sign-On Settings](./media/active-directory-saas-work-com-tutorial/IC794113.png "Single Sign-On Settings")

13. Dans la page **Single Sign on Settings** , procédez comme suit :
    
    ![SAML Enabled](./media/active-directory-saas-work-com-tutorial/IC781026.png "SAML Enabled")
    
    a. Sélectionnez **SAML Enabled**.
    
    b. Cliquez sur **Nouveau**.

14. Dans la section **SAML Single Sign-On Settings** , procédez comme suit :
    
    ![Paramètre d’authentification unique SAML](./media/active-directory-saas-work-com-tutorial/IC794114.png "SAML Single Sign-On Setting")
    
    a. Dans la zone de texte **Name** , tapez le nom de votre configuration.  
       
    > [!NOTE]
    > Le fait d’entrer une valeur pour **Name** renseigne automatiquement la zone de texte **API Name**.
    > 
    > 
    
    b. Sur la page **Configurer l’authentification unique sur Work.com** du portail Azure Classic, copiez la valeur **URL de l’émetteur**, puis collez-la dans la zone de texte **Issuer**.
    
    c. Pour charger le certificat téléchargé, cliquez sur **Parcourir**.
    
    d. Dans la zone de texte **Id d’entité**, saisissez **https://salesforce-jobscience.com**.
    
    e. Pour **SAML Identity Type (Type d’identité SAML)**, sélectionnez **Assertion contains the Federation ID from the User object (L’assertion contient l’ID de fédération de l’objet utilisateur)**.
    
    f. Pour **SAML Identity Location**, sélectionnez **Identity is in the NameIdentfier element of the Subject statement**.
    
    g. Dans le portail Azure Classic, dans la page **Configurer l’authentification unique sur Work.com**, copiez la valeur **URL de connexion distante**, puis collez-la dans la zone de texte **Identity Provider Login URL**.
    
    h. Dans le portail Azure Classic, dans la page **Configurer l’authentification unique sur Work.com**, copiez la valeur **URL de déconnexion distante**, puis collez-la dans la zone de texte **Identity Provider Logout URL**.
    
    i. Pour **Service Provider Initiated Request Binding**, sélectionnez **HTTP POST**.
    
    j. Cliquez sur **Enregistrer**.

15. Dans le volet de navigation gauche du portail classique Work.com, cliquez sur **Domain Management** pour développer la section associée, puis sur **My Domain** pour ouvrir la page **My Domain**. 
    
    ![My Domain](./media/active-directory-saas-work-com-tutorial/IC794115.png "My Domain")

16. Dans la section **Login Page Branding** de la page **My Domain**, cliquez sur **Edit**.
    
    ![Personnalisation de la page de connexion](./media/active-directory-saas-work-com-tutorial/IC767826.png "Login Page Branding")

17. Le nom des **SAML SSO Settings** s’affiche dans la section **Authentication Service** de la page **Login Page Branding**. Sélectionnez-le, puis cliquez sur **Save**.
    
    ![Personnalisation de la page de connexion](./media/active-directory-saas-work-com-tutorial/IC784366.png "Login Page Branding")

18. Dans le portail Azure Classic, sélectionnez la confirmation de la configuration de l’authentification unique, puis cliquez sur **Terminer** pour fermer la boîte de dialogue **Configurer l’authentification unique**.
    
    ![Configurer l’authentification unique](./media/active-directory-saas-work-com-tutorial/IC794116.png "Configure Single Sign-On")

## <a name="configuring-user-provisioning"></a>Configuration de l'approvisionnement des utilisateurs
Pour que les utilisateurs d’Azure AD puissent se connecter, leur accès doit être approvisionné dans Work.com.  
Dans le cas de Work.com, l’approvisionnement est une tâche manuelle.

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>Pour configurer l'approvisionnement des utilisateurs, procédez comme suit :
1. Connectez-vous à votre site d’entreprise Work.com en tant qu’administrateur.

2. Accédez à **Setup**.
   
    ![Paramétrage](./media/active-directory-saas-work-com-tutorial/IC794108.png "Setup")
3. Accédez à **Manage Users \> Users**.
   
    ![Gérer les utilisateurs](./media/active-directory-saas-work-com-tutorial/IC784369.png "Manage Users")

4. Cliquez sur **New User**.
   
    ![Tous les utilisateurs](./media/active-directory-saas-work-com-tutorial/IC794117.png "All Users")

5. Dans la section User Edit, procédez comme suit :
   
    ![Modification de l’utilisateur](./media/active-directory-saas-work-com-tutorial/IC794118.png "User Edit")
   
    a. Dans les zones de texte **Last Name**, **Alias**, **Email**, **Username** et **Nickname**, tapez les attributs d’un compte Azure Active Directory valide que vous souhaitez approvisionner.
   
    b. Sélectionnez **Role**, **User License** et **Profile**.
   
    c. Cliquez sur **Enregistrer**.  
      
    > [!NOTE]
    > Le titulaire du compte Azure AD reçoit un message électronique contenant un lien pour confirmer le compte avant qu’il ne soit activé.
    > 
    > 

## <a name="assigning-users"></a>Affectation d’utilisateurs
Pour tester votre configuration, vous devez autoriser les utilisateurs d’Azure AD concernés à accéder à votre application.

### <a name="to-assign-users-to-workcom-perform-the-following-steps"></a>Pour affecter des utilisateurs à Work.com, procédez comme suit :
1. Dans le portail Azure Classic, créez un compte de test.

2. Dans la page d’intégration d’applications Work.com, cliquez sur **Affecter des utilisateurs**.
   
    ![Affecter des utilisateurs](./media/active-directory-saas-work-com-tutorial/IC794119.png "Assign Users")

3. Sélectionnez votre utilisateur de test, cliquez sur **Affecter**, puis sur **Oui** pour confirmer votre affectation.
   
    ![Oui](./media/active-directory-saas-work-com-tutorial/IC767830.png "Yes")

À présent, vous devez patienter 10 minutes et vérifier que le compte a été synchronisé avec Work.com.

Si vous souhaitez tester vos paramètres d’authentification unique, ouvrez le volet d’accès. Pour plus d'informations sur le panneau d'accès, consultez [Présentation du panneau d’accès](active-directory-saas-access-panel-introduction.md).




<!--HONumber=Dec16_HO1-->


