---
title: Kierowanie ruchu za pomocą aplikacji rozproszonej geograficznie przy użyciu platformy Azure i usługi Azure Stack Hub
description: Dowiedz się, jak skierować ruch do określonych punktów końcowych za pomocą rozwiązania do obsługi aplikacji rozproszonych geograficznie przy użyciu platformy Azure i Azure Stack centrum.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 9fa2c351d2c13d85fe1adb17a35e165de96ea2a2
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895435"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="943db-103">Kierowanie ruchu za pomocą aplikacji rozproszonej geograficznie przy użyciu platformy Azure i usługi Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="943db-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="943db-104">Dowiedz się, jak skierować ruch do określonych punktów końcowych na podstawie różnych metryk przy użyciu wzorca aplikacji rozproszonych geograficznie.</span><span class="sxs-lookup"><span data-stu-id="943db-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="943db-105">Tworzenie profilu Traffic Manager przy użyciu geograficznej konfiguracji routingu i punktu końcowego zapewnia, że informacje są kierowane do punktów końcowych w oparciu o wymagania regionalne, przepisy firmowe i międzynarodowe oraz Twoje potrzeby dotyczące danych.</span><span class="sxs-lookup"><span data-stu-id="943db-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="943db-106">W tym rozwiązaniu utworzysz przykładowe środowisko w celu:</span><span class="sxs-lookup"><span data-stu-id="943db-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="943db-107">Tworzenie aplikacji rozproszonej geograficznie.</span><span class="sxs-lookup"><span data-stu-id="943db-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="943db-108">Użyj Traffic Manager, aby określić aplikację jako docelową.</span><span class="sxs-lookup"><span data-stu-id="943db-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="943db-109">Używanie wzorca aplikacji rozproszonych geograficznie</span><span class="sxs-lookup"><span data-stu-id="943db-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="943db-110">Ze wzorcem rozproszonym geograficznie aplikacja obejmuje regiony.</span><span class="sxs-lookup"><span data-stu-id="943db-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="943db-111">Możesz domyślnie być chmurą publiczną, ale niektórzy użytkownicy mogą wymagać, aby ich dane pozostawały w ich regionie.</span><span class="sxs-lookup"><span data-stu-id="943db-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="943db-112">Możesz kierować użytkowników do najbardziej odpowiedniej chmury na podstawie ich wymagań.</span><span class="sxs-lookup"><span data-stu-id="943db-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="943db-113">Problemy i kwestie do rozważenia</span><span class="sxs-lookup"><span data-stu-id="943db-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="943db-114">Zagadnienia dotyczące skalowalności</span><span class="sxs-lookup"><span data-stu-id="943db-114">Scalability considerations</span></span>

