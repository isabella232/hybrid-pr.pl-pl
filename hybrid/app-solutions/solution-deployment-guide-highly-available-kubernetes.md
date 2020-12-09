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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Wdrażanie klastra Kubernetes o wysokiej dostępności w centrum Azure Stack

W tym artykule przedstawiono sposób tworzenia środowiska klastra Kubernetes o wysokiej dostępności wdrożonego na wielu wystąpieniach centrów Azure Stack w różnych lokalizacjach fizycznych.

W tym przewodniku wdrażania rozwiązań dowiesz się, jak:

> [!div class="checklist"]
> - Pobieranie i przygotowywanie aparatu AKS
> - Nawiązywanie połączenia z maszyną wirtualną pomocnika aparatu AKS
> - Wdrażanie klastra Kubernetes
> - Nawiązywanie połączenia z klastrem Kubernetes
> - Łączenie Azure Pipelines z klastrem Kubernetes
> - Konfigurowanie monitorowania
> - Wdrażanie aplikacji
> - Automatyczne skalowanie aplikacji
> - Konfigurowanie usługi Traffic Manager
> - Upgrade Kubernetes (Uaktualnianie usługi Kubernetes)
> - Skalowanie Kubernetes

> [!Tip]  
> ![Filary hybrydowe](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub to rozszerzenie platformy Azure. Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.  
> 
> [Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych. Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.

## <a name="prerequisites"></a>Wymagania wstępne

Przed rozpoczęciem pracy z tym przewodnikiem wdrażania upewnij się, że:

- Zapoznaj się z artykułem [wzorca klastra Kubernetes o wysokiej dostępności](pattern-highly-available-kubernetes.md) .
- Zapoznaj się z zawartością [repozytorium pomocnika GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)zawierającego dodatkowe zasoby, do których odwołuje się ten artykuł.
- Mieć konto, które ma dostęp do [portalu użytkowników centrum Azure Stack](/azure-stack/user/azure-stack-use-portal)z co najmniej [uprawnieniami "Współautor"](/azure-stack/user/azure-stack-manage-permissions).

## <a name="download-and-prepare-aks-engine"></a>Pobierz i przygotuj aparat AKS

Aparat AKS to plik binarny, który może być używany z dowolnego hosta systemu Windows lub Linux, który może dotrzeć do punktów końcowych Azure Resource Manager centrum Azure Stack. W tym przewodniku opisano wdrażanie nowej maszyny wirtualnej z systemem Linux (lub Windows) w centrum Azure Stack. Będzie on używany później, gdy aparat AKS wdraża klastry Kubernetes.

> [!NOTE]
> Istnieje również możliwość użycia istniejącej maszyny wirtualnej z systemem Windows lub Linux do wdrożenia klastra Kubernetes w centrum Azure Stack przy użyciu aparatu AKS.

Szczegółowe procedury i wymagania dotyczące aparatu AKS są udokumentowane w tym miejscu:

* [Instalowanie aparatu AKS w systemie Linux w centrum Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (lub przy użyciu [systemu Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

Aparat AKS jest narzędziem pomocnika do wdrażania i obsługiwania (niezarządzanych) klastrów Kubernetes (w systemie Azure i w centrum Azure Stackm).

Szczegóły i różnice dotyczące aparatu AKS w centrum Azure Stack są opisane tutaj:

* [Co to jest aparat AKS na Azure Stack Hub?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Aparat AKS w centrum Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (w witrynie GitHub)

Przykładowe środowisko będzie używać Terraform do automatyzowania wdrożenia maszyny wirtualnej aparatu AKS. [Szczegóły i kod można znaleźć w repozytorium usługi GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

Wynikiem tego kroku jest nowa grupa zasobów w centrum Azure Stack, która zawiera maszynę wirtualną pomocnika aparatu AKS i powiązane zasoby:

![Zasoby maszyn wirtualnych aparatu AKS w centrum Azure Stack](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Jeśli musisz wdrożyć aparat AKS w odłączonym środowisku AIR-gapped, przejrzyj [rozłączone wystąpienia Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) , aby dowiedzieć się więcej.

W następnym kroku użyjemy nowo wdrożonej maszyny wirtualnej aparatu AKS do wdrożenia klastra Kubernetes.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Nawiązywanie połączenia z maszyną wirtualną pomocnika aparatu AKS

Najpierw musisz nawiązać połączenie z wcześniej utworzoną maszyną wirtualną pomocnika aparatu AKS.

Maszyna wirtualna powinna mieć publiczny adres IP i powinna być dostępna za pośrednictwem protokołu SSH (port 22/TCP).

![Strona przegląd aparatu maszyny wirtualnej w aparacie AKS](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Aby nawiązać połączenie z maszyną wirtualną z systemem Linux przy użyciu protokołu SSH, możesz użyć wybranego przez siebie narzędzia, takiego jak MobaXterm, lub program PowerShell w systemie Windows 10.

```console
ssh <username>@<ipaddress>
```

Po nawiązaniu połączenia Uruchom polecenie `aks-engine` . Przejdź do [obsługiwanych wersji aparatu AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) , aby dowiedzieć się więcej na temat aparatu AKS i wersji Kubernetes.

![AKS — przykład wiersza polecenia aparatu](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Wdrażanie klastra Kubernetes

Maszyna wirtualna pomocnika aparatu AKS nie utworzyła jeszcze klastra Kubernetes w centrum Azure Stack. Tworzenie klastra to pierwsza akcja do wykonania na maszynie wirtualnej pomocnika aparatu AKS.

Proces krok po kroku został opisany tutaj:

* [Wdrażanie klastra Kubernetes za pomocą aparatu AKS w centrum Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

Końcowy wynik `aks-engine deploy` polecenia i przygotowania w poprzednich krokach to w pełni polecany klaster Kubernetes wdrożony w przestrzeni dzierżawców pierwszego wystąpienia centrum Azure Stack. Sam klaster składa się z składników IaaS platformy Azure, takich jak maszyny wirtualne, moduły równoważenia obciążenia, sieci wirtualnych, dyski i tak dalej.

![Portal IaaS Azure Stack składników klastra centrum](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Moduł równoważenia obciążenia platformy Azure (punkt końcowy interfejsu API K8s)
2) Węzły procesu roboczego (Pula agentów)
3) Węzły główne

Klaster jest teraz uruchomiony i w następnym kroku połączymy się z nim.

## <a name="connect-to-the-kubernetes-cluster"></a>Nawiązywanie połączenia z klastrem Kubernetes

Teraz można nawiązać połączenie z utworzonym wcześniej klastrem Kubernetes za pośrednictwem protokołu SSH (przy użyciu klucza SSH określonego w ramach wdrożenia) lub za pośrednictwem `kubectl` (zalecane). Narzędzie wiersza polecenia Kubernetes `kubectl` jest dostępne dla systemów Windows, Linux i macOS [tutaj](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Jest już wstępnie zainstalowana i skonfigurowana na głównych węzłach klastra.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Wykonaj polecenia kubectl w węźle głównym](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Nie zaleca się używania węzła głównego jako serwera przesiadkowego do wykonywania zadań administracyjnych. `kubectl`Konfiguracja jest przechowywana w `.kube/config` węzłach głównych, a także na maszynie wirtualnej aparatu AKS. Konfigurację można skopiować na maszynę administracyjną z łącznością z klastrem Kubernetes i użyć `kubectl` polecenia. Ten `.kube/config` plik jest również używany później do konfigurowania połączenia usługi w Azure Pipelines.

> [!IMPORTANT]
> Zachowaj bezpieczeństwo tych plików, ponieważ zawierają one poświadczenia dla klastra Kubernetes. Osoba atakująca z dostępem do pliku ma wystarczającą ilość informacji, aby uzyskać dostęp administratora do niego. Wszystkie akcje wykonywane przy użyciu `.kube/config` pliku początkowego są wykonywane przy użyciu konta administratora klastra.

Możesz teraz wypróbować różne polecenia za pomocą programu `kubectl` , aby sprawdzić stan klastra.

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
> Kubernetes ma własny *i oparty na rolach model Access Control (RBAC)**, który umożliwia tworzenie szczegółowych definicji ról i powiązań ról. Jest to lepszy sposób na kontrolowanie dostępu do klastra zamiast przekazywania uprawnień administratora klastra.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Łączenie Azure Pipelines z klastrami Kubernetes

Aby połączyć Azure Pipelines z nowo wdrożonym klastrem Kubernetes, potrzebny jest plik konfiguracji polecenia ( `.kube/config` ), zgodnie z opisem w poprzednim kroku.

* Połącz się z jednym z węzłów głównych klastra Kubernetes.
* Skopiuj zawartość `.kube/config` pliku.
* Przejdź do pozycji Azure DevOps > ustawienia projektu > połączenia usługi, aby utworzyć nowe połączenie usługi "Kubernetes" (Użyj KubeConfig jako metody uwierzytelniania)

> [!IMPORTANT]
> Azure Pipelines (lub jej agenci kompilacji) muszą mieć dostęp do interfejsu API Kubernetes. Jeśli istnieje połączenie internetowe Azure Pipelines do centrum Azure Stack Kubernetes clusetr, należy wdrożyć samoobsługowy Azure Pipelines agenta kompilacji.

Podczas wdrażania agentów samoobsługowych dla Azure Pipelines można wdrożyć zarówno w centrum Azure Stack, jak i na komputerze z łącznością sieciową ze wszystkimi wymaganymi punktami końcowymi zarządzania. Szczegóły można znaleźć tutaj:

* [Azure Pipelines agenci](/azure/devops/pipelines/agents/agents) w [systemie Windows](/azure/devops/pipelines/agents/v2-windows) lub [Linux](/azure/devops/pipelines/agents/v2-linux)

Sekcja [uwagi dotyczące wdrażania wzorca (Ci/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) zawiera przepływ decyzyjny, który pomaga zrozumieć, czy należy używać agentów hostowanych przez firmę Microsoft czy agentów samodzielnych:

[![Agenci z samodzielnym przepływem decyzji](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

W tym przykładowym rozwiązaniu topologia zawiera samodzielny agent kompilacji dla każdego wystąpienia centrum Azure Stack. Agent może uzyskać dostęp do punktów końcowych zarządzania centrum Azure Stack i punktów końcowych interfejsu API klastra Kubernetes.

[![tylko ruch wychodzący](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Ten projekt spełnia typowe wymagania prawne, które ma mieć tylko połączenia wychodzące z rozwiązania aplikacji.

## <a name="configure-monitoring"></a>Konfigurowanie monitorowania

Do monitorowania kontenerów w rozwiązaniu można użyć [Azure monitor](/azure/azure-monitor/) kontenerów. Punkty te Azure Monitor do klastra Kubernetes wdrożonego przez aparat AKS w centrum Azure Stack.

Istnieją dwa sposoby włączania Azure Monitor w klastrze. Oba metody wymagają skonfigurowania obszaru roboczego Log Analytics na platformie Azure.

* [Metoda 1](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) używa wykresu Helm
* [Metoda druga](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) w ramach specyfikacji klastra aparatu AKS

W topologii przykładowej użyto metody "Metoda 1", która umożliwia automatyzację procesu, a aktualizacje można łatwiej instalować.

W następnym kroku potrzebny jest obszar roboczy usługi Azure LogAnalytics (identyfikator i klucz), `Helm` (wersja 3) i `kubectl` na komputerze.

Helm to Menedżer pakietów Kubernetes, dostępny jako plik binarny, który jest uruchamiany w systemach macOS, Windows i Linux. Można go pobrać tutaj: [Helm.sh](https://helm.sh/docs/intro/quickstart/) Helm opiera się na pliku konfiguracyjnym Kubernetes używanym dla tego `kubectl` polecenia.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

To polecenie spowoduje zainstalowanie agenta Azure Monitor w klastrze Kubernetes:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Agent pakietu Operations Management Suite (OMS) w klastrze Kubernetes wyśle dane monitorowania do obszaru roboczego usługi Azure Log Analytics (przy użyciu wychodzącego protokołu HTTPS). Teraz możesz używać Azure Monitor, aby uzyskać dokładniejsze informacje o klastrach Kubernetes w centrum Azure Stack. Ten projekt to zaawansowany sposób demonstrujący możliwości analizy, które mogą być automatycznie wdrażane z klastrami aplikacji.

[![Klastry centrum Azure Stack w usłudze Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Azure Monitor szczegóły klastra](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Jeśli Azure Monitor nie wyświetla żadnych danych centrum Azure Stack, upewnij się, że zostały wykonane instrukcje dotyczące [sposobu dodawania rozwiązania AzureMonitor-Containers do obszaru roboczego usługi Azure Loganalytics](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) .

## <a name="deploy-the-application"></a>Wdrażanie aplikacji

Przed zainstalowaniem naszej przykładowej aplikacji należy wykonać kolejną czynność, aby skonfigurować kontroler transferu danych przychodzących oparty na Nginx w naszym klastrze Kubernetes. Kontroler transferu danych przychodzących jest używany jako moduł równoważenia obciążenia warstwy 7 do kierowania ruchu w naszym klastrze na podstawie hosta, ścieżki lub protokołu. Nginx — ruch przychodzący jest dostępny jako wykres Helm. Aby uzyskać szczegółowe instrukcje, zapoznaj się z [repozytorium usługi GitHub Chart Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

Nasza przykładowa aplikacja jest również spakowana jako wykres Helm, taki jak [Agent monitorowania platformy Azure](#configure-monitoring) w poprzednim kroku. W związku z tym jest to proste, aby wdrożyć aplikację w naszym klastrze Kubernetes. [Pliki wykresów Helm można znaleźć w repozytorium usługi GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) .

Przykładowa aplikacja to aplikacja o trzech warstwach wdrożona w klastrze Kubernetes na każdym z dwóch wystąpień centrów Azure Stack. Aplikacja używa bazy danych MongoDB. Możesz dowiedzieć się więcej o tym, jak uzyskać dane zreplikowane w wielu wystąpieniach w ramach wzorców [danych i magazynu](pattern-highly-available-kubernetes.md#data-and-storage-considerations).

Po wdrożeniu wykresu Helm dla aplikacji zobaczysz wszystkie trzy warstwy aplikacji reprezentowane jako wdrożenia i zestawy stanowe (dla bazy danych) za pomocą jednego pod:

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

Po stronie usługi znajdziesz kontroler transferu danych przychodzących oparty na Nginx i jego publiczny adres IP:

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

"Zewnętrzny adres IP" to nasz "punkt końcowy aplikacji". Jest to sposób, w jaki użytkownicy będą łączyć się z otwieraniem aplikacji i będą również używane jako punkt końcowy dla następnego kroku [konfigurowania Traffic Manager](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Automatyczne skalowanie aplikacji
Opcjonalnie można skonfigurować skalowanie w [poziomie](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) w górę lub w dół na podstawie określonych metryk, takich jak użycie procesora CPU. Następujące polecenie spowoduje utworzenie skalowania w poziomie w pionie, które zachowuje 1 do 10 replik z zasobników objętych klasyfikacją — wdrażanie w sieci Web. Program HPA zwiększy i zmniejszy liczbę replik (za pośrednictwem wdrożenia), aby zachować średnie użycie procesora CPU we wszystkich zasobnikach o wielkości 80%.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Bieżący stan automatycznego skalowania można sprawdzić, uruchamiając następujące:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Konfigurowanie usługi Traffic Manager

Aby dystrybuować ruch między dwoma (lub więcej) wdrożeniami aplikacji, użyjemy [Traffic Manager platformy Azure](/azure/traffic-manager/traffic-manager-overview). Azure Traffic Manager to oparty na systemie DNS moduł równoważenia obciążenia na platformie Azure.

> [!NOTE]
> Traffic Manager używa systemu DNS do kierowania żądań klientów do najbardziej odpowiedniego punktu końcowego usługi na podstawie metody routingu ruchu i kondycji punktów końcowych.

Zamiast korzystać z usługi Azure Traffic Manager można także korzystać z innych rozwiązań opartych na globalnych rozwiązaniach do równoważenia obciążenia hostowanych lokalnie. W przykładowym scenariuszu będziemy używać platformy Azure Traffic Manager do dystrybucji ruchu między dwoma wystąpieniami naszej aplikacji. Mogą one działać na Azure Stack wystąpieniach centrów w tych samych lub różnych lokalizacjach:

![lokalny Menedżer ruchu](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

Na platformie Azure skonfigurujemy Traffic Manager tak, aby wskazywały dwa różne wystąpienia naszej aplikacji:

[![Profil punktu końcowego TM](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Jak widać, dwa punkty końcowe wskazują dwa wystąpienia wdrożonej aplikacji z [poprzedniej sekcji](#deploy-the-application).

W tym momencie:
- Infrastruktura Kubernetes została utworzona wraz z kontrolerem transferu danych przychodzących.
- Klastry zostały wdrożone w dwóch wystąpieniach centrów Azure Stack.
- Monitorowanie zostało skonfigurowane.
- Usługa Azure Traffic Manager będzie równoważyć obciążenie ruchu między dwoma wystąpieniami centrum Azure Stack.
- Na bazie tej infrastruktury Przykładowa aplikacja trójwarstwowej została wdrożona w sposób zautomatyzowany przy użyciu wykresów Helm. 

Rozwiązanie powinno być teraz dostępne dla użytkowników.

Istnieją również pewne zagadnienia operacyjne po wdrożeniu, które są omówione w następnych dwóch sekcjach.

## <a name="upgrade-kubernetes"></a>Upgrade Kubernetes (Uaktualnianie usługi Kubernetes)

Podczas uaktualniania klastra Kubernetes należy wziąć pod uwagę następujące zagadnienia:

- Uaktualnianie klastra Kubernetes to złożona operacja typu dzień 2, którą można wykonać za pomocą aparatu AKS. Aby uzyskać więcej informacji, zobacz [Uaktualnianie klastra Kubernetes w centrum Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- Aparat AKS umożliwia uaktualnianie klastrów do nowszych wersji obrazów Kubernetes i podstawowych systemu operacyjnego. Aby uzyskać więcej informacji, zobacz [kroki uaktualniania do nowszej wersji Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- Można również uaktualnić tylko węzły podnieśce do nowszej wersji obrazu systemu operacyjnego. Aby uzyskać więcej informacji, zobacz [Procedura uaktualniania tylko obrazu systemu operacyjnego](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Nowsze obrazy podstawowe systemu operacyjnego zawierają aktualizacje zabezpieczeń i jądra. Jest on odpowiedzialny za monitorowanie dostępności nowszych wersji Kubernetes i obrazów systemu operacyjnego. Operator powinien zaplanować i wykonać te uaktualnienia przy użyciu aparatu AKS. Podstawowe obrazy systemu operacyjnego należy pobrać z witryny Centrum Azure Stack Hub przez operatora Azure Stack Hub.

## <a name="scale-kubernetes"></a>Skalowanie Kubernetes

Skala jest kolejną operacją dnia 2, którą można organizować przy użyciu aparatu AKS.

Polecenie Skala ponownie używa pliku konfiguracji klastra (apimodel.json) w katalogu wyjściowym jako dane wejściowe dla nowego wdrożenia Azure Resource Manager. Aparat AKS wykonuje operację skalowania w odniesieniu do określonej puli agentów. Po zakończeniu operacji skalowania aparat AKS aktualizuje definicję klastra w tym samym apimodel.jsw pliku. Definicja klastra odzwierciedla nową liczbę węzłów w celu odzwierciedlenia zaktualizowanej bieżącej konfiguracji klastra.

- [Skalowanie klastra Kubernetes w centrum Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Następne kroki

- Dowiedz się więcej na temat [zagadnień dotyczących projektowania aplikacji hybrydowych](overview-app-design-considerations.md)
- Przejrzyj i Zaproponuj ulepszenia [kodu dla tego przykładu w witrynie GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).