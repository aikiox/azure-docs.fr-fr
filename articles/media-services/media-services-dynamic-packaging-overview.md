---
title: "Vue d’ensemble de l’empaquetage dynamique | Microsoft Docs"
description: Cette rubrique donne une vue d&quot;ensemble de l&quot;empaquetage dynamique.
author: Juliako
manager: erikre
editor: 
services: media-services
documentationcenter: 
ms.assetid: 0d9e4f54-5daa-45c1-bfaa-cf09ca89b812
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/24/2016
ms.author: juliako
translationtype: Human Translation
ms.sourcegitcommit: 0c77ee0f612c1cbef551a129a22cf3f125e6f29d
ms.openlocfilehash: 004cea5afb7ed6853043841c2c820b49d8a31584


---
# <a name="dynamic-packaging"></a>Empaquetage dynamique
## <a name="overview"></a>Vue d'ensemble
Vous pouvez utiliser Microsoft Azure Media Services pour distribuer de nombreux formats de fichiers sources multimédias, formats de diffusion en continu de contenu multimédia et formats de protection de contenu à diverses technologies clientes (par exemple, iOS, XBOX, Silverlight, Windows 8). Ces clients comprennent différents protocoles. Par exemple, iOS nécessite un format HTTP Live Streaming (HLS) V4, tandis que Silverlight et Xbox nécessitent le format Smooth Streaming. Si vous voulez transmettre un ensemble de fichiers MP4 (ISO Base Media 14496-12) ou de fichiers Smooth Streaming à vitesse de transmission adaptative, à des clients qui utilisent le format MPEG DASH, HLS ou Smooth Streaming, nous vous recommandons de tirer profit de l'empaquetage dynamique Media Services.

Avec l'empaquetage dynamique, il vous suffit de créer un élément multimédia contenant un ensemble de fichiers MP4 ou de fichiers Smooth Streaming à vitesse de transmission adaptative. Ensuite, en fonction du format spécifié dans le manifeste ou la demande de fragment, le serveur de diffusion en continu à la demande s'assure que vous recevez le flux conforme au protocole choisi. Par conséquent, il vous suffit de stocker et de payer les fichiers dans un seul format de stockage. Le service Media Services se charge de créer et de fournir la réponse appropriée en fonction des demandes des clients.

Le diagramme suivant illustre le flux traditionnel d'encodage et d'empaquetage statique.

![Encodage statique](./media/media-services-dynamic-packaging-overview/media-services-static-packaging.png)

Le diagramme suivant illustre le flux d'empaquetage dynamique.

![Encodage dynamique](./media/media-services-dynamic-packaging-overview/media-services-dynamic-packaging.png)


## <a name="common-scenario"></a>Scénario courant
1. Téléchargez un fichier d'entrée (appelé fichier mezzanine). Par exemple, H.264, MP4 ou WMV (pour obtenir la liste des formats pris en charge, consultez [Formats pris en charge par Media Encoder Standard](media-services-media-encoder-standard-formats.md)).
2. Encodez votre fichier mezzanine en ensembles de fichiers MP4 à vitesse de transmission adaptative H.264.
3. Publiez l'élément multimédia qui contient l'ensemble de fichiers MP4 à vitesse de transmission adaptative en créant le localisateur à la demande.
4. Générez les URL de diffusion en continu pour accéder à votre contenu et le diffuser en continu.

## <a name="preparing-assets-for-dynamic-streaming"></a>Préparation des éléments multimédias pour une diffusion en continu dynamique
Pour préparer un élément multimédia pour sa diffusion en continu dynamique, vous disposez de deux options :

1. [Chargez un fichier maître](media-services-dotnet-upload-files.md).
2. [Utilisez l’encodeur Media Encoder Standard pour produire des ensembles de fichiers MP4 à vitesse de transmission adaptative H.264](media-services-dotnet-encode-with-media-encoder-standard.md).
3. [Diffusez votre contenu](media-services-deliver-content-overview.md).

## <a name="a-idunsupportedformatsaformats-that-are-not-supported-by-dynamic-packaging"></a><a id="unsupported_formats"></a>Formats non pris en charge pour l'empaquetage dynamique
Les formats de fichiers sources suivants ne sont pas pris en charge par l'empaquetage dynamique.

* Fichiers MP4 Dolby Digital
* Fichiers Smooth Streaming Dolby Digital

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]




<!--HONumber=Jan17_HO2-->


