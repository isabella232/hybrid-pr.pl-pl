---
title: Wzorzec przekazywania hybrydowego na platformie Azure i w centrum Azure Stack
description: Użyj wzorca przekaźnika hybrydowego na platformie Azure i Azure Stack Hub, aby nawiązać połączenie z zasobami brzegowymi chronionymi przez zapory.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911149"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="1a2b5-103">Wzorzec przekaźnika hybrydowego</span><span class="sxs-lookup"><span data-stu-id="1a2b5-103">Hybrid relay pattern</span></span>

<span data-ttu-id="1a2b5-104">Dowiedz się, jak łączyć się z zasobami lub urządzeniami brzegowymi chronionymi przez zapory przy użyciu wzorca przekaźnika hybrydowego i Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="1a2b5-105">Kontekst i problem</span><span class="sxs-lookup"><span data-stu-id="1a2b5-105">Context and problem</span></span>

<span data-ttu-id="1a2b5-106">Urządzenia brzegowe często znajdują się za zaporą firmową lub urządzeniem NAT.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="1a2b5-107">Mimo że są one bezpieczne, mogą nie być w stanie komunikować się z chmurą publiczną lub urządzeniami brzegowymi w innych sieciach firmowych.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="1a2b5-108">Może być konieczne udostępnienie określonych portów i funkcji użytkownikom w chmurze publicznej w bezpieczny sposób.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="1a2b5-109">Rozwiązanie</span><span class="sxs-lookup"><span data-stu-id="1a2b5-109">Solution</span></span>

