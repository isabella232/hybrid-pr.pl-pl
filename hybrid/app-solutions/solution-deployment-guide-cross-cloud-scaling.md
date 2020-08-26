---
title: Wdrażanie aplikacji, która skaluje wiele chmur na platformie Azure i w Azure Stack Hub
description: Dowiedz się, jak wdrożyć aplikację, która skaluje wiele chmur na platformie Azure i Azure Stack centrum.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886819"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="68342-103">Wdrażanie aplikacji, która skaluje wiele chmur przy użyciu platformy Azure i usługi Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="68342-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="68342-104">Dowiedz się, jak utworzyć rozwiązanie międzychmurowe, aby zapewnić ręcznie wyzwolony proces przełączania z hostowanej aplikacji sieci Web Azure Stack Hub do hostowanej aplikacji sieci Web platformy Azure z funkcją automatycznego skalowania za pomocą usługi Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="68342-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="68342-105">Ten proces zapewnia elastyczne i skalowalne narzędzie w chmurze w ramach obciążenia.</span><span class="sxs-lookup"><span data-stu-id="68342-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="68342-106">Ten wzorzec może nie być gotowy do uruchamiania aplikacji w chmurze publicznej.</span><span class="sxs-lookup"><span data-stu-id="68342-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="68342-107">Niemniej jednak firma może nie być ekonomicznie wykonalna w celu utrzymania pojemności wymaganej w środowisku lokalnym w celu obsłużenia podaży na żądanie dla aplikacji.</span><span class="sxs-lookup"><span data-stu-id="68342-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="68342-108">Twoja dzierżawa może korzystać z elastyczności chmury publicznej za pomocą rozwiązania lokalnego.</span><span class="sxs-lookup"><span data-stu-id="68342-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="68342-109">W tym rozwiązaniu utworzysz przykładowe środowisko w celu:</span><span class="sxs-lookup"><span data-stu-id="68342-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="68342-110">Utwórz wielowęzłową aplikację sieci Web.</span><span class="sxs-lookup"><span data-stu-id="68342-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="68342-111">Skonfiguruj proces ciągłego wdrażania (CD) i Zarządzaj nim.</span><span class="sxs-lookup"><span data-stu-id="68342-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="68342-112">Opublikuj aplikację sieci Web w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="68342-113">Utwórz wydanie.</span><span class="sxs-lookup"><span data-stu-id="68342-113">Create a release.</span></span>
> - <span data-ttu-id="68342-114">Dowiedz się, jak monitorować i śledzić wdrożenia.</span><span class="sxs-lookup"><span data-stu-id="68342-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="68342-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="68342-115">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="68342-116">Microsoft Azure Stack Hub to rozszerzenie platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="68342-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="68342-117">Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.</span><span class="sxs-lookup"><span data-stu-id="68342-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="68342-118">[Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych.</span><span class="sxs-lookup"><span data-stu-id="68342-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="68342-119">Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.</span><span class="sxs-lookup"><span data-stu-id="68342-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="68342-120">Wymagania wstępne</span><span class="sxs-lookup"><span data-stu-id="68342-120">Prerequisites</span></span>

- <span data-ttu-id="68342-121">Subskrypcja platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="68342-121">Azure subscription.</span></span> <span data-ttu-id="68342-122">W razie konieczności przed rozpoczęciem Utwórz [bezpłatne konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) .</span><span class="sxs-lookup"><span data-stu-id="68342-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="68342-123">Zintegrowany system Azure Stack Hub lub wdrożenie Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="68342-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="68342-124">Aby uzyskać instrukcje dotyczące instalowania centrum Azure Stack, zobacz [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="68342-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install.md).</span></span>
  - <span data-ttu-id="68342-125">W przypadku skryptu automatyzacji po wdrożeniu ASDK przejdź do: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="68342-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="68342-126">Instalacja może wymagać kilku godzin.</span><span class="sxs-lookup"><span data-stu-id="68342-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="68342-127">Wdróż [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) usługi PaaS Services w usłudze Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="68342-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="68342-128">[Utwórz plany/oferty](/azure-stack/operator/service-plan-offer-subscription-overview.md) w ramach środowiska Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="68342-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="68342-129">[Utwórz subskrypcję dzierżawy](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) w ramach środowiska Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="68342-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="68342-130">Utwórz aplikację internetową w ramach subskrypcji dzierżawy.</span><span class="sxs-lookup"><span data-stu-id="68342-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="68342-131">Zanotuj nowy adres URL aplikacji sieci Web do późniejszego użycia.</span><span class="sxs-lookup"><span data-stu-id="68342-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="68342-132">Wdróż Azure Pipelines maszynę wirtualną w ramach subskrypcji dzierżawy.</span><span class="sxs-lookup"><span data-stu-id="68342-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="68342-133">Wymagana jest maszyna wirtualna z systemem Windows Server 2016 z platformą .NET 3,5.</span><span class="sxs-lookup"><span data-stu-id="68342-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="68342-134">Ta maszyna wirtualna zostanie utworzona w ramach subskrypcji dzierżawy na Azure Stack Hub jako prywatny agent kompilacji.</span><span class="sxs-lookup"><span data-stu-id="68342-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="68342-135">[System Windows Server 2016 z obrazem maszyny wirtualnej SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) jest dostępny w witrynie Centrum Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="68342-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image.md) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="68342-136">Jeśli ten obraz nie jest dostępny, należy skontaktować się z operatorem centrum Azure Stack, aby upewnić się, że został on dodany do środowiska.</span><span class="sxs-lookup"><span data-stu-id="68342-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="68342-137">Problemy i kwestie do rozważenia</span><span class="sxs-lookup"><span data-stu-id="68342-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="68342-138">Skalowalność</span><span class="sxs-lookup"><span data-stu-id="68342-138">Scalability</span></span>

<span data-ttu-id="68342-139">Kluczowym elementem skalowania między chmurami jest możliwość dostarczania natychmiastowych i na żądanie skalowania między publiczną i lokalną infrastrukturą chmurową, zapewniając spójną i niezawodną usługę.</span><span class="sxs-lookup"><span data-stu-id="68342-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="68342-140">Dostępność</span><span class="sxs-lookup"><span data-stu-id="68342-140">Availability</span></span>

<span data-ttu-id="68342-141">Upewnij się, że aplikacje wdrożone lokalnie są skonfigurowane pod kątem wysokiej dostępności za poorednictwem konfiguracji sprzętu lokalnego i wdrożenia oprogramowania.</span><span class="sxs-lookup"><span data-stu-id="68342-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="68342-142">Możliwości zarządzania</span><span class="sxs-lookup"><span data-stu-id="68342-142">Manageability</span></span>

<span data-ttu-id="68342-143">Rozwiązanie międzychmurowe zapewnia bezproblemowe zarządzanie i przyjazny interfejs między środowiskami.</span><span class="sxs-lookup"><span data-stu-id="68342-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="68342-144">Program PowerShell jest zalecany w przypadku zarządzania na wielu platformach.</span><span class="sxs-lookup"><span data-stu-id="68342-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="68342-145">Skalowanie między chmurami</span><span class="sxs-lookup"><span data-stu-id="68342-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="68342-146">Pobierz domenę niestandardową i skonfiguruj system DNS</span><span class="sxs-lookup"><span data-stu-id="68342-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="68342-147">Zaktualizuj plik strefy DNS dla domeny.</span><span class="sxs-lookup"><span data-stu-id="68342-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="68342-148">Usługa Azure AD sprawdzi własność niestandardowej nazwy domeny.</span><span class="sxs-lookup"><span data-stu-id="68342-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="68342-149">Użyj [Azure DNS](/azure/dns/dns-getstarted-portal) dla systemu azure/Microsoft 365/zewnętrznych rekordów DNS na platformie Azure lub Dodaj wpis DNS w [innym rejestratorze DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="68342-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="68342-150">Zarejestruj domenę niestandardową z rejestratorem publicznym.</span><span class="sxs-lookup"><span data-stu-id="68342-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="68342-151">Zaloguj się do rejestratora nazw domen dla domeny.</span><span class="sxs-lookup"><span data-stu-id="68342-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="68342-152">Aby można było zaktualizować DNS, może być wymagane zatwierdzenie administratora.</span><span class="sxs-lookup"><span data-stu-id="68342-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="68342-153">Zaktualizuj plik strefy DNS dla domeny przez dodanie wpisu DNS dostarczonego przez usługę Azure AD.</span><span class="sxs-lookup"><span data-stu-id="68342-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="68342-154">(Wpis DNS nie dotyczy zachowań routingu e-mail ani usług hostingu sieci Web).</span><span class="sxs-lookup"><span data-stu-id="68342-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="68342-155">Tworzenie domyślnej wielowęzłowej aplikacji sieci Web w centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="68342-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="68342-156">Skonfiguruj hybrydową ciągłą integrację i ciągłe wdrażanie (CI/CD), aby wdrażać aplikacje sieci Web na platformie Azure i w Azure Stack Hub oraz w celu samowypychania zmian w obu chmurach.</span><span class="sxs-lookup"><span data-stu-id="68342-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="68342-157">Wymagane są Azure Stack centrum z odpowiednimi obrazami do uruchomienia (system Windows Server i SQL) oraz wdrożenie App Service.</span><span class="sxs-lookup"><span data-stu-id="68342-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="68342-158">Aby uzyskać więcej informacji, zapoznaj się z dokumentacją App Service [wymagania wstępne dotyczące wdrażania App Service w centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="68342-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="68342-159">Dodaj kod do Azure Repos</span><span class="sxs-lookup"><span data-stu-id="68342-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="68342-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="68342-160">Azure Repos</span></span>

1. <span data-ttu-id="68342-161">Zaloguj się do Azure Repos przy użyciu konta, które ma prawa do tworzenia projektu na Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="68342-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="68342-162">Hybrydowej ciągłej integracji/ciągłego dostarczania można użyć zarówno w kodzie aplikacji, jak i w kodzie infrastruktury.</span><span class="sxs-lookup"><span data-stu-id="68342-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="68342-163">Używaj [szablonów Azure Resource Manager](https://azure.microsoft.com/resources/templates/) zarówno do programowania w chmurze prywatnej, jak i hostowanej.</span><span class="sxs-lookup"><span data-stu-id="68342-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Nawiązywanie połączenia z projektem w Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="68342-165">**Sklonuj repozytorium** , tworząc i otwierając domyślną aplikację sieci Web.</span><span class="sxs-lookup"><span data-stu-id="68342-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Klonowanie repozytorium w usłudze Azure Web App](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="68342-167">Utwórz samodzielne wdrożenie aplikacji sieci Web dla App Services w obu chmurach</span><span class="sxs-lookup"><span data-stu-id="68342-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="68342-168">Edytuj plik **WebApplication. csproj** .</span><span class="sxs-lookup"><span data-stu-id="68342-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="68342-169">Wybierz `Runtimeidentifier` i Dodaj `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="68342-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="68342-170">(Zobacz dokumentację [wdrożenia prewartą](/dotnet/core/deploying/deploy-with-vs#simpleSelf) ).</span><span class="sxs-lookup"><span data-stu-id="68342-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Edytuj plik projektu aplikacji sieci Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="68342-172">Zaewidencjonuj kod, aby Azure Repos przy użyciu Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="68342-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="68342-173">Upewnij się, że kod aplikacji został zaewidencjonowany do Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="68342-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="68342-174">Tworzenie definicji kompilacji</span><span class="sxs-lookup"><span data-stu-id="68342-174">Create the build definition</span></span>

1. <span data-ttu-id="68342-175">Zaloguj się do Azure Pipelines, aby potwierdzić możliwość tworzenia definicji kompilacji.</span><span class="sxs-lookup"><span data-stu-id="68342-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="68342-176">Kod Add **-r Win10-x64** .</span><span class="sxs-lookup"><span data-stu-id="68342-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="68342-177">Jest to konieczne do wyzwolenia wdrożenia samodzielnego z platformą .NET Core.</span><span class="sxs-lookup"><span data-stu-id="68342-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Dodawanie kodu do aplikacji sieci Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="68342-179">Uruchom kompilację.</span><span class="sxs-lookup"><span data-stu-id="68342-179">Run the build.</span></span> <span data-ttu-id="68342-180">Proces [kompilacji samodzielnego wdrażania](/dotnet/core/deploying/deploy-with-vs#simpleSelf) spowoduje opublikowanie artefaktów działających na platformie Azure i w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="68342-181">Korzystanie z agenta hostowanego na platformie Azure</span><span class="sxs-lookup"><span data-stu-id="68342-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="68342-182">Korzystanie z hostowanego agenta kompilacji w Azure Pipelines jest wygodną opcją kompilowania i wdrażania aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="68342-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="68342-183">Konserwacja i uaktualnienia są wykonywane automatycznie przez Microsoft Azure, co umożliwia ciągły i nieprzerwany cykl programowania.</span><span class="sxs-lookup"><span data-stu-id="68342-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="68342-184">Zarządzanie i Konfigurowanie procesu CD</span><span class="sxs-lookup"><span data-stu-id="68342-184">Manage and configure the CD process</span></span>

<span data-ttu-id="68342-185">Azure Pipelines i Azure DevOps Services zapewniają wysoce konfigurowalny i zarządzany potok dla wydań w wielu środowiskach, takich jak programowanie, przemieszczanie, pytania i odpowiedzi oraz środowiska produkcyjne; obejmuje to wymaganie zatwierdzeń na określonych etapach.</span><span class="sxs-lookup"><span data-stu-id="68342-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="68342-186">Utwórz definicję wydania</span><span class="sxs-lookup"><span data-stu-id="68342-186">Create release definition</span></span>

1. <span data-ttu-id="68342-187">Wybierz przycisk **Plus** , aby dodać nową wersję na karcie **wersje** w sekcji **kompilacja i wersja** Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="68342-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Tworzenie definicji wydania](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="68342-189">Zastosuj szablon wdrażania Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="68342-189">Apply the Azure App Service Deployment template.</span></span>

   ![Zastosuj szablon wdrożenia Azure App Service](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="68342-191">W obszarze **Dodawanie artefaktu**Dodaj artefakt dla aplikacji Azure Cloud Build.</span><span class="sxs-lookup"><span data-stu-id="68342-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Dodawanie artefaktu do kompilacji w chmurze platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="68342-193">Na karcie potok wybierz pozycję **faza, link zadania** środowiska i ustaw wartości środowiska chmury platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="68342-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Ustawianie wartości środowiska chmury platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="68342-195">Ustaw **nazwę środowiska** i wybierz **subskrypcję platformy Azure** dla punktu końcowego w chmurze platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="68342-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Wybierz subskrypcję platformy Azure dla punktu końcowego w chmurze platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="68342-197">W obszarze **nazwa usługi App Service**Ustaw wymaganą nazwę usługi Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="68342-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Ustawianie nazwy usługi Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="68342-199">Wprowadź wartość "hostowana program VS2017" w obszarze **Kolejka agenta** dla środowiska hostowanego w chmurze platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="68342-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Ustawianie kolejki agentów dla środowiska hostowanego w chmurze platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="68342-201">W menu Wdróż Azure App Service wybierz prawidłowy **pakiet lub folder** dla środowiska.</span><span class="sxs-lookup"><span data-stu-id="68342-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="68342-202">Wybierz pozycję **OK** , aby określić **lokalizację folderu**.</span><span class="sxs-lookup"><span data-stu-id="68342-202">Select **OK** to **folder location**.</span></span>
  
      ![Wybierz pakiet lub folder dla środowiska Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Wybierz pakiet lub folder dla środowiska Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="68342-205">Zapisz wszystkie zmiany i wróć do **potoku wydania**.</span><span class="sxs-lookup"><span data-stu-id="68342-205">Save all changes and go back to **release pipeline**.</span></span>

    ![Zapisz zmiany w potoku wydania](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="68342-207">Dodaj nowy artefakt, wybierając kompilację dla aplikacji Centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Dodawanie nowego artefaktu dla aplikacji Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="68342-209">Dodaj jeszcze jedno środowisko, stosując wdrożenie Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="68342-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Dodawanie środowiska do wdrożenia Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="68342-211">Nazwij nowe środowisko "Azure Stack".</span><span class="sxs-lookup"><span data-stu-id="68342-211">Name the new environment "Azure Stack".</span></span>

    ![Nazwa środowiska w Azure App Service wdrożenia](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="68342-213">Znajdź środowisko Azure Stack na karcie **zadanie** .</span><span class="sxs-lookup"><span data-stu-id="68342-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Środowisko Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="68342-215">Wybierz subskrypcję dla punktu końcowego Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Wybierz subskrypcję dla punktu końcowego Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="68342-217">Jako nazwę usługi App Service Ustaw nazwę aplikacji sieci Web Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="68342-218">![Ustaw Azure Stack nazwę aplikacji sieci Web](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="68342-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="68342-219">Wybierz agenta Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-219">Select the Azure Stack agent.</span></span>

    ![Wybierz agenta Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="68342-221">W sekcji Wdróż Azure App Service wybierz prawidłowy **pakiet lub folder** dla środowiska.</span><span class="sxs-lookup"><span data-stu-id="68342-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="68342-222">Wybierz pozycję **OK** , aby określić lokalizację folderu.</span><span class="sxs-lookup"><span data-stu-id="68342-222">Select **OK** to folder location.</span></span>

    ![Wybierz folder do wdrożenia Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Wybierz folder do wdrożenia Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="68342-225">W obszarze Karta zmienna Dodaj zmienną o nazwie `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , ustaw jej wartość na **true**i zakres na Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Dodawanie zmiennej do wdrożenia aplikacji platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="68342-227">Wybierz ikonę wyzwalacza **ciągłego** wdrażania w obu artefaktach i Włącz wyzwalacz wdrażania **Kontynuuj** .</span><span class="sxs-lookup"><span data-stu-id="68342-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Wybierz wyzwalacz ciągłego wdrażania](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="68342-229">Wybierz ikonę warunki **przed wdrożeniem** w środowisku Azure Stack i ustaw wyzwalacz na **po wydaniu.**</span><span class="sxs-lookup"><span data-stu-id="68342-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Wybieranie warunków przed wdrożeniem](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="68342-231">Zapisz wszystkie zmiany.</span><span class="sxs-lookup"><span data-stu-id="68342-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="68342-232">Niektóre ustawienia zadań mogą zostać automatycznie zdefiniowane jako [zmienne środowiskowe](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) podczas tworzenia definicji wydania na podstawie szablonu.</span><span class="sxs-lookup"><span data-stu-id="68342-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="68342-233">Tych ustawień nie można modyfikować w ustawieniach zadania; Zamiast tego należy wybrać element środowiska nadrzędnego, aby edytować te ustawienia.</span><span class="sxs-lookup"><span data-stu-id="68342-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="68342-234">Publikowanie w centrum Azure Stack przy użyciu programu Visual Studio</span><span class="sxs-lookup"><span data-stu-id="68342-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="68342-235">Tworząc punkty końcowe, kompilacja Azure DevOps Services może wdrożyć aplikacje usługi platformy Azure w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="68342-236">Azure Pipelines nawiązuje połączenie z agentem kompilacji, który łączy się z centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="68342-237">Zaloguj się do Azure DevOps Services i przejdź do strony Ustawienia aplikacji.</span><span class="sxs-lookup"><span data-stu-id="68342-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="68342-238">W obszarze **Ustawienia**wybierz pozycję **zabezpieczenia**.</span><span class="sxs-lookup"><span data-stu-id="68342-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="68342-239">W obszarze **grupy VSTS**wybierz pozycję **twórcy punktów końcowych**.</span><span class="sxs-lookup"><span data-stu-id="68342-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="68342-240">Na karcie **Członkowie** wybierz pozycję **Dodaj**.</span><span class="sxs-lookup"><span data-stu-id="68342-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="68342-241">W obszarze **Dodawanie użytkowników i grup**wprowadź nazwę użytkownika i wybierz tego użytkownika z listy użytkowników.</span><span class="sxs-lookup"><span data-stu-id="68342-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="68342-242">Wybierz pozycję **Save changes** (Zapisz zmiany).</span><span class="sxs-lookup"><span data-stu-id="68342-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="68342-243">Na liście **grupy VSTS** wybierz pozycję **administratorzy punktów końcowych**.</span><span class="sxs-lookup"><span data-stu-id="68342-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="68342-244">Na karcie **Członkowie** wybierz pozycję **Dodaj**.</span><span class="sxs-lookup"><span data-stu-id="68342-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="68342-245">W obszarze **Dodawanie użytkowników i grup**wprowadź nazwę użytkownika i wybierz tego użytkownika z listy użytkowników.</span><span class="sxs-lookup"><span data-stu-id="68342-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="68342-246">Wybierz pozycję **Save changes** (Zapisz zmiany).</span><span class="sxs-lookup"><span data-stu-id="68342-246">Select **Save changes**.</span></span>

<span data-ttu-id="68342-247">Teraz, gdy istnieją informacje o punkcie końcowym, Azure Pipelines do połączenia z centrum Azure Stack jest gotowy do użycia.</span><span class="sxs-lookup"><span data-stu-id="68342-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="68342-248">Agent kompilacji w Azure Stack Hub otrzymuje instrukcje od Azure Pipelines, a następnie Agent przekazuje informacje o punkcie końcowym komunikacji z centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="68342-249">Opracowywanie kompilacji aplikacji</span><span class="sxs-lookup"><span data-stu-id="68342-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="68342-250">Wymagane są Azure Stack centrum z odpowiednimi obrazami do uruchomienia (system Windows Server i SQL) oraz wdrożenie App Service.</span><span class="sxs-lookup"><span data-stu-id="68342-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="68342-251">Aby uzyskać więcej informacji, zobacz [wymagania wstępne dotyczące wdrażania App Service w centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span><span class="sxs-lookup"><span data-stu-id="68342-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

<span data-ttu-id="68342-252">Użyj [szablonów Azure Resource Manager](https://azure.microsoft.com/resources/templates/) , takich jak kod aplikacji sieci web z Azure Repos do wdrożenia w obu chmurach.</span><span class="sxs-lookup"><span data-stu-id="68342-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="68342-253">Dodawanie kodu do projektu Azure Repos</span><span class="sxs-lookup"><span data-stu-id="68342-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="68342-254">Zaloguj się do Azure Repos przy użyciu konta, które ma prawa do tworzenia projektu w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="68342-255">**Sklonuj repozytorium** , tworząc i otwierając domyślną aplikację sieci Web.</span><span class="sxs-lookup"><span data-stu-id="68342-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="68342-256">Utwórz samodzielne wdrożenie aplikacji sieci Web dla App Services w obu chmurach</span><span class="sxs-lookup"><span data-stu-id="68342-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="68342-257">Edytuj plik **WebApplication. csproj** : Wybierz `Runtimeidentifier` , a następnie Dodaj `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="68342-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="68342-258">Aby uzyskać więcej informacji, zobacz dokumentację dotyczącą [wdrażania samodzielnego](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .</span><span class="sxs-lookup"><span data-stu-id="68342-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="68342-259">Użyj Team Explorer, aby sprawdzić kod w Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="68342-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="68342-260">Upewnij się, że kod aplikacji został zaewidencjonowany w Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="68342-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="68342-261">Tworzenie definicji kompilacji</span><span class="sxs-lookup"><span data-stu-id="68342-261">Create the build definition</span></span>

1. <span data-ttu-id="68342-262">Zaloguj się do Azure Pipelines przy użyciu konta, które może utworzyć definicję kompilacji.</span><span class="sxs-lookup"><span data-stu-id="68342-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="68342-263">Przejdź do strony **Kompilowanie aplikacji sieci Web** dla projektu.</span><span class="sxs-lookup"><span data-stu-id="68342-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="68342-264">W **argumentach**Add **-r Win10-x64** Code.</span><span class="sxs-lookup"><span data-stu-id="68342-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="68342-265">To dodanie jest wymagane do wyzwolenia wdrożenia samodzielnego z platformą .NET Core.</span><span class="sxs-lookup"><span data-stu-id="68342-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="68342-266">Uruchom kompilację.</span><span class="sxs-lookup"><span data-stu-id="68342-266">Run the build.</span></span> <span data-ttu-id="68342-267">Proces [kompilacji samodzielnego wdrożenia](/dotnet/core/deploying/deploy-with-vs#simpleSelf) spowoduje opublikowanie artefaktów, które mogą być uruchamiane na platformie Azure i w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="68342-268">Korzystanie z agenta kompilacji hostowanej na platformie Azure</span><span class="sxs-lookup"><span data-stu-id="68342-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="68342-269">Korzystanie z hostowanego agenta kompilacji w Azure Pipelines jest wygodną opcją kompilowania i wdrażania aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="68342-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="68342-270">Konserwacja i uaktualnienia są wykonywane automatycznie przez Microsoft Azure, co umożliwia ciągły i nieprzerwany cykl programowania.</span><span class="sxs-lookup"><span data-stu-id="68342-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="68342-271">Konfigurowanie procesu ciągłego wdrażania (CD)</span><span class="sxs-lookup"><span data-stu-id="68342-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="68342-272">Azure Pipelines i Azure DevOps Services zapewniają wysoce konfigurowalny i zarządzany potok dla wersji w wielu środowiskach, takich jak programowanie, przemieszczanie, gwarancja jakości i środowisko produkcyjne.</span><span class="sxs-lookup"><span data-stu-id="68342-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="68342-273">Ten proces może obejmować wymaganie zatwierdzeń na określonych etapach cyklu życia aplikacji.</span><span class="sxs-lookup"><span data-stu-id="68342-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="68342-274">Utwórz definicję wydania</span><span class="sxs-lookup"><span data-stu-id="68342-274">Create release definition</span></span>

<span data-ttu-id="68342-275">Tworzenie definicji wydania jest ostatnim krokiem w procesie kompilacji aplikacji.</span><span class="sxs-lookup"><span data-stu-id="68342-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="68342-276">Ta definicja wydania służy do tworzenia wydania i wdrożenia kompilacji.</span><span class="sxs-lookup"><span data-stu-id="68342-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="68342-277">Zaloguj się do Azure Pipelines i przejdź do pozycji **kompilacja i wydanie** dla projektu.</span><span class="sxs-lookup"><span data-stu-id="68342-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="68342-278">Na karcie **wersje** wybierz opcję **[+]** , a następnie wybierz pozycję **Utwórz definicję wydania**.</span><span class="sxs-lookup"><span data-stu-id="68342-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="68342-279">W obszarze **Wybierz szablon**wybierz pozycję **wdrożenie Azure App Service**, a następnie wybierz pozycję **Zastosuj**.</span><span class="sxs-lookup"><span data-stu-id="68342-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="68342-280">Na stronie **Dodawanie artefaktu**ze **źródła (definicji kompilacji)** wybierz aplikację Azure Cloud Build.</span><span class="sxs-lookup"><span data-stu-id="68342-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="68342-281">Na karcie **potok** wybierz link **1 faza**, **1 zadanie** , aby **wyświetlić zadania środowiska**.</span><span class="sxs-lookup"><span data-stu-id="68342-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="68342-282">Na karcie **zadania** wprowadź wartość Azure jako **nazwę środowiska** , a następnie wybierz pozycję AzureCloud Traders — Web EP z listy **subskrypcji platformy Azure** .</span><span class="sxs-lookup"><span data-stu-id="68342-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="68342-283">Wprowadź **nazwę usługi Azure App Service**, która znajduje się `northwindtraders` w następnym przechwyceniu ekranu.</span><span class="sxs-lookup"><span data-stu-id="68342-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="68342-284">W przypadku fazy agenta wybierz pozycję **hostowana program VS2017** z listy **kolejek agentów** .</span><span class="sxs-lookup"><span data-stu-id="68342-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="68342-285">W obszarze **wdróż Azure App Service**wybierz prawidłowy **pakiet lub folder** dla środowiska.</span><span class="sxs-lookup"><span data-stu-id="68342-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="68342-286">W obszarze **Wybierz plik lub folder**wybierz pozycję **OK** , aby określić **lokalizację**.</span><span class="sxs-lookup"><span data-stu-id="68342-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="68342-287">Zapisz wszystkie zmiany i wróć do **potoku**.</span><span class="sxs-lookup"><span data-stu-id="68342-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="68342-288">Na karcie **potok** wybierz pozycję **Dodaj artefakt**i wybierz pozycję **NorthwindCloud Traders-statek** ze **źródła (definicji kompilacji)** .</span><span class="sxs-lookup"><span data-stu-id="68342-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="68342-289">Po **wybraniu szablonu**Dodaj inne środowisko.</span><span class="sxs-lookup"><span data-stu-id="68342-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="68342-290">Wybierz **wdrożenie Azure App Service** a następnie wybierz pozycję **Zastosuj**.</span><span class="sxs-lookup"><span data-stu-id="68342-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="68342-291">Wprowadź `Azure Stack Hub` jako **nazwę środowiska**.</span><span class="sxs-lookup"><span data-stu-id="68342-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="68342-292">Na karcie **zadania** Znajdź i wybierz pozycję Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="68342-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="68342-293">Z listy **subskrypcja platformy Azure** wybierz pozycję **AzureStack Traders — statek EP** dla punktu końcowego centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="68342-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="68342-294">Wprowadź nazwę aplikacji sieci Web Azure Stack Hub jako **nazwę usługi App Service**.</span><span class="sxs-lookup"><span data-stu-id="68342-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="68342-295">W obszarze **wybór agenta**wybierz pozycję **AzureStack-b Douglas Fir** z listy **kolejek agentów** .</span><span class="sxs-lookup"><span data-stu-id="68342-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="68342-296">W polu **wdróż Azure App Service**wybierz prawidłowy **pakiet lub folder** dla środowiska.</span><span class="sxs-lookup"><span data-stu-id="68342-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="68342-297">Na stronie **Wybieranie pliku lub folderu**wybierz pozycję **OK** dla **lokalizacji**folderu.</span><span class="sxs-lookup"><span data-stu-id="68342-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="68342-298">Na karcie **zmienna** Znajdź zmienną o nazwie `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` .</span><span class="sxs-lookup"><span data-stu-id="68342-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="68342-299">Ustaw wartość zmiennej na **true**, a następnie ustaw jej zakres na **Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="68342-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="68342-300">Na karcie **potok** wybierz ikonę **wyzwalacza ciągłego wdrażania** dla artefaktu sieci Web NorthwindCloud Traders i ustaw **wyzwalacz ciągłego wdrażania** na **włączone**.</span><span class="sxs-lookup"><span data-stu-id="68342-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="68342-301">Wykonaj tę samą czynność dla artefaktu **NorthwindCloud Traders-zbiornika** .</span><span class="sxs-lookup"><span data-stu-id="68342-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="68342-302">W przypadku środowiska Azure Stack Hub wybierz ikonę **warunki przed wdrożeniem** Ustaw wyzwalacz na **po wydaniu**.</span><span class="sxs-lookup"><span data-stu-id="68342-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="68342-303">Zapisz wszystkie zmiany.</span><span class="sxs-lookup"><span data-stu-id="68342-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="68342-304">Niektóre ustawienia zadań zlecenia są automatycznie definiowane jako [zmienne środowiskowe](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) podczas tworzenia definicji wydania na podstawie szablonu.</span><span class="sxs-lookup"><span data-stu-id="68342-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="68342-305">Tych ustawień nie można modyfikować w ustawieniach zadania, ale można je modyfikować w elementach środowiska nadrzędnego.</span><span class="sxs-lookup"><span data-stu-id="68342-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="68342-306">Tworzenie wersji</span><span class="sxs-lookup"><span data-stu-id="68342-306">Create a release</span></span>

1. <span data-ttu-id="68342-307">Na karcie **potok** Otwórz listę **wersja** i wybierz pozycję **Utwórz wydanie**.</span><span class="sxs-lookup"><span data-stu-id="68342-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="68342-308">Wprowadź opis wydania, sprawdź, czy wybrano poprawne artefakty, a następnie wybierz pozycję **Utwórz**.</span><span class="sxs-lookup"><span data-stu-id="68342-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="68342-309">Po kilku chwilach pojawi się transparent wskazujący, że nowa wersja została utworzona, a nazwa wydania jest wyświetlana jako link.</span><span class="sxs-lookup"><span data-stu-id="68342-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="68342-310">Wybierz łącze, aby wyświetlić stronę podsumowania wydania.</span><span class="sxs-lookup"><span data-stu-id="68342-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="68342-311">Na stronie Podsumowanie wydania są wyświetlane szczegółowe informacje o wersji.</span><span class="sxs-lookup"><span data-stu-id="68342-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="68342-312">W poniższym przechwyceniu ekranu dla wersji "Release-2" sekcja **środowiska** pokazuje **stan wdrożenia** platformy Azure jako "w toku", a stan centrum Azure Stack to "powodzenie".</span><span class="sxs-lookup"><span data-stu-id="68342-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="68342-313">Gdy stan wdrożenia środowiska platformy Azure zmieni się na "powodzenie", zostanie wyświetlony transparent wskazujący, że wydanie jest gotowe do zatwierdzenia.</span><span class="sxs-lookup"><span data-stu-id="68342-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="68342-314">Gdy wdrożenie oczekuje lub nie powiodło się, wyświetlana jest ikona informacji o niebieskie **(i)** .</span><span class="sxs-lookup"><span data-stu-id="68342-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="68342-315">Umieść kursor nad ikoną, aby wyświetlić okno podręczne zawierające przyczynę opóźnienia lub niepowodzenia.</span><span class="sxs-lookup"><span data-stu-id="68342-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="68342-316">Inne widoki, takie jak lista wydań, wyświetlają również ikonę wskazującą, że zatwierdzenie jest w stanie oczekiwania.</span><span class="sxs-lookup"><span data-stu-id="68342-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="68342-317">W oknie podręcznym dla tej ikony zostanie wyświetlona nazwa środowiska i więcej szczegółów dotyczących wdrożenia.</span><span class="sxs-lookup"><span data-stu-id="68342-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="68342-318">W łatwy sposób administrator widzi ogólny postęp wydań i widzi, które wydania oczekują na zatwierdzenie.</span><span class="sxs-lookup"><span data-stu-id="68342-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="68342-319">Monitorowanie i śledzenie wdrożeń</span><span class="sxs-lookup"><span data-stu-id="68342-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="68342-320">Na stronie Podsumowanie **wersji 2** wybierz pozycję **dzienniki**.</span><span class="sxs-lookup"><span data-stu-id="68342-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="68342-321">Podczas wdrażania na tej stronie jest wyświetlany dziennik na żywo od agenta.</span><span class="sxs-lookup"><span data-stu-id="68342-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="68342-322">W okienku po lewej stronie jest wyświetlany stan każdej operacji we wdrożeniu dla każdego środowiska.</span><span class="sxs-lookup"><span data-stu-id="68342-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="68342-323">Wybierz ikonę osoby w kolumnie **Akcja** dla zatwierdzenia przed wdrożeniem lub po wdrożeniu, aby zobaczyć, kto zatwierdził (lub odrzucił) wdrożenie i podaną przez nie wiadomość.</span><span class="sxs-lookup"><span data-stu-id="68342-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="68342-324">Po zakończeniu wdrożenia cały plik dziennika zostanie wyświetlony w okienku po prawej stronie.</span><span class="sxs-lookup"><span data-stu-id="68342-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="68342-325">Wybierz dowolny **krok** w okienku po lewej stronie, aby wyświetlić plik dziennika dla pojedynczego kroku, na przykład **zainicjuj zadanie**.</span><span class="sxs-lookup"><span data-stu-id="68342-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="68342-326">Możliwość wyświetlania pojedynczych dzienników ułatwia śledzenie i debugowanie części ogólnego wdrożenia.</span><span class="sxs-lookup"><span data-stu-id="68342-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="68342-327">**Zapisz** plik dziennika dla kroku lub **Pobierz wszystkie dzienniki w formacie ZIP**.</span><span class="sxs-lookup"><span data-stu-id="68342-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="68342-328">Otwórz kartę **Podsumowanie** , aby wyświetlić ogólne informacje o wersji.</span><span class="sxs-lookup"><span data-stu-id="68342-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="68342-329">Ten widok przedstawia szczegółowe informacje o kompilacji, środowiskach, w których zostały wdrożone, o stanie wdrożenia i innych informacjach o wersji.</span><span class="sxs-lookup"><span data-stu-id="68342-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="68342-330">Wybierz łącze środowiska (**Azure** lub **Azure Stack Hub**), aby wyświetlić informacje o istniejących i oczekujących wdrożeniach w określonym środowisku.</span><span class="sxs-lookup"><span data-stu-id="68342-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="68342-331">Użyj tych widoków, aby szybko sprawdzić, czy ta sama kompilacja została wdrożona w obu środowiskach.</span><span class="sxs-lookup"><span data-stu-id="68342-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="68342-332">Otwórz **wdrożoną aplikację produkcyjną** w przeglądarce.</span><span class="sxs-lookup"><span data-stu-id="68342-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="68342-333">Na przykład dla witryny sieci Web usługi Azure App Services Otwórz adres URL `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="68342-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="68342-334">Integracja z platformą Azure i usługą Azure Stack Hub zapewnia skalowalne rozwiązanie dla wielu chmur</span><span class="sxs-lookup"><span data-stu-id="68342-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="68342-335">Elastyczna i niezawodna usługa w chmurze zapewnia bezpieczeństwo danych, tworzenie kopii zapasowych, spójność i szybka dostępność, skalowalność magazynu i dystrybucji oraz Routing zgodny ze geograficzną.</span><span class="sxs-lookup"><span data-stu-id="68342-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="68342-336">Ten proces jest wyzwalany ręcznie, zapewniając niezawodne i wydajne przełączanie między hostowanymi aplikacjami sieci Web i natychmiastową dostępnością kluczowych danych.</span><span class="sxs-lookup"><span data-stu-id="68342-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="68342-337">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="68342-337">Next steps</span></span>

- <span data-ttu-id="68342-338">Aby dowiedzieć się więcej o wzorcach chmury platformy Azure, zobacz [wzorce projektowe w chmurze](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="68342-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
