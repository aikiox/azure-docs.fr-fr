---
title: "Didacticiel : intégration d’Azure Active Directory à IBM Kenexa Survey Enterprise | Microsoft Docs"
description: "Découvrez comment configurer l’authentification unique entre Azure Active Directory et IBM Kenexa Survey Enterprise."
services: active-directory
documentationcenter: 
author: jeevansd
manager: femila
editor: 
ms.assetid: c7aac6da-f4bf-419e-9e1a-16b460641a52
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/20/2016
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: dcda8b30adde930ab373a087d6955b900365c4cc
ms.openlocfilehash: 4bf0847e41b10862fdfdaa662ab7ce48a1cf747a


---
# <a name="tutorial-azure-active-directory-integration-with-ibm-kenexa-survey-enterprise"></a>Didacticiel : intégration d’Azure Active Directory à IBM Kenexa Survey Enterprise
Dans ce didacticiel, vous allez apprendre à intégrer IBM Kenexa Survey Enterprise à Azure Active Directory (Azure AD).

L’intégration du logiciel IBM Kenexa Survey Enterprise à Azure AD vous offre les avantages suivants :

* Dans Azure AD, vous pouvez contrôler qui a accès à IBM Kenexa Survey Enterprise.
* Vous pouvez autoriser les utilisateurs à se connecter automatiquement à IBM Kenexa Survey Enterprise (via l’authentification unique) avec leur compte Azure AD.
* Vous pouvez gérer vos comptes à un emplacement central : le portail Azure Classic.

Pour en savoir plus sur l’intégration des applications SaaS avec Azure AD, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](active-directory-appssoaccess-whatis.md).

## <a name="prerequisites"></a>Configuration requise
Pour configurer l’intégration d’Azure AD à IBM Kenexa Survey Enterprise, vous avez besoin des éléments suivants :

* Un abonnement Azure AD
* Un abonnement IBM Kenexa Survey Enterprise pour lequel l’authentification unique est activée

> [!NOTE]
> Pour tester les étapes de ce didacticiel, nous déconseillons l’utilisation d’un environnement de production.
> 
> 

Vous devez en outre suivre les recommandations ci-dessous :

