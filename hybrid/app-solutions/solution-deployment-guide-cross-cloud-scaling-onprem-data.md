---
title: Wdróż aplikację hybrydową przy użyciu danych lokalnych, które skaluje się z wielu chmur
description: Dowiedz się, jak wdrożyć aplikację korzystającą z danych lokalnych i skalować wiele chmur przy użyciu usług Azure i Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0989859fd68847932d3e69defee59740a2bffd44
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895401"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="31cd3-103">Wdróż aplikację hybrydową przy użyciu danych lokalnych, które skaluje się z wielu chmur</span><span class="sxs-lookup"><span data-stu-id="31cd3-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="31cd3-104">W tym przewodniku po rozwiązaniu pokazano, jak wdrożyć aplikację hybrydową, która obejmuje zarówno platformę Azure, jak i centrum Azure Stack i używa jednego lokalnego źródła danych.</span><span class="sxs-lookup"><span data-stu-id="31cd3-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="31cd3-105">Korzystając z rozwiązania w chmurze hybrydowej, można połączyć zalety chmury prywatnej z skalowalnością chmury publicznej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="31cd3-106">Deweloperzy mogą również korzystać z ekosystemu deweloperów firmy Microsoft i stosować swoje umiejętności w środowiskach chmurowych i lokalnych.</span><span class="sxs-lookup"><span data-stu-id="31cd3-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="31cd3-107">Przegląd i założenia</span><span class="sxs-lookup"><span data-stu-id="31cd3-107">Overview and assumptions</span></span>

<span data-ttu-id="31cd3-108">Postępuj zgodnie z tym samouczkiem, aby skonfigurować przepływ pracy, który umożliwia deweloperom wdrożenie identycznej aplikacji sieci Web w chmurze publicznej i prywatnej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="31cd3-109">Ta aplikacja może uzyskiwać dostęp do sieci z obsługą routingu bez Internetu hostowanej w chmurze prywatnej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="31cd3-110">Te aplikacje sieci Web są monitorowane i w przypadku wzrostu ruchu program modyfikuje rekordy DNS w celu przekierowania ruchu do chmury publicznej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="31cd3-111">Gdy ruch spadnie do poziomu przed skokiem, ruch jest kierowany z powrotem do chmury prywatnej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="31cd3-112">Ten samouczek obejmuje następujące zadania:</span><span class="sxs-lookup"><span data-stu-id="31cd3-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="31cd3-113">Wdróż serwer bazy danych SQL Server połączony hybrydowo.</span><span class="sxs-lookup"><span data-stu-id="31cd3-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="31cd3-114">Połącz aplikację sieci Web na platformie Azure w sieci hybrydowej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="31cd3-115">Skonfiguruj system DNS do skalowania między chmurami.</span><span class="sxs-lookup"><span data-stu-id="31cd3-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="31cd3-116">Skonfiguruj certyfikaty SSL na potrzeby skalowania między chmurami.</span><span class="sxs-lookup"><span data-stu-id="31cd3-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="31cd3-117">Skonfiguruj i Wdróż aplikację internetową.</span><span class="sxs-lookup"><span data-stu-id="31cd3-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="31cd3-118">Utwórz profil Traffic Manager i skonfiguruj go do skalowania między chmurami.</span><span class="sxs-lookup"><span data-stu-id="31cd3-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="31cd3-119">Skonfiguruj Application Insights monitorowania i generowania alertów w przypadku zwiększonego ruchu.</span><span class="sxs-lookup"><span data-stu-id="31cd3-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="31cd3-120">Skonfiguruj automatyczne przełączanie ruchu między globalnymi centrami Azure i Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="31cd3-121">![Diagram filarów hybrydowych](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="31cd3-121">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="31cd3-122">Microsoft Azure Stack Hub to rozszerzenie platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="31cd3-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="31cd3-123">Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.</span><span class="sxs-lookup"><span data-stu-id="31cd3-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="31cd3-124">[Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych.</span><span class="sxs-lookup"><span data-stu-id="31cd3-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="31cd3-125">Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.</span><span class="sxs-lookup"><span data-stu-id="31cd3-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="31cd3-126">Założenia</span><span class="sxs-lookup"><span data-stu-id="31cd3-126">Assumptions</span></span>

<span data-ttu-id="31cd3-127">W tym samouczku założono, że masz podstawową wiedzę na temat globalnego systemu Azure i centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="31cd3-128">Jeśli chcesz dowiedzieć się więcej przed rozpoczęciem pracy z samouczkiem, zapoznaj się z następującymi artykułami:</span><span class="sxs-lookup"><span data-stu-id="31cd3-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="31cd3-129">Wprowadzenie do platformy Azure</span><span class="sxs-lookup"><span data-stu-id="31cd3-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="31cd3-130">Podstawowe pojęcia Azure Stack centrum</span><span class="sxs-lookup"><span data-stu-id="31cd3-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview)

<span data-ttu-id="31cd3-131">W tym samouczku przyjęto również założenie, że masz subskrypcję platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="31cd3-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="31cd3-132">Jeśli nie masz subskrypcji, przed rozpoczęciem [Utwórz bezpłatne konto](https://azure.microsoft.com/free/) .</span><span class="sxs-lookup"><span data-stu-id="31cd3-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="31cd3-133">Wymagania wstępne</span><span class="sxs-lookup"><span data-stu-id="31cd3-133">Prerequisites</span></span>

