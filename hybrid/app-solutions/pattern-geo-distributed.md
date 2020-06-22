---
title: Wzorzec aplikacji rozproszonej geograficznie w centrum Azure Stack
description: Dowiedz się więcej na temat wzorca aplikacji rozproszonej geograficznej dla inteligentnej krawędzi za pomocą platformy Azure i usługi Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910883"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="9a630-103">Wzorzec aplikacji rozproszonej geograficznie</span><span class="sxs-lookup"><span data-stu-id="9a630-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="9a630-104">Dowiedz się, jak dostarczać punkty końcowe aplikacji w wielu regionach i kierować ruchem użytkowników w oparciu o lokalizacje i wymagania dotyczące zgodności.</span><span class="sxs-lookup"><span data-stu-id="9a630-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="9a630-105">Kontekst i problem</span><span class="sxs-lookup"><span data-stu-id="9a630-105">Context and problem</span></span>

<span data-ttu-id="9a630-106">Organizacje o szerokim zasięgu lokalizacje geograficzne dążą do bezpiecznego i dokładnego rozpowszechniania i zapewniania dostępu do danych przy jednoczesnym zapewnieniu wymaganych poziomów zabezpieczeń, zgodności i wydajności dla poszczególnych użytkowników, lokalizacji i urządzeń.</span><span class="sxs-lookup"><span data-stu-id="9a630-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="9a630-107">Rozwiązanie</span><span class="sxs-lookup"><span data-stu-id="9a630-107">Solution</span></span>

