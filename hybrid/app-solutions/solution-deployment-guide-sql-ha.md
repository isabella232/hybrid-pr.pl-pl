---
title: Wdróż grupę dostępności SQL Server 2016 na platformie Azure i w centrum Azure Stack
description: Dowiedz się, jak wdrożyć grupę dostępności SQL Server 2016 na platformie Azure i w centrum Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852477"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="d28cb-103">Wdróż grupę dostępności SQL Server 2016 na platformie Azure i w centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="d28cb-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="d28cb-104">Ten artykuł przeprowadzi Cię przez automatyczne wdrożenie podstawowego klastra o wysokiej dostępności (HA) SQL Server 2016 przedsiębiorstwa z asynchroniczną lokacją odzyskiwania po awarii w dwóch środowiskach Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="d28cb-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="d28cb-105">Aby dowiedzieć się więcej o SQL Server 2016 i wysokiej dostępności, zobacz [zawsze włączone grupy dostępności: rozwiązanie wysokiej dostępności i odzyskiwania po awarii](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="d28cb-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="d28cb-106">W tym rozwiązaniu utworzysz przykładowe środowisko w celu:</span><span class="sxs-lookup"><span data-stu-id="d28cb-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="d28cb-107">Organizuj wdrożenie w dwóch centrach Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d28cb-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="d28cb-108">Użyj platformy Docker, aby zminimalizować problemy zależności z profilami interfejsu API platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="d28cb-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="d28cb-109">Wdróż klaster Enterprise o wysokiej dostępności SQL Server 2016 przedsiębiorstwa z lokacją odzyskiwania po awarii.</span><span class="sxs-lookup"><span data-stu-id="d28cb-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="d28cb-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="d28cb-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="d28cb-111">Microsoft Azure Stack Hub to rozszerzenie platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="d28cb-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="d28cb-112">Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.</span><span class="sxs-lookup"><span data-stu-id="d28cb-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="d28cb-113">[Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych.</span><span class="sxs-lookup"><span data-stu-id="d28cb-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="d28cb-114">Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.</span><span class="sxs-lookup"><span data-stu-id="d28cb-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="d28cb-115">Architektura dla SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="d28cb-115">Architecture for SQL Server 2016</span></span>

