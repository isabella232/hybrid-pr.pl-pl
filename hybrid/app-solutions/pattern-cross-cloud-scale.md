---
title: Wzorzec skalowania między chmurami w centrum Azure Stack
description: Dowiedz się, jak utworzyć skalowalną aplikację międzychmurową na platformie Azure i w centrum Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911135"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="a933a-103">Wzorzec skalowania między chmurami</span><span class="sxs-lookup"><span data-stu-id="a933a-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="a933a-104">Automatycznie Dodaj zasoby do istniejącej aplikacji, aby zwiększyć obciążenie.</span><span class="sxs-lookup"><span data-stu-id="a933a-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="a933a-105">Kontekst i problem</span><span class="sxs-lookup"><span data-stu-id="a933a-105">Context and problem</span></span>

<span data-ttu-id="a933a-106">Twoja aplikacja nie może zwiększyć wydajności, aby sprostać nieoczekiwanym wzrostom popytu.</span><span class="sxs-lookup"><span data-stu-id="a933a-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="a933a-107">Ten brak skalowalności powoduje, że użytkownicy nie docierają do aplikacji w czasie szczytowego użycia.</span><span class="sxs-lookup"><span data-stu-id="a933a-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="a933a-108">Aplikacja może obsłużyć określoną liczbę użytkowników.</span><span class="sxs-lookup"><span data-stu-id="a933a-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="a933a-109">Przedsiębiorstwa globalne wymagają bezpiecznych, niezawodnych i dostępnych aplikacji opartych na chmurze.</span><span class="sxs-lookup"><span data-stu-id="a933a-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="a933a-110">Zwiększenie popytu i użycie odpowiedniej infrastruktury do obsługi tego żądania ma kluczowe znaczenie.</span><span class="sxs-lookup"><span data-stu-id="a933a-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="a933a-111">Firmy nie mogą zrównoważyć kosztów i konserwacji dzięki zabezpieczeniom danych biznesowych, magazynowaniu i dostępności w czasie rzeczywistym.</span><span class="sxs-lookup"><span data-stu-id="a933a-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="a933a-112">Uruchomienie aplikacji w chmurze publicznej może nie być możliwe.</span><span class="sxs-lookup"><span data-stu-id="a933a-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="a933a-113">Niemniej jednak firma może nie być ekonomicznie wykonalna w celu utrzymania pojemności wymaganej w środowisku lokalnym w celu obsłużenia podaży na żądanie dla aplikacji.</span><span class="sxs-lookup"><span data-stu-id="a933a-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="a933a-114">Korzystając z tego wzorca, można wykorzystać elastyczność chmury publicznej z rozwiązaniem lokalnym.</span><span class="sxs-lookup"><span data-stu-id="a933a-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="a933a-115">Rozwiązanie</span><span class="sxs-lookup"><span data-stu-id="a933a-115">Solution</span></span>

<span data-ttu-id="a933a-116">Wzorzec skalowania między chmurami rozszerza aplikację znajdującą się w chmurze lokalnej z zasobami chmury publicznej.</span><span class="sxs-lookup"><span data-stu-id="a933a-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="a933a-117">Wzorzec jest wyzwalany przez zwiększenie lub zmniejszenie zapotrzebowania, a odpowiednio dodaje lub usuwa zasoby w chmurze.</span><span class="sxs-lookup"><span data-stu-id="a933a-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="a933a-118">Te zasoby zapewniają nadmiarowość, szybką dostępność i Routing zgodny ze geograficzną.</span><span class="sxs-lookup"><span data-stu-id="a933a-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Wzorzec skalowania między chmurami](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="a933a-120">Ten wzorzec dotyczy tylko bezstanowych składników aplikacji.</span><span class="sxs-lookup"><span data-stu-id="a933a-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="a933a-121">Składniki</span><span class="sxs-lookup"><span data-stu-id="a933a-121">Components</span></span>

<span data-ttu-id="a933a-122">Wzorzec skalowania między chmurami składa się z następujących składników.</span><span class="sxs-lookup"><span data-stu-id="a933a-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="a933a-123">Poza chmurą</span><span class="sxs-lookup"><span data-stu-id="a933a-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="a933a-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="a933a-124">Traffic Manager</span></span>