* Vous ne devez pas utiliser votre environnement de production, sauf si cela est nécessaire.
* Si vous n’avez pas d’environnement d’essai Azure AD, vous pouvez obtenir un essai d’un mois [ici](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Description du scénario
Dans ce didacticiel, vous testez l’authentification unique Azure AD dans un environnement de test. Le scénario décrit dans ce didacticiel se compose des deux sections principales suivantes :

1. Ajout du logiciel IBM Kenexa Survey Enterprise à partir de la galerie
2. Configuration et test de l’authentification unique Azure AD

## <a name="adding-ibm-kenexa-survey-enterprise-from-the-gallery"></a>Ajout du logiciel IBM Kenexa Survey Enterprise à partir de la galerie
Pour configurer son intégration à Azure AD, vous devez ajouter IBM Kenexa Survey Enterprise à votre liste d’applications SaaS gérées, à partir de la galerie.

**Pour ajouter IBM Kenexa Survey Enterprise à partir de la galerie, procédez comme suit :**

1. Dans le volet de navigation gauche du **portail Azure Classic**, cliquez sur **Active Directory**. 
   
    ![Active Directory][1]
2. Dans la liste **Annuaire** , sélectionnez l'annuaire pour lequel vous voulez activer l'intégration d'annuaire.
3. Pour ouvrir la vue des applications, dans la vue d'annuaire, cliquez sur **Applications** dans le menu du haut.
   
    ![Applications][2]
4. Cliquez sur **Ajouter** en bas de la page.
   
    ![Applications][3]
5. Dans la boîte de dialogue **Que voulez-vous faire ?**, cliquez sur **Ajouter une application à partir de la galerie**.
   
    ![Applications][4]
6. Dans la zone de recherche, tapez **IBM Kenexa Survey Enterprise**.
   
    ![Création d’un utilisateur de test Azure AD](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_01.png)
7. Dans le volet des résultats, sélectionnez **IBM Kenexa Survey Enterprise**, puis cliquez sur **Terminer** pour ajouter l’application.
   
    ![Création d’un utilisateur de test Azure AD](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_02.png)

## <a name="configuring-and-testing-azure-ad-single-sign-on"></a>Configuration et test de l’authentification unique Azure AD
Dans cette section, vous allez configurer et tester l’authentification unique Azure AD avec IBM Kenexa Survey Enterprise, en tirant parti d’un utilisateur de test appelé « Britta Simon ».

Pour que l’authentification unique fonctionne, Azure AD doit savoir qui est l’utilisateur IBM Kenexa Survey Enterprise équivalent dans Azure AD. En d’autres termes, une relation doit être établie entre un utilisateur Azure AD et un utilisateur IBM Kenexa Survey Enterprise associé.

Pour cela, affectez la valeur de **nom d’utilisateur** dans Azure AD comme valeur de **nom d’utilisateur** dans IBM Kenexa Survey Enterprise.

Pour configurer et tester l’authentification unique Azure AD avec IBM Kenexa Survey Enterprise, vous devez suivre les indications des sections suivantes :

1. **[Configuration de l’authentification unique Azure AD](#configuring-azure-ad-single-sign-on)** pour permettre à vos utilisateurs d’utiliser cette fonctionnalité.
2. **[Création d’un utilisateur de test Azure AD](#creating-an-azure-ad-test-user)** pour tester l’authentification unique Azure AD avec Britta Simon.
3. **[Création d’un utilisateur de test IBM Kenexa Survey Enterprise](#creating-an-kenexasurvey-test-user)** pour avoir un équivalent de l’utilisateur Britta Simon dans IBM Kenexa Survey Enterprise, qui soit lié à la représentation Azure AD associée.
4. **[Affectation de l’utilisateur de test Azure AD](#assigning-the-azure-ad-test-user)** pour permettre à Britta Simon d’utiliser l’authentification unique Azure AD.
5. **[Testing Single Sign-On](#testing-single-sign-on)** pour vérifier si la configuration fonctionne.

### <a name="configuring-azure-ad-single-sign-on"></a>Configuration de l’authentification unique Azure AD
Dans cette section, vous allez activer l’authentification unique Azure AD dans le portail Azure Classic et configurer l’authentification unique dans votre application IBM Kenexa Survey Enterprise.

**Pour configurer l’authentification unique Azure AD avec IBM Kenexa Survey Enterprise, procédez comme suit :**

1. Dans le portail Classic, sur la page d’intégration d’applications **IBM Kenexa Survey Enterprise**, cliquez sur **Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configurer l’authentification unique**.
   
    ![Configurer l’authentification unique][6] 
2. Sur la page **Comment voulez-vous que les utilisateurs se connectent à IBM Kenexa Survey Enterprise ?**, sélectionnez **Authentification unique avec Azure AD**, puis cliquez sur **Suivant**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_03.png)
3. Sur la page **Configurer les paramètres d’application** , procédez comme suit :
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_04.png)
   
    a. Dans la zone de texte **Identificateur**, tapez une URL au format suivant : `https://surveys.kenexa.com/<company code>`
   
    b. Dans la zone de texte **URL de réponse** , tapez une URL au format suivant : `https://surveys.kenexa.com/<company code>/tools/sso.asp`
   
    c. Cliquez sur **Suivant**.
   
   > [!NOTE]
   > Notez qu’il ne s’agit pas des valeurs réelles. Vous devez mettre à jour ces valeurs avec l’identificateur et l’URL de réponse réels. Contactez l’équipe de support technique IBM Kenexa Survey Enterprise pour obtenir ces valeurs.
   > 
   > 
4. Dans la page **Configurer l’authentification unique sur IBM Kenexa Survey Enterprise**, cliquez sur **Télécharger le certificat**, puis enregistrez le fichier sur votre ordinateur :
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_05.png) 
5. Pour obtenir la configuration de l’authentification unique pour votre application, contactez l’équipe de support IBM Kenexa et fournissez-lui les éléments suivants :
   
    •  Le fichier de certificat téléchargé
   
    • L’**URL de l’émetteur**
   
    • L’**URL d’authentification unique SAML**
   
    •  **L’URL du service de déconnexion unique**
   
   > [!NOTE]
   > Remarque : la valeur de revendication NameID de la réponse doit correspondre à l’ID d’authentification unique configuré au sein du système Kenexa. Nous vous invitons à collaborer avec l’équipe de support technique Kenexa pour mapper l’identificateur d’utilisateur approprié dans votre organisation, en tant qu’ID d’authentification unique. Par défaut, Azure AD définit la valeur NameIdentifier en tant que valeur UPN. Pour modifier ce paramètre, accédez à l’onglet Attribut, comme indiqué dans la capture d’écran ci-dessous. L’intégration ne fonctionnera qu’une fois le mappage correct effectué. 
   > 
   > 
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_51.png)
6. Dans le portail classique, sélectionnez la confirmation de la configuration de l’authentification unique, puis cliquez sur **Suivant**.
   
    ![Authentification unique Azure AD][10]
7. Sur la page **Confirmation de l’authentification unique**, cliquez sur **Terminer**.  
   
    ![Authentification unique Azure AD][11]
8. Dans le portail Azure Classic, dans la page d’intégration d’application **IBM Kenexa Survey Enterprise**, dans le menu situé en haut, cliquez sur **Attributs**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_06.png)
9. Dans la boîte de dialogue **Attributs du jeton SAML** , procédez comme suit :
   
    a. Sélectionnez l’attribut de **NameIdentifier** et cliquez sur l’icône **Modifier**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_07.png)
   
    b. À partir de la liste **Valeur de l’attribut**, tapez la valeur d’attribut d’ID d’authentification unique configurée dans le système Kenexa.
   
    c. Cliquez sur **Terminé**

### <a name="creating-an-azure-ad-test-user"></a>Création d’un utilisateur de test Azure AD
L’objectif de cette section est de créer un utilisateur de test appelé Britta Simon dans le portail classique.