<span data-ttu-id="1a2b5-110">Wzorzec przekaźnika hybrydowego używa Azure Relay do ustanowienia tunelu WebSockets między dwoma punktami końcowymi, które nie mogą się bezpośrednio komunikować.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="1a2b5-111">Urządzenia, które nie są lokalne, ale muszą połączyć się z lokalnym punktem końcowym, będą łączyć się z punktem końcowym w chmurze publicznej.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="1a2b5-112">Ten punkt końcowy przekierowuje ruch ze wstępnie zdefiniowanych tras za pośrednictwem bezpiecznego kanału.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="1a2b5-113">Punkt końcowy wewnątrz środowiska lokalnego odbiera ruch i kieruje go do poprawnego miejsca docelowego.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![Architektura rozwiązania hybrydowego wzorca przekaźnika](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="1a2b5-115">Oto jak działa wzorzec przekaźnika hybrydowego:</span><span class="sxs-lookup"><span data-stu-id="1a2b5-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="1a2b5-116">Urządzenie nawiązuje połączenie z maszyną wirtualną na platformie Azure przy użyciu wstępnie zdefiniowanego portu.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="1a2b5-117">Ruch jest przekazywany do Azure Relay na platformie Azure.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="1a2b5-118">Maszyna wirtualna w Azure Stack Hub, która ustanowiła już połączenie o długim czasie życia Azure Relay, odbiera ruch i przekazuje go do miejsca docelowego.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="1a2b5-119">Usługa lokalna lub punkt końcowy przetwarza żądanie.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="1a2b5-120">Składniki</span><span class="sxs-lookup"><span data-stu-id="1a2b5-120">Components</span></span>

<span data-ttu-id="1a2b5-121">To rozwiązanie używa następujących składników:</span><span class="sxs-lookup"><span data-stu-id="1a2b5-121">This solution uses the following components:</span></span>

| <span data-ttu-id="1a2b5-122">Warstwa</span><span class="sxs-lookup"><span data-stu-id="1a2b5-122">Layer</span></span> | <span data-ttu-id="1a2b5-123">Składnik</span><span class="sxs-lookup"><span data-stu-id="1a2b5-123">Component</span></span> | <span data-ttu-id="1a2b5-124">Opis</span><span class="sxs-lookup"><span data-stu-id="1a2b5-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="1a2b5-125">Azure</span><span class="sxs-lookup"><span data-stu-id="1a2b5-125">Azure</span></span> | <span data-ttu-id="1a2b5-126">Maszyna wirtualna platformy Azure</span><span class="sxs-lookup"><span data-stu-id="1a2b5-126">Azure VM</span></span> | <span data-ttu-id="1a2b5-127">Maszyna wirtualna platformy Azure udostępnia publicznie dostępny punkt końcowy dla zasobu lokalnego.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="1a2b5-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="1a2b5-128">Azure Relay</span></span> | <span data-ttu-id="1a2b5-129">[Azure Relay](/azure/azure-relay/) zapewnia infrastrukturę do obsługi tunelu i połączenia między maszyną wirtualną platformy Azure i Azure Stack węzłem maszyny wirtualnej.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="1a2b5-130">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="1a2b5-130">Azure Stack Hub</span></span> | <span data-ttu-id="1a2b5-131">Wystąpienia obliczeniowe</span><span class="sxs-lookup"><span data-stu-id="1a2b5-131">Compute</span></span> | <span data-ttu-id="1a2b5-132">Maszyna wirtualna w Azure Stack Hub udostępnia po stronie serwera tunelu przekaźnika hybrydowego.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="1a2b5-133">Magazyn</span><span class="sxs-lookup"><span data-stu-id="1a2b5-133">Storage</span></span> | <span data-ttu-id="1a2b5-134">Klaster aparatu AKS wdrożony w centrum Azure Stack udostępnia skalowalny, odporny na awarię aparat do uruchomienia kontenera interfejs API rozpoznawania twarzy.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="1a2b5-135">Problemy i kwestie do rozważenia</span><span class="sxs-lookup"><span data-stu-id="1a2b5-135">Issues and considerations</span></span>

<span data-ttu-id="1a2b5-136">Podczas decydowania o sposobie wdrożenia tego rozwiązania należy wziąć pod uwagę następujące kwestie:</span><span class="sxs-lookup"><span data-stu-id="1a2b5-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="1a2b5-137">Skalowalność</span><span class="sxs-lookup"><span data-stu-id="1a2b5-137">Scalability</span></span>

<span data-ttu-id="1a2b5-138">Ten wzorzec zezwala tylko na mapowania portów 1:1 na kliencie i serwerze.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="1a2b5-139">Na przykład jeśli port 80 jest tunelowany dla jednej usługi w punkcie końcowym platformy Azure, nie można go użyć dla innej usługi.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="1a2b5-140">Mapowania portów powinny być odpowiednio planowane.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="1a2b5-141">Azure Relay i maszyny wirtualne powinny być odpowiednio skalowane w celu obsługi ruchu.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="1a2b5-142">Dostępność</span><span class="sxs-lookup"><span data-stu-id="1a2b5-142">Availability</span></span>

<span data-ttu-id="1a2b5-143">Te tunele i połączenia nie są nadmiarowe.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="1a2b5-144">Aby zapewnić wysoką dostępność, można zaimplementować błąd sprawdzania kodu.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="1a2b5-145">Inną opcją jest posiadanie puli maszyn wirtualnych połączonych z Azure Relay za modułem równoważenia obciążenia.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="1a2b5-146">Możliwości zarządzania</span><span class="sxs-lookup"><span data-stu-id="1a2b5-146">Manageability</span></span>

<span data-ttu-id="1a2b5-147">To rozwiązanie może obejmować wiele urządzeń i lokalizacji, które mogą uzyskać nieporęczny.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="1a2b5-148">Usługi IoT platformy Azure mogą automatycznie przenosić nowe lokalizacje i urządzenia w tryb online i aktualizować je na bieżąco.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="1a2b5-149">Zabezpieczenia</span><span class="sxs-lookup"><span data-stu-id="1a2b5-149">Security</span></span>

<span data-ttu-id="1a2b5-150">Ten wzorzec, jak pokazano, umożliwia swobodnego dostęp do portu na urządzeniu wewnętrznym z krawędzi.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="1a2b5-151">Rozważ dodanie mechanizmu uwierzytelniania do usługi na urządzeniu wewnętrznym lub przed punktem końcowym przekaźnika hybrydowego.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1a2b5-152">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="1a2b5-152">Next steps</span></span>

<span data-ttu-id="1a2b5-153">Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:</span><span class="sxs-lookup"><span data-stu-id="1a2b5-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="1a2b5-154">Ten wzorzec używa Azure Relay.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="1a2b5-155">Aby uzyskać więcej informacji, zobacz [dokumentację Azure Relay](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="1a2b5-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="1a2b5-156">Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na wszelkie dodatkowe pytania.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="1a2b5-157">Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="1a2b5-158">Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika wdrażania rozwiązań przekazywania hybrydowego](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="1a2b5-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="1a2b5-159">Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.</span><span class="sxs-lookup"><span data-stu-id="1a2b5-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>