<span data-ttu-id="a933a-125">Na diagramie znajduje się poza grupą chmury publicznej, ale musi być w stanie koordynować ruch zarówno w lokalnym centrum danych, jak i w chmurze publicznej.</span><span class="sxs-lookup"><span data-stu-id="a933a-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="a933a-126">Moduł równoważenia obciążenia zapewnia wysoką dostępność aplikacji przez monitorowanie punktów końcowych i udostępnianie ponownej dystrybucji trybu failover, jeśli jest to wymagane.</span><span class="sxs-lookup"><span data-stu-id="a933a-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="a933a-127">System nazw domen (DNS)</span><span class="sxs-lookup"><span data-stu-id="a933a-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="a933a-128">System nazw domen (DNS) jest odpowiedzialny za tłumaczenie (lub rozwiązanie) nazwy witryny sieci Web lub usługi na adres IP.</span><span class="sxs-lookup"><span data-stu-id="a933a-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="a933a-129">Chmura</span><span class="sxs-lookup"><span data-stu-id="a933a-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="a933a-130">Hostowany serwer kompilacji</span><span class="sxs-lookup"><span data-stu-id="a933a-130">Hosted build server</span></span>

<span data-ttu-id="a933a-131">Środowisko do hostowania potoku kompilacji.</span><span class="sxs-lookup"><span data-stu-id="a933a-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="a933a-132">Zasoby aplikacji</span><span class="sxs-lookup"><span data-stu-id="a933a-132">App resources</span></span>

<span data-ttu-id="a933a-133">Zasoby aplikacji muszą mieć możliwość skalowania w poziomie i skalowalności, tak jak w przypadku zestawów skalowania maszyn wirtualnych i kontenerów.</span><span class="sxs-lookup"><span data-stu-id="a933a-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="a933a-134">Niestandardowa nazwa domeny</span><span class="sxs-lookup"><span data-stu-id="a933a-134">Custom domain name</span></span>

<span data-ttu-id="a933a-135">Użyj niestandardowej nazwy domeny dla żądań routingu globalizowania.</span><span class="sxs-lookup"><span data-stu-id="a933a-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="a933a-136">Publiczne adresy IP</span><span class="sxs-lookup"><span data-stu-id="a933a-136">Public IP addresses</span></span>

<span data-ttu-id="a933a-137">Publiczne adresy IP są używane do kierowania ruchu przychodzącego za pomocą usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.</span><span class="sxs-lookup"><span data-stu-id="a933a-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="a933a-138">Chmura lokalna</span><span class="sxs-lookup"><span data-stu-id="a933a-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="a933a-139">Hostowany serwer kompilacji</span><span class="sxs-lookup"><span data-stu-id="a933a-139">Hosted build server</span></span>

<span data-ttu-id="a933a-140">Środowisko do hostowania potoku kompilacji.</span><span class="sxs-lookup"><span data-stu-id="a933a-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="a933a-141">Zasoby aplikacji</span><span class="sxs-lookup"><span data-stu-id="a933a-141">App resources</span></span>

<span data-ttu-id="a933a-142">Zasoby aplikacji muszą mieć możliwość skalowania w poziomie i skalowania, takich jak zestawy skalowania maszyn wirtualnych i kontenery.</span><span class="sxs-lookup"><span data-stu-id="a933a-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="a933a-143">Niestandardowa nazwa domeny</span><span class="sxs-lookup"><span data-stu-id="a933a-143">Custom domain name</span></span>

<span data-ttu-id="a933a-144">Użyj niestandardowej nazwy domeny dla żądań routingu globalizowania.</span><span class="sxs-lookup"><span data-stu-id="a933a-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="a933a-145">Publiczne adresy IP</span><span class="sxs-lookup"><span data-stu-id="a933a-145">Public IP addresses</span></span>

<span data-ttu-id="a933a-146">Publiczne adresy IP są używane do kierowania ruchu przychodzącego za pomocą usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.</span><span class="sxs-lookup"><span data-stu-id="a933a-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="a933a-147">Problemy i kwestie do rozważenia</span><span class="sxs-lookup"><span data-stu-id="a933a-147">Issues and considerations</span></span>

<span data-ttu-id="a933a-148">Podczas podejmowania decyzji o sposobie wdrożenia tego wzorca należy rozważyć następujące punkty:</span><span class="sxs-lookup"><span data-stu-id="a933a-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="a933a-149">Skalowalność</span><span class="sxs-lookup"><span data-stu-id="a933a-149">Scalability</span></span>

