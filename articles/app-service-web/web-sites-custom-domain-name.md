---
title: "Mapper un nom de domaine personnalisé à une application Azure"
description: "Découvrez comment mapper un nom de domaine personnalisé (domaine personnel) à votre application dans Azure App Service."
services: app-service
documentationcenter: 
author: cephalin
manager: wpickett
editor: jimbe
tags: top-support-issue
ms.assetid: 48644a39-107c-45fb-9cd3-c741974ff590
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/27/2016
ms.author: cephalin
translationtype: Human Translation
ms.sourcegitcommit: dcda8b30adde930ab373a087d6955b900365c4cc
ms.openlocfilehash: e25348cc1aa0ae284f0fcda7f11bdbe42ba1fa0a


---
# <a name="map-a-custom-domain-name-to-an-azure-app"></a>Mapper un nom de domaine personnalisé à une application Azure
[!INCLUDE [web-selector](../../includes/websites-custom-domain-selector.md)]

Cet article vous indique comment mapper manuellement un nom de domaine personnalisé à votre application web, à votre backend d’application mobile ou à votre application API dans [Azure App Service](../app-service/app-service-value-prop-what-is.md). 

Votre application est fournie avec un sous-domaine unique d’azurewebsites.net. Par exemple, si le nom de votre application est **contoso**, son nom de domaine est **contoso.azurewebsites.net**. Toutefois, vous pouvez mapper un nom de domaine personnalisé à l’application afin que son URL, par exemple `www.contoso.com`, reflète votre marque.