<span data-ttu-id="31cd3-134">Przed rozpoczęciem tego rozwiązania upewnij się, że spełniasz następujące wymagania:</span><span class="sxs-lookup"><span data-stu-id="31cd3-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="31cd3-135">Azure Stack Development Kit (ASDK) lub subskrypcja zintegrowanego systemu Azure Stack centrum.</span><span class="sxs-lookup"><span data-stu-id="31cd3-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="31cd3-136">Aby wdrożyć ASDK, postępuj zgodnie z instrukcjami podanymi w temacie [wdrażanie ASDK przy użyciu Instalatora](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="31cd3-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install).</span></span>
- <span data-ttu-id="31cd3-137">Instalacja centrum Azure Stack powinna mieć zainstalowane następujące elementy:</span><span class="sxs-lookup"><span data-stu-id="31cd3-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="31cd3-138">Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="31cd3-138">The Azure App Service.</span></span> <span data-ttu-id="31cd3-139">Współpracuj z operatorem Azure Stack Hub, aby wdrażać i konfigurować Azure App Service w danym środowisku.</span><span class="sxs-lookup"><span data-stu-id="31cd3-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="31cd3-140">Ten samouczek wymaga, aby App Service mieć co najmniej jedną dostępną rolę procesu roboczego (1).</span><span class="sxs-lookup"><span data-stu-id="31cd3-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="31cd3-141">Obraz systemu Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="31cd3-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="31cd3-142">Windows Server 2016 z obrazem Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="31cd3-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="31cd3-143">Odpowiednie plany i oferty.</span><span class="sxs-lookup"><span data-stu-id="31cd3-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="31cd3-144">Nazwa domeny dla aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="31cd3-144">A domain name for your web app.</span></span> <span data-ttu-id="31cd3-145">Jeśli nie masz nazwy domeny, możesz zakupić ją od dostawcy domeny, np. GoDaddy, Bluehost i unmotion.</span><span class="sxs-lookup"><span data-stu-id="31cd3-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="31cd3-146">Certyfikat SSL dla domeny z zaufanego urzędu certyfikacji, takiego jak LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="31cd3-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="31cd3-147">Aplikacja sieci Web, która komunikuje się z SQL Server bazą danych i obsługuje Application Insights.</span><span class="sxs-lookup"><span data-stu-id="31cd3-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="31cd3-148">Przykładową aplikację [dotnetcore-SQLDB-samouczka](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) można pobrać z witryny GitHub.</span><span class="sxs-lookup"><span data-stu-id="31cd3-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="31cd3-149">Sieć hybrydowa między siecią wirtualną platformy Azure a siecią wirtualną centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="31cd3-150">Aby uzyskać szczegółowe instrukcje, zobacz [Konfigurowanie łączności chmury hybrydowej za pomocą platformy Azure i centrum Azure Stack](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="31cd3-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="31cd3-151">Potok hybrydowej ciągłej integracji/ciągłego wdrażania (CI/CD) z prywatnym agentem kompilacji na Azure Stack centrum.</span><span class="sxs-lookup"><span data-stu-id="31cd3-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="31cd3-152">Aby uzyskać szczegółowe instrukcje, zobacz [Konfigurowanie tożsamości chmury hybrydowej przy użyciu platformy Azure i aplikacji centrum Azure Stack](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="31cd3-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="31cd3-153">Wdrażanie serwera bazy danych połączonego hybrydowo SQL Server</span><span class="sxs-lookup"><span data-stu-id="31cd3-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="31cd3-154">Zaloguj się do portalu użytkowników centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="31cd3-155">Na **pulpicie nawigacyjnym** wybierz pozycję **Marketplace**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack centrum Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="31cd3-157">W obszarze **Marketplace** wybierz pozycję **obliczenia**, a następnie wybierz pozycję **więcej**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="31cd3-158">W obszarze **więcej** wybierz **licencję bezpłatna SQL Server: SQL Server 2017 Developer w obrazie systemu Windows Server** .</span><span class="sxs-lookup"><span data-stu-id="31cd3-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Wybieranie obrazu maszyny wirtualnej w portalu użytkowników Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="31cd3-160">**Bezpłatna SQL Server Licencja: SQL Server 2017 Developer w systemie Windows Server**, wybierz pozycję **Utwórz**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="31cd3-161">Na stronie **podstawowe > Skonfiguruj ustawienia podstawowe**, podaj **nazwę** maszyny wirtualnej (VM), **nazwę użytkownika** dla SQL Server sa i **hasło** dla tego skojarzenia.</span><span class="sxs-lookup"><span data-stu-id="31cd3-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="31cd3-162">Z listy rozwijanej **subskrypcja** wybierz subskrypcję, która ma zostać wdrożona.</span><span class="sxs-lookup"><span data-stu-id="31cd3-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="31cd3-163">W obszarze **Grupa zasobów** Użyj **opcji wybierz istniejącą** i umieść maszynę wirtualną w tej samej grupie zasobów, co aplikacja sieci Web Centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Konfigurowanie podstawowych ustawień maszyny wirtualnej w portalu użytkownika Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="31cd3-165">W obszarze **rozmiar** wybierz rozmiar maszyny wirtualnej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="31cd3-166">W tym samouczku zalecamy A2_Standard lub DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="31cd3-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="31cd3-167">W obszarze **ustawienia > Skonfiguruj funkcje opcjonalne** skonfiguruj następujące ustawienia:</span><span class="sxs-lookup"><span data-stu-id="31cd3-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="31cd3-168">**Konto magazynu**: Utwórz nowe konto, jeśli będzie potrzebne.</span><span class="sxs-lookup"><span data-stu-id="31cd3-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="31cd3-169">**Sieć wirtualna**:</span><span class="sxs-lookup"><span data-stu-id="31cd3-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="31cd3-170">Upewnij się, że maszyna wirtualna SQL Server jest wdrożona w tej samej sieci wirtualnej co bramy sieci VPN.</span><span class="sxs-lookup"><span data-stu-id="31cd3-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="31cd3-171">**Publiczny adres IP**: Użyj ustawień domyślnych.</span><span class="sxs-lookup"><span data-stu-id="31cd3-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="31cd3-172">**Sieciowa Grupa zabezpieczeń**: (sieciowej grupy zabezpieczeń).</span><span class="sxs-lookup"><span data-stu-id="31cd3-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="31cd3-173">Utwórz nowy sieciowej grupy zabezpieczeń.</span><span class="sxs-lookup"><span data-stu-id="31cd3-173">Create a new NSG.</span></span>
   - <span data-ttu-id="31cd3-174">**Rozszerzenia i monitorowanie**: Zachowaj ustawienia domyślne.</span><span class="sxs-lookup"><span data-stu-id="31cd3-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="31cd3-175">**Konto magazynu diagnostyki**: Utwórz nowe konto, jeśli będzie potrzebne.</span><span class="sxs-lookup"><span data-stu-id="31cd3-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="31cd3-176">Wybierz **przycisk OK** , aby zapisać konfigurację.</span><span class="sxs-lookup"><span data-stu-id="31cd3-176">Select **OK** to save your configuration.</span></span>

     ![Skonfiguruj opcjonalne funkcje maszyny wirtualnej w portalu użytkowników w Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="31cd3-178">W obszarze **ustawienia SQL Server** skonfiguruj następujące ustawienia:</span><span class="sxs-lookup"><span data-stu-id="31cd3-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="31cd3-179">W przypadku **połączeń SQL** wybierz opcję **publiczny (Internet)**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="31cd3-180">W polu **port** pozostaw wartość domyślną **1433**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="31cd3-181">W obszarze **uwierzytelnianie SQL** wybierz pozycję **Włącz**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="31cd3-182">Po włączeniu uwierzytelniania SQL należy automatycznie wypełnić przy użyciu informacji "sqladmin", które zostały skonfigurowane w **podstawie**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="31cd3-183">W pozostałych ustawieniach Zachowaj ustawienia domyślne.</span><span class="sxs-lookup"><span data-stu-id="31cd3-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="31cd3-184">Wybierz przycisk **OK**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-184">Select **OK**.</span></span>

     ![Konfigurowanie ustawień SQL Server w portalu użytkowników Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="31cd3-186">Na stronie **Podsumowanie** Przejrzyj konfigurację maszyny wirtualnej, a następnie wybierz pozycję **OK** , aby rozpocząć wdrażanie.</span><span class="sxs-lookup"><span data-stu-id="31cd3-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Podsumowanie konfiguracji w portalu użytkowników w Azure Stack Hub](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="31cd3-188">Utworzenie nowej maszyny wirtualnej zajmuje trochę czasu.</span><span class="sxs-lookup"><span data-stu-id="31cd3-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="31cd3-189">STAN maszyn wirtualnych można wyświetlić na **maszynach wirtualnych**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Stan maszyn wirtualnych w portalu użytkowników w Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="31cd3-191">Tworzenie aplikacji sieci Web na platformie Azure i w centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="31cd3-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="31cd3-192">Azure App Service upraszcza uruchamianie aplikacji sieci Web i zarządzanie nią.</span><span class="sxs-lookup"><span data-stu-id="31cd3-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="31cd3-193">Ponieważ Azure Stack Hub jest spójna z platformą Azure, App Service może działać w obu środowiskach.</span><span class="sxs-lookup"><span data-stu-id="31cd3-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="31cd3-194">Będziesz używać App Service do hostowania aplikacji.</span><span class="sxs-lookup"><span data-stu-id="31cd3-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="31cd3-195">Tworzenie aplikacji sieci Web</span><span class="sxs-lookup"><span data-stu-id="31cd3-195">Create web apps</span></span>

1. <span data-ttu-id="31cd3-196">Utwórz aplikację sieci Web na platformie Azure, postępując zgodnie z instrukcjami w temacie [Zarządzanie planem App Service na platformie Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="31cd3-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="31cd3-197">Upewnij się, że aplikacja sieci Web została umieszczona w tej samej subskrypcji i grupie zasobów, co w sieci hybrydowej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="31cd3-198">Powtórz poprzedni krok (1) w Azure Stack centrum.</span><span class="sxs-lookup"><span data-stu-id="31cd3-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="31cd3-199">Dodawanie trasy dla centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="31cd3-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="31cd3-200">Aby umożliwić użytkownikom dostęp do aplikacji, App Service na Azure Stack Hub muszą być trasowane z publicznej sieci Internet.</span><span class="sxs-lookup"><span data-stu-id="31cd3-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="31cd3-201">Jeśli centrum Azure Stack jest dostępne z Internetu, zanotuj publiczny adres IP lub adres URL dla aplikacji sieci Web Centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="31cd3-202">Jeśli używasz ASDK, możesz [skonfigurować statyczne mapowanie NAT](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) , aby uwidocznić App Service poza środowiskiem wirtualnym.</span><span class="sxs-lookup"><span data-stu-id="31cd3-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="31cd3-203">Łączenie aplikacji sieci Web na platformie Azure z siecią hybrydową</span><span class="sxs-lookup"><span data-stu-id="31cd3-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="31cd3-204">Aby zapewnić łączność między frontonem sieci Web na platformie Azure a SQL Server bazą danych w centrum Azure Stack, aplikacja sieci Web musi być połączona z siecią hybrydową między platformą Azure i usługą Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="31cd3-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="31cd3-205">Aby włączyć łączność, należy:</span><span class="sxs-lookup"><span data-stu-id="31cd3-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="31cd3-206">Konfigurowanie połączenia punkt-lokacja.</span><span class="sxs-lookup"><span data-stu-id="31cd3-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="31cd3-207">Skonfiguruj aplikację internetową.</span><span class="sxs-lookup"><span data-stu-id="31cd3-207">Configure the web app.</span></span>
- <span data-ttu-id="31cd3-208">Zmodyfikuj bramę sieci lokalnej w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="31cd3-209">Konfigurowanie sieci wirtualnej platformy Azure na potrzeby połączeń punkt-lokacja</span><span class="sxs-lookup"><span data-stu-id="31cd3-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="31cd3-210">Brama sieci wirtualnej na stronie platformy Azure w sieci hybrydowej musi zezwalać na połączenia typu punkt-lokacja z usługą Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="31cd3-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="31cd3-211">W Azure Portal przejdź do strony bramy sieci wirtualnej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-211">In the Azure portal, go to the virtual network gateway page.</span></span> <span data-ttu-id="31cd3-212">W obszarze **Ustawienia** wybierz pozycję **Konfiguracja punktu do lokacji**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Opcja punkt-lokacja w bramie sieci wirtualnej platformy Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="31cd3-214">Wybierz pozycję **Konfiguruj teraz** , aby skonfigurować pozycję punkt-lokacja.</span><span class="sxs-lookup"><span data-stu-id="31cd3-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Konfiguracja punktu do lokacji w bramie sieci wirtualnej platformy Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="31cd3-216">Na stronie Konfiguracja **punktu do lokacji** Wprowadź zakres prywatnych adresów IP, który ma być używany w **puli adresów**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="31cd3-217">Upewnij się, że określony zakres nie pokrywa się z żadnym z zakresów adresów używanych już przez podsieci w globalnych składnikach platformy Azure lub Azure Stack Centrum sieci hybrydowej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="31cd3-218">W obszarze **Typ tunelu** Usuń zaznaczenie **sieci VPN protokołu IKEv2**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="31cd3-219">Wybierz pozycję **Zapisz** , aby zakończyć konfigurowanie punktu do lokacji.</span><span class="sxs-lookup"><span data-stu-id="31cd3-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Ustawienia punkt-lokacja w bramie sieci wirtualnej platformy Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="31cd3-221">Integrowanie aplikacji Azure App Service z siecią hybrydową</span><span class="sxs-lookup"><span data-stu-id="31cd3-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="31cd3-222">Aby połączyć aplikację z siecią wirtualną platformy Azure, postępuj zgodnie z instrukcjami w temacie [wymagana integracja](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)z siecią wirtualną.</span><span class="sxs-lookup"><span data-stu-id="31cd3-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="31cd3-223">Przejdź do pozycji **Ustawienia** dla planu App Service hostowania aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="31cd3-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="31cd3-224">W obszarze **Ustawienia** wybierz pozycję **Sieć**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-224">In **Settings**, select **Networking**.</span></span>

    ![Konfigurowanie sieci dla planu App Service](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="31cd3-226">W **integracja z siecią wirtualną** wybierz **pozycję kliknij tutaj, aby zarządzać**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Zarządzanie integracją sieci wirtualnej w ramach planu App Service](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="31cd3-228">Wybierz sieć wirtualną, którą chcesz skonfigurować.</span><span class="sxs-lookup"><span data-stu-id="31cd3-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="31cd3-229">W obszarze **adresy IP kierowane do sieci wirtualnej** Wprowadź zakres adresów IP dla sieci wirtualnej platformy Azure, sieć wirtualną Azure Stack Hub i przestrzenie adresowe punkt-lokacja.</span><span class="sxs-lookup"><span data-stu-id="31cd3-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="31cd3-230">Wybierz pozycję **Zapisz** , aby sprawdzić poprawność i zapisać te ustawienia.</span><span class="sxs-lookup"><span data-stu-id="31cd3-230">Select **Save** to validate and save these settings.</span></span>

    ![Zakresy adresów IP do rozesłania w ramach integracji Virtual Network](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="31cd3-232">Aby dowiedzieć się więcej o tym, jak App Service integrują się z usługą Azure sieci wirtualnych, zobacz [Integrowanie aplikacji z usługą azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="31cd3-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="31cd3-233">Konfigurowanie sieci wirtualnej Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="31cd3-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="31cd3-234">Bramę sieci lokalnej w sieci wirtualnej Azure Stack Hub należy skonfigurować do kierowania ruchem z zakresu adresów App Service punkt-lokacja.</span><span class="sxs-lookup"><span data-stu-id="31cd3-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="31cd3-235">W portalu Azure Stack Hub przejdź do pozycji **Brama sieci lokalnej**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-235">In the Azure Stack Hub portal, go to **Local network gateway**.</span></span> <span data-ttu-id="31cd3-236">W obszarze **Ustawienia** wybierz pozycję **Konfiguracja**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-236">Under **Settings**, select **Configuration**.</span></span>

    ![Opcja konfiguracji bramy w bramie sieci lokalnej centrum Azure Stack](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="31cd3-238">W **obszarze przestrzeń adresowa** Wprowadź zakres adresów punkt-lokacja dla bramy sieci wirtualnej na platformie Azure.</span><span class="sxs-lookup"><span data-stu-id="31cd3-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Przestrzeń adresowa punkt-lokacja w bramie sieci lokalnej centrum Azure Stack](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="31cd3-240">Wybierz pozycję **Zapisz** , aby sprawdzić poprawność i zapisać konfigurację.</span><span class="sxs-lookup"><span data-stu-id="31cd3-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="31cd3-241">Konfigurowanie systemu DNS do skalowania między chmurami</span><span class="sxs-lookup"><span data-stu-id="31cd3-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="31cd3-242">Po poprawnym skonfigurowaniu usługi DNS dla aplikacji w chmurze użytkownicy mogą uzyskiwać dostęp do globalnych wystąpień platformy Azure i Azure Stack w aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="31cd3-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="31cd3-243">Konfiguracja systemu DNS w tym samouczku umożliwia również usłudze Azure Traffic Manager kierowanie ruchu w przypadku zwiększenia lub zmniejszenia obciążenia.</span><span class="sxs-lookup"><span data-stu-id="31cd3-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="31cd3-244">Ten samouczek używa Azure DNS do zarządzania systemem DNS, ponieważ domeny App Service nie będą działały.</span><span class="sxs-lookup"><span data-stu-id="31cd3-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="31cd3-245">Tworzenie poddomen</span><span class="sxs-lookup"><span data-stu-id="31cd3-245">Create subdomains</span></span>

<span data-ttu-id="31cd3-246">Ponieważ Traffic Manager opiera się na rekordach CNAME systemu DNS, poddomena jest wymagana do prawidłowego kierowania ruchu do punktów końcowych.</span><span class="sxs-lookup"><span data-stu-id="31cd3-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="31cd3-247">Aby uzyskać więcej informacji na temat rekordów DNS i mapowania domen, zobacz [Mapowanie domen za pomocą Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="31cd3-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="31cd3-248">W przypadku punktu końcowego platformy Azure utworzysz poddomenę, za pomocą której użytkownicy mogą uzyskiwać dostęp do aplikacji sieci Web.</span><span class="sxs-lookup"><span data-stu-id="31cd3-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="31cd3-249">W tym samouczku można używać **App.Northwind.com**, ale należy dostosować tę wartość na podstawie własnej domeny.</span><span class="sxs-lookup"><span data-stu-id="31cd3-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="31cd3-250">Należy również utworzyć poddomenę z rekordem dla punktu końcowego Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="31cd3-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="31cd3-251">Możesz użyć **azurestack.Northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="31cd3-252">Konfigurowanie domeny niestandardowej na platformie Azure</span><span class="sxs-lookup"><span data-stu-id="31cd3-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="31cd3-253">Dodaj nazwę hosta **App.Northwind.com** do aplikacji sieci Web platformy Azure, [mapując rekord CNAME na Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="31cd3-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="31cd3-254">Konfigurowanie domen niestandardowych w centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="31cd3-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="31cd3-255">Dodaj nazwę hosta **azurestack.Northwind.com** do aplikacji sieci web centrum Azure Stack, [mapując rekord A na Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="31cd3-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="31cd3-256">Użyj adresu IP z obsługą routingu internetowego dla aplikacji App Service.</span><span class="sxs-lookup"><span data-stu-id="31cd3-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="31cd3-257">Dodaj nazwę hosta **App.Northwind.com** do aplikacji sieci web centrum Azure Stack, [mapując rekord CNAME na Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="31cd3-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="31cd3-258">Użyj nazwy hosta skonfigurowanej w poprzednim kroku (1) jako elementu docelowego rekordu CNAME.</span><span class="sxs-lookup"><span data-stu-id="31cd3-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="31cd3-259">Konfigurowanie certyfikatów SSL na potrzeby skalowania między chmurami</span><span class="sxs-lookup"><span data-stu-id="31cd3-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="31cd3-260">Ważne jest, aby zapewnić, że poufne dane zbierane przez aplikację sieci Web są zabezpieczane podczas przesyłania do i przechowywane w bazie danych SQL.</span><span class="sxs-lookup"><span data-stu-id="31cd3-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="31cd3-261">Skonfigurujesz aplikacje sieci Web na platformie Azure i Azure Stack, aby używać certyfikatów SSL dla całego ruchu przychodzącego.</span><span class="sxs-lookup"><span data-stu-id="31cd3-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="31cd3-262">Dodawanie protokołu SSL do platformy Azure i usługi Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="31cd3-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="31cd3-263">Aby dodać protokół SSL do platformy Azure:</span><span class="sxs-lookup"><span data-stu-id="31cd3-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="31cd3-264">Upewnij się, że pobrany certyfikat SSL jest prawidłowy dla utworzonej domeny podrzędnej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="31cd3-265">(Można używać certyfikatów symboli wieloznacznych).</span><span class="sxs-lookup"><span data-stu-id="31cd3-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="31cd3-266">W Azure Portal postępuj zgodnie z instrukcjami podanymi w sekcji **przygotowanie aplikacji sieci Web** i **POwiązaniu certyfikatu SSL** w artykule [Powiązywanie istniejącego niestandardowego certyfikatu protokołu SSL z usługą Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) .</span><span class="sxs-lookup"><span data-stu-id="31cd3-266">In the Azure portal, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="31cd3-267">Wybierz **protokół SSL oparty na SNI** jako **Typ protokołu SSL**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="31cd3-268">Przekieruj cały ruch do portu HTTPS.</span><span class="sxs-lookup"><span data-stu-id="31cd3-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="31cd3-269">Postępuj zgodnie z instrukcjami w sekcji   **Wymuszanie protokołu HTTPS** w artykule [Powiązywanie istniejącego niestandardowego certyfikatu protokołu SSL z usługą Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) .</span><span class="sxs-lookup"><span data-stu-id="31cd3-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="31cd3-270">Aby dodać protokół SSL do Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="31cd3-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="31cd3-271">Powtórz kroki 1-3, które były używane w przypadku platformy Azure, przy użyciu portalu Centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-271">Repeat steps 1-3 that you used for Azure, using the Azure Stack Hub portal.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="31cd3-272">Konfigurowanie i wdrażanie aplikacji internetowej</span><span class="sxs-lookup"><span data-stu-id="31cd3-272">Configure and deploy the web app</span></span>

<span data-ttu-id="31cd3-273">Skonfigurujesz kod aplikacji w taki sposób, aby zgłaszał dane telemetryczne do poprawnego wystąpienia Application Insights i skonfigurować aplikacje sieci Web przy użyciu właściwych parametrów połączenia.</span><span class="sxs-lookup"><span data-stu-id="31cd3-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="31cd3-274">Aby dowiedzieć się więcej na temat Application Insights, zobacz [co to jest Application Insights?](/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="31cd3-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="31cd3-275">Dodaj Application Insights</span><span class="sxs-lookup"><span data-stu-id="31cd3-275">Add Application Insights</span></span>

1. <span data-ttu-id="31cd3-276">Otwórz aplikację sieci Web w Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="31cd3-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="31cd3-277">[Dodaj Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) do projektu, aby przesłać dane telemetryczne, których Application Insights używa do tworzenia alertów w przypadku wzrostu lub zmniejszenia ruchu w sieci Web.</span><span class="sxs-lookup"><span data-stu-id="31cd3-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="31cd3-278">Konfigurowanie dynamicznych parametrów połączenia</span><span class="sxs-lookup"><span data-stu-id="31cd3-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="31cd3-279">Każde wystąpienie aplikacji sieci Web będzie używać innej metody do łączenia się z bazą danych SQL.</span><span class="sxs-lookup"><span data-stu-id="31cd3-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="31cd3-280">Aplikacja na platformie Azure używa prywatnego adresu IP maszyny wirtualnej SQL Server, a aplikacja w centrum Azure Stack używa publicznego adresu IP SQL Server maszyny wirtualnej.</span><span class="sxs-lookup"><span data-stu-id="31cd3-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="31cd3-281">W systemie zintegrowanym z centrum Azure Stack publiczny adres IP nie powinien być obsługiwany przez Internet.</span><span class="sxs-lookup"><span data-stu-id="31cd3-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="31cd3-282">Na ASDK publiczny adres IP nie jest w trakcie routingu poza ASDK.</span><span class="sxs-lookup"><span data-stu-id="31cd3-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="31cd3-283">Za pomocą zmiennych środowiskowych App Service można przekazać inne parametry połączenia do każdego wystąpienia aplikacji.</span><span class="sxs-lookup"><span data-stu-id="31cd3-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="31cd3-284">Otwórz aplikację w programie Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="31cd3-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="31cd3-285">Otwórz Start. cs i znajdź następujący blok kodu:</span><span class="sxs-lookup"><span data-stu-id="31cd3-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="31cd3-286">Zastąp poprzedni blok kodu poniższym kodem, który używa parametrów połączenia zdefiniowanych w *appsettings.jsw* pliku:</span><span class="sxs-lookup"><span data-stu-id="31cd3-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="31cd3-287">Konfigurowanie ustawień aplikacji App Service</span><span class="sxs-lookup"><span data-stu-id="31cd3-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="31cd3-288">Utwórz parametry połączenia dla systemu Azure i centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="31cd3-289">Ciągi powinny być takie same, z wyjątkiem adresów IP, które są używane.</span><span class="sxs-lookup"><span data-stu-id="31cd3-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="31cd3-290">Na platformie Azure i Azure Stack Hub Dodaj odpowiednie parametry połączenia [jako ustawienia aplikacji](/azure/app-service/web-sites-configure) w aplikacji sieci Web, używając `SQLCONNSTR\_` jako prefiksu w nazwie.</span><span class="sxs-lookup"><span data-stu-id="31cd3-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="31cd3-291">**Zapisz** ustawienia aplikacji sieci Web i ponownie uruchom aplikację.</span><span class="sxs-lookup"><span data-stu-id="31cd3-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="31cd3-292">Włącz automatyczne skalowanie na globalnym platformie Azure</span><span class="sxs-lookup"><span data-stu-id="31cd3-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="31cd3-293">Gdy tworzysz aplikację sieci Web w środowisku App Service, rozpocznie się ono z jednym wystąpieniem.</span><span class="sxs-lookup"><span data-stu-id="31cd3-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="31cd3-294">Możesz automatycznie skalować w poziomie, aby dodać wystąpienia w celu zapewnienia większej ilości zasobów obliczeniowych dla aplikacji.</span><span class="sxs-lookup"><span data-stu-id="31cd3-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="31cd3-295">Podobnie można automatycznie skalować i zmniejszać liczbę wystąpień potrzebnych aplikacji.</span><span class="sxs-lookup"><span data-stu-id="31cd3-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="31cd3-296">Musisz mieć App Service plan, aby skonfigurować skalowanie w poziomie i w poziomie.</span><span class="sxs-lookup"><span data-stu-id="31cd3-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="31cd3-297">Jeśli nie masz planu, utwórz go przed rozpoczęciem następnych kroków.</span><span class="sxs-lookup"><span data-stu-id="31cd3-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="31cd3-298">Włącz automatyczne skalowanie w poziomie</span><span class="sxs-lookup"><span data-stu-id="31cd3-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="31cd3-299">W Azure Portal Znajdź plan App Service dla witryn, które mają być skalowane w poziomie, a następnie wybierz pozycję **skalowanie w poziomie (plan App Service)**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-299">In the Azure portal, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Skalowanie w poziomie Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="31cd3-301">Wybierz pozycję **Włącz automatyczne skalowanie**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-301">Select **Enable autoscale**.</span></span>

    ![Włącz automatyczne skalowanie w Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="31cd3-303">Wprowadź nazwę **nazwy ustawienia skalowania automatycznego**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="31cd3-304">Dla **domyślnej** reguły automatycznego skalowania wybierz pozycję **skalowanie na podstawie metryki**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="31cd3-305">Ustaw **limity wystąpienia** na wartość **minimum: 1**, **maksimum: 10**, a **wartość domyślna: 1**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Konfigurowanie automatycznego skalowania w Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="31cd3-307">Wybierz pozycję **+ Dodaj regułę**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="31cd3-308">W polu **Źródło metryk** wybierz pozycję **bieżący zasób**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="31cd3-309">Użyj następujących kryteriów i akcji dla reguły.</span><span class="sxs-lookup"><span data-stu-id="31cd3-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="31cd3-310">Kryteria</span><span class="sxs-lookup"><span data-stu-id="31cd3-310">Criteria</span></span>

1. <span data-ttu-id="31cd3-311">W obszarze **agregacja czasu** wybierz pozycję **średnia**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="31cd3-312">W obszarze **Nazwa metryki** wybierz opcję **procent procesora CPU**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="31cd3-313">W obszarze **operator** wybierz pozycję **większe niż**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="31cd3-314">Ustaw **próg** na **50**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="31cd3-315">Ustaw **czas trwania** na **10**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="31cd3-316">Akcja</span><span class="sxs-lookup"><span data-stu-id="31cd3-316">Action</span></span>

1. <span data-ttu-id="31cd3-317">W obszarze **operacja** wybierz pozycję **Zwiększ liczbę według**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="31cd3-318">Ustaw **liczbę wystąpień** na **2**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="31cd3-319">Ustaw **chłodną** wartość na **5**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="31cd3-320">Wybierz pozycję **Dodaj**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-320">Select **Add**.</span></span>

5. <span data-ttu-id="31cd3-321">Wybierz pozycję **+ Dodaj regułę**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="31cd3-322">W polu **Źródło metryk** wybierz pozycję **bieżący zasób.**</span><span class="sxs-lookup"><span data-stu-id="31cd3-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="31cd3-323">Bieżący zasób będzie zawierać nazwę/identyfikator GUID planu App Service i lista rozwijana **Typ zasobu** i **zasób** będzie niedostępna.</span><span class="sxs-lookup"><span data-stu-id="31cd3-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="31cd3-324">Włącz automatyczne skalowanie w</span><span class="sxs-lookup"><span data-stu-id="31cd3-324">Enable automatic scale in</span></span>

<span data-ttu-id="31cd3-325">Po zmniejszeniu natężenia ruchu aplikacja internetowa platformy Azure może automatycznie zmniejszyć liczbę aktywnych wystąpień, aby obniżyć koszty.</span><span class="sxs-lookup"><span data-stu-id="31cd3-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="31cd3-326">Ta akcja jest mniej agresywna niż skalowanie w poziomie i minimalizuje wpływ na użytkowników aplikacji.</span><span class="sxs-lookup"><span data-stu-id="31cd3-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="31cd3-327">Przejdź do **domyślnego** warunku skalowania w poziomie, a następnie wybierz pozycję **+ Dodaj regułę**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="31cd3-328">Użyj następujących kryteriów i akcji dla reguły.</span><span class="sxs-lookup"><span data-stu-id="31cd3-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="31cd3-329">Kryteria</span><span class="sxs-lookup"><span data-stu-id="31cd3-329">Criteria</span></span>

1. <span data-ttu-id="31cd3-330">W obszarze **agregacja czasu** wybierz pozycję **średnia**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="31cd3-331">W obszarze **Nazwa metryki** wybierz opcję **procent procesora CPU**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="31cd3-332">W obszarze **operator** wybierz pozycję **mniejsze niż**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="31cd3-333">Ustaw **próg** na wartość **30**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="31cd3-334">Ustaw **czas trwania** na **10**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="31cd3-335">Akcja</span><span class="sxs-lookup"><span data-stu-id="31cd3-335">Action</span></span>

1. <span data-ttu-id="31cd3-336">W obszarze **operacja** wybierz pozycję **Zmniejsz liczbę według**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="31cd3-337">Ustaw **liczbę wystąpień** na **1**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="31cd3-338">Ustaw **chłodną** wartość na **5**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="31cd3-339">Wybierz pozycję **Dodaj**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="31cd3-340">Utwórz profil Traffic Manager i skonfiguruj skalowanie między chmurami</span><span class="sxs-lookup"><span data-stu-id="31cd3-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="31cd3-341">Utwórz profil Traffic Manager przy użyciu Azure Portal, a następnie skonfiguruj punkty końcowe, aby umożliwić skalowanie między chmurami.</span><span class="sxs-lookup"><span data-stu-id="31cd3-341">Create a Traffic Manager profile using the Azure portal, then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="31cd3-342">Utwórz profil Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="31cd3-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="31cd3-343">Wybierz pozycję **Utwórz zasób**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="31cd3-344">Wybierz pozycję **Sieć**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-344">Select **Networking**.</span></span>
3. <span data-ttu-id="31cd3-345">Wybierz **profil Traffic Manager** i skonfiguruj następujące ustawienia:</span><span class="sxs-lookup"><span data-stu-id="31cd3-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="31cd3-346">W polu **Nazwa** wprowadź nazwę profilu.</span><span class="sxs-lookup"><span data-stu-id="31cd3-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="31cd3-347">Ta nazwa **musi** być unikatowa w strefie trafficmanager.NET i służy do tworzenia nowej nazwy DNS (na przykład northwindstore.trafficmanager.NET).</span><span class="sxs-lookup"><span data-stu-id="31cd3-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="31cd3-348">W polu **Metoda routingu** wybierz opcję **ważone**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="31cd3-349">W obszarze **subskrypcja** wybierz subskrypcję, w której chcesz utworzyć ten profil.</span><span class="sxs-lookup"><span data-stu-id="31cd3-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="31cd3-350">W obszarze **Grupa zasobów** Utwórz nową grupę zasobów dla tego profilu.</span><span class="sxs-lookup"><span data-stu-id="31cd3-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="31cd3-351">W obszarze **Lokalizacja grupy zasobów** wybierz lokalizację grupy zasobów.</span><span class="sxs-lookup"><span data-stu-id="31cd3-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="31cd3-352">To ustawienie dotyczy lokalizacji grupy zasobów i nie ma wpływu na profil Traffic Manager, który został wdrożony globalnie.</span><span class="sxs-lookup"><span data-stu-id="31cd3-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="31cd3-353">Wybierz przycisk **Utwórz**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-353">Select **Create**.</span></span>

    ![Utwórz profil Traffic Manager](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="31cd3-355">Po ukończeniu globalnego wdrożenia profilu Traffic Manager zostanie on wyświetlony na liście zasobów dla grupy zasobów, w której został utworzony.</span><span class="sxs-lookup"><span data-stu-id="31cd3-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="31cd3-356">Dodawanie punktów końcowych usługi Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="31cd3-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="31cd3-357">Wyszukaj utworzony profil Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="31cd3-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="31cd3-358">Jeśli przejdziesz do grupy zasobów profilu, wybierz profil.</span><span class="sxs-lookup"><span data-stu-id="31cd3-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="31cd3-359">W oknie **profil Traffic Manager** w obszarze **Ustawienia** wybierz pozycję **punkty końcowe**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="31cd3-360">Wybierz pozycję **Dodaj**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-360">Select **Add**.</span></span>

4. <span data-ttu-id="31cd3-361">W obszarze **Dodaj punkt końcowy** Użyj następujących ustawień centrum Azure Stack:</span><span class="sxs-lookup"><span data-stu-id="31cd3-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="31cd3-362">W obszarze **Typ** wybierz pozycję **zewnętrzny punkt końcowy**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="31cd3-363">Wprowadź **nazwę** punktu końcowego.</span><span class="sxs-lookup"><span data-stu-id="31cd3-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="31cd3-364">W polu w **pełni kwalifikowana nazwa domeny (FQDN) lub adres IP** wprowadź zewnętrzny adres URL aplikacji sieci Web Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="31cd3-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="31cd3-365">W polu **waga** pozostaw wartość domyślną **1**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="31cd3-366">Ta waga powoduje, że cały ruch przechodzi do tego punktu końcowego, jeśli jest w dobrej kondycji.</span><span class="sxs-lookup"><span data-stu-id="31cd3-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="31cd3-367">Pozostaw pole wyboru **Dodaj jako wyłączone wyłączona** .</span><span class="sxs-lookup"><span data-stu-id="31cd3-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="31cd3-368">Wybierz **przycisk OK** , aby zapisać punkt końcowy Azure Stack centrum.</span><span class="sxs-lookup"><span data-stu-id="31cd3-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="31cd3-369">Następnie skonfigurujesz punkt końcowy platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="31cd3-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="31cd3-370">W obszarze **profil Traffic Manager** wybierz pozycję **punkty końcowe**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="31cd3-371">Wybierz pozycję **+Dodaj**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-371">Select **+Add**.</span></span>
3. <span data-ttu-id="31cd3-372">Na stronie **Dodawanie punktu końcowego** Użyj następujących ustawień platformy Azure:</span><span class="sxs-lookup"><span data-stu-id="31cd3-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="31cd3-373">W obszarze **Typ** wybierz pozycję **punkt końcowy platformy Azure**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="31cd3-374">Wprowadź **nazwę** punktu końcowego.</span><span class="sxs-lookup"><span data-stu-id="31cd3-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="31cd3-375">W obszarze **Typ zasobu docelowego** wybierz pozycję **App Service**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="31cd3-376">W polu **zasób docelowy** wybierz pozycję **Wybierz usługę App Service** , aby wyświetlić listę Web Apps w tej samej subskrypcji.</span><span class="sxs-lookup"><span data-stu-id="31cd3-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="31cd3-377">W obszarze **Zasoby** wybierz usługę aplikacji, którą chcesz dodać jako pierwszy punkt końcowy.</span><span class="sxs-lookup"><span data-stu-id="31cd3-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="31cd3-378">W obszarze **waga** wybierz pozycję **2**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="31cd3-379">To ustawienie powoduje, że cały ruch przechodzi do tego punktu końcowego, jeśli podstawowy punkt końcowy ma złą kondycję lub jeśli masz regułę/alert przekierowuje ruch po wyzwoleniu.</span><span class="sxs-lookup"><span data-stu-id="31cd3-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="31cd3-380">Pozostaw pole wyboru **Dodaj jako wyłączone wyłączona** .</span><span class="sxs-lookup"><span data-stu-id="31cd3-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="31cd3-381">Wybierz **przycisk OK** , aby zapisać punkt końcowy platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="31cd3-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="31cd3-382">Po skonfigurowaniu obu punktów końcowych są one wyświetlane w **Traffic Manager profilu** w przypadku wybrania **punktów końcowych**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="31cd3-383">W przykładzie na poniższym zrzucie ekranu przedstawiono dwa punkty końcowe, z informacjami o stanie i konfiguracją każdego z nich.</span><span class="sxs-lookup"><span data-stu-id="31cd3-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Punkty końcowe w profilu Traffic Manager](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a><span data-ttu-id="31cd3-385">Konfigurowanie Application Insights monitorowania i alertów na platformie Azure</span><span class="sxs-lookup"><span data-stu-id="31cd3-385">Set up Application Insights monitoring and alerting in Azure</span></span>

<span data-ttu-id="31cd3-386">Usługa Azure Application Insights umożliwia monitorowanie aplikacji i wysyłanie alertów na podstawie skonfigurowanych warunków.</span><span class="sxs-lookup"><span data-stu-id="31cd3-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="31cd3-387">Oto kilka przykładów: aplikacja jest niedostępna, występuje błędy lub pokazuje problemy z wydajnością.</span><span class="sxs-lookup"><span data-stu-id="31cd3-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="31cd3-388">Do tworzenia alertów służą metryki usługi Azure Application Insights.</span><span class="sxs-lookup"><span data-stu-id="31cd3-388">You'll use Azure Application Insights metrics to create alerts.</span></span> <span data-ttu-id="31cd3-389">Gdy te alerty wyzwalają, wystąpienie aplikacji sieci Web automatycznie przejdzie z centrum Azure Stack na platformę Azure w celu skalowania w poziomie, a następnie z powrotem do centrum Azure Stack w celu skalowania w poziomie.</span><span class="sxs-lookup"><span data-stu-id="31cd3-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="31cd3-390">Tworzenie alertu na podstawie metryki</span><span class="sxs-lookup"><span data-stu-id="31cd3-390">Create an alert from metrics</span></span>

<span data-ttu-id="31cd3-391">W Azure Portal przejdź do grupy zasobów w tym samouczku i wybierz wystąpienie Application Insights, aby otworzyć **Application Insights**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-391">In the Azure portal, go to the resource group for this tutorial, and select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="31cd3-393">Ten widok służy do tworzenia alertu skalowania w poziomie i alertu dotyczącego skalowania w poziomie.</span><span class="sxs-lookup"><span data-stu-id="31cd3-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="31cd3-394">Tworzenie alertu skalowania w poziomie</span><span class="sxs-lookup"><span data-stu-id="31cd3-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="31cd3-395">W obszarze **Konfiguracja** wybierz pozycję **alerty (klasyczne)**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="31cd3-396">Wybierz pozycję **Dodaj alert dotyczący metryki (klasyczny)**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="31cd3-397">W obszarze **Dodaj regułę** skonfiguruj następujące ustawienia:</span><span class="sxs-lookup"><span data-stu-id="31cd3-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="31cd3-398">W obszarze **Nazwa** wprowadź polecenie przebicie **na chmurę Azure**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="31cd3-399">**Opis** jest opcjonalny.</span><span class="sxs-lookup"><span data-stu-id="31cd3-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="31cd3-400">W obszarze alert **źródłowy**  >  **na** wybierz pozycję **metryki**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="31cd3-401">W obszarze **kryteria** wybierz swoją subskrypcję, grupę zasobów dla profilu Traffic Manager i nazwę profilu Traffic Manager dla zasobu.</span><span class="sxs-lookup"><span data-stu-id="31cd3-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="31cd3-402">W obszarze **Metryka** wybierz pozycję **Liczba żądań**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="31cd3-403">W obszarze **warunek** wybierz opcję **większe niż**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="31cd3-404">W obszarze **próg wprowadź wartość** **2**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="31cd3-405">W polu **okres** wybierz pozycję **w ciągu ostatnich 5 minut**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="31cd3-406">W obszarze **Powiadamiaj za pośrednictwem**:</span><span class="sxs-lookup"><span data-stu-id="31cd3-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="31cd3-407">Zaznacz pole wyboru dla **właścicieli, współautorów i czytelników poczty e-mail**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="31cd3-408">Wprowadź swój adres e-mail, aby uzyskać **dodatkowe adresy e-mail administratora**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="31cd3-409">Na pasku menu wybierz pozycję **Zapisz**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="31cd3-410">Tworzenie alertu dotyczącego skalowania w poziomie</span><span class="sxs-lookup"><span data-stu-id="31cd3-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="31cd3-411">W obszarze **Konfiguracja** wybierz pozycję **alerty (klasyczne)**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="31cd3-412">Wybierz pozycję **Dodaj alert dotyczący metryki (klasyczny)**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="31cd3-413">W obszarze **Dodaj regułę** skonfiguruj następujące ustawienia:</span><span class="sxs-lookup"><span data-stu-id="31cd3-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="31cd3-414">W obszarze **Nazwa** wprowadź **skalowanie z powrotem do Azure Stack Hub**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="31cd3-415">**Opis** jest opcjonalny.</span><span class="sxs-lookup"><span data-stu-id="31cd3-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="31cd3-416">W obszarze alert **źródłowy**  >  **na** wybierz pozycję **metryki**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="31cd3-417">W obszarze **kryteria** wybierz swoją subskrypcję, grupę zasobów dla profilu Traffic Manager i nazwę profilu Traffic Manager dla zasobu.</span><span class="sxs-lookup"><span data-stu-id="31cd3-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="31cd3-418">W obszarze **Metryka** wybierz pozycję **Liczba żądań**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="31cd3-419">W obszarze **warunek** wybierz pozycję **mniejsze niż**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="31cd3-420">W obszarze **próg wprowadź wartość** **2**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="31cd3-421">W polu **okres** wybierz pozycję **w ciągu ostatnich 5 minut**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="31cd3-422">W obszarze **Powiadamiaj za pośrednictwem**:</span><span class="sxs-lookup"><span data-stu-id="31cd3-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="31cd3-423">Zaznacz pole wyboru dla **właścicieli, współautorów i czytelników poczty e-mail**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="31cd3-424">Wprowadź swój adres e-mail, aby uzyskać **dodatkowe adresy e-mail administratora**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="31cd3-425">Na pasku menu wybierz pozycję **Zapisz**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="31cd3-426">Poniższy zrzut ekranu przedstawia alerty dotyczące skalowania w poziomie i skalowania w poziomie.</span><span class="sxs-lookup"><span data-stu-id="31cd3-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Alerty Application Insights (klasyczne)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="31cd3-428">Przekierowywanie ruchu między platformą Azure i usługą Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="31cd3-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="31cd3-429">Można skonfigurować ręczne lub automatyczne przełączanie ruchu aplikacji sieci Web między platformą Azure i Azure Stack centrum.</span><span class="sxs-lookup"><span data-stu-id="31cd3-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="31cd3-430">Konfigurowanie ręcznego przełączania między platformą Azure i usługą Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="31cd3-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="31cd3-431">Gdy witryna sieci Web osiągnie skonfigurowane progi, otrzymasz alert.</span><span class="sxs-lookup"><span data-stu-id="31cd3-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="31cd3-432">Wykonaj następujące kroki, aby ręcznie przekierować ruch do platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="31cd3-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="31cd3-433">W Azure Portal wybierz profil Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="31cd3-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Traffic Manager punkty końcowe w Azure Portal](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="31cd3-435">Wybierz **punkty końcowe**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="31cd3-436">Wybierz **punkt końcowy platformy Azure**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="31cd3-437">W obszarze **stan** wybierz pozycję **włączone**, a następnie wybierz pozycję **Zapisz**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Włącz punkt końcowy platformy Azure w Azure Portal](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="31cd3-439">W obszarze **punkty końcowe** dla profilu Traffic Manager wybierz pozycję **zewnętrzny punkt końcowy**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="31cd3-440">W obszarze **stan** wybierz pozycję **wyłączone**, a następnie wybierz pozycję **Zapisz**.</span><span class="sxs-lookup"><span data-stu-id="31cd3-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Wyłącz punkt końcowy Azure Stack Hub w programie Azure Portal](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="31cd3-442">Po skonfigurowaniu punktów końcowych ruch aplikacji przechodzi do aplikacji sieci Web platformy Azure w poziomie, a nie aplikacji sieci Web Centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="31cd3-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Punkty końcowe zostały zmienione w ruchu aplikacji sieci Web platformy Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="31cd3-444">Aby odwrócić przepływ z powrotem do Azure Stack Hub, wykonaj poprzednie kroki, aby:</span><span class="sxs-lookup"><span data-stu-id="31cd3-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="31cd3-445">Włącz punkt końcowy Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="31cd3-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="31cd3-446">Wyłącz punkt końcowy platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="31cd3-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="31cd3-447">Konfigurowanie automatycznego przełączania między platformą Azure i usługą Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="31cd3-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="31cd3-448">Można również użyć monitorowania Application Insights, jeśli aplikacja działa w środowisku [bezserwerowym](https://azure.microsoft.com/overview/serverless-computing/) udostępnianym przez Azure Functions.</span><span class="sxs-lookup"><span data-stu-id="31cd3-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="31cd3-449">W tym scenariuszu można skonfigurować Application Insights do używania elementu webhook, który wywołuje aplikację funkcji.</span><span class="sxs-lookup"><span data-stu-id="31cd3-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="31cd3-450">Ta aplikacja automatycznie włącza lub wyłącza punkt końcowy w odpowiedzi na alert.</span><span class="sxs-lookup"><span data-stu-id="31cd3-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="31cd3-451">Poniższe kroki służą do konfigurowania automatycznego przełączania ruchu.</span><span class="sxs-lookup"><span data-stu-id="31cd3-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="31cd3-452">Tworzenie aplikacji funkcji platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="31cd3-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="31cd3-453">Utwórz funkcję wyzwalaną przez protokół HTTP.</span><span class="sxs-lookup"><span data-stu-id="31cd3-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="31cd3-454">Zaimportuj zestawy SDK platformy Azure dla Menedżer zasobów, Web Apps i Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="31cd3-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="31cd3-455">Rozwijaj kod, aby:</span><span class="sxs-lookup"><span data-stu-id="31cd3-455">Develop code to:</span></span>

   - <span data-ttu-id="31cd3-456">Uwierzytelnianie w ramach subskrypcji platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="31cd3-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="31cd3-457">Użyj parametru, który przełącza Traffic Manager punkty końcowe, aby skierować ruch do platformy Azure lub Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="31cd3-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="31cd3-458">Zapisz swój kod i Dodaj adres URL aplikacji funkcji z odpowiednimi parametrami do sekcji **elementu webhook** w ustawieniach reguły alertu Application Insights.</span><span class="sxs-lookup"><span data-stu-id="31cd3-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="31cd3-459">Ruch jest automatycznie przekierowywany po uruchomieniu alertu Application Insights.</span><span class="sxs-lookup"><span data-stu-id="31cd3-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="31cd3-460">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="31cd3-460">Next steps</span></span>

- <span data-ttu-id="31cd3-461">Aby dowiedzieć się więcej o wzorcach chmury platformy Azure, zobacz [wzorce projektowe w chmurze](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="31cd3-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