<span data-ttu-id="a933a-150">Kluczowym elementem skalowania między chmurami jest możliwość dostarczania skalowania na żądanie.</span><span class="sxs-lookup"><span data-stu-id="a933a-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="a933a-151">Skalowanie musi nastąpić między chmurą publiczną a lokalną i zapewnić spójną, niezawodną usługę na żądanie.</span><span class="sxs-lookup"><span data-stu-id="a933a-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="a933a-152">Dostępność</span><span class="sxs-lookup"><span data-stu-id="a933a-152">Availability</span></span>

<span data-ttu-id="a933a-153">Upewnij się, że aplikacje wdrożone lokalnie są skonfigurowane pod kątem wysokiej dostępności za poorednictwem konfiguracji sprzętu lokalnego i wdrożenia oprogramowania.</span><span class="sxs-lookup"><span data-stu-id="a933a-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="a933a-154">Możliwości zarządzania</span><span class="sxs-lookup"><span data-stu-id="a933a-154">Manageability</span></span>

<span data-ttu-id="a933a-155">Wzorzec między chmurami zapewnia bezproblemowe zarządzanie i przyjazny interfejs między środowiskami.</span><span class="sxs-lookup"><span data-stu-id="a933a-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="a933a-156">Kiedy używać tego wzorca</span><span class="sxs-lookup"><span data-stu-id="a933a-156">When to use this pattern</span></span>

<span data-ttu-id="a933a-157">Użyj tego wzorca, aby:</span><span class="sxs-lookup"><span data-stu-id="a933a-157">Use this pattern:</span></span>

- <span data-ttu-id="a933a-158">Gdy trzeba zwiększyć pojemność aplikacji z nieoczekiwanymi wymaganiami lub okresowe zapotrzebowanie na żądanie.</span><span class="sxs-lookup"><span data-stu-id="a933a-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="a933a-159">Gdy nie chcesz inwestować w zasoby, które będą używane tylko podczas szczytów.</span><span class="sxs-lookup"><span data-stu-id="a933a-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="a933a-160">Płacisz za to, czego używasz.</span><span class="sxs-lookup"><span data-stu-id="a933a-160">Pay for what you use.</span></span>

<span data-ttu-id="a933a-161">Ten wzorzec nie jest zalecany w przypadku:</span><span class="sxs-lookup"><span data-stu-id="a933a-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="a933a-162">Twoje rozwiązanie wymaga od użytkowników nawiązywania połączenia przez Internet.</span><span class="sxs-lookup"><span data-stu-id="a933a-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="a933a-163">Twoja firma ma lokalne regulacje, które wymagają, aby połączenie pochodzące z wywołania Onsite było dostępne.</span><span class="sxs-lookup"><span data-stu-id="a933a-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="a933a-164">Twoja sieć ma stałe wąskie gardła, które ograniczają wydajność skalowania.</span><span class="sxs-lookup"><span data-stu-id="a933a-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="a933a-165">Twoje środowisko zostało odłączone od Internetu i nie może nawiązać połączenia z chmurą publiczną.</span><span class="sxs-lookup"><span data-stu-id="a933a-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a933a-166">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="a933a-166">Next steps</span></span>

<span data-ttu-id="a933a-167">Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:</span><span class="sxs-lookup"><span data-stu-id="a933a-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="a933a-168">Zobacz [Omówienie usługi Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) , aby dowiedzieć się więcej na temat działania tego modułu równoważenia obciążenia opartego na systemie DNS.</span><span class="sxs-lookup"><span data-stu-id="a933a-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="a933a-169">Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na dodatkowe pytania.</span><span class="sxs-lookup"><span data-stu-id="a933a-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="a933a-170">Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.</span><span class="sxs-lookup"><span data-stu-id="a933a-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="a933a-171">Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika wdrażania rozwiązań do skalowania między chmurami](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="a933a-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="a933a-172">Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.</span><span class="sxs-lookup"><span data-stu-id="a933a-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="a933a-173">Dowiesz się, jak utworzyć rozwiązanie międzychmurowe, aby zapewnić ręcznie wyzwolony proces przełączania z hostowanej aplikacji sieci Web Azure Stack Hub do aplikacji sieci Web hostowanej na platformie Azure.</span><span class="sxs-lookup"><span data-stu-id="a933a-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="a933a-174">Dowiesz się również, jak używać skalowania automatycznego za pomocą usługi Traffic Manager, zapewniając elastyczne i skalowalne narzędzie w chmurze w ramach obciążenia.</span><span class="sxs-lookup"><span data-stu-id="a933a-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