<span data-ttu-id="9a630-108">Wzorzec routingu ruchu geograficznego centrum Azure Stack lub aplikacje rozproszone geograficznie umożliwiają kierowanie ruchu do określonych punktów końcowych na podstawie różnych metryk.</span><span class="sxs-lookup"><span data-stu-id="9a630-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="9a630-109">Tworzenie Traffic Manager przy użyciu geograficznej konfiguracji routingu i punktu końcowego kieruje ruch do punktów końcowych na podstawie wymagań regionalnych, regulacji korporacyjnych i międzynarodowych oraz potrzeb dotyczących danych.</span><span class="sxs-lookup"><span data-stu-id="9a630-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Wzorzec rozproszony geograficznie](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="9a630-111">Składniki</span><span class="sxs-lookup"><span data-stu-id="9a630-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="9a630-112">Poza chmurą</span><span class="sxs-lookup"><span data-stu-id="9a630-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="9a630-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="9a630-113">Traffic Manager</span></span>

<span data-ttu-id="9a630-114">Na diagramie Traffic Manager znajduje się poza chmurą publiczną, ale musi być w stanie koordynować ruch zarówno w lokalnym centrum danych, jak i w chmurze publicznej.</span><span class="sxs-lookup"><span data-stu-id="9a630-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="9a630-115">Moduł równoważenia obciążenia kieruje ruch do lokalizacji geograficznych.</span><span class="sxs-lookup"><span data-stu-id="9a630-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="9a630-116">System nazw domen (DNS)</span><span class="sxs-lookup"><span data-stu-id="9a630-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="9a630-117">System nazw domen (DNS) jest odpowiedzialny za tłumaczenie (lub rozwiązanie) nazwy witryny sieci Web lub usługi na adres IP.</span><span class="sxs-lookup"><span data-stu-id="9a630-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="9a630-118">Chmura publiczna</span><span class="sxs-lookup"><span data-stu-id="9a630-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="9a630-119">Punkt końcowy w chmurze</span><span class="sxs-lookup"><span data-stu-id="9a630-119">Cloud Endpoint</span></span>

<span data-ttu-id="9a630-120">Publiczne adresy IP są używane do kierowania ruchu przychodzącego za pomocą usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.</span><span class="sxs-lookup"><span data-stu-id="9a630-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="9a630-121">Chmury lokalne</span><span class="sxs-lookup"><span data-stu-id="9a630-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="9a630-122">Lokalny punkt końcowy</span><span class="sxs-lookup"><span data-stu-id="9a630-122">Local endpoint</span></span>

<span data-ttu-id="9a630-123">Publiczne adresy IP są używane do kierowania ruchu przychodzącego za pomocą usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.</span><span class="sxs-lookup"><span data-stu-id="9a630-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="9a630-124">Problemy i kwestie do rozważenia</span><span class="sxs-lookup"><span data-stu-id="9a630-124">Issues and considerations</span></span>

<span data-ttu-id="9a630-125">Podczas podejmowania decyzji o sposobie wdrożenia tego wzorca należy rozważyć następujące punkty:</span><span class="sxs-lookup"><span data-stu-id="9a630-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="9a630-126">Skalowalność</span><span class="sxs-lookup"><span data-stu-id="9a630-126">Scalability</span></span>

<span data-ttu-id="9a630-127">Wzorzec obsługuje routing ruchu geograficznego zamiast skalowania w celu osiągnięcia wzrostu ruchu.</span><span class="sxs-lookup"><span data-stu-id="9a630-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="9a630-128">Można jednak połączyć ten wzorzec z innymi rozwiązaniami platformy Azure i lokalnymi.</span><span class="sxs-lookup"><span data-stu-id="9a630-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="9a630-129">Na przykład ten wzorzec może być używany z wzorcem skalowania między chmurami.</span><span class="sxs-lookup"><span data-stu-id="9a630-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="9a630-130">Dostępność</span><span class="sxs-lookup"><span data-stu-id="9a630-130">Availability</span></span>

<span data-ttu-id="9a630-131">Upewnij się, że aplikacje wdrożone lokalnie są skonfigurowane pod kątem wysokiej dostępności za poorednictwem konfiguracji sprzętu lokalnego i wdrożenia oprogramowania.</span><span class="sxs-lookup"><span data-stu-id="9a630-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="9a630-132">Możliwości zarządzania</span><span class="sxs-lookup"><span data-stu-id="9a630-132">Manageability</span></span>

<span data-ttu-id="9a630-133">Wzorzec zapewnia bezproblemowe zarządzanie i przyjazny interfejs między środowiskami.</span><span class="sxs-lookup"><span data-stu-id="9a630-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="9a630-134">Kiedy używać tego wzorca</span><span class="sxs-lookup"><span data-stu-id="9a630-134">When to use this pattern</span></span>

- <span data-ttu-id="9a630-135">Moja organizacja ma rozgałęzienia międzynarodowe wymagające niestandardowych zasad zabezpieczeń i dystrybucji.</span><span class="sxs-lookup"><span data-stu-id="9a630-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="9a630-136">Każda z biur organizacji pobiera dane pracowników, działalności i możliwości, wymagając działań związanych z raportowaniem na lokalne regulacje i strefę czasową.</span><span class="sxs-lookup"><span data-stu-id="9a630-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="9a630-137">Wymagania dotyczące dużej skali mogą być spełnione przez skalowanie w poziomie aplikacji, z wieloma wdrożeniami aplikacji wykonywanych w jednym regionie i między regionami w celu obsługi skrajnych wymagań dotyczących obciążenia.</span><span class="sxs-lookup"><span data-stu-id="9a630-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="9a630-138">Aplikacje muszą mieć wysoką dostępność i odpowiadać na żądania klientów nawet w przypadku awarii w jednym regionie.</span><span class="sxs-lookup"><span data-stu-id="9a630-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="9a630-139">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="9a630-139">Next steps</span></span>

<span data-ttu-id="9a630-140">Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:</span><span class="sxs-lookup"><span data-stu-id="9a630-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="9a630-141">Zobacz [Omówienie usługi Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) , aby dowiedzieć się więcej na temat działania tego modułu równoważenia obciążenia opartego na systemie DNS.</span><span class="sxs-lookup"><span data-stu-id="9a630-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="9a630-142">Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na dodatkowe pytania.</span><span class="sxs-lookup"><span data-stu-id="9a630-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="9a630-143">Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.</span><span class="sxs-lookup"><span data-stu-id="9a630-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="9a630-144">Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika po wdrożeniu rozwiązania aplikacji rozproszonej geograficznie](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="9a630-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="9a630-145">Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.</span><span class="sxs-lookup"><span data-stu-id="9a630-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="9a630-146">Dowiesz się, jak skierować ruch do określonych punktów końcowych na podstawie różnych metryk przy użyciu wzorca aplikacji rozproszonej geograficznie.</span><span class="sxs-lookup"><span data-stu-id="9a630-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="9a630-147">Tworzenie profilu Traffic Manager przy użyciu geograficznej konfiguracji routingu i punktu końcowego zapewnia, że informacje są kierowane do punktów końcowych w oparciu o wymagania regionalne, przepisy firmowe i międzynarodowe oraz Twoje potrzeby dotyczące danych.</span><span class="sxs-lookup"><span data-stu-id="9a630-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
