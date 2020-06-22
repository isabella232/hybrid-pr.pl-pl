---
title: Uczenie modelu uczenia maszynowego pod wzorcem krawędzi
description: Dowiedz się, jak przeprowadzić szkolenie modelu uczenia maszynowego na brzegu na platformie Azure i w centrum Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911098"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="7e4a6-103">Uczenie modelu uczenia maszynowego pod wzorcem krawędzi</span><span class="sxs-lookup"><span data-stu-id="7e4a6-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="7e4a6-104">Generuj modele przenośnej uczenia maszynowego (ML) na podstawie danych, które istnieją tylko lokalnie.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="7e4a6-105">Kontekst i problem</span><span class="sxs-lookup"><span data-stu-id="7e4a6-105">Context and problem</span></span>

<span data-ttu-id="7e4a6-106">Wiele organizacji chce odwiedzać szczegółowe informacje z lokalnych lub starszych danych przy użyciu narzędzi, które są zrozumiałe dla analityków danych.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="7e4a6-107">[Azure Machine Learning](/azure/machine-learning/) zapewnia natywne narzędzia chmurowe umożliwiające uczenie, dostrajanie i wdrażanie modeli i uczenia głębokiego.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="7e4a6-108">Jednak niektóre dane są zbyt duże do chmury lub nie można ich wysłać do chmury z przyczyn prawnych.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="7e4a6-109">Korzystając z tego wzorca, analityki danych mogą używać Azure Machine Learning do uczenia modeli przy użyciu danych lokalnych i obliczeń.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="7e4a6-110">Rozwiązanie</span><span class="sxs-lookup"><span data-stu-id="7e4a6-110">Solution</span></span>