![Créer un utilisateur Azure AD][20]

**Pour créer un utilisateur de test dans Azure AD, procédez comme suit :**

1. Dans le volet de navigation gauche du **portail Azure Classic**, cliquez sur **Active Directory**.
   
    ![Création d’un utilisateur de test Azure AD](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_09.png) 
2. Dans la liste **Annuaire** , sélectionnez l'annuaire pour lequel vous voulez activer l'intégration d'annuaire.
3. Pour afficher la liste des utilisateurs, dans le menu situé en haut, cliquez sur **Utilisateurs**.
   
    ![Création d’un utilisateur de test Azure AD](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_03.png) 
4. Pour ouvrir la boîte de dialogue **Ajouter un utilisateur**, cliquez sur l’option **Ajouter un utilisateur** figurant dans la barre d’outils du bas.
   
    ![Création d’un utilisateur de test Azure AD](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_04.png) 
5. Sur la page de boîte de dialogue **Dites-nous en plus sur cet utilisateur** , procédez comme suit :
   
    ![Création d’un utilisateur de test Azure AD](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_05.png) 
   
    a. Dans Type d’utilisateur, sélectionnez Nouvel utilisateur dans votre organisation.
   
    b. Dans la zone de texte **Nom d’utilisateur**, entrez **BrittaSimon**.
   
    c. Cliquez sur **Next**.
6. Sur la page de boîte de dialogue **Profil utilisateur** , procédez comme suit :
   
   ![Création d’un utilisateur de test Azure AD](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_06.png) 
   
   a. Dans la zone de texte **First Name**, tapez **Britta**.  
   
   b. Dans la zone de texte **Last Name**, tapez **Simon**.
   
   c. Dans la zone de texte **Nom d’affichage**, entrez **Britta Simon**.
   
   d. Dans la liste **Rôle**, sélectionnez **Utilisateur**.
   
   e. Cliquez sur **Next**.
7. Sur la page de boîte de dialogue **Obtenir un mot de passe temporaire**, cliquez sur **créer**.
   
    ![Création d’un utilisateur de test Azure AD](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_07.png) 
8. Sur la page de boîte de dialogue **Obtenir un mot de passe temporaire** , procédez comme suit :
   
    ![Création d’un utilisateur de test Azure AD](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_08.png) 
   
    a. Notez la valeur du **Nouveau mot de passe**.
   
    b. Cliquez sur **Terminé**.   

### <a name="creating-an-ibm-kenexa-survey-enterprise-test-user"></a>Création d’un utilisateur de test IBM Kenexa Survey Enterprise
Dans cette section, vous allez créer un utilisateur appelé Britta Simon dans IBM Kenexa Survey Enterprise. Nous vous invitons à collaborer avec l’équipe de support technique IBM Kenexa pour mapper l’ID d’authentification unique pour l’ensemble des utilisateurs. Par ailleurs, cet ID doit être mappé à la valeur NameIdentifier fournie par Azure AD. Vous pouvez modifier ces paramètres par défaut en accédant à l’onglet Attribut.

> [!NOTE]
> Si vous devez créer un utilisateur manuellement, contactez l’équipe de support technique IBM Kenexa Survey Enterprise.
> 
> 

### <a name="assigning-the-azure-ad-test-user"></a>Affectation de l’utilisateur de test Azure AD
Dans cette section, vous allez autoriser Britta Simon à utiliser l’authentification unique Azure en lui accordant l’accès à IBM Kenexa Survey Enterprise.

![Affecter des utilisateurs][200] 

**Pour affecter Britta Simon à IBM Kenexa Survey Enterprise, procédez comme suit :**

1. Pour ouvrir l’affichage des applications dans le portail Classic, dans l’affichage de l’annuaire, cliquez sur l’option **Applications** figurant dans le menu du haut.
   
    ![Affecter des utilisateurs][201] 
2. Dans la liste des applications, sélectionnez **IBM Kenexa Survey Enterprise**.
   
    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_50.png) 
3. Dans le menu situé en haut, cliquez sur **Utilisateurs**.
   
    ![Affecter des utilisateurs][203] 
4. Dans la liste Utilisateurs, sélectionnez **Britta Simon**.
5. Dans la barre d’outils située en bas, cliquez sur **Attribuer**.
   
    ![Affecter des utilisateurs][205]

### <a name="testing-single-sign-on"></a>Test de l’authentification unique
Dans cette section, vous allez tester la configuration de l’authentification unique Azure AD à l’aide du volet d’accès.

Lorsque vous cliquez sur la vignette IBM Kenexa Survey Enterprise dans le volet d’accès, vous devez être connecté automatiquement à votre application IBM Kenexa Survey Enterprise.

## <a name="additional-resources"></a>Ressources supplémentaires
* [Liste de didacticiels sur l’intégration d’applications SaaS avec Azure Active Directory](active-directory-saas-tutorial-list.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](active-directory-appssoaccess-whatis.md)

<!--Image references-->

[1]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_205.png



<!--HONumber=Dec16_HO2-->


