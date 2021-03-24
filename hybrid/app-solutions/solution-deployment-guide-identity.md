---
title: Konfigurowanie tożsamości chmury hybrydowej dla aplikacji platformy Azure i usługi Azure Stack Hub
description: Dowiedz się, jak skonfigurować tożsamość chmury hybrydowej dla platformy Azure i aplikacji Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895351"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="409ca-103">Konfigurowanie tożsamości chmury hybrydowej dla aplikacji platformy Azure i usługi Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="409ca-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="409ca-104">Dowiedz się, jak skonfigurować tożsamość chmury hybrydowej dla aplikacji platformy Azure i usługi Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="409ca-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="409ca-105">Dostępne są dwie opcje udzielania dostępu do aplikacji zarówno w globalnych centrach Azure, jak i Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="409ca-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="409ca-106">Gdy usługa Azure Stack Hub ma ciągłe połączenie z Internetem, możesz użyć Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="409ca-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="409ca-107">Gdy Azure Stack Hub zostanie odłączony od Internetu, możesz użyć usług federacyjnych w usłudze Azure Directory (AD FS).</span><span class="sxs-lookup"><span data-stu-id="409ca-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="409ca-108">Nazwy główne usługi są używane do udzielania dostępu do aplikacji Centrum Azure Stack w celu wdrożenia lub konfiguracji przy użyciu Azure Resource Manager w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="409ca-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="409ca-109">W tym rozwiązaniu utworzysz przykładowe środowisko w celu:</span><span class="sxs-lookup"><span data-stu-id="409ca-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="409ca-110">Ustanów tożsamość hybrydową w globalnych centrach Azure i Azure Stack</span><span class="sxs-lookup"><span data-stu-id="409ca-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="409ca-111">Pobierz token, aby uzyskać dostęp do interfejsu API usługi Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="409ca-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="409ca-112">Aby wykonać kroki opisane w tym rozwiązaniu, musisz mieć uprawnienia operatora centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="409ca-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="409ca-113">![Diagram filarów hybrydowych](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="409ca-113">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="409ca-114">Microsoft Azure Stack Hub to rozszerzenie platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="409ca-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="409ca-115">Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.</span><span class="sxs-lookup"><span data-stu-id="409ca-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="409ca-116">[Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych.</span><span class="sxs-lookup"><span data-stu-id="409ca-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="409ca-117">Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.</span><span class="sxs-lookup"><span data-stu-id="409ca-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="409ca-118">Tworzenie nazwy głównej usługi dla usługi Azure AD w portalu</span><span class="sxs-lookup"><span data-stu-id="409ca-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="409ca-119">Jeśli usługa Azure Stack Hub została wdrożona przy użyciu usługi Azure AD jako magazynu tożsamości, można utworzyć jednostki usługi tak samo jak w przypadku platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="409ca-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="409ca-120">[Użyj tożsamości aplikacji, aby uzyskać dostęp do zasobów](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) , pokazuje, jak wykonać kroki w portalu.</span><span class="sxs-lookup"><span data-stu-id="409ca-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="409ca-121">Przed rozpoczęciem upewnij się, że masz [wymagane uprawnienia usługi Azure AD](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) .</span><span class="sxs-lookup"><span data-stu-id="409ca-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="409ca-122">Tworzenie nazwy głównej usługi dla AD FS przy użyciu programu PowerShell</span><span class="sxs-lookup"><span data-stu-id="409ca-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="409ca-123">Jeśli wdrożono Azure Stack Hub z AD FS, można użyć programu PowerShell do utworzenia jednostki usługi, przypisania roli do dostępu i zalogowania się za pomocą programu PowerShell przy użyciu tej tożsamości.</span><span class="sxs-lookup"><span data-stu-id="409ca-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="409ca-124">[Do uzyskiwania dostępu do zasobów służy tożsamość aplikacji](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) . przedstawiono w nim sposób wykonywania wymaganych kroków przy użyciu programu PowerShell.</span><span class="sxs-lookup"><span data-stu-id="409ca-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="409ca-125">Korzystanie z interfejsu API Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="409ca-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="409ca-126">Rozwiązanie [interfejsu api Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use)  przeprowadzi Cię przez proces pobierania tokenu w celu uzyskania dostępu do interfejsu api usługi Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="409ca-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="409ca-127">Nawiązywanie połączenia z centrum Azure Stack przy użyciu programu PowerShell</span><span class="sxs-lookup"><span data-stu-id="409ca-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="409ca-128">Przewodnik Szybki Start [do rozpoczęcia pracy z programem PowerShell w usłudze Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) przeprowadzi Cię przez kroki wymagane do zainstalowania Azure PowerShell i nawiązania połączenia z instalacją centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="409ca-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="409ca-129">Wymagania wstępne</span><span class="sxs-lookup"><span data-stu-id="409ca-129">Prerequisites</span></span>

<span data-ttu-id="409ca-130">Wymagana jest instalacja centrum Azure Stack podłączona do usługi Azure AD z subskrypcją, do której można uzyskać dostęp.</span><span class="sxs-lookup"><span data-stu-id="409ca-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="409ca-131">Jeśli nie masz instalacji Azure Stack Hub, możesz użyć tych instrukcji, aby skonfigurować [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="409ca-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="409ca-132">Nawiązywanie połączenia z centrum Azure Stack przy użyciu kodu</span><span class="sxs-lookup"><span data-stu-id="409ca-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="409ca-133">Aby nawiązać połączenie z centrum Azure Stack przy użyciu kodu, użyj interfejsu API Azure Resource Manager punktów końcowych, aby pobrać punkty końcowe uwierzytelniania i grafu dla instalacji centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="409ca-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="409ca-134">Następnie Uwierzytelnij się przy użyciu żądań REST.</span><span class="sxs-lookup"><span data-stu-id="409ca-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="409ca-135">Przykładową aplikację kliencką można znaleźć w witrynie [GitHub](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="409ca-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="409ca-136">Jeśli zestaw Azure SDK dla wybranego języka nie obsługuje profilów interfejsu API platformy Azure, zestaw SDK może nie współpracować z centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="409ca-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="409ca-137">Aby dowiedzieć się więcej o profilach interfejsu API platformy Azure, zobacz artykuł [Zarządzanie wersjami profilów interfejsu API](/azure-stack/user/azure-stack-version-profiles) .</span><span class="sxs-lookup"><span data-stu-id="409ca-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="409ca-138">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="409ca-138">Next steps</span></span>

- <span data-ttu-id="409ca-139">Aby dowiedzieć się więcej o sposobie obsługi tożsamości w centrum Azure Stack, zobacz temat [Architektura tożsamości dla centrum Azure Stack](/azure-stack/operator/azure-stack-identity-architecture).</span><span class="sxs-lookup"><span data-stu-id="409ca-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).</span></span>
- <span data-ttu-id="409ca-140">Aby dowiedzieć się więcej o wzorcach chmury platformy Azure, zobacz [wzorce projektowe w chmurze](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="409ca-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