<span data-ttu-id="7e4a6-111">Szkolenia w ramach wzorca brzegowego korzystają z maszyny wirtualnej uruchomionej w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="7e4a6-112">Maszyna wirtualna jest zarejestrowana jako obiekt docelowy obliczeń w usłudze Azure ML, dzięki czemu dostęp do danych jest dostępny tylko lokalnie.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="7e4a6-113">W takim przypadku dane są przechowywane w magazynie obiektów Blob Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="7e4a6-114">Po przeszkoleniu modelu jest on rejestrowany w usłudze Azure ML, kontenerze i dodawany do Azure Container Registry wdrożenia.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="7e4a6-115">Dla tej iteracji wzorca maszyna wirtualna szkoleń Azure Stack Hub musi być dostępna za pośrednictwem publicznego Internetu.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="7e4a6-116">[![Model uczenia maszynowego na architekturze brzegowej](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="7e4a6-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="7e4a6-117">Oto jak działa wzorzec:</span><span class="sxs-lookup"><span data-stu-id="7e4a6-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="7e4a6-118">Maszyna wirtualna Azure Stack Hub jest wdrażana i rejestrowana jako obiekt docelowy obliczeń przy użyciu usługi Azure ML.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="7e4a6-119">Na platformie Azure ML jest tworzony eksperyment, który używa maszyny wirtualnej Azure Stack Hub jako elementu docelowego obliczeń.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="7e4a6-120">Po przeszkoleniu modelu jest on zarejestrowany i kontener.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="7e4a6-121">Model można teraz wdrożyć do lokalizacji lokalnych lub w chmurze.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="7e4a6-122">Składniki</span><span class="sxs-lookup"><span data-stu-id="7e4a6-122">Components</span></span>

<span data-ttu-id="7e4a6-123">To rozwiązanie używa następujących składników:</span><span class="sxs-lookup"><span data-stu-id="7e4a6-123">This solution uses the following components:</span></span>

| <span data-ttu-id="7e4a6-124">Warstwa</span><span class="sxs-lookup"><span data-stu-id="7e4a6-124">Layer</span></span> | <span data-ttu-id="7e4a6-125">Składnik</span><span class="sxs-lookup"><span data-stu-id="7e4a6-125">Component</span></span> | <span data-ttu-id="7e4a6-126">Opis</span><span class="sxs-lookup"><span data-stu-id="7e4a6-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="7e4a6-127">Azure</span><span class="sxs-lookup"><span data-stu-id="7e4a6-127">Azure</span></span> | <span data-ttu-id="7e4a6-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="7e4a6-128">Azure Machine Learning</span></span> | <span data-ttu-id="7e4a6-129">[Azure Machine Learning](/azure/machine-learning/) organizować szkolenia modelu ml.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="7e4a6-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="7e4a6-130">Azure Container Registry</span></span> | <span data-ttu-id="7e4a6-131">Usługa Azure ML pakuje model do kontenera i zapisuje go w [Azure Container Registry](/azure/container-registry/) do wdrożenia.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="7e4a6-132">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="7e4a6-132">Azure Stack Hub</span></span> | <span data-ttu-id="7e4a6-133">App Service</span><span class="sxs-lookup"><span data-stu-id="7e4a6-133">App Service</span></span> | <span data-ttu-id="7e4a6-134">[Azure Stack Hub z App Service](/azure-stack/operator/azure-stack-app-service-overview) stanowi podstawę dla składników na krawędzi.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="7e4a6-135">Wystąpienia obliczeniowe</span><span class="sxs-lookup"><span data-stu-id="7e4a6-135">Compute</span></span> | <span data-ttu-id="7e4a6-136">Maszyna wirtualna Azure Stack Hub z systemem Ubuntu z platformą Docker służy do uczenia modelu ML.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="7e4a6-137">Magazyn</span><span class="sxs-lookup"><span data-stu-id="7e4a6-137">Storage</span></span> | <span data-ttu-id="7e4a6-138">Dane prywatne mogą być hostowane w Azure Stack centrum obiektów BLOB Storage.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="7e4a6-139">Problemy i kwestie do rozważenia</span><span class="sxs-lookup"><span data-stu-id="7e4a6-139">Issues and considerations</span></span>

<span data-ttu-id="7e4a6-140">Podczas decydowania o sposobie wdrożenia tego rozwiązania należy wziąć pod uwagę następujące kwestie:</span><span class="sxs-lookup"><span data-stu-id="7e4a6-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="7e4a6-141">Skalowalność</span><span class="sxs-lookup"><span data-stu-id="7e4a6-141">Scalability</span></span>

<span data-ttu-id="7e4a6-142">Aby umożliwić skalowanie tego rozwiązania, należy utworzyć maszynę wirtualną o odpowiednim rozmiarze na Azure Stack Hub na potrzeby szkolenia.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="7e4a6-143">Dostępność</span><span class="sxs-lookup"><span data-stu-id="7e4a6-143">Availability</span></span>

<span data-ttu-id="7e4a6-144">Upewnij się, że skrypty szkoleniowe i maszyna wirtualna Azure Stack Hub mają dostęp do danych lokalnych używanych do szkoleń.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="7e4a6-145">Możliwości zarządzania</span><span class="sxs-lookup"><span data-stu-id="7e4a6-145">Manageability</span></span>

<span data-ttu-id="7e4a6-146">Upewnij się, że modele i eksperymenty są odpowiednio zarejestrowane, w wersji i otagowane, aby uniknąć pomyłek podczas wdrażania modelu.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="7e4a6-147">Zabezpieczenia</span><span class="sxs-lookup"><span data-stu-id="7e4a6-147">Security</span></span>

<span data-ttu-id="7e4a6-148">Ten wzorzec umożliwia usłudze Azure ML dostęp do danych poufnych w środowisku lokalnym.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="7e4a6-149">Upewnij się, że konto używane do protokołu SSH do maszyny wirtualnej Azure Stack Hub ma silne hasło i że skrypty szkoleniowe nie zachowają ani nie przekazują danych do chmury.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="7e4a6-150">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="7e4a6-150">Next steps</span></span>

<span data-ttu-id="7e4a6-151">Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:</span><span class="sxs-lookup"><span data-stu-id="7e4a6-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="7e4a6-152">Zapoznaj się z [dokumentacją Azure Machine Learning](/azure/machine-learning) , aby zapoznać się z tematem dotyczącym tablic i tematów pokrewnych.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="7e4a6-153">Zobacz [Azure Container Registry](/azure/container-registry/) , aby dowiedzieć się, jak tworzyć i przechowywać obrazy i zarządzać nimi na potrzeby wdrożeń kontenerów.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="7e4a6-154">Aby dowiedzieć się więcej na temat dostawcy zasobów i sposobu wdrażania, zapoznaj się z tematem [App Service w centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-overview) .</span><span class="sxs-lookup"><span data-stu-id="7e4a6-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="7e4a6-155">Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na dodatkowe pytania.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="7e4a6-156">Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="7e4a6-157">Gdy wszystko będzie gotowe do przetestowania przykładowego rozwiązania, przejdź do [modelu uczenia maszynowego w przewodniku wdrażania programu Edge](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="7e4a6-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="7e4a6-158">Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.</span><span class="sxs-lookup"><span data-stu-id="7e4a6-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