> [!NOTE]
> Consultez les [forums Azure](https://azure.microsoft.com/support/forums/)pour y trouver l’aide des experts Azure. Pour obtenir un niveau de support encore supérieur, accédez au [site Support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.
> 
> 

[!INCLUDE [introfooter](../../includes/custom-dns-web-site-intro-notes.md)]

## <a name="buy-a-new-custom-domain-in-azure-portal"></a>Acheter un nouveau domaine personnalisé sur le portail Azure
Si vous n’avez pas déjà acheté un nom de domaine personnalisé, vous pouvez en acheter un et le gérer directement dans les paramètres de votre application dans le [portail Azure](https://portal.azure.com). Cette option facilite le mappage d’un domaine personnalisé à votre application, qu’elle utilise ou non [Azure Traffic Manager](web-sites-traffic-manager-custom-domain-name.md) . 

Pour connaître les instructions à suivre, voir l’article [Acheter et configurer un nom de domaine personnalisé dans Azure App Service](custom-dns-web-site-buydomains-web-app.md).

## <a name="map-a-custom-domain-you-purchased-externally"></a>Mapper un domaine personnalisé acheté en externe
Si vous avez déjà acheté un domaine personnalisé auprès de [DNS Azure](https://azure.microsoft.com/services/dns/) ou d’un fournisseur tiers, la procédure de mappage du domaine personnalisé à votre application compte trois étapes principales :

1. [*(Enregistrement A uniquement)* Obtenez l’adresse IP de l’application](#vip).
2. [Créez les enregistrements DNS qui mappent votre domaine à votre application](#createdns). 
   * **Où** : l’outil de gestion de votre bureau d’enregistrement de domaines (par ex. DNS Azure, GoDaddy, etc.).
   * **Pourquoi**: pour que votre bureau d’enregistrement de domaines sache résoudre le domaine personnalisé souhaité à votre application Azure.
3. [Activez le nom de domaine personnalisé de votre application Azure](#enable).
   * **Où**: le [portail Azure](https://portal.azure.com).
   * **Pourquoi**: pour que votre application sache répondre aux requêtes effectuées auprès du nom de domaine personnalisé.
4. [Vérifiez la propagation DNS](#verify).

### <a name="types-of-domains-you-can-map"></a>Types de domaine que vous pouvez mapper
Azure App Service vous permet de mapper les catégories suivantes de domaines personnalisés à votre application.

* **Domaine racine** : nom du domaine que vous avez réservé auprès du bureau d’enregistrement de domaines (représenté généralement par l’enregistrement d’hôte `@`). 
  Par exemple, **contoso.com**.
* **Sous-domaine** : tout domaine se trouvant sous votre domaine racine. Par exemple, **www.contoso.com** (représenté par l’enregistrement d’hôte `www`).  Vous pouvez mapper différents sous-domaines d’un même domaine racine à différentes applications dans Azure.
* **Domaine avec un caractère générique** - [tout sous-domaine dont le nom DNS à l’extrême gauche est `*`](https://en.wikipedia.org/wiki/Wildcard_DNS_record) (par exemple, les enregistrements d’hôte `*` et `*.blogs`). Par exemple, **\*.contoso.com**.

### <a name="types-of-dns-records-you-can-use"></a>Types d’enregistrement DNS que vous pouvez utiliser
Selon vos besoins, vous pouvez utiliser deux types d’enregistrement DNS standard pour mapper votre domaine personnalisé : 

* [A](https://en.wikipedia.org/wiki/List_of_DNS_record_types#A) : mappe directement votre nom de domaine personnalisé à l’adresse IP virtuelle de l’application Azure. 
* [CNAME](https://en.wikipedia.org/wiki/CNAME_record) : mappe votre nom de domaine personnalisé au nom de domaine Azure de votre application, **&lt;*nom_application*>.azurewebsites.net**. 

L’avantage de l’enregistrement CNAME est qu’il persiste malgré les modifications de l’adresse IP. Si vous supprimez et recréez votre application, ou si vous repassez d’un niveau tarifaire plus élevé au niveau **Partagé** , l’adresse IP virtuelle de votre application peut être modifiée. Avec une modification de ce type, un enregistrement CNAME reste valide tandis qu’un enregistrement A doit être mis à jour. 

Le didacticiel vous montre les procédures à suivre pour utiliser les enregistrements A et CNAME.

> [!IMPORTANT]
> Ne créez pas d’enregistrement CNAME pour votre domaine racine (c’est-à-dire « d’enregistrement racine »). Pour plus d’informations, consultez l’article [Why can’t a CNAME record be used at the root domain (Pourquoi un enregistrement CNAME ne peut-il pas être utilisé dans le domaine racine)](http://serverfault.com/questions/613829/why-cant-a-cname-record-be-used-at-the-apex-aka-root-of-a-domain).
> Pour mapper un domaine racine à votre application Azure, utilisez plutôt un enregistrement A.
> 
> 

<a name="vip"></a>

## <a name="step-1-a-record-only-get-apps-ip-address"></a>Étape 1. *(Enregistrement A uniquement)* Obtenir l’adresse IP de l’application
Pour mapper un nom de domaine personnalisé à l’aide d’un enregistrement A, vous avez besoin de l’adresse IP de votre application Azure. Si vous préférez mapper à l’aide d’un enregistrement CNAME, ignorez cette étape et passez à la section suivante.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **App Services** dans le menu de gauche.
3. Cliquez sur votre application, puis sur **Domaines personnalisés**.
4. Prenez note de l’adresse IP située au-dessus de la section Noms d’hôtes.
   
   ![Mapper un nom de domaine personnalisé avec un enregistrement A : obtenir l’adresse IP de votre application Azure App Service](./media/web-sites-custom-domain-name/virtual-ip-address.png)
5. Laissez ce panneau du portail ouvert. Vous retournerez dans celui-ci une fois que vous aurez créé les enregistrements DNS.

<a name="createdns"></a>

## <a name="step-2-create-the-dns-records"></a>Étape 2. Créer les enregistrements DNS
Connectez-vous à votre bureau d'enregistrement de domaine et utilisez son utilitaire pour ajouter un enregistrement A ou CNAME. Comme l’interface utilisateur de chaque bureau d’enregistrement est légèrement différente, vous devez donc consulter la documentation de votre fournisseur. Voici néanmoins quelques recommandations générales.

1. Trouvez la page de gestion des enregistrements DNS. Recherchez la mention **Nom de domaine**, **DNS** ou **Gestion du nom de serveur**. Vous trouvez généralement un lien en affichant vos informations de compte, puis en recherchant un lien comme **Mes domaines**.
2. Recherchez un lien vous permettant d’ajouter ou de modifier des enregistrements DNS. Il s’agit probablement d’un lien de **fichier de zone** ou **d’enregistrements DNS**, ou d’un lien de configuration **avancé**.
3. Créez l’enregistrement et enregistrez vos modifications.
   * [Instructions relatives à un enregistrement A](#a).
   * [Instructions relatives à un enregistrement CNAME](#cname).

<a name="a"></a>

### <a name="create-an-a-record"></a>Créer un enregistrement A
Pour utiliser un enregistrement A à mapper à l’adresse IP de votre application Azure, vous devez créer un enregistrement A et un enregistrement TXT. L’enregistrement A concerne la résolution DNS proprement dite, et l’enregistrement TXT permet à Azure de vérifier que vous possédez le nom de domaine personnalisé. 

Configurez votre enregistrement A comme suit ((@ représente généralement le domaine racine) :

<table cellspacing="0" border="1">
  <tr>
    <th>Exemple de nom de domaine complet</th>
    <th>Hôte A</th>
    <th>Valeur A</th>
  </tr>
  <tr>
    <td>contoso.com (racine)</td>
    <td>@</td>
    <td>Adresse IP de <a href="#vip">l’étape 1</a></td>
  </tr>
  <tr>
    <td>www.contoso.com (sous-domaine)</td>
    <td>www</td>
    <td>Adresse IP de <a href="#vip">l’étape 1</a></td>
  </tr>
  <tr>
    <td>\*.contoso.com (caractère générique)</td>
    <td>\*</td>
    <td>Adresse IP de <a href="#vip">l’étape 1</a></td>
  </tr>
</table>

L’enregistrement TXT supplémentaire adopte la convention qui mappe de &lt;*sous-domaine*>.&lt;*domaine_racine*> à &lt;*nom_application*>.azurewebsites.net. Configurez votre enregistrement TXT comme suit :

<table cellspacing="0" border="1">
  <tr>
    <th>Exemple de nom de domaine complet</th>
    <th>Hôte TXT</th>
    <th>Valeur TXT</th>
  </tr>
  <tr>
    <td>contoso.com (racine)</td>
    <td>@</td>
    <td>&lt;<i>nom_application</i>>.azurewebsites.net</td>
  </tr>
  <tr>
    <td>www.contoso.com (sous-domaine)</td>
    <td>www</td>
    <td>&lt;<i>nom_application</i>>.azurewebsites.net</td>
  </tr>
  <tr>
    <td>\*.contoso.com (caractère générique)</td>
    <td>\*</td>
    <td>&lt;<i>nom_application</i>>.azurewebsites.net</td>
  </tr>
</table>

<a name="cname"></a>

### <a name="create-a-cname-record"></a>Créer un enregistrement CNAME
Si vous utilisez un enregistrement CNAME à mapper au nom de domaine par défaut de votre application Azure, vous n’avez pas besoin d’un enregistrement TXT supplémentaire, comme c’est le cas avec un enregistrement A. 

> [!IMPORTANT]
> Ne créez pas d’enregistrement CNAME pour votre domaine racine (c’est-à-dire « d’enregistrement racine »). Pour plus d’informations, consultez l’article [Why can’t a CNAME record be used at the root domain (Pourquoi un enregistrement CNAME ne peut-il pas être utilisé dans le domaine racine)](http://serverfault.com/questions/613829/why-cant-a-cname-record-be-used-at-the-apex-aka-root-of-a-domain).
> Pour mapper un domaine racine à votre application Azure, utilisez plutôt un [enregistrement A](#a) .
> 
> 

Configurez votre enregistrement CNAME comme suit ((@ représente généralement le domaine racine) :

<table cellspacing="0" border="1">
  <tr>
    <th>Exemple de nom de domaine complet</th>
    <th>Hôte CNAME</th>
    <th>Valeur CNAME</th>
  </tr>
  <tr>
    <td>www.contoso.com (sous-domaine)</td>
    <td>www</td>
    <td>&lt;<i>nom_application</i>>.azurewebsites.net</td>
  </tr>
  <tr>
    <td>\*.contoso.com (caractère générique)</td>
    <td>\*</td>
    <td>&lt;<i>nom_application</i>>.azurewebsites.net</td>
  </tr>
</table>

<a name="enable"></a>

## <a name="step-3-enable-the-custom-domain-name-for-your-app"></a>Étape 3. Activer le nom de domaine personnalisé de votre application
De retour dans le panneau **Domaines personnalisés** du portail Azure (voir [l’étape 1](#vip)), vous devez ajouter à la liste le nom de domaine complet (FQDN) de votre domaine personnalisé.

1. Si ce n’est pas déjà fait, connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le portail Azure, cliquez sur **App Services** dans le menu de gauche.
3. Cliquez sur votre application, puis sur **Domaines personnalisés** > **Ajouter un nom d’hôte**.
4. Ajoutez le nom de domaine complet de votre domaine personnalisé à la liste (par exemple, **www.contoso.com**).
   
   ![Mapper un nom de domaine personnalisé à une application Azure : Ajouter à la liste des noms de domaine](./media/web-sites-custom-domain-name/add-custom-domain.png)
   
   > [!NOTE]
   > Azure tente de vérifier le nom de domaine que vous utilisez ici. Assurez-vous qu’il s’agit du nom du domaine pour lequel vous avez créé un enregistrement DNS à [l’étape 2](#createdns). 
   > 
   > 
5. Cliquez sur **Valider**.
6. Lorsque vous cliquez sur **Valider** , Azure lance le processus de vérification du domaine. Ce processus vérifie la propriété du domaine ainsi que la disponibilité du nom du domaine, vous signale la réussite ou le détail de l’échec de l’opération et vous fournit des instructions de résolution des éventuelles erreurs.    
7. Lorsque la validation a été correctement effectuée, le bouton **Ajouter un nom d’hôte** est activé, et vous êtes alors en mesure d’attribuer un nom d’hôte. 
8. Une fois qu’Azure a terminé la configuration de votre nouveau nom de domaine, accédez à votre nom de domaine personnalisé dans un navigateur. Le navigateur devrait ouvrir votre application Azure, ce qui signifie que le nom de votre domaine personnalisé est correctement configuré.

## <a name="migrate-an-active-domain-with-no-downtime"></a>Migration d’un domaine actif sans interruption de service 

Lorsque vous migrez un site de production et son nom de domaine vers App Service, ce nom de domaine s’occupe déjà du trafic de production, et vous ne souhaitez pas d’interruption de service pour la résolution DNS lors du processus de migration. Dans ce cas, vous devez lier le nom de domaine à votre application Azure de façon préventive pour la vérification du domaine. Pour ce faire, suivez les étapes modifiées ci-dessous :

1. Commencez par créer un enregistrement TXT de vérification avec votre registre DNS en suivant la démarche de [l’étape 2. Créer les enregistrements DNS](#createdns).
L’enregistrement TXT supplémentaire adopte la convention qui mappe de &lt;*sous-domaine*>.&lt;*domaine_racine*> à &lt;*nom_application*>.azurewebsites.net.
Pour des exemples, consultez le tableau suivant :  
 
    <table cellspacing="0" border="1">
    <tr>
    <th>Exemple de nom de domaine complet</th>
    <th>Hôte TXT</th>
    <th>Valeur TXT</th>
    </tr>
    <tr>
    <td>contoso.com (racine)</td>
    <td>awverify.contoso.com</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
    </tr>
    <tr>
    <td>www.contoso.com (sous-domaine)</td>
    <td>awverify.www.contoso.com</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
    </tr>
    <tr>
    <td>\*.contoso.com (caractère générique)</td>
    <td>awverify.\*.contoso.com</td>
    <td>&lt;<i>appname</i>>.azurewebsites.net</td>
    </tr>
    </table>

2. Puis, ajoutez votre nom de domaine personnalisé à votre application Azure en suivant la démarche de [l’étape 3. Activer le nom de domaine personnalisé de votre application](#enable).

    Votre domaine personnalisé est maintenant activé dans votre application Azure. La seule chose qui vous reste à faire est de mettre à jour l’enregistrement DNS avec votre bureau d’enregistrement de domaine.

3. Enfin, mettez à jour l’enregistrement DNS de votre domaine pour pointer vers votre application Azure, comme indiqué dans [l’étape 2. Créer les enregistrements DNS](#createdns). 

    Le trafic utilisateur devrait être redirigé vers votre application Azure immédiatement après la propagation DNS.

<a name="verify"></a>

## <a name="verify-dns-propagation"></a>Vérifier la propagation DNS
Une fois les étapes de configuration terminées, la propagation des modifications peut prendre un certain temps, suivant votre fournisseur DNS. Vous pouvez vérifier que la propagation DNS fonctionne normalement à partir du site [http://digwebinterface.com/](http://digwebinterface.com/). Une fois sur le site, spécifiez les noms d’hôte dans la zone de texte et cliquez sur **Dig**. Consultez les résultats pour vérifier si les modifications récentes ont pris effet.  

![Mapper un nom de domaine personnalisé à une application Azure : Vérifier la propagation DNS](./media/web-sites-custom-domain-name/1-digwebinterface.png)

> [!NOTE]
> La propagation des entrées DNS peut prendre 48 heures (parfois plus). Même si vous avez tout configuré correctement, vous devez faire preuve de patience.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
Apprenez à sécuriser votre nom de domaine personnalisé avec HTTPS en [achetant un certificat SSL dans Azure](web-sites-purchase-ssl-web-site.md) ou [à l’aide d’un certificat SSL depuis un autre emplacement](web-sites-configure-ssl-certificate.md).

> [!NOTE]
> Si vous voulez vous familiariser avec Azure App Service avant d’ouvrir un compte Azure, accédez à la page [Essayer App Service](http://go.microsoft.com/fwlink/?LinkId=523751), où vous pourrez créer immédiatement une application web temporaire dans App Service. Aucune carte de crédit n’est requise ; vous ne prenez aucun engagement.
> 
> 

[Prise en main d’Azure DNS](../dns/dns-getstarted-create-dnszone.md)  
[Créer des enregistrements DNS pour une application web dans un domaine personnalisé](../dns/dns-web-sites-custom-domain.md)  
[Délégation de domaine à Azure DNS](../dns/dns-domain-delegation.md)

<!-- Images -->
[subdomain]: media/web-sites-custom-domain-name/azurewebsites-subdomain.png



<!--HONumber=Dec16_HO2-->


