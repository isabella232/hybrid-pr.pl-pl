---
title: Wzorzec Kubernetes o wysokiej dostępności przy użyciu platformy Azure i usługi Azure Stack Hub
description: Dowiedz się, w jaki sposób rozwiązanie klastrowe Kubernetes zapewnia wysoką dostępność przy użyciu platformy Azure i Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 454cc0a0531882b7a8ec050a461420ce13eebcfe
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911793"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Wzorzec klastra Kubernetes o wysokiej dostępności

W tym artykule opisano sposób tworzenia i obsługi infrastruktury opartej na Kubernetes o wysokiej dostępności przy użyciu aparatu usługi Azure Kubernetes Service (AKS) w centrum Azure Stack. Ten scenariusz jest typowy dla organizacji o znaczeniu krytycznym w środowiskach o wysokim stopniu ograniczonej i regulowanej. Organizacje w domenach, takie jak finanse, obronne i rządowe.

## <a name="context-and-problem"></a>Kontekst i problem

Wiele organizacji opracowuje rozwiązania natywne w chmurze, które wykorzystują najnowocześniejsze usługi i technologie, takie jak Kubernetes. Mimo że platforma Azure udostępnia centra danych w większości regionów świata, czasami istnieją przypadki użycia krawędzi i scenariusze, w których krytyczne aplikacje muszą działać w określonej lokalizacji. Zagadnienia obejmują:

- Czułość lokalizacji
- Opóźnienie między systemami aplikacji i lokalnymi
- Ochrona przepustowości
- Łączność
- Wymagania prawne lub statutowe

Platforma Azure, w połączeniu z centrum Azure Stack, rozwiązuje największe problemy. Poniżej opisano szeroki zestaw opcji, decyzji i zagadnień dotyczących pomyślnej implementacji usługi Kubernetes działającej w centrum Azure Stack.

## <a name="solution"></a>Rozwiązanie

W tym wzorcu przyjęto założenie, że musimy zająć się ścisłym zestawem ograniczeń. Aplikacja musi działać lokalnie, a wszystkie dane osobowe nie mogą dotrzeć do usług w chmurze publicznej. Monitorowanie i inne dane nieosobowe mogą być wysyłane do platformy Azure i przetwarzane w tym miejscu. Do usług zewnętrznych, takich jak publiczna Container Registry lub inne, można uzyskać dostęp, ale mogą one być filtrowane przez zaporę lub serwer proxy.

Przykładowa aplikacja pokazana tutaj (oparta na usłudze [Azure Kubernetes Service Workshop](/learn/modules/aks-workshop/)) została zaprojektowana tak, aby korzystała z rozwiązań natywnych Kubernetes, jeśli to możliwe. Ten projekt umożliwia uniknięcie blokady dostawcy zamiast korzystania z usług natywnych platformy. Przykładowo aplikacja używa własnego zaplecza bazy danych MongoDB, zamiast usługi PaaS lub zewnętrznej bazy danych.

[![Hybrydowy wzorzec aplikacji](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

Na powyższym diagramie przedstawiono architekturę aplikacji przykładowej aplikacji działającej na Kubernetes w centrum Azure Stack. Aplikacja składa się z kilku składników, w tym:

 1) Klaster Kubernetes oparty na aparacie AKS w centrum Azure Stack.
 2) [Menedżer certyfikatów](https://www.jetstack.io/cert-manager/), który zapewnia pakiet narzędzi do zarządzania certyfikatami w usłudze Kubernetes, używany do automatycznego żądania certyfikatów od do szyfrowania.
 3) Przestrzeń nazw Kubernetes, która zawiera składniki aplikacji dla frontonu (ratings-Web), API (ratings-API) i Database (ratings-MongoDB).
 4) Kontroler transferu danych przychodzących, który kieruje ruch HTTP/HTTPS do punktów końcowych w klastrze Kubernetes.

Przykładowa aplikacja służy do ilustrowania architektury aplikacji. Wszystkie składniki są przykładami. Architektura zawiera tylko jedno wdrożenie aplikacji. Aby zapewnić wysoką dostępność (HA), należy uruchomić wdrożenie co najmniej dwa razy w dwóch różnych wystąpieniach Azure Stack Hub — mogą one być uruchamiane w tej samej lokalizacji lub w dwóch (lub więcej) różnych lokacjach:

![Architektura infrastruktury](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Usługi, takie jak Azure Container Registry, Azure Monitor i inne, są hostowane poza centrum Azure Stack na platformie Azure lub lokalnie. Ten projekt hybrydowy chroni rozwiązanie przed awarią pojedynczego wystąpienia centrum Azure Stack.

## <a name="components"></a>Składniki

Ogólna architektura programu składa się z następujących składników:

**Azure Stack Hub** to rozszerzenie platformy Azure, które może uruchamiać obciążenia w środowisku lokalnym, dostarczając usługi platformy Azure w centrum danych. Przejdź do [omówienia Azure Stack Hub](/azure-stack/operator/azure-stack-overview) , aby dowiedzieć się więcej.

Aparat **usługi Kubernetes platformy Azure (aparat AKS)** jest aparatem za zarządzaną ofertą usług Kubernetes, Azure Kubernetes Service (AKS), która jest obecnie dostępna na platformie Azure. W przypadku Azure Stack Hub aparat AKS umożliwia wdrażanie, skalowanie i uaktualnianie w pełni zarządzanych, samoobsługowych klastrów Kubernetes przy użyciu funkcji IaaS centrum Azure Stack. Przejdź do [omówienia aparatu AKS](https://github.com/Azure/aks-engine) , aby dowiedzieć się więcej.

Przejdź do [znanych problemów i ograniczeń](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) , aby dowiedzieć się więcej o różnicach między aparatem AKS na platformie Azure i aparatem AKS w centrum Azure Stack.

**Usługa Azure Virtual Network (VNET)** służy do zapewnienia infrastruktury sieci na każdym centrum Azure Stack dla Virtual Machines (VM) obsługującego infrastrukturę klastra Kubernetes.

**Azure Load Balancer** jest używany dla punktu końcowego interfejsu API Kubernetes i Nginx transferu danych przychodzących. Moduł równoważenia obciążenia kieruje ruch zewnętrzny (na przykład internetowy) do węzłów i maszyn wirtualnych oferujących konkretną usługę.

**Azure Container Registry (ACR)** służy do przechowywania prywatnych obrazów platformy Docker i wykresów Helm, które są wdrażane w klastrze. Aparat AKS może uwierzytelniać się za pomocą Container Registry przy użyciu tożsamości usługi Azure AD. Kubernetes nie wymaga ACR. Możesz użyć innych rejestrów kontenerów, takich jak Docker Hub.

**Azure Repos** to zestaw narzędzi do kontroli wersji, których można użyć do zarządzania kodem. Możesz również korzystać z usługi GitHub lub innych repozytoriów opartych na git. Przejdź do [omówienia Azure Repos](/azure/devops/repos/get-started/what-is-repos) , aby dowiedzieć się więcej.

**Azure Pipelines** jest częścią Azure DevOps Services i uruchamia zautomatyzowane kompilacje, testy i wdrożenia. Możesz również używać rozwiązań CI/CD innych firm, takich jak Jenkins. Przejdź do [omówienia usługi Azure Pipeline](/azure/devops/pipelines/get-started/what-is-azure-pipelines) , aby dowiedzieć się więcej.

**Azure monitor** zbiera i przechowuje metryki i dzienniki, w tym metryki platformy dla usług platformy Azure w ramach telemetrii rozwiązania i aplikacji. Te dane służą do monitorowania aplikacji, konfigurowania alertów i pulpitów nawigacyjnych oraz wykonywania analizy głównych przyczyn błędów. Azure Monitor integruje się z usługą Kubernetes, aby zbierać metryki z kontrolerów, węzłów i kontenerów, a także dzienników kontenerów i dzienników węzłów głównych. Przejdź do [omówienia Azure monitor](/azure/azure-monitor/overview) , aby dowiedzieć się więcej.

**Usługa Azure Traffic Manager** to oparty na systemie DNS moduł równoważenia obciążenia, który umożliwia optymalne dystrybuowanie ruchu do usług w różnych regionach platformy Azure lub wdrożeniach centrów Azure Stack. Traffic Manager zapewnia również wysoką dostępność i czas odpowiedzi. Punkty końcowe aplikacji muszą być dostępne z zewnątrz. Dostępne są również inne rozwiązania lokalne.

**Kontroler** transferu danych przychodzących Kubernetes ujawnia trasy http (S) do usług w klastrze Kubernetes. W tym celu można użyć Nginx lub dowolnego odpowiedniego kontrolera transferu danych przychodzących.

**Helm** to Menedżer pakietów dla wdrożenia Kubernetes, który umożliwia łączenie różnych obiektów Kubernetes, takich jak wdrożenia, usługi, wpisy tajne, w jednym "wykresie". Możesz publikować, wdrażać i kontrolować Zarządzanie wersjami oraz aktualizować obiekt wykresu. Azure Container Registry może służyć jako repozytorium do przechowywania spakowanych wykresów Helm.

## <a name="design-considerations"></a>Zagadnienia dotyczące projektowania

Ten wzorzec jest zgodny z kilkoma zagadnieniami wysokiego poziomu wyjaśnionymi bardziej szczegółowo w następnych sekcjach tego artykułu:

- Aplikacja używa rozwiązań natywnych Kubernetes, aby uniknąć zablokowania dostawcy.
- Aplikacja używa architektury mikrousług.
- Centrum Azure Stack nie wymaga ruchu przychodzącego, ale umożliwia wychodzące połączenie z Internetem.

Te zalecenia praktyczne dotyczą również rzeczywistych obciążeń i scenariuszy.

## <a name="scalability-considerations"></a>Zagadnienia dotyczące skalowalności

Skalowalność jest ważna, aby zapewnić użytkownikom spójny, niezawodny i wydajny dostęp do aplikacji.

Przykładowy scenariusz obejmuje skalowalność wielu warstw stosu aplikacji. Poniżej przedstawiono ogólne omówienie różnych warstw:

| Poziom architektury | Mową | Jak to zrobić? |
| --- | --- | ---
| Aplikacja | Aplikacja | Skalowanie w poziomie na podstawie liczby zasobników/replik/Container Instances * |
| Klaster | Klaster Kubernetes | Liczba węzłów (z zakresu od 1 do 50), maszyna wirtualna — rozmiar jednostki SKU i pule węzłów (aparat AKS w centrum Azure Stack obecnie obsługuje tylko jedną pulę węzłów); Korzystanie z polecenia skalowania aparatu AKS (ręczne) |
| Infrastruktura | Azure Stack Hub | Liczba węzłów, pojemności i jednostek skalowania w ramach wdrożenia centrum Azure Stack |

\* Korzystanie z Kubernetes "poziome pod skalowaniem automatycznym (HPA); Automatyczne skalowanie oparte na pomiarach lub skalowanie w pionie przez zmianę wielkości wystąpień kontenera (procesor/pamięć).

**Centrum Azure Stack (poziom infrastruktury)**

Infrastruktura Centrum Azure Stack jest podstawą tej implementacji, ponieważ usługa Azure Stack Hub działa na sprzęcie fizycznym w centrum danych. Podczas wybierania sprzętu centrum należy wybrać opcje dla procesora CPU, gęstości pamięci, konfiguracji magazynu i liczby serwerów. Aby dowiedzieć się więcej o skalowalności centrum Azure Stack, zapoznaj się z następującymi zasobami:

- [Planowanie pojemności dla centrum Azure Stack — Omówienie](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Dodawanie dodatkowych węzłów jednostki skalowania w Azure Stack Hub](/azure-stack/operator/azure-stack-add-scale-node)

**Klaster Kubernetes (poziom klastra)**

Sam klaster Kubernetes składa się z programu i jest oparty na platformie Azure (stack) IaaS Components, w tym w przypadku zasobów obliczeniowych, magazynu i sieci. Rozwiązania Kubernetes obejmują węzły główne i procesy robocze, które są wdrażane jako maszyny wirtualne na platformie Azure (i w centrum Azure Stackm).

- [Węzły płaszczyzny kontroli](/azure/aks/concepts-clusters-workloads#control-plane) (Master) zapewniają podstawowe usługi Kubernetes i aranżację obciążeń aplikacji.
- [Węzły procesu roboczego](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (Worker) uruchamiają obciążenia aplikacji.

W przypadku wybrania rozmiarów maszyn wirtualnych dla początkowego wdrożenia należy wziąć pod uwagę kilka kwestii:  

- **Koszt** — w przypadku planowania węzłów procesu roboczego Pamiętaj o całkowitym koszcie za maszynę wirtualną, która zostanie naliczone. Na przykład jeśli obciążenia aplikacji wymagają ograniczonych zasobów, należy zaplanować wdrożenie maszyn wirtualnych o mniejszych rozmiarach. Usługa Azure Stack Hub, taka jak Azure, jest zwykle rozliczana na podstawie zużycia, więc odpowiednie rozmiary maszyn wirtualnych dla ról Kubernetes ma kluczowe znaczenie dla optymalizacji kosztów użycia. 

- **Skalowalność** — skalowalność klastra jest osiągana przez skalowanie w i wypełnianie liczby węzłów głównych i procesów roboczych lub przez dodanie dodatkowych pul węzłów (niedostępnych w centrum Azure Stackm dzisiaj). Skalowanie klastra może odbywać się na podstawie danych wydajności zebranych za pomocą usługi Container Insights (Azure Monitor + Log Analytics). 

    Jeśli aplikacja wymaga więcej (lub mniej) zasobów, można skalować w poziomie bieżące węzły (lub w dół) dla obu węzłów (od 1 do 50 węzłów). Jeśli potrzebujesz więcej niż 50 węzłów, możesz utworzyć dodatkowy klaster w oddzielnej subskrypcji. Nie można skalować w górę rzeczywistych maszyn wirtualnych do innego rozmiaru maszyny wirtualnej bez ponownego wdrożenia klastra.

    Skalowanie jest wykonywane ręcznie przy użyciu maszyny wirtualnej pomocnika aparatu AKS, która została wcześniej użyta do wdrożenia klastra Kubernetes. Aby uzyskać więcej informacji, zobacz [skalowanie klastrów Kubernetes](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Przydziały** — Uwzględnij [limity przydziału](/azure-stack/operator/azure-stack-quota-types) SKONFIGUROWANE podczas planowania wdrożenia AKS w centrum Azure Stack. Upewnij się, że każda [subskrypcja](/azure-stack/operator/service-plan-offer-subscription-overview) ma odpowiednie plany i skonfigurowane limity przydziału. Subskrypcja będzie musiała obsłużyć ilość zasobów obliczeniowych, magazynu i innych usług potrzebnych dla klastrów w miarę skalowania w poziomie.

- **Obciążenia aplikacji** — zapoznaj się z [pojęciami dotyczącymi klastrów i obciążeń](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) w Kubernetes podstawowych koncepcji dotyczących dokumentu usługi Azure Kubernetes. Ten artykuł ułatwi Określanie rozmiaru maszyny wirtualnej w zależności od potrzeb obliczeniowych i pamięci aplikacji.  

**Aplikacja (poziom aplikacji)**

W warstwie aplikacji używamy funkcji [automatycznego skalowania w poziomie Kubernetes (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/). HPA może zwiększyć lub zmniejszyć liczbę replik (pod/Container Instances) w naszym wdrożeniu na podstawie różnych metryk (takich jak użycie procesora CPU).

Kolejną opcją jest skalowanie wystąpień kontenerów w pionie. Można to osiągnąć przez zmianę ilości procesora i pamięci wymaganej do określonego wdrożenia. Aby dowiedzieć się więcej, zobacz [Zarządzanie zasobami dla kontenerów](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) w witrynie Kubernetes.IO.

## <a name="networking-and-connectivity-considerations"></a>Zagadnienia dotyczące sieci i łączności

Sieci i łączności mają również wpływ na trzy warstwy wymienione wcześniej dla Kubernetes w centrum Azure Stack. W poniższej tabeli przedstawiono warstwy i usługi, które zawierają:

| Warstwa | Mową | Co? |
| --- | --- | ---
| Aplikacja | Aplikacja | Jak jest dostępna aplikacja? Czy będzie on widoczny dla Internetu? |
| Klaster | Klaster Kubernetes | Interfejs API Kubernetes, maszyna wirtualna z aparatem AKS, ściąganie obrazów kontenera (ruch wychodzący), wysyłanie danych monitorowania i telemetrię (ruch wychodzący) |
| Infrastruktura | Azure Stack Hub | Ułatwienia dostępu do punktów końcowych zarządzania centrum Azure Stack, takich jak portal i punkty końcowe Azure Resource Manager. |

**Aplikacja**

W przypadku warstwy aplikacji Najważniejszym zagadnieniem jest to, czy aplikacja jest udostępniana i dostępna z Internetu. Z perspektywy Kubernetes dostępność Internetu oznacza ujawnienie wdrożenia lub użycie usługi Kubernetes lub kontrolera transferu danych przychodzących.

> [!NOTE]
> Zalecamy użycie kontrolerów transferu danych przychodzących, aby udostępnić usługi Kubernetes Services, ponieważ liczba publicznych adresów IP frontonu w Azure Stack Hub jest ograniczona do 5. Ten projekt ogranicza także liczbę usług Kubernetes (z typem modułu równoważenia obciążenia) do 5, który będzie zbyt mały dla wielu wdrożeń. Przejdź do [dokumentacji aparatu AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) , aby dowiedzieć się więcej.

Udostępnianie aplikacji przy użyciu publicznego adresu IP za pośrednictwem Load Balancer lub kontroler transferu danych przychodzących nie nessecarily oznacza, że aplikacja jest teraz dostępna za pośrednictwem Internetu. Istnieje możliwość, że centrum Azure Stack ma publiczny adres IP, który jest widoczny tylko w lokalnym intranecie, a nie wszystkie publiczne adresy IP są naprawdę dostępne z Internetu.

Poprzedni blok uwzględnia ruch przychodzący do aplikacji. Innym tematem, który należy wziąć pod uwagę w przypadku pomyślnego wdrożenia Kubernetes, jest ruch wychodzący/wyjściowy. Poniżej przedstawiono kilka przypadków użycia, które wymagają ruchu wychodzącego:

- Ściąganie obrazów kontenera przechowywanych w DockerHub lub Azure Container Registry
- Pobieranie wykresów Helm
- Emitowanie danych Application Insights (lub innych danych monitorowania)

Niektóre środowiska korporacyjne mogą wymagać użycia _przezroczystych_ lub _nieprzezroczystych_ serwerów proxy. Te serwery wymagają określonej konfiguracji w różnych składnikach klastra. Dokumentacja aparatu AKS zawiera różne szczegóły dotyczące sposobu obsługi sieciowych serwerów proxy. Aby uzyskać więcej informacji, zobacz [aparat AKS i serwery proxy](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Na koniec ruch między klastrami musi przepływać między wystąpieniami centrów Azure Stack. Przykładowe wdrożenie składa się z pojedynczych klastrów Kubernetes działających na poszczególnych wystąpieniach centrów Azure Stack. Ruch między nimi, taki jak ruch związany z replikacją między dwiema bazami danych, to "ruch zewnętrzny". Ruch zewnętrzny musi być kierowany za pośrednictwem sieci VPN typu lokacja-lokacja lub publicznych adresów IP centrum Azure Stack, aby połączyć Kubernetes na dwóch wystąpieniach centrów Azure Stack:

![ruch między i wewnątrz klastra](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Klaster**  

Klaster Kubernetes nie musi być dostępny za pośrednictwem Internetu. Istotną częścią jest interfejs API Kubernetes używany do obsługi klastra, na przykład przy użyciu `kubectl` . Punkt końcowy interfejsu API Kubernetes musi być dostępny dla wszystkich użytkowników, którzy działają w klastrze, lub wdrażać na niej aplikacje i usługi. Ten temat jest bardziej szczegółowy z perspektywy DevOps w sekcji [uwagi dotyczące wdrażania (Ci/CD)](#deployment-cicd-considerations) poniżej.

Na poziomie klastra istnieją również pewne zagadnienia dotyczące ruchu wychodzącego:

- Aktualizacje węzła (dla Ubuntu)
- Dane monitorowania (wysyłane do usługi Azure LogAnalytics)
- Inni agenci wymagający ruchu wychodzącego (specyficzne dla każdego środowiska wdrażania)

Przed wdrożeniem klastra Kubernetes przy użyciu aparatu AKS Zaplanuj ostateczny projekt sieci. Zamiast tworzyć dedykowane Virtual Network, można bardziej wydajniej wdrożyć klaster w istniejącej sieci. Można na przykład użyć istniejącego połączenia sieci VPN typu lokacja-lokacja, które zostało już skonfigurowane w środowisku centrum Azure Stack.

**Infrastruktura**  

Infrastruktura dotyczy dostępu do punktów końcowych zarządzania centrum Azure Stack. Punkty końcowe obejmują Portal dzierżawców i administratorów oraz punkty końcowe Azure Resource Manager administratora i dzierżawy. Punkty końcowe są wymagane do działania centrum Azure Stack i jego podstawowych usług.

## <a name="data-and-storage-considerations"></a>Zagadnienia dotyczące danych i magazynowania

Dwa wystąpienia naszej aplikacji zostaną wdrożone w dwóch poszczególnych klastrach Kubernetes w dwóch wystąpieniach centrów Azure Stack. Ten projekt wymaga, aby rozważyć sposób replikowania i synchronizowania danych między nimi.

Na platformie Azure mamy wbudowaną funkcję replikowania magazynu między wieloma regionami i strefami w chmurze. Obecnie z centrum Azure Stack nie ma natywnych metod replikowania magazynu między dwoma różnymi wystąpieniami centrów Azure Stack — tworzą one dwie niezależne chmury bez konieczności zarządzania nimi jako zestaw. Planowanie odporności aplikacji działających w ramach usługi Azure Stack Hub wymusza tę niezależność w projekcie i wdrożeniach aplikacji.

W większości przypadków replikacja magazynu nie będzie konieczna dla aplikacji odpornej i wysokiej dostępności wdrożonej w systemie AKS. Należy jednak wziąć pod uwagę niezależny magazyn na wystąpienie Azure Stack Hub w projekcie aplikacji. Jeśli ten projekt jest przeznaczonym do wdrożenia rozwiązania w centrum Azure Stack, dostępne są rozwiązania od partnerów firmy Microsoft, które udostępniają załączniki do magazynu. Załączniki magazynu zapewniają rozwiązanie replikacji magazynu w wielu centrach Azure Stack i na platformie Azure. Aby uzyskać więcej informacji, zobacz [rozwiązania partnerskie](#partner-solutions).

W naszej architekturze są brane pod uwagę następujące warstwy:

**Konfiguracja**

Konfiguracja obejmuje konfigurację centrum Azure Stack, aparatu AKS i klastra Kubernetes. Konfiguracja powinna być zautomatyzowana jak najwięcej i przechowywana jako infrastruktura jako kod w systemie kontroli wersji opartej na usłudze git, takim jak Azure DevOps lub GitHub. Tych ustawień nie można łatwo synchronizować między wieloma wdrożeniami. Dlatego zalecamy przechowywanie i stosowanie konfiguracji z zewnątrz oraz przy użyciu potoku DevOps.

**Aplikacja**

Aplikacja powinna być przechowywana w repozytorium opartym na systemie Git. Za każdym razem, gdy istnieje nowe wdrożenie, zmiany aplikacji lub odzyskiwania po awarii, można je łatwo wdrożyć przy użyciu Azure Pipelines.

**Dane**

Dane to najważniejsze zagadnienia w większości projektów aplikacji. Dane aplikacji muszą pozostać zsynchronizowane między różnymi wystąpieniami aplikacji. Dane wymagają również strategii tworzenia kopii zapasowych i odzyskiwania po awarii, jeśli wystąpi awaria.

Osiągnięcie tego projektu jest zależne od opcji technologicznych. Poniżej przedstawiono przykłady rozwiązań dotyczących implementowania bazy danych w trybie wysokiej dostępności na Azure Stack centrum:

- [Wdróż grupę dostępności SQL Server 2016 na platformie Azure i w centrum Azure Stack](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Wdróż rozwiązanie MongoDB o wysokiej dostępności na platformie Azure i w centrum Azure Stack](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Zagadnienia dotyczące pracy z danymi w wielu lokalizacjach to jeszcze bardziej skomplikowany sposób rozwiązania o wysokiej dostępności i odporności. Rozważ następujące kwestie:

- Opóźnienie i łączność sieciowa między centrami Azure Stack.
- Dostępność tożsamości dla usług i uprawnień. Każde wystąpienie centrum Azure Stack integruje się z zewnętrznym katalogiem. Podczas wdrażania należy użyć jednej z Azure Active Directory (Azure AD) lub Active Directory Federation Services (AD FS). W związku z tym istnieje możliwość użycia pojedynczej tożsamości, która może współdziałać z wieloma niezależnymi wystąpieniami Azure Stack Hub.

## <a name="business-continuity-and-disaster-recovery"></a>Ciągłość działania i odzyskiwanie po awarii

Ciągłość działania i odzyskiwanie po awarii (BCDR) to ważny temat w centrum Azure Stack i na platformie Azure. Główną różnicą jest to, że w centrum Azure Stack, operator musi zarządzać całym procesem BCDR. Na platformie Azure części BCDR są zarządzane automatycznie przez firmę Microsoft.

BCDR wpływa na te same obszary, które wymieniono w poprzedniej sekcji [zagadnienia dotyczące danych i magazynu](#data-and-storage-considerations):

- Infrastruktura/konfiguracja
- Dostępność aplikacji
- Dane aplikacji

Jak wspomniano w poprzedniej sekcji, obszary te są odpowiedzialne za operator Azure Stack Hub i mogą się różnić między organizacjami. Zaplanuj BCDR zgodnie z dostępnymi narzędziami i procesami.

**Infrastruktura i konfiguracja**

Ta sekcja dotyczy infrastruktury fizycznej i logicznej oraz konfiguracji centrum Azure Stack. Obejmuje ona akcje administratora i miejsca dzierżawy.

Operator Azure Stack Hub (lub administrator) jest odpowiedzialny za konserwację wystąpień centrum Azure Stack. Obejmuje to między innymi składniki, takie jak sieć, magazyn, tożsamość i inne tematy, które znajdują się poza zakresem tego artykułu. Aby dowiedzieć się więcej na temat konkretnych operacji Azure Stack Hub, zobacz następujące zasoby:

- [Odzyskiwanie danych w centrum Azure Stack przy użyciu usługi Infrastructure Backup](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Włączanie tworzenia kopii zapasowych dla centrum Azure Stack w portalu administratora](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Odzyskiwanie sprawności po utracie danych w wyniku katastrofy](/azure-stack/operator/azure-stack-backup-recover-data)
- [Najlepsze rozwiązania dotyczące usługi Infrastructure Backup](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub to platforma i sieć szkieletowa, w której zostaną wdrożone aplikacje Kubernetes. Właścicielem aplikacji Kubernetes jest użytkownik programu Azure Stack Hub z dostępem udzielonym do wdrożenia infrastruktury aplikacji wymaganej dla rozwiązania. Infrastruktura aplikacji w tym przypadku oznacza klaster Kubernetes wdrożony przy użyciu aparatu AKS i otaczających usług. Te składniki zostaną wdrożone w centrum Azure Stack, ograniczone przez ofertę Azure Stack centrum. Upewnij się, że oferta zaakceptowana przez właściciela aplikacji Kubernetes ma wystarczającą pojemność (wyrażoną w przydziałach Azure Stack centrum), aby wdrożyć całe rozwiązanie. Zgodnie z zaleceniami w poprzedniej sekcji wdrożenie aplikacji powinno być zautomatyzowane przy użyciu potoków w postaci infrastruktury jako kodu i wdrożenia, takich jak Azure DevOps Azure Pipelines.

Aby uzyskać więcej informacji na temat Azure Stack ofert i przydziałów centrum, zobacz [Azure Stack Omówienie usług centrum, planów, ofert i subskrypcji](/azure-stack/operator/service-plan-offer-subscription-overview)

Ważne jest bezpieczne zapisywanie i przechowywanie konfiguracji aparatu AKS, w tym danych wyjściowych. Te pliki zawierają poufne informacje służące do uzyskiwania dostępu do klastra Kubernetes, dlatego muszą być chronione przed ujawnieniem dla administratorów innych niż administratorzy.

**Dostępność aplikacji**

Aplikacja nie powinna polegać na kopiach zapasowych wdrożonego wystąpienia. Zgodnie ze standardowymi metodami, należy ponownie wdrożyć aplikację w ramach wzorców infrastruktury jako kodu. Na przykład Wdróż ponownie przy użyciu usługi Azure DevOps Azure Pipelines. Procedura BCDR powinna dotyczyć ponownego wdrożenia aplikacji w tym samym lub innym klastrze Kubernetes.

**Dane aplikacji**

Dane aplikacji są częścią krytyczną, aby zminimalizować utratę danych. W poprzedniej sekcji opisano metody replikowania i synchronizowania danych między dwoma (lub więcej) wystąpieniami aplikacji. W zależności od infrastruktury bazy danych (MySQL, MongoDB, MSSQL lub innych) służącej do przechowywania danych będą dostępne różne metody dostępności bazy danych i tworzenia kopii zapasowych, które można wybrać.

Zalecanym sposobem osiągnięcia integralności jest użycie:
- Natywne rozwiązanie do tworzenia kopii zapasowych dla określonej bazy danych.
- Rozwiązanie do tworzenia kopii zapasowych, które oficjalnie obsługuje tworzenie kopii zapasowych i odzyskiwanie bazy danych używanej przez aplikację.

> [!IMPORTANT]
> Nie przechowuj danych kopii zapasowej w tym samym wystąpieniu centrum Azure Stack, w którym znajdują się dane aplikacji. Pełna awaria wystąpienia centrum Azure Stack również spowoduje naruszenie kopii zapasowych.

## <a name="availability-considerations"></a>Zagadnienia dotyczące dostępności

Kubernetes w centrum Azure Stack wdrożonym za pośrednictwem aparatu AKS nie jest usługą zarządzaną. Jest to zautomatyzowane wdrożenie i konfiguracja klastra Kubernetes przy użyciu usługi Azure Infrastructure as-a-Service (IaaS). W związku z tym zapewnia taką samą dostępność jak podstawowa infrastruktura.

Infrastruktura Centrum Azure Stack jest już odporna na awarie i oferuje funkcje takie jak zestawy dostępności do dystrybucji składników w wielu [domenach błędów i aktualizacji](/azure-stack/user/azure-stack-vm-considerations#high-availability). Jednak podstawowa technologia (klaster trybu failover) nadal wiąże się z czasem przestoju w przypadku maszyn wirtualnych na serwerze fizycznym, których dotyczy problem, jeśli wystąpi awaria sprzętowa.

Dobrym sposobem jest wdrożenie klastra Kubernetes produkcyjny, a także obciążenia do dwóch (lub więcej) klastrów. Te klastry powinny znajdować się w różnych lokalizacjach lub centrach danych oraz wykorzystują takie technologie jak Azure Traffic Manager, które umożliwiają kierowanie użytkowników w oparciu o czas odpowiedzi klastra lub na podstawie lokalizacji geograficznej.

![Używanie Traffic Manager do kontrolowania przepływów ruchu](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Klienci, którzy mają jeden klaster Kubernetes zazwyczaj nawiązują połączenie z adresem IP usługi lub nazwą DNS danej aplikacji. W przypadku wdrożenia obejmującego wiele klastrów klienci powinni łączyć się z Traffic Manager nazwą DNS, która wskazuje usługi/ruch przychodzący w każdym klastrze Kubernetes.

![Kierowanie do klastra lokalnego przy użyciu Traffic Manager](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Ten wzorzec jest również [najlepszym rozwiązaniem dla (zarządzanych) klastrów AKS na platformie Azure](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment).

Sam klaster Kubernetes wdrożony za pośrednictwem aparatu AKS powinien zawierać co najmniej trzy węzły główne i dwa węzły procesu roboczego.

## <a name="identity-and-security-considerations"></a>Zagadnienia dotyczące tożsamości i zabezpieczeń

Tożsamość i zabezpieczenia są ważnymi tematami. Szczególnie gdy rozwiązanie obejmuje niezależne wystąpienia centrów Azure Stack. Kubernetes i Azure (w tym centrum Azure Stack) mają różne mechanizmy kontroli dostępu opartej na rolach (RBAC):

- Usługa Azure RBAC kontroluje dostęp do zasobów na platformie Azure (i w centrum Azure Stack), w tym możliwość tworzenia nowych zasobów platformy Azure. Uprawnienia można przypisać do użytkowników, grup lub podmiotów usługi. (Jednostka usługi jest tożsamością zabezpieczeń używaną przez aplikacje).
- Kubernetes formant RBAC kontroluje uprawnienia do interfejsu API Kubernetes. Na przykład tworzenie i wystawianie zasobników to akcje, które mogą być autoryzowane (lub odrzucane) do użytkownika za pomocą RBAC. Aby przypisać uprawnienia Kubernetes użytkownikom, tworzysz role i powiązania ról.

**Tożsamość centrum Azure Stack i kontrola RBAC**

Azure Stack Hub udostępnia dwie opcje dostawcy tożsamości. Używany dostawca zależy od środowiska i od tego, czy działa w środowisku podłączonym lub rozłączonym:

- Azure AD — może być używana tylko w podłączonym środowisku.
- Usługi ADFS do tradycyjnego Active Directory lasu — mogą być używane zarówno w środowisku podłączonym, jak i rozłączonym.

Dostawca tożsamości zarządza użytkownikami i grupami, w tym uwierzytelnianie i autoryzację w celu uzyskiwania dostępu do zasobów. Można udzielić dostępu do zasobów centrum Azure Stack, takich jak subskrypcje, grupy zasobów i poszczególne zasoby, takie jak maszyny wirtualne lub moduły równoważenia obciążenia. Aby mieć spójny model dostępu, należy rozważyć użycie tych samych grup (bezpośrednich lub zagnieżdżonych) dla wszystkich centrów Azure Stack. Oto przykład konfiguracji:

![Zagnieżdżone grupy usługi AAD za pomocą usługi Azure Stack Hub](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

Przykład zawiera dedykowaną grupę (przy użyciu usługi AAD lub ADFS) do określonego celu. Na przykład, aby zapewnić uprawnienia współautora dla grupy zasobów zawierającej infrastrukturę klastra Kubernetes w określonym wystąpieniu centrum Azure Stack (w tym miejscu "Współautor K8s klastra"). Te grupy są następnie zagnieżdżane w grupie ogólnej zawierającej "podgrupy" poszczególnych centrów Azure Stack.

Nasz przykładowy użytkownik będzie miał teraz uprawnienia "Współautor" do obu grup zasobów, które zawierają cały zestaw zasobów infrastruktury Kubernetes. Użytkownik będzie miał dostęp do zasobów w obu wystąpieniach usługi Azure Stack Hub, ponieważ wystąpienia te korzystają z tego samego dostawcy tożsamości.

> [!IMPORTANT]
> Te uprawnienia mają wpływ tylko na centrum Azure Stack i niektóre z zasobów wdrożonych na tym komputerze. Użytkownik mający ten poziom dostępu może wykonywać wiele szkód, ale nie może uzyskać dostępu do maszyn wirtualnych Kubernetes IaaS ani do interfejsu API Kubernetes bez dodatkowego dostępu do wdrożenia Kubernetes.

**Kubernetes tożsamość i RBAC**

Klaster Kubernetes domyślnie nie używa tego samego dostawcy tożsamości co centrum Azure Stack. Maszyny wirtualne obsługujące klaster Kubernetes, główny i węzły procesu roboczego używają klucza SSH określonego podczas wdrażania klastra. Ten klucz SSH jest wymagany do nawiązania połączenia z tymi węzłami przy użyciu protokołu SSH.

Interfejs API Kubernetes (na przykład dostęp za pomocą programu `kubectl` ) jest również chroniony przez konta usług, w tym domyślne konto usługi "administrator klastra". Poświadczenia dla tego konta usługi są początkowo przechowywane w `.kube/config` pliku w węzłach głównych Kubernetes.

**Zarządzanie kluczami tajnymi i poświadczenia aplikacji**

Aby przechowywać wpisy tajne, takie jak parametry połączenia lub poświadczenia bazy danych, można wybrać kilka opcji, w tym:

- W usłudze Azure Key Vault
- Wpisy tajne usługi Kubernetes
- rozwiązania innych firm, takie jak magazyn HashiCorp (działające na Kubernetes)

Nie przechowuj wpisów tajnych ani poświadczeń w postaci zwykłego tekstu w plikach konfiguracyjnych, kodzie aplikacji ani w skryptach. Nie należy przechowywać ich w systemie kontroli wersji. Zamiast tego Automatyzacja wdrażania powinna pobrać wpisy tajne zgodnie z potrzebami.

## <a name="patch-and-update"></a>Poprawka i aktualizacja

Proces **poprawek i aktualizacji (PNU)** w usłudze Azure Kubernetes Service jest częściowo zautomatyzowany. Uaktualnienia wersji Kubernetes są wyzwalane ręcznie, podczas gdy aktualizacje zabezpieczeń są stosowane automatycznie. Te aktualizacje mogą obejmować poprawki zabezpieczeń systemu operacyjnego lub aktualizacje jądra. AKS nie uruchamia automatycznie ponownie tych węzłów systemu Linux w celu ukończenia procesu aktualizacji. 

Proces PNU klastra Kubernetes wdrożony przy użyciu aparatu AKS na Azure Stack Hub jest niezarządzany i jest odpowiedzialny za operator klastra. 

Aparat AKSa pomaga z dwoma najważniejszymi zadaniami:

- [Uaktualnij do nowszej wersji obrazu systemu operacyjnego Kubernetes i podstawowej](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Uaktualnij tylko podstawowy obraz systemu operacyjnego](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Nowsze obrazy podstawowe systemu operacyjnego zawierają najnowsze poprawki zabezpieczeń systemu operacyjnego i aktualizacje jądra. 

Mechanizm [nienadzorowanego uaktualniania](https://wiki.debian.org/UnattendedUpgrades) automatycznie instaluje aktualizacje zabezpieczeń wydane przed udostępnieniem nowej wersji obrazu podstawowego systemu operacyjnego w portalu Azure Stack Hub. Uaktualnienie nienadzorowane jest domyślnie włączone i automatycznie instaluje aktualizacje zabezpieczeń, ale nie uruchamia ponownie węzłów klastra Kubernetes. Ponowny rozruch węzłów można zautomatyzować za pomocą polecenia "Open Source [ **K** Ubernetes reboot **D** aemon (kured) **RE**](/azure/aks/node-updates-kured)". Kured czujki dla węzłów systemu Linux, które wymagają ponownego uruchomienia, a następnie automatycznie obsłużyć ponowne planowanie procesów uruchamiania i ponownego uruchamiania węzła.

## <a name="deployment-cicd-considerations"></a>Zagadnienia dotyczące wdrażania (CI/CD)

Usługa Azure i Azure Stack Hub uwidaczniają te same Azure Resource Manager interfejsy API REST. Te interfejsy API są podobne do wszystkich innych chmur platformy Azure (Azure, Azure Chiny 21Vianet, Azure Government). Mogą wystąpić różnice w wersjach interfejsu API między chmurami, a usługa Azure Stack Hub udostępnia tylko podzestaw usług. Identyfikator URI punktu końcowego zarządzania jest również różny dla każdej chmury i każdego wystąpienia Azure Stack Hub.

Oprócz niewielkich różnic między wymienionymi Azure Resource Manager interfejsy API REST zapewniają spójny sposób współpracy z usługą Azure i Azure Stack Hub. Ten sam zestaw narzędzi może być używany w tym miejscu, jak w przypadku dowolnej innej chmury platformy Azure. Do wdrażania i organizowania usług w usłudze Azure Stack Hub można używać platformy Azure DevOps, narzędzi takich jak Jenkins lub PowerShell.

**Zagadnienia do rozważenia**

Jedną z głównych różnic, gdy chodzi o wdrażanie w centrum Azure Stack, jest pytanie o dostępność Internetu. Dostępność internetowa określa, czy należy wybrać agenta kompilacji hostowanego przez firmę Microsoft lub samodzielnego dla zadań CI/CD.

Agent samoobsługowy może działać na Azure Stack centrum (jako maszyna wirtualna IaaS) lub w podsieci sieciowej, która ma dostęp do centrum Azure Stack. Przejdź do [Azure Pipelines agentów](/azure/devops/pipelines/agents/agents) , aby dowiedzieć się więcej o różnicach.

Poniższa ilustracja ułatwia podjęcie decyzji o tym, czy potrzebujesz samodzielnie hostowanego lub obsługiwanego przez firmę Microsoft agenta kompilacji:

![Samodzielne agenci kompilacji tak lub nie](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Czy punkty końcowe zarządzania centrum Azure Stack są dostępne za pośrednictwem Internetu?
  - Tak: możemy użyć Azure Pipelines z agentami hostowanymi przez firmę Microsoft w celu nawiązania połączenia z centrum Azure Stack.
  - Nie: potrzebujemy własnych agentów, którzy mogą łączyć się z punktami końcowymi zarządzania centrum Azure Stack.
- Czy nasz klaster Kubernetes jest dostępny za pośrednictwem Internetu?
  - Tak: możemy używać Azure Pipelines z agentami obsługiwanymi przez firmę Microsoft do bezpośredniej współpracy z punktem końcowym interfejsu API Kubernetes.
  - Nie: potrzebujemy własnych agentów, którzy mogą łączyć się z punktem końcowym interfejsu API klastra Kubernetes.

W scenariuszach, w których punkty końcowe zarządzania centrum Azure Stack i interfejs API Kubernetes są dostępne za pośrednictwem Internetu, wdrożenie może korzystać z agenta hostowanego przez firmę Microsoft. To wdrożenie spowoduje zastosowanie architektury aplikacji w następujący sposób:

[![Omówienie architektury publicznej](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Jeśli Azure Resource Manager punkty końcowe, interfejs API Kubernetes lub oba nie są bezpośrednio dostępne za pośrednictwem Internetu, możemy wykorzystać samoobsługowy agent kompilacji do uruchomienia kroków potoku. Ten projekt wymaga mniej łączności i może zostać wdrożony tylko przy użyciu lokalnej łączności sieciowej do Azure Resource Manager punktów końcowych i interfejsu API Kubernetes:

[![Omówienie architektury Premium](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Co z niepołączonymi scenariuszami?** W scenariuszach, w których Azure Stack Hub lub Kubernetes lub oba z nich nie mają punktów końcowych zarządzania dostępnych z Internetu, nadal można używać usługi Azure DevOps dla wdrożeń. Możesz użyć samodzielnie hostowanej puli agentów (jest to Agent DevOps działający lokalnie lub w centrum Azure Stack) lub całkowicie hostowane Azure DevOps Server w środowisku lokalnym. Agent samoobsługowy musi tylko wychodzące połączenie z Internetem HTTPS (TCP/443).

Wzorzec może korzystać z klastra Kubernetes (wdrożonego i zorganizowanego z aparatem AKS) w każdym wystąpieniu Azure Stack Hub. Obejmuje ona aplikację składającą się z frontonu, usług zaplecza (na przykład MongoDB) i kontrolera danych wejściowych opartych na Nginx. Zamiast korzystać z bazy danych hostowanej w klastrze K8s, można wykorzystać "zewnętrzne magazyny danych". Opcje bazy danych obejmują MySQL, SQL Server lub dowolny rodzaj bazy danych hostowanej poza centrum Azure Stack lub w IaaS. Takie konfiguracje nie należą do zakresu.

## <a name="partner-solutions"></a>Rozwiązania partnerskie

Istnieją rozwiązania partnerskie firmy Microsoft, które mogą zwiększyć możliwości Centrum Azure Stack. Te rozwiązania okazały się przydatne w przypadku wdrożeń aplikacji działających w klastrach Kubernetes.  

## <a name="storage-and-data-solutions"></a>Magazyn i rozwiązania dotyczące danych

Zgodnie z opisem w temacie [zagadnienia dotyczące danych i magazynowania](#data-and-storage-considerations), centrum Azure Stack obecnie nie ma natywnego rozwiązania do replikowania magazynu między wieloma wystąpieniami. W przeciwieństwie do platformy Azure, możliwość replikowania magazynu w wielu regionach nie istnieje. W centrum Azure Stack każde wystąpienie jest własną odrębną chmurą. Dostępne są jednak rozwiązania partnerów firmy Microsoft, które umożliwiają replikację magazynu między centrami Azure Stack i platformą Azure. 

**SCALITY**

[Scality](https://www.scality.com/) zapewnia magazyn w skali sieci Web, który ma napędzane firmy cyfrowe od 2009. PIERŚCIEŃ Scality, nasz magazyn zdefiniowany przez oprogramowanie, włącza serwery z procesorami x86 do nieograniczonej puli magazynów dla dowolnego typu danych — plik i obiekt — na skalę petabajtów.

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) upraszcza magazyn przedsiębiorstwa z nieograniczonym skalowalnym magazynem, który konsoliduje duże zestawy danych do jednego, łatwo zarządzanego środowiska.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat pojęć wprowadzonych w tym artykule:

- [Skalowanie między chmurami](pattern-cross-cloud-scale.md) i [wzorce aplikacji rozproszonych geograficznie](pattern-geo-distributed.md) w centrum Azure Stack.
- [Architektura mikrousług w usłudze Azure Kubernetes Service (AKS)](/azure/architecture/reference-architectures/microservices/aks).

Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika wdrażania klastra Kubernetes o wysokiej dostępności](solution-deployment-guide-highly-available-kubernetes.md). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.