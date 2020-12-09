---
title: Wdróż klaster Kubernetes o wysokiej dostępności w centrum Azure Stack
description: Dowiedz się, jak wdrożyć rozwiązanie klastrowe Kubernetes w celu zapewnienia wysokiej dostępności przy użyciu usług Azure i Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911737"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="67599-103">Wdrażanie klastra Kubernetes o wysokiej dostępności w centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="67599-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="67599-104">W tym artykule przedstawiono sposób tworzenia środowiska klastra Kubernetes o wysokiej dostępności wdrożonego na wielu wystąpieniach centrów Azure Stack w różnych lokalizacjach fizycznych.</span><span class="sxs-lookup"><span data-stu-id="67599-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="67599-105">W tym przewodniku wdrażania rozwiązań dowiesz się, jak:</span><span class="sxs-lookup"><span data-stu-id="67599-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="67599-106">Pobieranie i przygotowywanie aparatu AKS</span><span class="sxs-lookup"><span data-stu-id="67599-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="67599-107">Nawiązywanie połączenia z maszyną wirtualną pomocnika aparatu AKS</span><span class="sxs-lookup"><span data-stu-id="67599-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="67599-108">Wdrażanie klastra Kubernetes</span><span class="sxs-lookup"><span data-stu-id="67599-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="67599-109">Nawiązywanie połączenia z klastrem Kubernetes</span><span class="sxs-lookup"><span data-stu-id="67599-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="67599-110">Łączenie Azure Pipelines z klastrem Kubernetes</span><span class="sxs-lookup"><span data-stu-id="67599-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="67599-111">Konfigurowanie monitorowania</span><span class="sxs-lookup"><span data-stu-id="67599-111">Configure monitoring</span></span>
> - <span data-ttu-id="67599-112">Wdrażanie aplikacji</span><span class="sxs-lookup"><span data-stu-id="67599-112">Deploy application</span></span>
> - <span data-ttu-id="67599-113">Automatyczne skalowanie aplikacji</span><span class="sxs-lookup"><span data-stu-id="67599-113">Autoscale application</span></span>
> - <span data-ttu-id="67599-114">Konfigurowanie usługi Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="67599-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="67599-115">Upgrade Kubernetes (Uaktualnianie usługi Kubernetes)</span><span class="sxs-lookup"><span data-stu-id="67599-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="67599-116">Skalowanie Kubernetes</span><span class="sxs-lookup"><span data-stu-id="67599-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="67599-117">![Filary hybrydowe](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="67599-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="67599-118">Microsoft Azure Stack Hub to rozszerzenie platformy Azure.</span><span class="sxs-lookup"><span data-stu-id="67599-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="67599-119">Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.</span><span class="sxs-lookup"><span data-stu-id="67599-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="67599-120">[Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych.</span><span class="sxs-lookup"><span data-stu-id="67599-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="67599-121">Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.</span><span class="sxs-lookup"><span data-stu-id="67599-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="67599-122">Wymagania wstępne</span><span class="sxs-lookup"><span data-stu-id="67599-122">Prerequisites</span></span>

<span data-ttu-id="67599-123">Przed rozpoczęciem pracy z tym przewodnikiem wdrażania upewnij się, że:</span><span class="sxs-lookup"><span data-stu-id="67599-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="67599-124">Zapoznaj się z artykułem [wzorca klastra Kubernetes o wysokiej dostępności](pattern-highly-available-kubernetes.md) .</span><span class="sxs-lookup"><span data-stu-id="67599-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="67599-125">Zapoznaj się z zawartością [repozytorium pomocnika GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)zawierającego dodatkowe zasoby, do których odwołuje się ten artykuł.</span><span class="sxs-lookup"><span data-stu-id="67599-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="67599-126">Mieć konto, które ma dostęp do [portalu użytkowników centrum Azure Stack](/azure-stack/user/azure-stack-use-portal)z co najmniej [uprawnieniami "Współautor"](/azure-stack/user/azure-stack-manage-permissions).</span><span class="sxs-lookup"><span data-stu-id="67599-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="67599-127">Pobierz i przygotuj aparat AKS</span><span class="sxs-lookup"><span data-stu-id="67599-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="67599-128">Aparat AKS to plik binarny, który może być używany z dowolnego hosta systemu Windows lub Linux, który może dotrzeć do punktów końcowych Azure Resource Manager centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="67599-129">W tym przewodniku opisano wdrażanie nowej maszyny wirtualnej z systemem Linux (lub Windows) w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="67599-130">Będzie on używany później, gdy aparat AKS wdraża klastry Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="67599-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="67599-131">Istnieje również możliwość użycia istniejącej maszyny wirtualnej z systemem Windows lub Linux do wdrożenia klastra Kubernetes w centrum Azure Stack przy użyciu aparatu AKS.</span><span class="sxs-lookup"><span data-stu-id="67599-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="67599-132">Szczegółowe procedury i wymagania dotyczące aparatu AKS są udokumentowane w tym miejscu:</span><span class="sxs-lookup"><span data-stu-id="67599-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="67599-133">[Instalowanie aparatu AKS w systemie Linux w centrum Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (lub przy użyciu [systemu Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="67599-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="67599-134">Aparat AKS jest narzędziem pomocnika do wdrażania i obsługiwania (niezarządzanych) klastrów Kubernetes (w systemie Azure i w centrum Azure Stackm).</span><span class="sxs-lookup"><span data-stu-id="67599-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="67599-135">Szczegóły i różnice dotyczące aparatu AKS w centrum Azure Stack są opisane tutaj:</span><span class="sxs-lookup"><span data-stu-id="67599-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="67599-136">Co to jest aparat AKS na Azure Stack Hub?</span><span class="sxs-lookup"><span data-stu-id="67599-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="67599-137">[Aparat AKS w centrum Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (w witrynie GitHub)</span><span class="sxs-lookup"><span data-stu-id="67599-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="67599-138">Przykładowe środowisko będzie używać Terraform do automatyzowania wdrożenia maszyny wirtualnej aparatu AKS.</span><span class="sxs-lookup"><span data-stu-id="67599-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="67599-139">[Szczegóły i kod można znaleźć w repozytorium usługi GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="67599-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="67599-140">Wynikiem tego kroku jest nowa grupa zasobów w centrum Azure Stack, która zawiera maszynę wirtualną pomocnika aparatu AKS i powiązane zasoby:</span><span class="sxs-lookup"><span data-stu-id="67599-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Zasoby maszyn wirtualnych aparatu AKS w centrum Azure Stack](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="67599-142">Jeśli musisz wdrożyć aparat AKS w odłączonym środowisku AIR-gapped, przejrzyj [rozłączone wystąpienia Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) , aby dowiedzieć się więcej.</span><span class="sxs-lookup"><span data-stu-id="67599-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="67599-143">W następnym kroku użyjemy nowo wdrożonej maszyny wirtualnej aparatu AKS do wdrożenia klastra Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="67599-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="67599-144">Nawiązywanie połączenia z maszyną wirtualną pomocnika aparatu AKS</span><span class="sxs-lookup"><span data-stu-id="67599-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="67599-145">Najpierw musisz nawiązać połączenie z wcześniej utworzoną maszyną wirtualną pomocnika aparatu AKS.</span><span class="sxs-lookup"><span data-stu-id="67599-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="67599-146">Maszyna wirtualna powinna mieć publiczny adres IP i powinna być dostępna za pośrednictwem protokołu SSH (port 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="67599-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Strona przegląd aparatu maszyny wirtualnej w aparacie AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="67599-148">Aby nawiązać połączenie z maszyną wirtualną z systemem Linux przy użyciu protokołu SSH, możesz użyć wybranego przez siebie narzędzia, takiego jak MobaXterm, lub program PowerShell w systemie Windows 10.</span><span class="sxs-lookup"><span data-stu-id="67599-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="67599-149">Po nawiązaniu połączenia Uruchom polecenie `aks-engine` .</span><span class="sxs-lookup"><span data-stu-id="67599-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="67599-150">Przejdź do [obsługiwanych wersji aparatu AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) , aby dowiedzieć się więcej na temat aparatu AKS i wersji Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="67599-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![AKS — przykład wiersza polecenia aparatu](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="67599-152">Wdrażanie klastra Kubernetes</span><span class="sxs-lookup"><span data-stu-id="67599-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="67599-153">Maszyna wirtualna pomocnika aparatu AKS nie utworzyła jeszcze klastra Kubernetes w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="67599-154">Tworzenie klastra to pierwsza akcja do wykonania na maszynie wirtualnej pomocnika aparatu AKS.</span><span class="sxs-lookup"><span data-stu-id="67599-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="67599-155">Proces krok po kroku został opisany tutaj:</span><span class="sxs-lookup"><span data-stu-id="67599-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="67599-156">Wdrażanie klastra Kubernetes za pomocą aparatu AKS w centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="67599-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="67599-157">Końcowy wynik `aks-engine deploy` polecenia i przygotowania w poprzednich krokach to w pełni polecany klaster Kubernetes wdrożony w przestrzeni dzierżawców pierwszego wystąpienia centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="67599-158">Sam klaster składa się z składników IaaS platformy Azure, takich jak maszyny wirtualne, moduły równoważenia obciążenia, sieci wirtualnych, dyski i tak dalej.</span><span class="sxs-lookup"><span data-stu-id="67599-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Portal IaaS Azure Stack składników klastra centrum](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="67599-160">Moduł równoważenia obciążenia platformy Azure (punkt końcowy interfejsu API K8s)</span><span class="sxs-lookup"><span data-stu-id="67599-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="67599-161">Węzły procesu roboczego (Pula agentów)</span><span class="sxs-lookup"><span data-stu-id="67599-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="67599-162">Węzły główne</span><span class="sxs-lookup"><span data-stu-id="67599-162">Master Nodes</span></span>

<span data-ttu-id="67599-163">Klaster jest teraz uruchomiony i w następnym kroku połączymy się z nim.</span><span class="sxs-lookup"><span data-stu-id="67599-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="67599-164">Nawiązywanie połączenia z klastrem Kubernetes</span><span class="sxs-lookup"><span data-stu-id="67599-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="67599-165">Teraz można nawiązać połączenie z utworzonym wcześniej klastrem Kubernetes za pośrednictwem protokołu SSH (przy użyciu klucza SSH określonego w ramach wdrożenia) lub za pośrednictwem `kubectl` (zalecane).</span><span class="sxs-lookup"><span data-stu-id="67599-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="67599-166">Narzędzie wiersza polecenia Kubernetes `kubectl` jest dostępne dla systemów Windows, Linux i macOS [tutaj](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span><span class="sxs-lookup"><span data-stu-id="67599-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="67599-167">Jest już wstępnie zainstalowana i skonfigurowana na głównych węzłach klastra.</span><span class="sxs-lookup"><span data-stu-id="67599-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Wykonaj polecenia kubectl w węźle głównym](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="67599-169">Nie zaleca się używania węzła głównego jako serwera przesiadkowego do wykonywania zadań administracyjnych.</span><span class="sxs-lookup"><span data-stu-id="67599-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="67599-170">`kubectl`Konfiguracja jest przechowywana w `.kube/config` węzłach głównych, a także na maszynie wirtualnej aparatu AKS.</span><span class="sxs-lookup"><span data-stu-id="67599-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="67599-171">Konfigurację można skopiować na maszynę administracyjną z łącznością z klastrem Kubernetes i użyć `kubectl` polecenia.</span><span class="sxs-lookup"><span data-stu-id="67599-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="67599-172">Ten `.kube/config` plik jest również używany później do konfigurowania połączenia usługi w Azure Pipelines.</span><span class="sxs-lookup"><span data-stu-id="67599-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="67599-173">Zachowaj bezpieczeństwo tych plików, ponieważ zawierają one poświadczenia dla klastra Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="67599-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="67599-174">Osoba atakująca z dostępem do pliku ma wystarczającą ilość informacji, aby uzyskać dostęp administratora do niego.</span><span class="sxs-lookup"><span data-stu-id="67599-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="67599-175">Wszystkie akcje wykonywane przy użyciu `.kube/config` pliku początkowego są wykonywane przy użyciu konta administratora klastra.</span><span class="sxs-lookup"><span data-stu-id="67599-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="67599-176">Możesz teraz wypróbować różne polecenia za pomocą programu `kubectl` , aby sprawdzić stan klastra.</span><span class="sxs-lookup"><span data-stu-id="67599-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> <span data-ttu-id="67599-177">Kubernetes ma własny \*i oparty na rolach model Access Control (RBAC)\*\*, który umożliwia tworzenie szczegółowych definicji ról i powiązań ról.</span><span class="sxs-lookup"><span data-stu-id="67599-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="67599-178">Jest to lepszy sposób na kontrolowanie dostępu do klastra zamiast przekazywania uprawnień administratora klastra.</span><span class="sxs-lookup"><span data-stu-id="67599-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="67599-179">Łączenie Azure Pipelines z klastrami Kubernetes</span><span class="sxs-lookup"><span data-stu-id="67599-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="67599-180">Aby połączyć Azure Pipelines z nowo wdrożonym klastrem Kubernetes, potrzebny jest plik konfiguracji polecenia ( `.kube/config` ), zgodnie z opisem w poprzednim kroku.</span><span class="sxs-lookup"><span data-stu-id="67599-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="67599-181">Połącz się z jednym z węzłów głównych klastra Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="67599-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="67599-182">Skopiuj zawartość `.kube/config` pliku.</span><span class="sxs-lookup"><span data-stu-id="67599-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="67599-183">Przejdź do pozycji Azure DevOps > ustawienia projektu > połączenia usługi, aby utworzyć nowe połączenie usługi "Kubernetes" (Użyj KubeConfig jako metody uwierzytelniania)</span><span class="sxs-lookup"><span data-stu-id="67599-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="67599-184">Azure Pipelines (lub jej agenci kompilacji) muszą mieć dostęp do interfejsu API Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="67599-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="67599-185">Jeśli istnieje połączenie internetowe Azure Pipelines do centrum Azure Stack Kubernetes clusetr, należy wdrożyć samoobsługowy Azure Pipelines agenta kompilacji.</span><span class="sxs-lookup"><span data-stu-id="67599-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="67599-186">Podczas wdrażania agentów samoobsługowych dla Azure Pipelines można wdrożyć zarówno w centrum Azure Stack, jak i na komputerze z łącznością sieciową ze wszystkimi wymaganymi punktami końcowymi zarządzania.</span><span class="sxs-lookup"><span data-stu-id="67599-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="67599-187">Szczegóły można znaleźć tutaj:</span><span class="sxs-lookup"><span data-stu-id="67599-187">See the details here:</span></span>

* <span data-ttu-id="67599-188">[Azure Pipelines agenci](/azure/devops/pipelines/agents/agents) w [systemie Windows](/azure/devops/pipelines/agents/v2-windows) lub [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="67599-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="67599-189">Sekcja [uwagi dotyczące wdrażania wzorca (Ci/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) zawiera przepływ decyzyjny, który pomaga zrozumieć, czy należy używać agentów hostowanych przez firmę Microsoft czy agentów samodzielnych:</span><span class="sxs-lookup"><span data-stu-id="67599-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="67599-190">[![Agenci z samodzielnym przepływem decyzji](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="67599-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="67599-191">W tym przykładowym rozwiązaniu topologia zawiera samodzielny agent kompilacji dla każdego wystąpienia centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="67599-192">Agent może uzyskać dostęp do punktów końcowych zarządzania centrum Azure Stack i punktów końcowych interfejsu API klastra Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="67599-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="67599-193">[![tylko ruch wychodzący](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="67599-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="67599-194">Ten projekt spełnia typowe wymagania prawne, które ma mieć tylko połączenia wychodzące z rozwiązania aplikacji.</span><span class="sxs-lookup"><span data-stu-id="67599-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="67599-195">Konfigurowanie monitorowania</span><span class="sxs-lookup"><span data-stu-id="67599-195">Configure monitoring</span></span>

<span data-ttu-id="67599-196">Do monitorowania kontenerów w rozwiązaniu można użyć [Azure monitor](/azure/azure-monitor/) kontenerów.</span><span class="sxs-lookup"><span data-stu-id="67599-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="67599-197">Punkty te Azure Monitor do klastra Kubernetes wdrożonego przez aparat AKS w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="67599-198">Istnieją dwa sposoby włączania Azure Monitor w klastrze.</span><span class="sxs-lookup"><span data-stu-id="67599-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="67599-199">Oba metody wymagają skonfigurowania obszaru roboczego Log Analytics na platformie Azure.</span><span class="sxs-lookup"><span data-stu-id="67599-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="67599-200">[Metoda 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) używa wykresu Helm</span><span class="sxs-lookup"><span data-stu-id="67599-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="67599-201">[Metoda druga](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) w ramach specyfikacji klastra aparatu AKS</span><span class="sxs-lookup"><span data-stu-id="67599-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="67599-202">W topologii przykładowej użyto metody "Metoda 1", która umożliwia automatyzację procesu, a aktualizacje można łatwiej instalować.</span><span class="sxs-lookup"><span data-stu-id="67599-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="67599-203">W następnym kroku potrzebny jest obszar roboczy usługi Azure LogAnalytics (identyfikator i klucz), `Helm` (wersja 3) i `kubectl` na komputerze.</span><span class="sxs-lookup"><span data-stu-id="67599-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="67599-204">Helm to Menedżer pakietów Kubernetes, dostępny jako plik binarny, który jest uruchamiany w systemach macOS, Windows i Linux.</span><span class="sxs-lookup"><span data-stu-id="67599-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="67599-205">Można go pobrać tutaj: [Helm.sh](https://helm.sh/docs/intro/quickstart/) Helm opiera się na pliku konfiguracyjnym Kubernetes używanym dla tego `kubectl` polecenia.</span><span class="sxs-lookup"><span data-stu-id="67599-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="67599-206">To polecenie spowoduje zainstalowanie agenta Azure Monitor w klastrze Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="67599-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="67599-207">Agent pakietu Operations Management Suite (OMS) w klastrze Kubernetes wyśle dane monitorowania do obszaru roboczego usługi Azure Log Analytics (przy użyciu wychodzącego protokołu HTTPS).</span><span class="sxs-lookup"><span data-stu-id="67599-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="67599-208">Teraz możesz używać Azure Monitor, aby uzyskać dokładniejsze informacje o klastrach Kubernetes w centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="67599-209">Ten projekt to zaawansowany sposób demonstrujący możliwości analizy, które mogą być automatycznie wdrażane z klastrami aplikacji.</span><span class="sxs-lookup"><span data-stu-id="67599-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="67599-210">[![Klastry centrum Azure Stack w usłudze Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="67599-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="67599-211">[![Azure Monitor szczegóły klastra](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="67599-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="67599-212">Jeśli Azure Monitor nie wyświetla żadnych danych centrum Azure Stack, upewnij się, że zostały wykonane instrukcje dotyczące [sposobu dodawania rozwiązania AzureMonitor-Containers do obszaru roboczego usługi Azure Loganalytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) .</span><span class="sxs-lookup"><span data-stu-id="67599-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="67599-213">Wdrażanie aplikacji</span><span class="sxs-lookup"><span data-stu-id="67599-213">Deploy the application</span></span>

<span data-ttu-id="67599-214">Przed zainstalowaniem naszej przykładowej aplikacji należy wykonać kolejną czynność, aby skonfigurować kontroler transferu danych przychodzących oparty na Nginx w naszym klastrze Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="67599-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="67599-215">Kontroler transferu danych przychodzących jest używany jako moduł równoważenia obciążenia warstwy 7 do kierowania ruchu w naszym klastrze na podstawie hosta, ścieżki lub protokołu.</span><span class="sxs-lookup"><span data-stu-id="67599-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="67599-216">Nginx — ruch przychodzący jest dostępny jako wykres Helm.</span><span class="sxs-lookup"><span data-stu-id="67599-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="67599-217">Aby uzyskać szczegółowe instrukcje, zapoznaj się z [repozytorium usługi GitHub Chart Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="67599-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="67599-218">Nasza przykładowa aplikacja jest również spakowana jako wykres Helm, taki jak [Agent monitorowania platformy Azure](#configure-monitoring) w poprzednim kroku.</span><span class="sxs-lookup"><span data-stu-id="67599-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="67599-219">W związku z tym jest to proste, aby wdrożyć aplikację w naszym klastrze Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="67599-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="67599-220">[Pliki wykresów Helm można znaleźć w repozytorium usługi GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) .</span><span class="sxs-lookup"><span data-stu-id="67599-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="67599-221">Przykładowa aplikacja to aplikacja o trzech warstwach wdrożona w klastrze Kubernetes na każdym z dwóch wystąpień centrów Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="67599-222">Aplikacja używa bazy danych MongoDB.</span><span class="sxs-lookup"><span data-stu-id="67599-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="67599-223">Możesz dowiedzieć się więcej o tym, jak uzyskać dane zreplikowane w wielu wystąpieniach w ramach wzorców [danych i magazynu](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span><span class="sxs-lookup"><span data-stu-id="67599-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="67599-224">Po wdrożeniu wykresu Helm dla aplikacji zobaczysz wszystkie trzy warstwy aplikacji reprezentowane jako wdrożenia i zestawy stanowe (dla bazy danych) za pomocą jednego pod:</span><span class="sxs-lookup"><span data-stu-id="67599-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

<span data-ttu-id="67599-225">Po stronie usługi znajdziesz kontroler transferu danych przychodzących oparty na Nginx i jego publiczny adres IP:</span><span class="sxs-lookup"><span data-stu-id="67599-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

<span data-ttu-id="67599-226">"Zewnętrzny adres IP" to nasz "punkt końcowy aplikacji".</span><span class="sxs-lookup"><span data-stu-id="67599-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="67599-227">Jest to sposób, w jaki użytkownicy będą łączyć się z otwieraniem aplikacji i będą również używane jako punkt końcowy dla następnego kroku [konfigurowania Traffic Manager](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="67599-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="67599-228">Automatyczne skalowanie aplikacji</span><span class="sxs-lookup"><span data-stu-id="67599-228">Autoscale the application</span></span>
<span data-ttu-id="67599-229">Opcjonalnie można skonfigurować skalowanie w [poziomie](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) w górę lub w dół na podstawie określonych metryk, takich jak użycie procesora CPU.</span><span class="sxs-lookup"><span data-stu-id="67599-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="67599-230">Następujące polecenie spowoduje utworzenie skalowania w poziomie w pionie, które zachowuje 1 do 10 replik z zasobników objętych klasyfikacją — wdrażanie w sieci Web.</span><span class="sxs-lookup"><span data-stu-id="67599-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="67599-231">Program HPA zwiększy i zmniejszy liczbę replik (za pośrednictwem wdrożenia), aby zachować średnie użycie procesora CPU we wszystkich zasobnikach o wielkości 80%.</span><span class="sxs-lookup"><span data-stu-id="67599-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="67599-232">Bieżący stan automatycznego skalowania można sprawdzić, uruchamiając następujące:</span><span class="sxs-lookup"><span data-stu-id="67599-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="67599-233">Konfigurowanie usługi Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="67599-233">Configure Traffic Manager</span></span>

<span data-ttu-id="67599-234">Aby dystrybuować ruch między dwoma (lub więcej) wdrożeniami aplikacji, użyjemy [Traffic Manager platformy Azure](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="67599-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="67599-235">Azure Traffic Manager to oparty na systemie DNS moduł równoważenia obciążenia na platformie Azure.</span><span class="sxs-lookup"><span data-stu-id="67599-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="67599-236">Traffic Manager używa systemu DNS do kierowania żądań klientów do najbardziej odpowiedniego punktu końcowego usługi na podstawie metody routingu ruchu i kondycji punktów końcowych.</span><span class="sxs-lookup"><span data-stu-id="67599-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="67599-237">Zamiast korzystać z usługi Azure Traffic Manager można także korzystać z innych rozwiązań opartych na globalnych rozwiązaniach do równoważenia obciążenia hostowanych lokalnie.</span><span class="sxs-lookup"><span data-stu-id="67599-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="67599-238">W przykładowym scenariuszu będziemy używać platformy Azure Traffic Manager do dystrybucji ruchu między dwoma wystąpieniami naszej aplikacji.</span><span class="sxs-lookup"><span data-stu-id="67599-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="67599-239">Mogą one działać na Azure Stack wystąpieniach centrów w tych samych lub różnych lokalizacjach:</span><span class="sxs-lookup"><span data-stu-id="67599-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![lokalny Menedżer ruchu](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="67599-241">Na platformie Azure skonfigurujemy Traffic Manager tak, aby wskazywały dwa różne wystąpienia naszej aplikacji:</span><span class="sxs-lookup"><span data-stu-id="67599-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="67599-242">[![Profil punktu końcowego TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="67599-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="67599-243">Jak widać, dwa punkty końcowe wskazują dwa wystąpienia wdrożonej aplikacji z [poprzedniej sekcji](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="67599-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="67599-244">W tym momencie:</span><span class="sxs-lookup"><span data-stu-id="67599-244">At this point:</span></span>
- <span data-ttu-id="67599-245">Infrastruktura Kubernetes została utworzona wraz z kontrolerem transferu danych przychodzących.</span><span class="sxs-lookup"><span data-stu-id="67599-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="67599-246">Klastry zostały wdrożone w dwóch wystąpieniach centrów Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="67599-247">Monitorowanie zostało skonfigurowane.</span><span class="sxs-lookup"><span data-stu-id="67599-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="67599-248">Usługa Azure Traffic Manager będzie równoważyć obciążenie ruchu między dwoma wystąpieniami centrum Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="67599-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="67599-249">Na bazie tej infrastruktury Przykładowa aplikacja trójwarstwowej została wdrożona w sposób zautomatyzowany przy użyciu wykresów Helm.</span><span class="sxs-lookup"><span data-stu-id="67599-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="67599-250">Rozwiązanie powinno być teraz dostępne dla użytkowników.</span><span class="sxs-lookup"><span data-stu-id="67599-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="67599-251">Istnieją również pewne zagadnienia operacyjne po wdrożeniu, które są omówione w następnych dwóch sekcjach.</span><span class="sxs-lookup"><span data-stu-id="67599-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="67599-252">Upgrade Kubernetes (Uaktualnianie usługi Kubernetes)</span><span class="sxs-lookup"><span data-stu-id="67599-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="67599-253">Podczas uaktualniania klastra Kubernetes należy wziąć pod uwagę następujące zagadnienia:</span><span class="sxs-lookup"><span data-stu-id="67599-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="67599-254">Uaktualnianie klastra Kubernetes to złożona operacja typu dzień 2, którą można wykonać za pomocą aparatu AKS.</span><span class="sxs-lookup"><span data-stu-id="67599-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="67599-255">Aby uzyskać więcej informacji, zobacz [Uaktualnianie klastra Kubernetes w centrum Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="67599-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="67599-256">Aparat AKS umożliwia uaktualnianie klastrów do nowszych wersji obrazów Kubernetes i podstawowych systemu operacyjnego.</span><span class="sxs-lookup"><span data-stu-id="67599-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="67599-257">Aby uzyskać więcej informacji, zobacz [kroki uaktualniania do nowszej wersji Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="67599-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="67599-258">Można również uaktualnić tylko węzły podnieśce do nowszej wersji obrazu systemu operacyjnego.</span><span class="sxs-lookup"><span data-stu-id="67599-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="67599-259">Aby uzyskać więcej informacji, zobacz [Procedura uaktualniania tylko obrazu systemu operacyjnego](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="67599-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="67599-260">Nowsze obrazy podstawowe systemu operacyjnego zawierają aktualizacje zabezpieczeń i jądra.</span><span class="sxs-lookup"><span data-stu-id="67599-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="67599-261">Jest on odpowiedzialny za monitorowanie dostępności nowszych wersji Kubernetes i obrazów systemu operacyjnego.</span><span class="sxs-lookup"><span data-stu-id="67599-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="67599-262">Operator powinien zaplanować i wykonać te uaktualnienia przy użyciu aparatu AKS.</span><span class="sxs-lookup"><span data-stu-id="67599-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="67599-263">Podstawowe obrazy systemu operacyjnego należy pobrać z witryny Centrum Azure Stack Hub przez operatora Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="67599-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="67599-264">Skalowanie Kubernetes</span><span class="sxs-lookup"><span data-stu-id="67599-264">Scale Kubernetes</span></span>

<span data-ttu-id="67599-265">Skala jest kolejną operacją dnia 2, którą można organizować przy użyciu aparatu AKS.</span><span class="sxs-lookup"><span data-stu-id="67599-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="67599-266">Polecenie Skala ponownie używa pliku konfiguracji klastra (apimodel.json) w katalogu wyjściowym jako dane wejściowe dla nowego wdrożenia Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="67599-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="67599-267">Aparat AKS wykonuje operację skalowania w odniesieniu do określonej puli agentów.</span><span class="sxs-lookup"><span data-stu-id="67599-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="67599-268">Po zakończeniu operacji skalowania aparat AKS aktualizuje definicję klastra w tym samym apimodel.jsw pliku.</span><span class="sxs-lookup"><span data-stu-id="67599-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="67599-269">Definicja klastra odzwierciedla nową liczbę węzłów w celu odzwierciedlenia zaktualizowanej bieżącej konfiguracji klastra.</span><span class="sxs-lookup"><span data-stu-id="67599-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="67599-270">Skalowanie klastra Kubernetes w centrum Azure Stack</span><span class="sxs-lookup"><span data-stu-id="67599-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="67599-271">Następne kroki</span><span class="sxs-lookup"><span data-stu-id="67599-271">Next steps</span></span>

- <span data-ttu-id="67599-272">Dowiedz się więcej na temat [zagadnień dotyczących projektowania aplikacji hybrydowych](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="67599-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="67599-273">Przejrzyj i Zaproponuj ulepszenia [kodu dla tego przykładu w witrynie GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="67599-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>