![Centrum Azure Stack SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="d28cb-117">Wymagania wstępne dotyczące SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="d28cb-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="d28cb-118">Dwa połączone systemy Azure Stack Hub zintegrowane (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="d28cb-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="d28cb-119">To wdrożenie nie działa na Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="d28cb-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="d28cb-120">Aby dowiedzieć się więcej o centrum Azure Stack, zobacz [omówienie Azure Stack](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="d28cb-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="d28cb-121">Subskrypcja dzierżawy w każdym centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d28cb-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="d28cb-122">**Zanotuj każdy identyfikator subskrypcji i punkt końcowy Azure Resource Manager dla poszczególnych centrów Azure Stack.**</span><span class="sxs-lookup"><span data-stu-id="d28cb-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="d28cb-123">Nazwa główna usługi Azure Active Directory (Azure AD), która ma uprawnienia do subskrypcji dzierżawy w poszczególnych centrach Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d28cb-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="d28cb-124">Jeśli centra Azure Stack są wdrażane na różnych dzierżawach usługi Azure AD, może być konieczne utworzenie dwóch jednostek usługi.</span><span class="sxs-lookup"><span data-stu-id="d28cb-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="d28cb-125">Aby dowiedzieć się, jak utworzyć jednostkę usługi dla Azure Stack Hub, zobacz [Tworzenie nazw głównych usług, aby umożliwić aplikacjom dostęp do Azure Stack zasobów centrum](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="d28cb-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="d28cb-126">**Zanotuj każdy identyfikator aplikacji jednostki usługi, klucz tajny klienta i nazwę dzierżawy (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="d28cb-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="d28cb-127">SQL Server 2016 Enterprise jest zespolona z każdym rynkiem Azure Stack centrum.</span><span class="sxs-lookup"><span data-stu-id="d28cb-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="d28cb-128">Aby dowiedzieć się więcej na temat zespalania portalu Marketplace, zobacz artykuł [Pobieranie elementów z witryny Marketplace do centrum Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="d28cb-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="d28cb-129">**Upewnij się, że Twoja organizacja ma odpowiednie licencje SQL.**</span><span class="sxs-lookup"><span data-stu-id="d28cb-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="d28cb-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) zainstalowany na komputerze lokalnym.</span><span class="sxs-lookup"><span data-stu-id="d28cb-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="d28cb-131">Pobierz obraz platformy Docker</span><span class="sxs-lookup"><span data-stu-id="d28cb-131">Get the Docker image</span></span>

<span data-ttu-id="d28cb-132">Obrazy platformy Docker dla każdego wdrożenia eliminują problemy zależności między różnymi wersjami Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="d28cb-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="d28cb-133">Upewnij się, że Docker for Windows używa kontenerów systemu Windows.</span><span class="sxs-lookup"><span data-stu-id="d28cb-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="d28cb-134">Uruchom następujący skrypt w wierszu polecenia z podwyższonym poziomem uprawnień, aby uzyskać kontener platformy Docker ze skryptami wdrażania.</span><span class="sxs-lookup"><span data-stu-id="d28cb-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="d28cb-135">Wdróż grupę dostępności</span><span class="sxs-lookup"><span data-stu-id="d28cb-135">Deploy the availability group</span></span>

1. <span data-ttu-id="d28cb-136">Po pomyślnym ściągnięciu obrazu kontenera Uruchom obraz.</span><span class="sxs-lookup"><span data-stu-id="d28cb-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="d28cb-137">Po rozpoczęciu kontenera otrzymasz w kontenerze odpowiedni Terminal programu PowerShell z podwyższonym poziomem uprawnień.</span><span class="sxs-lookup"><span data-stu-id="d28cb-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="d28cb-138">Zmień katalogi, aby przejść do skryptu wdrażania.</span><span class="sxs-lookup"><span data-stu-id="d28cb-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="d28cb-139">Uruchom wdrożenie.</span><span class="sxs-lookup"><span data-stu-id="d28cb-139">Run the deployment.</span></span> <span data-ttu-id="d28cb-140">Podaj poświadczenia i nazwy zasobów, jeśli jest to potrzebne.</span><span class="sxs-lookup"><span data-stu-id="d28cb-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="d28cb-141">HA odnosi się do centrum Azure Stack, w którym zostanie wdrożony klaster HA.</span><span class="sxs-lookup"><span data-stu-id="d28cb-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="d28cb-142">Program DR odwołuje się do centrum Azure Stack, w którym zostanie wdrożony klaster odzyskiwania po awarii.</span><span class="sxs-lookup"><span data-stu-id="d28cb-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. <span data-ttu-id="d28cb-143">Wpisz `Y` polecenie, aby zezwolić na instalację dostawcy NuGet, co spowoduje rozpoczęcie instalacji profilu interfejsu API "2018-03-01-hybrydowe".</span><span class="sxs-lookup"><span data-stu-id="d28cb-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="d28cb-144">Poczekaj na zakończenie wdrażania zasobów.</span><span class="sxs-lookup"><span data-stu-id="d28cb-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="d28cb-145">Po zakończeniu wdrażania zasobów DR Zamknij kontener.</span><span class="sxs-lookup"><span data-stu-id="d28cb-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="d28cb-146">Zbadaj wdrożenie, wyświetlając zasoby w każdym portalu Centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="d28cb-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="d28cb-147">Połącz się z jednym z wystąpień programu SQL Server w środowisku HA i sprawdź grupę dostępności za pomocą SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="d28cb-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="d28cb-149">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="d28cb-149">Next steps</span></span>

- <span data-ttu-id="d28cb-150">Użyj SQL Server Management Studio, aby ręcznie przełączyć klaster w tryb failover.</span><span class="sxs-lookup"><span data-stu-id="d28cb-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="d28cb-151">Zobacz [przeprowadzenie wymuszonej ręcznej pracy awaryjnej grupy dostępności zawsze włączone (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="d28cb-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="d28cb-152">Dowiedz się więcej o aplikacjach hybrydowych w chmurze.</span><span class="sxs-lookup"><span data-stu-id="d28cb-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="d28cb-153">Zobacz [hybrydowe rozwiązania w chmurze.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="d28cb-153">See [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="d28cb-154">Użyj własnych danych lub zmodyfikuj kod w tym przykładzie w witrynie [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="d28cb-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>