---
title: Wdrażanie rozwiązania wykrywania FootFall opartego na AI na platformie Azure i w centrum Azure Stack
description: Dowiedz się, jak wdrożyć rozwiązanie FootFall Detection (AI) do analizowania ruchu gościa w sklepach detalicznych przy użyciu platformy Azure i usługi Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6913cc522da447092dad0af24e148a3b2576495c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910897"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="8aa0c-103">Wdrażanie rozwiązania wykrywania FootFall opartego na formacie AI przy użyciu platformy Azure i usługi Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="8aa0c-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="8aa0c-104">W tym artykule opisano sposób wdrażania rozwiązań opartych na formacie AI, które generują szczegółowe informacje z rzeczywistych działań na świecie przy użyciu platformy Azure, Azure Stack Hub i zestawu Custom Vision AI.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="8aa0c-105">W tym rozwiązaniu dowiesz się, jak:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="8aa0c-106">Wdróż zbiory natywnych aplikacji w chmurze (CNAB) na krawędzi.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="8aa0c-107">Wdróż aplikację, która obejmuje granice chmury.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="8aa0c-108">Użyj zestawu SDK Custom Vision AI do wnioskowania na brzegu.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="8aa0c-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="8aa0c-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="8aa0c-110">Microsoft Azure Stack Hub to rozszerzenie platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="8aa0c-111">Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="8aa0c-112">[Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="8aa0c-113">Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="8aa0c-114">Wymagania wstępne</span><span class="sxs-lookup"><span data-stu-id="8aa0c-114">Prerequisites</span></span>

<span data-ttu-id="8aa0c-115">Przed rozpoczęciem pracy z tym przewodnikiem wdrażania upewnij się, że:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="8aa0c-116">Zapoznaj się z tematem dotyczącym [wzorca wykrywania FootFall](pattern-retail-footfall-detection.md) .</span><span class="sxs-lookup"><span data-stu-id="8aa0c-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="8aa0c-117">Uzyskaj dostęp użytkowników do Azure Stack Development Kit (ASDK) lub zintegrowanego wystąpienia systemowego centrum Azure Stack z:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="8aa0c-118">Azure App Service na zainstalowaniu [dostawcy zasobów centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-overview.md) .</span><span class="sxs-lookup"><span data-stu-id="8aa0c-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="8aa0c-119">Potrzebujesz dostępu operatora do wystąpienia centrum Azure Stack lub skontaktuj się z administratorem, aby zainstalować program.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="8aa0c-120">Subskrypcja oferty zapewniającej przydział App Service i magazynu.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="8aa0c-121">Aby utworzyć ofertę, musisz mieć dostęp do operatora.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="8aa0c-122">Uzyskaj dostęp do subskrypcji platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="8aa0c-123">Jeśli nie masz subskrypcji platformy Azure, przed rozpoczęciem Utwórz [konto bezpłatnej wersji próbnej](https://azure.microsoft.com/free/) .</span><span class="sxs-lookup"><span data-stu-id="8aa0c-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="8aa0c-124">Utwórz dwie jednostki usługi w katalogu:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="8aa0c-125">Jeden skonfigurowany do użycia z zasobami platformy Azure z dostępem w zakresie subskrypcji platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="8aa0c-126">Jeden skonfigurowany do użycia z zasobami centrum Azure Stack, z dostępem w zakresie subskrypcji centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="8aa0c-127">Aby dowiedzieć się więcej na temat tworzenia jednostek usługi i autoryzowania dostępu, zobacz [Korzystanie z tożsamości aplikacji w celu uzyskania dostępu do zasobów](/azure-stack/operator/azure-stack-create-service-principals.md).</span><span class="sxs-lookup"><span data-stu-id="8aa0c-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="8aa0c-128">Jeśli wolisz używać interfejsu wiersza polecenia platformy Azure, zobacz [Tworzenie jednostki usługi platformy Azure przy użyciu interfejsu wiersza polecenia platformy Azure](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="8aa0c-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span></span>
- <span data-ttu-id="8aa0c-129">Wdróż usługę Azure Cognitive Services na platformie Azure lub w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="8aa0c-130">Najpierw [Dowiedz się więcej o Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="8aa0c-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="8aa0c-131">Następnie odwiedź stronę [wdrażanie usługi Azure Cognitive Services w usłudze Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) , aby wdrożyć Cognitive Services na Azure Stack centrum.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="8aa0c-132">Najpierw musisz zarejestrować się w celu uzyskania dostępu do wersji zapoznawczej.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="8aa0c-133">Klonowanie lub pobieranie nieskonfigurowanego zestawu Azure Custom Vision AI dev Kit.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="8aa0c-134">Aby uzyskać szczegółowe informacje, zobacz [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="8aa0c-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="8aa0c-135">Utwórz konto Power BI.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="8aa0c-136">Klucz subskrypcji usługi Azure Cognitive Services interfejs API rozpoznawania twarzy i adres URL punktu końcowego.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="8aa0c-137">Możesz skorzystać z usługi [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) bezpłatna wersja próbna.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="8aa0c-138">Lub postępuj zgodnie z instrukcjami w temacie [Tworzenie konta Cognitive Services](/azure/cognitive-services/cognitive-services-apis-create-account).</span><span class="sxs-lookup"><span data-stu-id="8aa0c-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="8aa0c-139">Zainstaluj następujące zasoby programistyczne:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="8aa0c-140">Interfejs wiersza polecenia platformy Azure 2.0</span><span class="sxs-lookup"><span data-stu-id="8aa0c-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="8aa0c-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="8aa0c-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="8aa0c-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="8aa0c-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="8aa0c-143">Za pomocą Porter można wdrażać aplikacje w chmurze przy użyciu dodanych manifestów pakietu CNAB.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="8aa0c-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="8aa0c-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="8aa0c-145">Narzędzia usługi Azure IoT dla Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="8aa0c-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="8aa0c-146">Rozszerzenie języka Python dla Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="8aa0c-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="8aa0c-147">Python</span><span class="sxs-lookup"><span data-stu-id="8aa0c-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="8aa0c-148">Wdrażanie aplikacji w chmurze hybrydowej</span><span class="sxs-lookup"><span data-stu-id="8aa0c-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="8aa0c-149">Najpierw użyj interfejsu wiersza polecenia Porter, aby wygenerować zestaw poświadczeń, a następnie wdróż aplikację w chmurze.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="8aa0c-150">Klonowanie lub Pobieranie przykładowego kodu rozwiązania z https://github.com/azure-samples/azure-intelligent-edge-patterns .</span><span class="sxs-lookup"><span data-stu-id="8aa0c-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="8aa0c-151">Porter wygeneruje zestaw poświadczeń, które będą automatyzować wdrażanie aplikacji.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="8aa0c-152">Przed uruchomieniem polecenia generowania poświadczeń upewnij się, że dostępne są następujące elementy:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="8aa0c-153">Jednostka usługi do uzyskiwania dostępu do zasobów platformy Azure, w tym identyfikatora jednostki usługi, klucza i usługi DNS dzierżawy.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="8aa0c-154">Identyfikator subskrypcji subskrypcji platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="8aa0c-155">Jednostka usługi do uzyskiwania dostępu do zasobów centrum Azure Stack, w tym identyfikatora podmiotu zabezpieczeń, klucza i usługi DNS dzierżawy.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="8aa0c-156">Identyfikator subskrypcji dla subskrypcji centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="8aa0c-157">Adres URL klucza i punktu końcowego usługi Azure Cognitive Services interfejs API rozpoznawania twarzy.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="8aa0c-158">Uruchom proces generowania poświadczeń Porter i postępuj zgodnie z monitami:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="8aa0c-159">Porter wymaga również zestawu parametrów do uruchomienia.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="8aa0c-160">Utwórz plik tekstowy parametrów i wprowadź następujące pary nazwa/wartość.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="8aa0c-161">Skontaktuj się z administratorem centrum Azure Stack, jeśli potrzebujesz pomocy przy dowolnych wymaganych wartościach.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="8aa0c-162">Ta `resource suffix` wartość służy do upewnienia się, że zasoby wdrożenia mają unikatowe nazwy na platformie Azure.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="8aa0c-163">Musi to być unikatowy ciąg liter i cyfr, nie dłuższa niż 8 znaków.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="8aa0c-164">Zapisz plik tekstowy i zanotuj jego ścieżkę.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="8aa0c-165">Teraz możesz przystąpić do wdrażania aplikacji w chmurze hybrydowej za pomocą Porter.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="8aa0c-166">Uruchom polecenie instalacji i obejrzyj, jak zasoby są wdrażane na platformie Azure i w centrum Azure Stack:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="8aa0c-167">Po zakończeniu wdrażania należy pamiętać o następujących wartościach:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="8aa0c-168">Parametry połączenia aparatu.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-168">The camera's connection string.</span></span>
    - <span data-ttu-id="8aa0c-169">Parametry połączenia konta magazynu obrazu.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="8aa0c-170">Nazwy grup zasobów.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="8aa0c-171">Przygotuj Custom Vision AI DevKit</span><span class="sxs-lookup"><span data-stu-id="8aa0c-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="8aa0c-172">Następnie skonfiguruj zestaw Custom Vision AI dev Kit, jak pokazano w [przewodniku szybki start dla programu Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span><span class="sxs-lookup"><span data-stu-id="8aa0c-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="8aa0c-173">Należy również skonfigurować i przetestować aparat przy użyciu parametrów połączenia podanych w poprzednim kroku.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="8aa0c-174">Wdrażanie aplikacji aparatu</span><span class="sxs-lookup"><span data-stu-id="8aa0c-174">Deploy the camera app</span></span>

<span data-ttu-id="8aa0c-175">Użyj interfejsu wiersza polecenia Porter, aby wygenerować zestaw poświadczeń, a następnie wdróż aplikację aparatu.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="8aa0c-176">Porter wygeneruje zestaw poświadczeń, które będą automatyzować wdrażanie aplikacji.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="8aa0c-177">Przed uruchomieniem polecenia generowania poświadczeń upewnij się, że dostępne są następujące elementy:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="8aa0c-178">Jednostka usługi do uzyskiwania dostępu do zasobów platformy Azure, w tym identyfikatora jednostki usługi, klucza i usługi DNS dzierżawy.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="8aa0c-179">Identyfikator subskrypcji subskrypcji platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="8aa0c-180">Parametry połączenia konta magazynu obrazu podane podczas wdrażania aplikacji w chmurze.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="8aa0c-181">Uruchom proces generowania poświadczeń Porter i postępuj zgodnie z monitami:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="8aa0c-182">Porter wymaga również zestawu parametrów do uruchomienia.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="8aa0c-183">Utwórz plik tekstowy parametrów i wprowadź następujący tekst.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="8aa0c-184">Skontaktuj się z administratorem centrum Azure Stack, jeśli nie znasz pewnych wymaganych wartości.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="8aa0c-185">Ta `deployment suffix` wartość służy do upewnienia się, że zasoby wdrożenia mają unikatowe nazwy na platformie Azure.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="8aa0c-186">Musi to być unikatowy ciąg liter i cyfr, nie dłuższa niż 8 znaków.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="8aa0c-187">Zapisz plik tekstowy i zanotuj jego ścieżkę.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="8aa0c-188">Teraz możesz przystąpić do wdrażania aplikacji aparatu fotograficznego za pomocą Porter.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="8aa0c-189">Uruchom polecenie install i obejrzyj, jak zostanie utworzone wdrożenie IoT Edge.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="8aa0c-190">Sprawdź, czy wdrożenie aparatu fotograficznego zostało ukończone przez wyświetlenie kanału informacyjnego aparatu w lokalizacji `https://<camera-ip>:3000/` , gdzie `<camara-ip>` jest adresem IP aparatu.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="8aa0c-191">Ten krok może potrwać do 10 minut.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="8aa0c-192">Konfigurowanie Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="8aa0c-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="8aa0c-193">Teraz, gdy dane przepływają do Azure Stream Analytics z aparatu, musimy ręcznie autoryzować ją do komunikowania się z Power BI.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="8aa0c-194">W Azure Portal Otwórz **wszystkie zasoby**i zadanie *Process-FootFall \[ \] yoursuffix* .</span><span class="sxs-lookup"><span data-stu-id="8aa0c-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="8aa0c-195">W sekcji **Topologia zadania** okienka zadania usługi Stream Analytics wybierz opcję **Dane wyjściowe**.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="8aa0c-196">Wybierz **docelowy przepływ** danych wyjściowych.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="8aa0c-197">Wybierz pozycję **Odnów autoryzację** i zaloguj się na swoim koncie Power BI.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Monit odnawiania autoryzacji w Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="8aa0c-199">Zapisz ustawienia danych wyjściowych.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-199">Save the output settings.</span></span>

6. <span data-ttu-id="8aa0c-200">Przejdź do okienka **Przegląd** i wybierz pozycję **Rozpocznij** , aby rozpocząć wysyłanie danych do Power BI.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="8aa0c-201">Wybierz wartość **Teraz** jako godzinę rozpoczęcia generowania danych wyjściowych zadania, a następnie wybierz pozycję **Uruchom**.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="8aa0c-202">Stan zadania możesz wyświetlić na pasku powiadomień.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="8aa0c-203">Tworzenie pulpitu nawigacyjnego Power BI</span><span class="sxs-lookup"><span data-stu-id="8aa0c-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="8aa0c-204">Gdy zadanie zakończy się pomyślnie, przejdź do [Power BI](https://powerbi.com/) i zaloguj się przy użyciu konta służbowego.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="8aa0c-205">Jeśli wyniki zapytania zadania Stream Analytics są wyprowadzane, utworzony zestaw danych *FootFall-DataSet* istnieje na karcie **zestawy** danych.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="8aa0c-206">W obszarze roboczym Power BI wybierz pozycję **+ Utwórz** , aby utworzyć nowy pulpit nawigacyjny o nazwie *FootFall Analysis.*</span><span class="sxs-lookup"><span data-stu-id="8aa0c-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="8aa0c-207">W górnej części okna wybierz pozycję **Dodaj kafelek**.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="8aa0c-208">Następnie wybierz pozycje **Niestandardowe dane przesyłane strumieniowo** i **Dalej**.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="8aa0c-209">Wybierz **zestaw danych FootFall-DataSet** w **zestawach**danych.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="8aa0c-210">Wybierz **kartę** z listy rozwijanej **typ wizualizacji** , a następnie Dodaj opcję **wiek** do **pól**.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="8aa0c-211">Wybierz pozycję **Dalej**, aby wprowadzić nazwę kafelka, a następnie wybierz pozycję **Zastosuj**, aby utworzyć kafelek.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="8aa0c-212">W razie potrzeby możesz dodać dodatkowe pola i karty.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="8aa0c-213">Testowanie rozwiązania</span><span class="sxs-lookup"><span data-stu-id="8aa0c-213">Test Your Solution</span></span>

<span data-ttu-id="8aa0c-214">Obserwuj, jak dane w kartach utworzonych w Power BI zmieniają się w zależności od aparatu.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="8aa0c-215">Po zarejestrowaniu wnioskowania może upłynąć do 20 sekund.</span><span class="sxs-lookup"><span data-stu-id="8aa0c-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="8aa0c-216">Usuwanie rozwiązania</span><span class="sxs-lookup"><span data-stu-id="8aa0c-216">Remove Your Solution</span></span>

<span data-ttu-id="8aa0c-217">Jeśli chcesz usunąć swoje rozwiązanie, uruchom następujące polecenia przy użyciu Porter, używając tych samych plików parametrów, które zostały utworzone dla wdrożenia:</span><span class="sxs-lookup"><span data-stu-id="8aa0c-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="8aa0c-218">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="8aa0c-218">Next steps</span></span>

- <span data-ttu-id="8aa0c-219">Dowiedz się więcej o [zagadnienia dotyczące projektowania aplikacji hybrydowych]. (overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="8aa0c-219">Learn more about [Hybrid app design considerations].(overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="8aa0c-220">Przejrzyj i Zaproponuj ulepszenia [kodu dla tego przykładu w witrynie GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span><span class="sxs-lookup"><span data-stu-id="8aa0c-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