<span data-ttu-id="943db-115">Rozwiązanie, które będziesz kompilować w tym artykule, nie uwzględnia skalowalności.</span><span class="sxs-lookup"><span data-stu-id="943db-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="943db-116">Jednak jeśli są używane w połączeniu z innymi rozwiązaniami Azure i lokalnymi, można obsłużyć wymagania dotyczące skalowalności.</span><span class="sxs-lookup"><span data-stu-id="943db-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="943db-117">Aby uzyskać informacje na temat tworzenia rozwiązania hybrydowego z skalowaniem automatycznym za pomocą usługi Traffic Manager, zobacz [Tworzenie rozwiązań do skalowania między chmurami przy użyciu platformy Azure](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="943db-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="943db-118">Zagadnienia dotyczące dostępności</span><span class="sxs-lookup"><span data-stu-id="943db-118">Availability considerations</span></span>

<span data-ttu-id="943db-119">Podobnie jak w przypadku problemów z skalowalnością, to rozwiązanie nie jest bezpośrednio związane z dostępnością.</span><span class="sxs-lookup"><span data-stu-id="943db-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="943db-120">Jednak rozwiązania platformy Azure i lokalne można zaimplementować w ramach tego rozwiązania, aby zapewnić wysoką dostępność dla wszystkich składników.</span><span class="sxs-lookup"><span data-stu-id="943db-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="943db-121">Kiedy używać tego wzorca</span><span class="sxs-lookup"><span data-stu-id="943db-121">When to use this pattern</span></span>

- <span data-ttu-id="943db-122">Organizacja ma rozgałęzienia międzynarodowe wymagające niestandardowych zasad zabezpieczeń i dystrybucji.</span><span class="sxs-lookup"><span data-stu-id="943db-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="943db-123">Każda z biur organizacji pobiera dane pracowników, firm i materiałów, które wymagają działania raportowania na lokalne regulacje i strefy czasowe.</span><span class="sxs-lookup"><span data-stu-id="943db-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="943db-124">Wymagania dotyczące wysokiego rozmiaru są spełnione przez skalowanie w poziomie aplikacji z wieloma wdrożeniami aplikacji w jednym regionie i w różnych regionach, aby obsługiwać skrajne wymagania dotyczące obciążenia.</span><span class="sxs-lookup"><span data-stu-id="943db-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="943db-125">Planowanie topologii</span><span class="sxs-lookup"><span data-stu-id="943db-125">Planning the topology</span></span>

<span data-ttu-id="943db-126">Przed rozpoczęciem tworzenia rozproszonej aplikacji można znać następujące kwestie:</span><span class="sxs-lookup"><span data-stu-id="943db-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="943db-127">**Domena niestandardowa dla aplikacji:** Jaka jest nazwa domeny niestandardowej, która będzie używana przez klientów w celu uzyskania dostępu do aplikacji?</span><span class="sxs-lookup"><span data-stu-id="943db-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="943db-128">W przypadku przykładowej aplikacji niestandardowa nazwa domeny to *www \. scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="943db-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="943db-129">**Traffic Manager domeny:** Nazwa domeny jest wybierana podczas tworzenia [profilu usługi Azure Traffic Manager](/azure/traffic-manager/traffic-manager-manage-profiles).</span><span class="sxs-lookup"><span data-stu-id="943db-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="943db-130">Ta nazwa jest połączona z sufiksem *trafficmanager.NET* w celu zarejestrowania wpisu domeny, który jest zarządzany przez Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="943db-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="943db-131">W przypadku przykładowej aplikacji wybrana nazwa to *skalowalne — Demonstracja*.</span><span class="sxs-lookup"><span data-stu-id="943db-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="943db-132">W związku z tym pełna nazwa domeny, która jest zarządzana przez Traffic Manager, to *Scalable-ASE-demo.trafficmanager.NET*.</span><span class="sxs-lookup"><span data-stu-id="943db-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="943db-133">**Strategia skalowania rozmiaru aplikacji:** Zdecyduj, czy rozmiar aplikacji będzie dystrybuowany między wieloma środowiskami App Service w jednym regionie, wielu regionach, czy też z obu obu sposobów.</span><span class="sxs-lookup"><span data-stu-id="943db-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="943db-134">Decyzja powinna być oparta na oczekiwaniach, w których nastąpi ruch klienta i jak również pozostała część aplikacji obsługującej infrastrukturę zaplecza może skalować.</span><span class="sxs-lookup"><span data-stu-id="943db-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="943db-135">Na przykład w przypadku aplikacji bezstanowej 100% aplikacja może być w znacznym stopniu skalowana przy użyciu kombinacji wielu App Serviceych środowisk w regionie świadczenia usługi Azure, pomnożonych przez App Service środowiska wdrożone w wielu regionach świadczenia usługi Azure.</span><span class="sxs-lookup"><span data-stu-id="943db-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="943db-136">Dzięki 15 i globalnym regionom platformy Azure dostępnym do wyboru klienci mogą naprawdę kompilować skalę na całym świecie.</span><span class="sxs-lookup"><span data-stu-id="943db-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="943db-137">W przypadku przykładowej aplikacji używanej w tym miejscu trzy środowiska App Service zostały utworzone w jednym regionie świadczenia usługi Azure (Południowo-środkowe stany USA).</span><span class="sxs-lookup"><span data-stu-id="943db-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="943db-138">**Konwencja nazewnictwa dla środowisk App Service:** Każde środowisko App Service wymaga unikatowej nazwy.</span><span class="sxs-lookup"><span data-stu-id="943db-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="943db-139">Poza jednym lub dwoma środowiskami App Service warto mieć konwencję nazewnictwa ułatwiającą identyfikację każdego środowiska App Service.</span><span class="sxs-lookup"><span data-stu-id="943db-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="943db-140">W przypadku przykładowej aplikacji użytej w tym miejscu użyto prostej konwencji nazewnictwa.</span><span class="sxs-lookup"><span data-stu-id="943db-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="943db-141">Nazwy trzech środowisk App Service to *fe1ase*, *fe2ase* i *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="943db-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="943db-142">**Konwencja nazewnictwa dla aplikacji:** Ponieważ zostanie wdrożonych wiele wystąpień aplikacji, wymagana jest nazwa dla każdego wystąpienia wdrożonej aplikacji.</span><span class="sxs-lookup"><span data-stu-id="943db-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="943db-143">Za pomocą App Service Environment dla aplikacji zaawansowanych można używać tej samej nazwy aplikacji w wielu środowiskach.</span><span class="sxs-lookup"><span data-stu-id="943db-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="943db-144">Ponieważ każde środowisko App Service ma unikatowy sufiks domeny, deweloperzy mogą skorzystać z dokładnej nazwy aplikacji w każdym środowisku.</span><span class="sxs-lookup"><span data-stu-id="943db-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="943db-145">Na przykład deweloper może mieć aplikacje o nazwie w następujący sposób: *MyApp.Foo1.p.azurewebsites.NET*, *MyApp.foo2.p.azurewebsites.NET*, *MyApp.Foo3.p.azurewebsites.NET* itd.</span><span class="sxs-lookup"><span data-stu-id="943db-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="943db-146">W przypadku aplikacji użytej w tym miejscu każde wystąpienie aplikacji ma unikatową nazwę.</span><span class="sxs-lookup"><span data-stu-id="943db-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="943db-147">Używane nazwy wystąpień aplikacji to *webfrontend1*, *webfrontend2* i *webfrontend3*.</span><span class="sxs-lookup"><span data-stu-id="943db-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="943db-148">![Diagram filarów hybrydowych](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="943db-148">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="943db-149">Microsoft Azure Stack Hub to rozszerzenie platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="943db-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="943db-150">Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.</span><span class="sxs-lookup"><span data-stu-id="943db-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="943db-151">[Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych.</span><span class="sxs-lookup"><span data-stu-id="943db-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="943db-152">Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.</span><span class="sxs-lookup"><span data-stu-id="943db-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="943db-153">Część 1. Tworzenie aplikacji rozproszonej geograficznie</span><span class="sxs-lookup"><span data-stu-id="943db-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="943db-154">W tej części utworzysz aplikację sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="943db-155">Twórz aplikacje sieci Web i Publikuj.</span><span class="sxs-lookup"><span data-stu-id="943db-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="943db-156">Dodaj kod do Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="943db-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="943db-157">Wskaż kompilację aplikacji w wielu celach w chmurze.</span><span class="sxs-lookup"><span data-stu-id="943db-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="943db-158">Zarządzanie procesem CD i konfigurowanie go.</span><span class="sxs-lookup"><span data-stu-id="943db-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="943db-159">Wymagania wstępne</span><span class="sxs-lookup"><span data-stu-id="943db-159">Prerequisites</span></span>

<span data-ttu-id="943db-160">Wymagana jest subskrypcja platformy Azure i instalacja centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="943db-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="943db-161">Kroki aplikacji rozproszonej geograficznie</span><span class="sxs-lookup"><span data-stu-id="943db-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="943db-162">Uzyskaj domenę niestandardową i skonfiguruj system DNS</span><span class="sxs-lookup"><span data-stu-id="943db-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="943db-163">Zaktualizuj plik strefy DNS dla domeny.</span><span class="sxs-lookup"><span data-stu-id="943db-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="943db-164">Usługa Azure AD może następnie zweryfikować własność niestandardowej nazwy domeny.</span><span class="sxs-lookup"><span data-stu-id="943db-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="943db-165">Użyj [Azure DNS](/azure/dns/dns-getstarted-portal) dla systemu azure/Microsoft 365/zewnętrznych rekordów DNS na platformie Azure lub Dodaj wpis DNS w [innym rejestratorze DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="943db-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="943db-166">Zarejestruj domenę niestandardową z rejestratorem publicznym.</span><span class="sxs-lookup"><span data-stu-id="943db-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="943db-167">Zaloguj się do rejestratora nazw domen dla domeny.</span><span class="sxs-lookup"><span data-stu-id="943db-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="943db-168">Aby można było zaktualizować DNS, może być wymagane zatwierdzenie administratora.</span><span class="sxs-lookup"><span data-stu-id="943db-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="943db-169">Zaktualizuj plik strefy DNS dla domeny przez dodanie wpisu DNS dostarczonego przez usługę Azure AD.</span><span class="sxs-lookup"><span data-stu-id="943db-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="943db-170">Wpis DNS nie zmienia zachowań, takich jak Routing poczty lub hosting internetowy.</span><span class="sxs-lookup"><span data-stu-id="943db-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="943db-171">Tworzenie aplikacji sieci Web i publikowanie</span><span class="sxs-lookup"><span data-stu-id="943db-171">Create web apps and publish</span></span>

<span data-ttu-id="943db-172">Skonfiguruj hybrydową ciągłą integrację/ciągłe dostarczanie (CI/CD), aby wdrożyć aplikację sieci Web na platformie Azure i w Azure Stack Hub, a następnie przeprowadź autowypychanie zmian w obu chmurach.</span><span class="sxs-lookup"><span data-stu-id="943db-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="943db-173">Wymagane są Azure Stack centrum z odpowiednimi obrazami do uruchomienia (system Windows Server i SQL) oraz wdrożenie App Service.</span><span class="sxs-lookup"><span data-stu-id="943db-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="943db-174">Aby uzyskać więcej informacji, zobacz [wymagania wstępne dotyczące wdrażania App Service w centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="943db-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="943db-175">Dodaj kod do Azure Repos</span><span class="sxs-lookup"><span data-stu-id="943db-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="943db-176">Zaloguj się do programu Visual Studio przy użyciu **konta, które ma prawa do tworzenia projektu** na Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="943db-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="943db-177">Ciągłej integracji/ciągłego wdrażania można użyć zarówno w kodzie aplikacji, jak i w kodzie infrastruktury.</span><span class="sxs-lookup"><span data-stu-id="943db-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="943db-178">Używaj [szablonów Azure Resource Manager](https://azure.microsoft.com/resources/templates/) zarówno do programowania w chmurze prywatnej, jak i hostowanej.</span><span class="sxs-lookup"><span data-stu-id="943db-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Nawiązywanie połączenia z projektem w programie Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="943db-180">**Sklonuj repozytorium** , tworząc i otwierając domyślną aplikację sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Klonowanie repozytorium w programie Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="943db-182">Utwórz wdrożenie aplikacji sieci Web w obu chmurach</span><span class="sxs-lookup"><span data-stu-id="943db-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="943db-183">Edytuj plik **WebApplication. csproj** : Wybierz `Runtimeidentifier` i Dodaj `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="943db-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="943db-184">(Zobacz dokumentację [wdrożenia prewartą](/dotnet/core/deploying/deploy-with-vs#simpleSelf) ).</span><span class="sxs-lookup"><span data-stu-id="943db-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Edytuj plik projektu aplikacji sieci Web w programie Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="943db-186">**Zaewidencjonuj kod, aby Azure Repos** przy użyciu Team Explorer.</span><span class="sxs-lookup"><span data-stu-id="943db-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="943db-187">Upewnij się, że **kod aplikacji** został zaewidencjonowany do Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="943db-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="943db-188">Tworzenie definicji kompilacji</span><span class="sxs-lookup"><span data-stu-id="943db-188">Create the build definition</span></span>

1. <span data-ttu-id="943db-189">**Zaloguj się do Azure Pipelines** , aby potwierdzić możliwość tworzenia definicji kompilacji.</span><span class="sxs-lookup"><span data-stu-id="943db-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="943db-190">Dodaj `-r win10-x64` kod.</span><span class="sxs-lookup"><span data-stu-id="943db-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="943db-191">Jest to konieczne do wyzwolenia wdrożenia samodzielnego z platformą .NET Core.</span><span class="sxs-lookup"><span data-stu-id="943db-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Dodawanie kodu do definicji kompilacji w Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="943db-193">**Uruchom kompilację**.</span><span class="sxs-lookup"><span data-stu-id="943db-193">**Run the build**.</span></span> <span data-ttu-id="943db-194">Proces [kompilacji samodzielnego wdrożenia](/dotnet/core/deploying/deploy-with-vs#simpleSelf) spowoduje opublikowanie artefaktów, które mogą być uruchamiane na platformie Azure i w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="943db-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="943db-195">Korzystanie z agenta hostowanego na platformie Azure</span><span class="sxs-lookup"><span data-stu-id="943db-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="943db-196">Korzystanie z hostowanego agenta w Azure Pipelines jest wygodną opcją kompilowania i wdrażania aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="943db-197">Konserwacja i uaktualnienia są wykonywane automatycznie przez Microsoft Azure, co umożliwia nieprzerwane programowanie, testowanie i wdrażanie.</span><span class="sxs-lookup"><span data-stu-id="943db-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="943db-198">Zarządzanie i Konfigurowanie procesu CD</span><span class="sxs-lookup"><span data-stu-id="943db-198">Manage and configure the CD process</span></span>

<span data-ttu-id="943db-199">Azure DevOps Services zapewnić wysoce konfigurowalny i zarządzany potok dla wydań w wielu środowiskach, takich jak programowanie, przemieszczanie, zarządzanie PYTANIAmi i środowiska produkcyjne; obejmuje to wymaganie zatwierdzeń na określonych etapach.</span><span class="sxs-lookup"><span data-stu-id="943db-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="943db-200">Utwórz definicję wydania</span><span class="sxs-lookup"><span data-stu-id="943db-200">Create release definition</span></span>

1. <span data-ttu-id="943db-201">Wybierz przycisk **Plus** , aby dodać nową wersję na karcie **wersje** w sekcji **kompilacja i wersja** Azure DevOps Services.</span><span class="sxs-lookup"><span data-stu-id="943db-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Tworzenie definicji wydania w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="943db-203">Zastosuj szablon wdrażania Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="943db-203">Apply the Azure App Service Deployment template.</span></span>

   ![Zastosuj szablon wdrożenia Azure App Service w programie Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="943db-205">W obszarze **Dodawanie artefaktu** Dodaj artefakt dla aplikacji Azure Cloud Build.</span><span class="sxs-lookup"><span data-stu-id="943db-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Dodaj artefakt do usługi Azure Cloud Build w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="943db-207">Na karcie potok wybierz pozycję **faza, link zadania** środowiska i ustaw wartości środowiska chmury platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="943db-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Ustaw wartości środowiska chmury platformy Azure w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="943db-209">Ustaw **nazwę środowiska** i wybierz **subskrypcję platformy Azure** dla punktu końcowego w chmurze platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="943db-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Wybierz subskrypcję platformy Azure dla punktu końcowego w chmurze platformy Azure w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="943db-211">W obszarze **nazwa usługi App Service** Ustaw wymaganą nazwę usługi Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="943db-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Ustaw nazwę usługi Azure App Service w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="943db-213">Wprowadź wartość "hostowana program VS2017" w obszarze **Kolejka agenta** dla środowiska hostowanego w chmurze platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="943db-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Ustaw kolejkę agentów dla środowiska hostowanego w chmurze platformy Azure w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="943db-215">W menu Wdróż Azure App Service wybierz prawidłowy **pakiet lub folder** dla środowiska.</span><span class="sxs-lookup"><span data-stu-id="943db-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="943db-216">Wybierz pozycję **OK** , aby określić **lokalizację folderu**.</span><span class="sxs-lookup"><span data-stu-id="943db-216">Select **OK** to **folder location**.</span></span>
  
      ![Wybierz pakiet lub folder dla środowiska Azure App Service w programie Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Okno dialogowe selektora folderów 1](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="943db-219">Zapisz wszystkie zmiany i wróć do **potoku wydania**.</span><span class="sxs-lookup"><span data-stu-id="943db-219">Save all changes and go back to **release pipeline**.</span></span>

    ![Zapisz zmiany w potoku wydania w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="943db-221">Dodaj nowy artefakt, wybierając kompilację dla aplikacji Centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="943db-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Dodawanie nowego artefaktu dla aplikacji Azure Stack Hub w programie Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="943db-223">Dodaj jeszcze jedno środowisko, stosując wdrożenie Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="943db-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Dodawanie środowiska do wdrożenia Azure App Service w programie Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="943db-225">Nazwij nowe środowisko Azure Stack centrum.</span><span class="sxs-lookup"><span data-stu-id="943db-225">Name the new environment Azure Stack Hub.</span></span>

    ![Nazwa środowiska w Azure App Service wdrożenia w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="943db-227">Znajdź środowisko centrum Azure Stack w obszarze Karta **zadań** .</span><span class="sxs-lookup"><span data-stu-id="943db-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Środowisko Azure Stack Hub w Azure DevOps Services w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="943db-229">Wybierz subskrypcję dla punktu końcowego Azure Stack centrum.</span><span class="sxs-lookup"><span data-stu-id="943db-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Wybierz subskrypcję punktu końcowego Azure Stack Hub w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="943db-231">Określ nazwę aplikacji sieci Web Azure Stack Hub jako nazwę usługi App Service.</span><span class="sxs-lookup"><span data-stu-id="943db-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Ustaw Azure Stack nazwę aplikacji sieci Web Hub w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="943db-233">Wybierz agenta Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="943db-233">Select the Azure Stack Hub agent.</span></span>

    ![Wybierz agenta Azure Stack Hub w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="943db-235">W sekcji Wdróż Azure App Service wybierz prawidłowy **pakiet lub folder** dla środowiska.</span><span class="sxs-lookup"><span data-stu-id="943db-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="943db-236">Wybierz pozycję **OK** , aby określić lokalizację folderu.</span><span class="sxs-lookup"><span data-stu-id="943db-236">Select **OK** to folder location.</span></span>

    ![Wybierz folder do wdrożenia Azure App Service w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Okno dialogowe selektora folderów 2](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="943db-239">W obszarze Karta zmienna Dodaj zmienną o nazwie `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , ustaw jej wartość na **true** i zakres na Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="943db-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Dodawanie zmiennej do wdrożenia aplikacji platformy Azure w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="943db-241">Wybierz ikonę wyzwalacza **ciągłego** wdrażania w obu artefaktach i Włącz wyzwalacz wdrażania **Kontynuuj** .</span><span class="sxs-lookup"><span data-stu-id="943db-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Wybierz wyzwalacz ciągłego wdrażania w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="943db-243">Wybierz ikonę warunki **przed wdrożeniem** w środowisku centrum Azure Stack i ustaw wyzwalacz na wartość **po wydaniu.**</span><span class="sxs-lookup"><span data-stu-id="943db-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Wybierz warunki przed wdrożeniem w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="943db-245">Zapisz wszystkie zmiany.</span><span class="sxs-lookup"><span data-stu-id="943db-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="943db-246">Niektóre ustawienia zadań mogą zostać automatycznie zdefiniowane jako [zmienne środowiskowe](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) podczas tworzenia definicji wydania na podstawie szablonu.</span><span class="sxs-lookup"><span data-stu-id="943db-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="943db-247">Tych ustawień nie można modyfikować w ustawieniach zadania; Zamiast tego należy wybrać element środowiska nadrzędnego, aby edytować te ustawienia.</span><span class="sxs-lookup"><span data-stu-id="943db-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="943db-248">Część 2: Aktualizowanie opcji aplikacji sieci Web</span><span class="sxs-lookup"><span data-stu-id="943db-248">Part 2: Update web app options</span></span>

<span data-ttu-id="943db-249">[Azure App Service](/azure/app-service/overview) zapewnia wysoce skalowalną, samoobsługową usługę hostingu w sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="943db-251">Zamapuj istniejącą niestandardową nazwę DNS na platformę Azure Web Apps.</span><span class="sxs-lookup"><span data-stu-id="943db-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="943db-252">Użyj **rekordu CNAME** i **rekordu a,** Aby zmapować niestandardową nazwę DNS na App Service.</span><span class="sxs-lookup"><span data-stu-id="943db-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="943db-253">Mapowanie istniejącej niestandardowej nazwy DNS na aplikacje internetowe platformy Azure</span><span class="sxs-lookup"><span data-stu-id="943db-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="943db-254">Użyj rekordu CNAME dla wszystkich niestandardowych nazw DNS, z wyjątkiem domeny głównej (na przykład northwind.com).</span><span class="sxs-lookup"><span data-stu-id="943db-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="943db-255">Aby przeprowadzić migrację aktywnej witryny oraz jej nazwy domeny DNS do usługi App Service, zobacz [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain) (Migrowanie aktywnej nazwy DNS do usługi Azure App Service).</span><span class="sxs-lookup"><span data-stu-id="943db-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="943db-256">Wymagania wstępne</span><span class="sxs-lookup"><span data-stu-id="943db-256">Prerequisites</span></span>

<span data-ttu-id="943db-257">Aby ukończyć to rozwiązanie:</span><span class="sxs-lookup"><span data-stu-id="943db-257">To complete this solution:</span></span>

- <span data-ttu-id="943db-258">[Utwórz aplikację App Service](/azure/app-service/)lub użyj aplikacji utworzonej dla innego rozwiązania.</span><span class="sxs-lookup"><span data-stu-id="943db-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="943db-259">Kup nazwę domeny i Zapewnij dostęp do rejestru DNS dostawcy domeny.</span><span class="sxs-lookup"><span data-stu-id="943db-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="943db-260">Zaktualizuj plik strefy DNS dla domeny.</span><span class="sxs-lookup"><span data-stu-id="943db-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="943db-261">Usługa Azure AD sprawdzi własność niestandardowej nazwy domeny.</span><span class="sxs-lookup"><span data-stu-id="943db-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="943db-262">Użyj [Azure DNS](/azure/dns/dns-getstarted-portal) dla systemu azure/Microsoft 365/zewnętrznych rekordów DNS na platformie Azure lub Dodaj wpis DNS w [innym rejestratorze DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span><span class="sxs-lookup"><span data-stu-id="943db-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="943db-263">Zarejestruj domenę niestandardową z rejestratorem publicznym.</span><span class="sxs-lookup"><span data-stu-id="943db-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="943db-264">Zaloguj się do rejestratora nazw domen dla domeny.</span><span class="sxs-lookup"><span data-stu-id="943db-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="943db-265">(Do aktualizacji DNS może być wymagana zatwierdzona administrator).</span><span class="sxs-lookup"><span data-stu-id="943db-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="943db-266">Zaktualizuj plik strefy DNS dla domeny przez dodanie wpisu DNS dostarczonego przez usługę Azure AD.</span><span class="sxs-lookup"><span data-stu-id="943db-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="943db-267">Aby na przykład dodać wpisy DNS dla northwindcloud.com i www \. northwindcloud.com, skonfiguruj ustawienia DNS dla domeny katalogu głównego northwindcloud.com.</span><span class="sxs-lookup"><span data-stu-id="943db-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="943db-268">Nazwę domeny można zakupić przy użyciu [Azure Portal](/azure/app-service/manage-custom-dns-buy-domain).</span><span class="sxs-lookup"><span data-stu-id="943db-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="943db-269">Aby zamapować niestandardową nazwę DNS na aplikację internetową, dla tej aplikacji internetowej musisz mieć płatną warstwę [planu usługi App Service](https://azure.microsoft.com/pricing/details/app-service/) (**Współdzielona**, **Podstawowa**, **Standardowa** lub **Premium**).</span><span class="sxs-lookup"><span data-stu-id="943db-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="943db-270">Tworzenie i mapowanie rekordów CNAME i A</span><span class="sxs-lookup"><span data-stu-id="943db-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="943db-271">Uzyskiwanie dostępu do rekordów DNS u dostawcy domen</span><span class="sxs-lookup"><span data-stu-id="943db-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="943db-272">Użyj Azure DNS, aby skonfigurować niestandardową nazwę DNS dla Web Apps platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="943db-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="943db-273">Aby uzyskać więcej informacji, zobacz [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain) (Korzystanie z usługi Azure DNS w celu udostępnienia niestandardowych ustawień domeny dla usługi platformy Azure).</span><span class="sxs-lookup"><span data-stu-id="943db-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="943db-274">Zaloguj się do witryny sieci Web głównego dostawcy.</span><span class="sxs-lookup"><span data-stu-id="943db-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="943db-275">Znajdź stronę służącą do zarządzania rekordami DNS.</span><span class="sxs-lookup"><span data-stu-id="943db-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="943db-276">Każdy dostawca domeny ma własny interfejs rekordów DNS.</span><span class="sxs-lookup"><span data-stu-id="943db-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="943db-277">Poszukaj obszarów witryny z etykietą **Nazwa domeny**, **DNS** lub **Zarządzanie serwerami nazw**.</span><span class="sxs-lookup"><span data-stu-id="943db-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="943db-278">Strona rekordów DNS może być wyświetlana w obszarze **Moje domeny**.</span><span class="sxs-lookup"><span data-stu-id="943db-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="943db-279">Znajdź łącze o nazwie **plik strefy**, **rekordy DNS** lub **Konfiguracja zaawansowana**.</span><span class="sxs-lookup"><span data-stu-id="943db-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="943db-280">Poniższy zrzut ekranu przedstawia przykład strony rekordów DNS:</span><span class="sxs-lookup"><span data-stu-id="943db-280">The following screenshot is an example of a DNS records page:</span></span>

![Przykładowa strona rekordów DNS](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="943db-282">W obszarze rejestrator nazw domen wybierz pozycję **Dodaj lub Utwórz** , aby utworzyć rekord.</span><span class="sxs-lookup"><span data-stu-id="943db-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="943db-283">Niektórzy dostawcy udostępniają różne linki na potrzeby dodawania różnych typów rekordów.</span><span class="sxs-lookup"><span data-stu-id="943db-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="943db-284">Zapoznaj się z dokumentacją dostawcy.</span><span class="sxs-lookup"><span data-stu-id="943db-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="943db-285">Dodaj rekord CNAME, aby zmapować poddomenę na domyślną nazwę hosta aplikacji.</span><span class="sxs-lookup"><span data-stu-id="943db-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="943db-286">W przypadku \. przykładowej domeny northwindcloud.com www Dodaj rekord CNAME, który mapuje nazwę na `<app_name>.azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="943db-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="943db-287">Po dodaniu rekordu CNAME Strona rekordów DNS wygląda podobnie do poniższego przykładu:</span><span class="sxs-lookup"><span data-stu-id="943db-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Nawigacja w portalu do aplikacji platformy Azure](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="943db-289">Włączanie mapowania rekordów CNAME na platformie Azure</span><span class="sxs-lookup"><span data-stu-id="943db-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="943db-290">Na nowej karcie Zaloguj się do Azure Portal.</span><span class="sxs-lookup"><span data-stu-id="943db-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="943db-291">Wybierz pozycję App Services.</span><span class="sxs-lookup"><span data-stu-id="943db-291">Go to App Services.</span></span>

3. <span data-ttu-id="943db-292">Wybierz pozycję aplikacja sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-292">Select web app.</span></span>

4. <span data-ttu-id="943db-293">W lewym obszarze nawigacji na stronie aplikacji w witrynie Azure Portal wybierz pozycję **Domeny niestandardowe**.</span><span class="sxs-lookup"><span data-stu-id="943db-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="943db-294">Wybierz **+** ikonę obok pozycji **Dodaj nazwę hosta**.</span><span class="sxs-lookup"><span data-stu-id="943db-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="943db-295">Wpisz w pełni kwalifikowaną nazwę domeny, na przykład `www.northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="943db-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="943db-296">Wybierz przycisk **Weryfikuj**.</span><span class="sxs-lookup"><span data-stu-id="943db-296">Select **Validate**.</span></span>

8. <span data-ttu-id="943db-297">Jeśli to wskazane, Dodaj dodatkowe rekordy innych typów ( `A` lub `TXT` ) do rekordów DNS rejestratorów nazw domen.</span><span class="sxs-lookup"><span data-stu-id="943db-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="943db-298">Platforma Azure dostarczy wartości i typy tych rekordów:</span><span class="sxs-lookup"><span data-stu-id="943db-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="943db-299">a.</span><span class="sxs-lookup"><span data-stu-id="943db-299">a.</span></span>  <span data-ttu-id="943db-300">Rekord **A** do zmapowania adresu IP aplikacji.</span><span class="sxs-lookup"><span data-stu-id="943db-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="943db-301">b.</span><span class="sxs-lookup"><span data-stu-id="943db-301">b.</span></span>  <span data-ttu-id="943db-302">Rekord **TXT** do zmapowania domyślnej nazwy hosta aplikacji `<app_name>.azurewebsites.net`.</span><span class="sxs-lookup"><span data-stu-id="943db-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="943db-303">App Service używa tego rekordu tylko w czasie konfiguracji do sprawdzenia własności domeny niestandardowej.</span><span class="sxs-lookup"><span data-stu-id="943db-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="943db-304">Po weryfikacji Usuń rekord TXT.</span><span class="sxs-lookup"><span data-stu-id="943db-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="943db-305">Wykonaj to zadanie na karcie rejestrator domen i ponownie sprawdź poprawność, dopóki nie zostanie aktywowany przycisk **Dodaj nazwę hosta** .</span><span class="sxs-lookup"><span data-stu-id="943db-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="943db-306">Upewnij się, że **Typ rekordu nazwy hosta** jest ustawiony na wartość **CNAME** (www.example.com lub dowolna poddomena).</span><span class="sxs-lookup"><span data-stu-id="943db-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="943db-307">Wybierz przycisk **Dodaj nazwę hosta**.</span><span class="sxs-lookup"><span data-stu-id="943db-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="943db-308">Wpisz w pełni kwalifikowaną nazwę domeny, na przykład `northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="943db-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="943db-309">Wybierz przycisk **Weryfikuj**.</span><span class="sxs-lookup"><span data-stu-id="943db-309">Select **Validate**.</span></span> <span data-ttu-id="943db-310">**Dodatek** jest aktywowany.</span><span class="sxs-lookup"><span data-stu-id="943db-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="943db-311">Upewnij się, że **Typ rekordu nazwy hosta** jest ustawiony na **rekord** (example.com).</span><span class="sxs-lookup"><span data-stu-id="943db-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="943db-312">**Dodaj nazwę hosta**.</span><span class="sxs-lookup"><span data-stu-id="943db-312">**Add hostname**.</span></span>

    <span data-ttu-id="943db-313">Może upłynąć trochę czasu, zanim nowe nazwy hostów zostaną odzwierciedlone na stronie **domeny niestandardowe** aplikacji.</span><span class="sxs-lookup"><span data-stu-id="943db-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="943db-314">Spróbuj odświeżyć przeglądarkę, aby zaktualizować dane.</span><span class="sxs-lookup"><span data-stu-id="943db-314">Try refreshing the browser to update the data.</span></span>
  
    ![Niestandardowe domeny](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="943db-316">Jeśli wystąpi błąd, w dolnej części strony pojawi się powiadomienie o błędzie weryfikacji.</span><span class="sxs-lookup"><span data-stu-id="943db-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Błąd weryfikacji domeny](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="943db-318">Powyższe kroki można powtórzyć, aby zmapować domenę symboli wieloznacznych ( \* . northwindcloud.com).</span><span class="sxs-lookup"><span data-stu-id="943db-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="943db-319">Pozwala to dodać do tej usługi aplikacji dodatkowe domeny podrzędne bez konieczności tworzenia oddzielnego rekordu CNAME dla każdej z nich.</span><span class="sxs-lookup"><span data-stu-id="943db-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="943db-320">Postępuj zgodnie z instrukcjami rejestratora, aby skonfigurować to ustawienie.</span><span class="sxs-lookup"><span data-stu-id="943db-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="943db-321">Testowanie w przeglądarce</span><span class="sxs-lookup"><span data-stu-id="943db-321">Test in a browser</span></span>

<span data-ttu-id="943db-322">Przejdź do nazw DNS skonfigurowanych wcześniej (na przykład `northwindcloud.com` lub `www.northwindcloud.com` ).</span><span class="sxs-lookup"><span data-stu-id="943db-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="943db-323">Część 3: wiązanie niestandardowego certyfikatu SSL</span><span class="sxs-lookup"><span data-stu-id="943db-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="943db-324">W tej części będziemy:</span><span class="sxs-lookup"><span data-stu-id="943db-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="943db-325">Powiąż niestandardowy certyfikat SSL z App Service.</span><span class="sxs-lookup"><span data-stu-id="943db-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="943db-326">Wymuszanie protokołu HTTPS dla aplikacji.</span><span class="sxs-lookup"><span data-stu-id="943db-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="943db-327">Automatyzacja powiązania certyfikatu SSL ze skryptami.</span><span class="sxs-lookup"><span data-stu-id="943db-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="943db-328">W razie potrzeby uzyskaj certyfikat SSL klienta w Azure Portal i powiąż go z aplikacją sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="943db-329">Aby uzyskać więcej informacji, zobacz [samouczek App Service Certificates](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="943db-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="943db-330">Wymagania wstępne</span><span class="sxs-lookup"><span data-stu-id="943db-330">Prerequisites</span></span>

<span data-ttu-id="943db-331">Aby ukończyć to rozwiązanie:</span><span class="sxs-lookup"><span data-stu-id="943db-331">To complete this  solution:</span></span>

- [<span data-ttu-id="943db-332">Utwórz aplikację App Service.</span><span class="sxs-lookup"><span data-stu-id="943db-332">Create an App Service app.</span></span>](/azure/app-service/)
- [<span data-ttu-id="943db-333">Zamapuj niestandardową nazwę DNS na aplikację sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="943db-334">Uzyskaj certyfikat SSL z zaufanego urzędu certyfikacji i Użyj klucza do podpisania żądania.</span><span class="sxs-lookup"><span data-stu-id="943db-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="943db-335">Wymagania dotyczące certyfikatu protokołu SSL</span><span class="sxs-lookup"><span data-stu-id="943db-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="943db-336">Aby używać certyfikatu w usłudze App Service, musi on spełniać wszystkie następujące wymagania:</span><span class="sxs-lookup"><span data-stu-id="943db-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="943db-337">Podpisany przez zaufany urząd certyfikacji.</span><span class="sxs-lookup"><span data-stu-id="943db-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="943db-338">Eksportowany jako chroniony hasłem plik PFX.</span><span class="sxs-lookup"><span data-stu-id="943db-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="943db-339">Zawiera klucz prywatny o długości co najmniej 2048 bitów.</span><span class="sxs-lookup"><span data-stu-id="943db-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="943db-340">Zawiera wszystkie certyfikaty pośrednie w łańcuchu certyfikatów.</span><span class="sxs-lookup"><span data-stu-id="943db-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="943db-341">**Certyfikaty kryptografii krzywej eliptycznej (ECC)** działają z App Service, ale nie są uwzględnione w tym przewodniku.</span><span class="sxs-lookup"><span data-stu-id="943db-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="943db-342">Zapoznaj się z urzędem certyfikacji, aby uzyskać pomoc w tworzeniu certyfikatów ECC.</span><span class="sxs-lookup"><span data-stu-id="943db-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="943db-343">Przygotowywanie aplikacji sieci Web</span><span class="sxs-lookup"><span data-stu-id="943db-343">Prepare the web app</span></span>

<span data-ttu-id="943db-344">Aby powiązać niestandardowy certyfikat SSL z aplikacją internetową, [plan App Service](https://azure.microsoft.com/pricing/details/app-service/) musi znajdować się w warstwie **podstawowa**, **standardowa** lub **Premium** .</span><span class="sxs-lookup"><span data-stu-id="943db-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="943db-345">Logowanie do platformy Azure</span><span class="sxs-lookup"><span data-stu-id="943db-345">Sign in to Azure</span></span>

1. <span data-ttu-id="943db-346">Otwórz [Azure Portal](https://portal.azure.com/) i przejdź do aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="943db-347">Z menu po lewej stronie wybierz pozycję **App Services**, a następnie wybierz nazwę aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Wybierz aplikację sieci Web w Azure Portal](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="943db-349">Sprawdzanie warstwy cenowej</span><span class="sxs-lookup"><span data-stu-id="943db-349">Check the pricing tier</span></span>

1. <span data-ttu-id="943db-350">W okienku nawigacji po lewej stronie aplikacji sieci Web przewiń do sekcji **Ustawienia** i wybierz pozycję **Skaluj w górę (plan App Service)**.</span><span class="sxs-lookup"><span data-stu-id="943db-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Menu skalowania w aplikacji sieci Web](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="943db-352">Upewnij się, że aplikacja sieci Web nie znajduje się w warstwie **bezpłatna** lub **współdzielona** .</span><span class="sxs-lookup"><span data-stu-id="943db-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="943db-353">Bieżąca warstwa aplikacji sieci Web zostanie wyróżniona w ciemnoniebieskim polu.</span><span class="sxs-lookup"><span data-stu-id="943db-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![Sprawdzanie warstwy cenowej w aplikacji sieci Web](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="943db-355">Niestandardowy protokół SSL nie jest obsługiwany w warstwie **bezpłatna** ani **współdzielona** .</span><span class="sxs-lookup"><span data-stu-id="943db-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="943db-356">Aby przeprowadzić skalowanie, wykonaj kroki opisane w następnej sekcji lub stronie **Wybierz warstwę cenową** , a następnie przejdź do [przekazywania i powiązania certyfikatu SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="943db-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="943db-357">Skalowanie w górę planu usługi App Service</span><span class="sxs-lookup"><span data-stu-id="943db-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="943db-358">Wybierz jedną z następujących warstw: **Podstawowa**, **Standardowa** lub **Premium**.</span><span class="sxs-lookup"><span data-stu-id="943db-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="943db-359">Wybierz pozycję **Wybierz**.</span><span class="sxs-lookup"><span data-stu-id="943db-359">Select **Select**.</span></span>

![Wybierz warstwę cenową dla aplikacji sieci Web](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="943db-361">Operacja skalowania zostanie zakończona, gdy zostanie wyświetlone powiadomienie.</span><span class="sxs-lookup"><span data-stu-id="943db-361">The scale operation is complete when notification is displayed.</span></span>

![Powiadomienie o skalowaniu w górę](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="943db-363">Powiązywanie certyfikatu SSL i scalanie certyfikatów pośrednich</span><span class="sxs-lookup"><span data-stu-id="943db-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="943db-364">Scal wiele certyfikatów w łańcuchu.</span><span class="sxs-lookup"><span data-stu-id="943db-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="943db-365">**Otwórz każdy certyfikat** otrzymany w edytorze tekstu.</span><span class="sxs-lookup"><span data-stu-id="943db-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="943db-366">Utwórz plik dla scalonego certyfikatu o nazwie *mergedcertificate. CRT*.</span><span class="sxs-lookup"><span data-stu-id="943db-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="943db-367">W edytorze tekstów skopiuj zawartość każdego certyfikatu do tego pliku.</span><span class="sxs-lookup"><span data-stu-id="943db-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="943db-368">Kolejność certyfikatów powinna być zgodna z kolejnością w łańcuchu certyfikatów, poczynając od Twojego certyfikatu i kończąc na certyfikacie głównym.</span><span class="sxs-lookup"><span data-stu-id="943db-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="943db-369">Wygląda to następująco:</span><span class="sxs-lookup"><span data-stu-id="943db-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="943db-370">Eksportowanie certyfikatu do pliku PFX</span><span class="sxs-lookup"><span data-stu-id="943db-370">Export certificate to PFX</span></span>

<span data-ttu-id="943db-371">Wyeksportuj scalony certyfikat SSL przy użyciu klucza prywatnego wygenerowanego przez certyfikat.</span><span class="sxs-lookup"><span data-stu-id="943db-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="943db-372">Plik klucza prywatnego jest tworzony za pośrednictwem OpenSSL.</span><span class="sxs-lookup"><span data-stu-id="943db-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="943db-373">Aby wyeksportować certyfikat do pliku PFX, uruchom następujące polecenie i Zastąp symbole zastępcze `<private-key-file>` oraz `<merged-certificate-file>` ścieżkę klucza prywatnego i scalony plik certyfikatu:</span><span class="sxs-lookup"><span data-stu-id="943db-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="943db-374">Po wyświetleniu monitu Zdefiniuj hasło eksportu służące do przekazywania certyfikatu SSL w celu App Service późniejszej.</span><span class="sxs-lookup"><span data-stu-id="943db-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="943db-375">Gdy usługi IIS lub **Certreq.exe** są używane do generowania żądania certyfikatu, należy zainstalować certyfikat na komputerze lokalnym, a następnie [wyeksportować certyfikat do pliku PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="943db-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="943db-376">Przekaż certyfikat SSL</span><span class="sxs-lookup"><span data-stu-id="943db-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="943db-377">Wybierz pozycję **Ustawienia protokołu SSL** w lewym obszarze nawigacji aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="943db-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="943db-378">Wybierz pozycję **Przekaż certyfikat**.</span><span class="sxs-lookup"><span data-stu-id="943db-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="943db-379">W **pliku certyfikatu PFX** wybierz pozycję plik PFX.</span><span class="sxs-lookup"><span data-stu-id="943db-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="943db-380">W polu **hasło certyfikatu** wpisz hasło utworzone podczas EKSPORTOWANIA pliku PFX.</span><span class="sxs-lookup"><span data-stu-id="943db-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="943db-381">Wybierz pozycję **Przekaż**.</span><span class="sxs-lookup"><span data-stu-id="943db-381">Select **Upload**.</span></span>

    ![Przekaż certyfikat SSL](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="943db-383">Po zakończeniu przekazywania certyfikatu App Service zostanie on wyświetlony na stronie **Ustawienia protokołu SSL** .</span><span class="sxs-lookup"><span data-stu-id="943db-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![Ustawienia protokołu SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="943db-385">Wiązanie certyfikatu protokołu SSL</span><span class="sxs-lookup"><span data-stu-id="943db-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="943db-386">W sekcji **powiązania SSL** wybierz pozycję **Dodaj powiązanie**.</span><span class="sxs-lookup"><span data-stu-id="943db-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="943db-387">Jeśli certyfikat został przekazany, ale nie pojawia się na liście nazwy domen na liście rozwijanej **Nazwa hosta** , spróbuj odświeżyć stronę przeglądarki.</span><span class="sxs-lookup"><span data-stu-id="943db-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="943db-388">Na stronie **Dodawanie powiązania SSL** Użyj listy rozwijanej, aby wybrać nazwę domeny do zabezpieczenia i certyfikat do użycia.</span><span class="sxs-lookup"><span data-stu-id="943db-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="943db-389">W obszarze **Typ protokołu SSL** wybierz, czy ma być używane [**Oznaczanie nazwy serwera (SNI, Server Name Indication)**](https://en.wikipedia.org/wiki/Server_Name_Indication), czy też protokół SSL oparty na protokole IP.</span><span class="sxs-lookup"><span data-stu-id="943db-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="943db-390">**Protokół SSL oparty na SNI**: można dodać wiele powiązań SSL opartych na SNI.</span><span class="sxs-lookup"><span data-stu-id="943db-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="943db-391">Ta opcja umożliwia zabezpieczenie wielu domen na tym samym adresie IP za pomocą wielu certyfikatów protokołu SSL.</span><span class="sxs-lookup"><span data-stu-id="943db-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="943db-392">Większość nowoczesnych przeglądarek (w tym programy Internet Explorer, Chrome, Firefox i Opera) obsługuje funkcję SNI. Bardziej szczegółowe informacje dotyczące obsługi przeglądarek możesz znaleźć w artykule [Server Name Indication (Oznaczanie nazwy serwera)](https://wikipedia.org/wiki/Server_Name_Indication).</span><span class="sxs-lookup"><span data-stu-id="943db-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="943db-393">**Protokół SSL oparty na** protokole IP: można dodać tylko jedno powiązanie SSL oparte na adresie IP.</span><span class="sxs-lookup"><span data-stu-id="943db-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="943db-394">Ta opcja umożliwia zabezpieczenie dedykowanego publicznego adresu IP za pomocą tylko jednego certyfikatu protokołu SSL.</span><span class="sxs-lookup"><span data-stu-id="943db-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="943db-395">Aby zabezpieczyć wiele domen, zabezpiecz je wszystkie przy użyciu tego samego certyfikatu SSL.</span><span class="sxs-lookup"><span data-stu-id="943db-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="943db-396">Protokół SSL oparty na protokole IP jest tradycyjną opcją dla powiązania SSL.</span><span class="sxs-lookup"><span data-stu-id="943db-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="943db-397">Wybierz pozycję **Dodaj powiązanie**.</span><span class="sxs-lookup"><span data-stu-id="943db-397">Select **Add Binding**.</span></span>

    ![Dodawanie powiązania SSL](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="943db-399">Po zakończeniu przekazywania certyfikatu App Service zostanie on wyświetlony w sekcjach **powiązania protokołu SSL** .</span><span class="sxs-lookup"><span data-stu-id="943db-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Zakończono przekazywanie powiązań SSL](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="943db-401">Ponowne mapowanie rekordu A dla Połączenie SSL z adresu IP</span><span class="sxs-lookup"><span data-stu-id="943db-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="943db-402">Jeśli w aplikacji sieci Web nie jest używany protokół SSL oparty na protokole IP, przejdź do [testowania protokołu HTTPS dla domeny niestandardowej](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="943db-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="943db-403">Domyślnie aplikacja sieci Web używa udostępnionego publicznego adresu IP.</span><span class="sxs-lookup"><span data-stu-id="943db-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="943db-404">Jeśli certyfikat jest powiązany z protokołem SSL opartym na protokole IP, App Service tworzy nowy i dedykowany adres IP dla aplikacji internetowej.</span><span class="sxs-lookup"><span data-stu-id="943db-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="943db-405">W przypadku zmapowania rekordu do aplikacji sieci Web należy zaktualizować rejestr domeny za pomocą dedykowanego adresu IP.</span><span class="sxs-lookup"><span data-stu-id="943db-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="943db-406">Strona **domena niestandardowa** jest aktualizowana przy użyciu nowego, dedykowanego adresu IP.</span><span class="sxs-lookup"><span data-stu-id="943db-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="943db-407">Skopiuj ten [adres IP](/azure/app-service/app-service-web-tutorial-custom-domain), a następnie ponownie zamapuj [rekord A](/azure/app-service/app-service-web-tutorial-custom-domain) na ten nowy adres IP.</span><span class="sxs-lookup"><span data-stu-id="943db-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="943db-408">Testowanie protokołu HTTPS</span><span class="sxs-lookup"><span data-stu-id="943db-408">Test HTTPS</span></span>

<span data-ttu-id="943db-409">W różnych przeglądarkach przejdź do, `https://<your.custom.domain>` Aby upewnić się, że aplikacja sieci Web jest obsługiwana.</span><span class="sxs-lookup"><span data-stu-id="943db-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![Przejdź do aplikacji sieci Web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="943db-411">Jeśli wystąpią błędy walidacji certyfikatu, certyfikat z podpisem własnym może być przyczyną lub certyfikaty pośrednie mogły zostać wyłączone podczas eksportowania do pliku PFX.</span><span class="sxs-lookup"><span data-stu-id="943db-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="943db-412">Wymuszanie protokołu HTTPS</span><span class="sxs-lookup"><span data-stu-id="943db-412">Enforce HTTPS</span></span>

<span data-ttu-id="943db-413">Domyślnie każdy użytkownik może uzyskać dostęp do aplikacji sieci Web przy użyciu protokołu HTTP.</span><span class="sxs-lookup"><span data-stu-id="943db-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="943db-414">Można przekierować wszystkie żądania HTTP do portu HTTPS.</span><span class="sxs-lookup"><span data-stu-id="943db-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="943db-415">Na stronie aplikacja sieci Web wybierz pozycję **Ustawienia SL**.</span><span class="sxs-lookup"><span data-stu-id="943db-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="943db-416">Następnie w pozycji **Tylko HTTPS** wybierz opcję **Włączone**.</span><span class="sxs-lookup"><span data-stu-id="943db-416">Then, in **HTTPS Only**, select **On**.</span></span>

![Wymuszanie protokołu HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="943db-418">Po zakończeniu operacji przejdź do dowolnego adresu URL protokołu HTTP, który wskazuje aplikację.</span><span class="sxs-lookup"><span data-stu-id="943db-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="943db-419">Na przykład:</span><span class="sxs-lookup"><span data-stu-id="943db-419">For example:</span></span>

- <span data-ttu-id="943db-420"> https://<app_name>. azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="943db-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="943db-421">Wymuszanie protokołu TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="943db-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="943db-422">Aplikacja domyślnie zezwala na [protokół TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0, który nie jest już traktowany jako bezpieczny przez standardy branżowe (takie jak [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span><span class="sxs-lookup"><span data-stu-id="943db-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="943db-423">Aby wymusić nowsze wersje protokołu TLS, wykonaj następujące kroki:</span><span class="sxs-lookup"><span data-stu-id="943db-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="943db-424">Na stronie aplikacja sieci Web w lewym okienku nawigacji wybierz pozycję **Ustawienia protokołu SSL**.</span><span class="sxs-lookup"><span data-stu-id="943db-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="943db-425">W polu **wersja protokołu TLS** wybierz pozycję minimalna wersja protokołu TLS.</span><span class="sxs-lookup"><span data-stu-id="943db-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![Wymuszanie protokołu TLS 1.1 lub 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="943db-427">Tworzenie profilu usługi Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="943db-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="943db-428">Wybierz kolejno pozycje **Utwórz zasób**  >  **Sieć**  >  **Traffic Manager**  >  **Utwórz** profil.</span><span class="sxs-lookup"><span data-stu-id="943db-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="943db-429">W obszarze **Tworzenie profilu usługi Traffic Manager** podaj następujące informacje:</span><span class="sxs-lookup"><span data-stu-id="943db-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="943db-430">W polu **Nazwa** Podaj nazwę profilu.</span><span class="sxs-lookup"><span data-stu-id="943db-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="943db-431">Ta nazwa musi być unikatowa w obrębie strefy manager.net ruchu i powoduje, że w polu Nazwa DNS trafficmanager.net, która jest używana do uzyskiwania dostępu do profilu Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="943db-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="943db-432">W obszarze **Metoda routingu** wybierz **metodę routingu geograficznego**.</span><span class="sxs-lookup"><span data-stu-id="943db-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="943db-433">W obszarze **subskrypcja** wybierz subskrypcję, w ramach której chcesz utworzyć ten profil.</span><span class="sxs-lookup"><span data-stu-id="943db-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="943db-434">W obszarze **Grupy zasobów** utwórz nową grupę zasobów, w której zostanie umieszczony ten profil.</span><span class="sxs-lookup"><span data-stu-id="943db-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="943db-435">W obszarze **Lokalizacja grupy zasobów** wybierz lokalizację grupy zasobów.</span><span class="sxs-lookup"><span data-stu-id="943db-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="943db-436">To ustawienie dotyczy lokalizacji grupy zasobów i nie ma wpływu na profil Traffic Manager wdrożony globalnie.</span><span class="sxs-lookup"><span data-stu-id="943db-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="943db-437">Wybierz przycisk **Utwórz**.</span><span class="sxs-lookup"><span data-stu-id="943db-437">Select **Create**.</span></span>

    7. <span data-ttu-id="943db-438">Po ukończeniu globalnego wdrożenia profilu Traffic Manager jest on wyświetlany w odpowiedniej grupie zasobów jako jeden z zasobów.</span><span class="sxs-lookup"><span data-stu-id="943db-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Grupy zasobów w profilu tworzenia Traffic Manager](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="943db-440">Dodawanie punktów końcowych usługi Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="943db-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="943db-441">Na pasku wyszukiwania portalu Wyszukaj nazwę **profilu Traffic Manager** utworzoną w poprzedniej sekcji, a następnie wybierz profil usługi Traffic Manager w wyświetlonych wynikach.</span><span class="sxs-lookup"><span data-stu-id="943db-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="943db-442">W obszarze **profil Traffic Manager** w sekcji **Ustawienia** wybierz pozycję **punkty końcowe**.</span><span class="sxs-lookup"><span data-stu-id="943db-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="943db-443">Wybierz pozycję **Dodaj**.</span><span class="sxs-lookup"><span data-stu-id="943db-443">Select **Add**.</span></span>

4. <span data-ttu-id="943db-444">Dodawanie punktu końcowego Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="943db-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="943db-445">W obszarze **Typ** wybierz pozycję **zewnętrzny punkt końcowy**.</span><span class="sxs-lookup"><span data-stu-id="943db-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="943db-446">Podaj **nazwę** dla tego punktu końcowego, najlepiej nazwę centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="943db-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="943db-447">Dla w pełni kwalifikowanej nazwy domeny (**FQDN**) Użyj zewnętrznego adresu URL dla aplikacji sieci Web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="943db-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="943db-448">W obszarze mapowanie geograficzne wybierz region/kontynent, w którym znajduje się zasób.</span><span class="sxs-lookup"><span data-stu-id="943db-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="943db-449">Na przykład **Europa.**</span><span class="sxs-lookup"><span data-stu-id="943db-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="943db-450">W wyświetlonym menu rozwijanym Kraj/region wybierz kraj, który ma zastosowanie do tego punktu końcowego.</span><span class="sxs-lookup"><span data-stu-id="943db-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="943db-451">Na przykład **Niemcy**.</span><span class="sxs-lookup"><span data-stu-id="943db-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="943db-452">Pozycję **Dodaj jako wyłączone** pozostaw niezaznaczoną.</span><span class="sxs-lookup"><span data-stu-id="943db-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="943db-453">Wybierz przycisk **OK**.</span><span class="sxs-lookup"><span data-stu-id="943db-453">Select **OK**.</span></span>

12. <span data-ttu-id="943db-454">Dodawanie punkt końcowy platformy Azure:</span><span class="sxs-lookup"><span data-stu-id="943db-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="943db-455">W obszarze **Typ** wybierz pozycję **punkt końcowy platformy Azure**.</span><span class="sxs-lookup"><span data-stu-id="943db-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="943db-456">Podaj **nazwę** punktu końcowego.</span><span class="sxs-lookup"><span data-stu-id="943db-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="943db-457">W obszarze **Typ zasobu docelowego** wybierz pozycję **App Service**.</span><span class="sxs-lookup"><span data-stu-id="943db-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="943db-458">W polu **zasób docelowy** wybierz pozycję **Wybierz usługę App Service** , aby wyświetlić listę Web Apps w ramach tej samej subskrypcji.</span><span class="sxs-lookup"><span data-stu-id="943db-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="943db-459">W obszarze **zasób** wybierz usługę App Service używaną jako pierwszy punkt końcowy.</span><span class="sxs-lookup"><span data-stu-id="943db-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="943db-460">W obszarze mapowanie geograficzne wybierz region/kontynent, w którym znajduje się zasób.</span><span class="sxs-lookup"><span data-stu-id="943db-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="943db-461">Na przykład **Ameryka Północna/środkowe Ameryki/Karaiby.**</span><span class="sxs-lookup"><span data-stu-id="943db-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="943db-462">W wyświetlonym menu rozwijanym Kraj/region, pozostaw to pole puste, aby wybrać wszystkie powyższe grupowanie regionalne.</span><span class="sxs-lookup"><span data-stu-id="943db-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="943db-463">Pozycję **Dodaj jako wyłączone** pozostaw niezaznaczoną.</span><span class="sxs-lookup"><span data-stu-id="943db-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="943db-464">Wybierz przycisk **OK**.</span><span class="sxs-lookup"><span data-stu-id="943db-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="943db-465">Utwórz co najmniej jeden punkt końcowy z zakresem geograficznym (World), który będzie używany jako domyślny punkt końcowy dla zasobu.</span><span class="sxs-lookup"><span data-stu-id="943db-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="943db-466">Po zakończeniu dodawania obu punktów końcowych są one wyświetlane w **profilu Traffic Manager** wraz z ich stanem monitorowania w **trybie online**.</span><span class="sxs-lookup"><span data-stu-id="943db-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Stan punktu końcowego profilu Traffic Manager](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="943db-468">Globalny Enterprise opiera się na możliwościach dystrybucji geograficznej platformy Azure</span><span class="sxs-lookup"><span data-stu-id="943db-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="943db-469">Kierowanie ruchu danych za pośrednictwem platformy Azure Traffic Manager i punkty końcowe specyficzne dla lokalizacji geograficznej umożliwiają globalnym przedsiębiorstwom przestrzeganie przepisów regionalnych i utrzymywanie zgodności z danymi, co jest niezwykle ważne dla sukcesu lokalnych i zdalnych lokalizacji roboczych.</span><span class="sxs-lookup"><span data-stu-id="943db-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="943db-470">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="943db-470">Next steps</span></span>

- <span data-ttu-id="943db-471">Aby dowiedzieć się więcej o wzorcach chmury platformy Azure, zobacz [wzorce projektowe w chmurze](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="943db-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
