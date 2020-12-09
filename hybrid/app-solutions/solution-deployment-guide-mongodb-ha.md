---
title: Wdróż rozwiązanie MongoDB o wysokiej dostępności na platformie Azure i w centrum Azure Stack
description: Dowiedz się, jak wdrożyć rozwiązanie MongoDB o wysokiej dostępności na platformie Azure i w centrum Azure Stack
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 624f032def509d8e42d55807d72176e5fce85910
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901511"
---
# <a name="deploy-a-highly-available-mongodb-solution-across-two-azure-stack-hub-environments"></a><span data-ttu-id="f842f-103">Wdróż rozwiązanie MongoDB o wysokiej dostępności w dwóch środowiskach centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="f842f-103">Deploy a highly available MongoDB solution across two Azure Stack Hub environments</span></span>

<span data-ttu-id="f842f-104">Ten artykuł przeprowadzi Cię przez automatyczne wdrożenie podstawowego klastra o wysokiej dostępności (HA) MongoDB z lokacją odzyskiwania po awarii w dwóch środowiskach centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f842f-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="f842f-105">Aby dowiedzieć się więcej na temat MongoDB i wysokiej dostępności, zobacz [elementy członkowskie zestawu replik](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="f842f-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="f842f-106">W tym rozwiązaniu utworzysz przykładowe środowisko, aby:</span><span class="sxs-lookup"><span data-stu-id="f842f-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="f842f-107">Organizuj wdrożenie w dwóch centrach Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f842f-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="f842f-108">Użyj platformy Docker, aby zminimalizować problemy zależności z profilami interfejsu API platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="f842f-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="f842f-109">Wdróż klaster MongoDB o wysokiej dostępności z lokacją odzyskiwania po awarii.</span><span class="sxs-lookup"><span data-stu-id="f842f-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="f842f-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="f842f-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="f842f-111">Microsoft Azure Stack Hub to rozszerzenie platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="f842f-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="f842f-112">Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.</span><span class="sxs-lookup"><span data-stu-id="f842f-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="f842f-113">[Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych.</span><span class="sxs-lookup"><span data-stu-id="f842f-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="f842f-114">Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.</span><span class="sxs-lookup"><span data-stu-id="f842f-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="f842f-115">Architektura MongoDB z centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="f842f-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Architektura MongoDB o wysokiej dostępności w centrum Azure Stack](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="f842f-117">Wymagania wstępne dotyczące usługi MongoDB z centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="f842f-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="f842f-118">Dwa połączone systemy Azure Stack Hub zintegrowane (Azure Stack Hub).</span><span class="sxs-lookup"><span data-stu-id="f842f-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="f842f-119">To wdrożenie nie działa na Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="f842f-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="f842f-120">Aby dowiedzieć się więcej na temat Azure Stack Hub, zobacz [co to jest centrum Azure Stack?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="f842f-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="f842f-121">Subskrypcja dzierżawy w każdym centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f842f-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="f842f-122">**Zanotuj każdy identyfikator subskrypcji i punkt końcowy Azure Resource Manager dla poszczególnych centrów Azure Stack.**</span><span class="sxs-lookup"><span data-stu-id="f842f-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="f842f-123">Nazwa główna usługi Azure Active Directory (Azure AD), która ma uprawnienia do subskrypcji dzierżawy w poszczególnych centrach Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f842f-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="f842f-124">Jeśli centra Azure Stack są wdrażane na różnych dzierżawach usługi Azure AD, może być konieczne utworzenie dwóch jednostek usługi.</span><span class="sxs-lookup"><span data-stu-id="f842f-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="f842f-125">Aby dowiedzieć się, jak utworzyć jednostkę usługi dla Azure Stack Hub, zobacz [Korzystanie z tożsamości aplikacji w celu uzyskiwania dostępu do zasobów centrum Azure Stack](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="f842f-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="f842f-126">**Zanotuj każdy identyfikator aplikacji jednostki usługi, klucz tajny klienta i nazwę dzierżawy (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="f842f-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="f842f-127">Ubuntu 16,04 został zespolony z każdym z rynków centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="f842f-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="f842f-128">Aby dowiedzieć się więcej na temat zespalania portalu Marketplace, zobacz artykuł [Pobieranie elementów z witryny Marketplace do centrum Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="f842f-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="f842f-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) zainstalowany na komputerze lokalnym.</span><span class="sxs-lookup"><span data-stu-id="f842f-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="f842f-130">Pobierz obraz platformy Docker</span><span class="sxs-lookup"><span data-stu-id="f842f-130">Get the Docker image</span></span>

<span data-ttu-id="f842f-131">Obrazy platformy Docker dla każdego wdrożenia eliminują problemy zależności między różnymi wersjami Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="f842f-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="f842f-132">Upewnij się, że Docker for Windows używa kontenerów systemu Windows.</span><span class="sxs-lookup"><span data-stu-id="f842f-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="f842f-133">Uruchom następujące polecenie w wierszu polecenia z podwyższonym poziomem uprawnień, aby uzyskać kontener platformy Docker ze skryptami wdrażania.</span><span class="sxs-lookup"><span data-stu-id="f842f-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="f842f-134">Wdrażanie klastrów</span><span class="sxs-lookup"><span data-stu-id="f842f-134">Deploy the clusters</span></span>

1. <span data-ttu-id="f842f-135">Po pomyślnym ściągnięciu obrazu kontenera Uruchom obraz.</span><span class="sxs-lookup"><span data-stu-id="f842f-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="f842f-136">Po rozpoczęciu kontenera otrzymasz w kontenerze odpowiedni Terminal programu PowerShell z podwyższonym poziomem uprawnień.</span><span class="sxs-lookup"><span data-stu-id="f842f-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="f842f-137">Zmień katalogi, aby przejść do skryptu wdrażania.</span><span class="sxs-lookup"><span data-stu-id="f842f-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="f842f-138">Uruchom wdrożenie.</span><span class="sxs-lookup"><span data-stu-id="f842f-138">Run the deployment.</span></span> <span data-ttu-id="f842f-139">Podaj poświadczenia i nazwy zasobów, jeśli jest to potrzebne.</span><span class="sxs-lookup"><span data-stu-id="f842f-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="f842f-140">HA odnosi się do centrum Azure Stack, w którym zostanie wdrożony klaster HA.</span><span class="sxs-lookup"><span data-stu-id="f842f-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="f842f-141">Program DR odwołuje się do centrum Azure Stack, w którym zostanie wdrożony klaster odzyskiwania po awarii.</span><span class="sxs-lookup"><span data-stu-id="f842f-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="f842f-142">Wpisz `Y` polecenie, aby zezwolić na instalację dostawcy NuGet, co spowoduje rozpoczęcie instalacji profilu interfejsu API "2018-03-01-hybrydowe".</span><span class="sxs-lookup"><span data-stu-id="f842f-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="f842f-143">Zasoby HA zostaną wdrożone jako pierwsze.</span><span class="sxs-lookup"><span data-stu-id="f842f-143">The HA resources will deploy first.</span></span> <span data-ttu-id="f842f-144">Monitoruj wdrożenie i poczekaj na jego zakończenie.</span><span class="sxs-lookup"><span data-stu-id="f842f-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="f842f-145">Gdy zostanie wyświetlony komunikat z informacją, że wdrożenie HA zostało zakończone, można sprawdzić Portal centrum Azure Stack HA, aby zobaczyć wdrożone zasoby.</span><span class="sxs-lookup"><span data-stu-id="f842f-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="f842f-146">Kontynuuj wdrażanie zasobów DR i zdecyduj, czy chcesz włączyć pole skoku w centrum Azure Stack DR, aby móc korzystać z klastra.</span><span class="sxs-lookup"><span data-stu-id="f842f-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="f842f-147">Poczekaj na zakończenie wdrożenia zasobu DR.</span><span class="sxs-lookup"><span data-stu-id="f842f-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="f842f-148">Po zakończeniu wdrażania zasobów DR Zamknij kontener.</span><span class="sxs-lookup"><span data-stu-id="f842f-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="f842f-149">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="f842f-149">Next steps</span></span>

- <span data-ttu-id="f842f-150">Jeśli włączono maszynę wirtualną pola skoku w centrum Azure Stack DR, możesz nawiązać połączenie za pośrednictwem protokołu SSH i korzystać z klastra MongoDB przez zainstalowanie interfejsu wiersza polecenia Mongo.</span><span class="sxs-lookup"><span data-stu-id="f842f-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="f842f-151">Aby dowiedzieć się więcej na temat współpracy z usługą MongoDB, zobacz [powłoka Mongo](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="f842f-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="f842f-152">Aby dowiedzieć się więcej na temat hybrydowych aplikacji w chmurze, zobacz [hybrydowe rozwiązania w chmurze.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="f842f-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="f842f-153">Zmodyfikuj kod do tego przykładu w serwisie [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="f842f-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
