---
title: Wzorzec Kubernetes o wysokiej dostępności korzystający z platformy Azure i Azure Stack Hub
description: Dowiedz się, jak rozwiązanie klastra Kubernetes zapewnia wysoką dostępność przy użyciu platformy Azure i Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281316"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Wzorzec klastra Kubernetes o wysokiej dostępności

W tym artykule opisano sposób projektowania i obsługi infrastruktury opartej na platformie Kubernetes o wysokiej Azure Kubernetes Service (AKS) na platformie Azure Stack Hub. Ten scenariusz jest często spotykany w organizacjach z obciążeniami o krytycznym znaczeniu w środowiskach z ograniczeniami i regulacjami. Organizacje w domenach, takich jak finanse, obronność i instytucje rządowe.

## <a name="context-and-problem"></a>Kontekst i problem

Wiele organizacji opracowuje rozwiązania natywne dla chmury, które wykorzystują najnowocześniejsze usługi i technologie, takie jak Kubernetes. Mimo że platforma Azure zapewnia centra danych w większości regionów świata, czasami istnieją przypadki użycia brzegowe i scenariusze, w których aplikacje krytyczne dla działania firmy muszą działać w określonej lokalizacji. Zagadnienia do rozważenia:

- Czułość lokalizacji
- Opóźnienie między aplikacją i systemami lokalnymi
- Przepustowość
- Łączność
- Wymagania prawne lub prawne

Platforma Azure w połączeniu z Azure Stack Hub rozwiązuje większość z tych problemów. Poniżej opisano szeroki zestaw opcji, decyzji i kwestii dotyczących pomyślnej implementacji rozwiązania Kubernetes uruchomionego na platformie Azure Stack Hub.

## <a name="solution"></a>Rozwiązanie

W tym wzorcu przyjęto założenie, że musimy poradzić sobie ze ścisłym zestawem ograniczeń. Aplikacja musi działać lokalnie, a wszystkie dane osobowe nie mogą docierać do usług w chmurze publicznej. Dane monitorowania i inne dane inne niż piI mogą być wysyłane na platformę Azure i tam przetwarzane. Usługi zewnętrzne, takie jak Container Registry publiczne lub inne, są dostępne, ale mogą być filtrowane przez zaporę lub serwer proxy.

Pokazana tutaj przykładowa aplikacja (oparta na Azure Kubernetes Service [Workshop)](/learn/modules/aks-workshop/)jest przeznaczona do używania rozwiązań natywnych dla rozwiązania Kubernetes zawsze, gdy jest to możliwe. Ten projekt pozwala uniknąć blokady dostawcy zamiast korzystania z usług natywnych dla platformy. Na przykład aplikacja używa własnego zaplecza bazy danych MongoDB zamiast usługi PaaS lub zewnętrznej bazy danych.

[![Hybrydowy wzorzec aplikacji](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

Na powyższym diagramie przedstawiono architekturę aplikacji przykładowej działającej na kubernetes na Azure Stack Hub. Aplikacja składa się z kilku składników, w tym:

 1) Klaster Kubernetes oparty na aksiecie AKS na Azure Stack Hub.
 2) [cert-manager](https://www.jetstack.io/cert-manager/), który udostępnia zestaw narzędzi do zarządzania certyfikatami na platformie Kubernetes, używany do automatycznego żądania certyfikatów z usługi Let's Encrypt.
 3) Przestrzeń nazw Kubernetes zawierająca składniki aplikacji dla frontonia (ratings-web), interfejsu API (ratings-api) i bazy danych (ratings-mongodb).
 4) Kontroler ruchu przychodzących, który kieruje ruch HTTP/HTTPS do punktów końcowych w klastrze Kubernetes.

Przykładowa aplikacja służy do zilustrowania architektury aplikacji. Wszystkie składniki są przykładami. Architektura zawiera tylko jedno wdrożenie aplikacji. W celu osiągnięcia wysokiej dostępności (HA) będziemy uruchamiać wdrożenie co najmniej dwa razy w dwóch różnych wystąpieniach usługi Azure Stack Hub — mogą one działać w tej samej lokalizacji lub w co najmniej dwóch różnych lokacjach:

![Architektura infrastruktury](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Usługi takie Azure Container Registry, Azure Monitor i inne są hostowane poza Azure Stack Hub na platformie Azure lub lokalnie. Ten projekt hybrydowy chroni rozwiązanie przed ową Azure Stack Hub wystąpienia.

## <a name="components"></a>Składniki

Ogólna architektura składa się z następujących składników:

**Azure Stack Hub** to rozszerzenie platformy Azure, które może uruchamiać obciążenia w środowisku lokalnym, zapewniając usługi platformy Azure w centrum danych. Przejdź do [Azure Stack Hub omówienie,](/azure-stack/operator/azure-stack-overview) aby dowiedzieć się więcej.

**Azure Kubernetes Service Engine (AKS Engine)** to aparat zarządzanych ofert usługi Kubernetes, Azure Kubernetes Service (AKS), który jest obecnie dostępny na platformie Azure. Na Azure Stack Hub usługa AKS Engine umożliwia wdrażanie, skalowanie i uaktualnianie w pełni funkcjonalnych, samodzielnie zarządzanych klastrów Kubernetes przy użyciu możliwości usługi Azure Stack Hub IaaS firmy Azure Stack Hub. Aby dowiedzieć się więcej, przejdź do tematu Omówienie aparatu [AKS.](https://github.com/Azure/aks-engine)

Przejdź do [tematu Znane problemy i ograniczenia,](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) aby dowiedzieć się więcej o różnicach między aparatem AKS na platformie Azure a aparatem AKS na platformie Azure Stack Hub.

**Usługa Azure Virtual Network (VNet)** służy do zapewnienia infrastruktury sieciowej na każdym serwerze Azure Stack Hub dla maszyn wirtualnych Virtual Machines hostującym infrastrukturę klastra Kubernetes.

**Azure Load Balancer** jest używana dla punktu końcowego interfejsu API kubernetes i kontrolera ruchu przychodzących Nginx. Usługa równoważenia obciążenia kieruje zewnętrzny (na przykład internet) ruch do węzłów i maszyn wirtualnych oferujących określoną usługę.

**Azure Container Registry (ACR)** służy do przechowywania prywatnych obrazów platformy Docker i wykresów Helm, które są wdrażane w klastrze. Aparat usługi AKS może uwierzytelniać się za pomocą Container Registry przy użyciu tożsamości usługi Azure AD. Kubernetes nie wymaga ACR. Możesz użyć innych rejestrów kontenerów, takich jak Docker Hub.

**Azure Repos** to zestaw narzędzi kontroli wersji, których można użyć do zarządzania kodem. Możesz również użyć GitHub lub innych repozytoriów opartych na usłudze Git. Przejdź do [Azure Repos Omówienie,](/azure/devops/repos/get-started/what-is-repos) aby dowiedzieć się więcej.

**Azure Pipelines** jest częścią Azure DevOps Services i uruchamia zautomatyzowane kompilacje, testy i wdrożenia. Można również użyć rozwiązań do cicha/cd innych firm, takich jak Jenkins. Aby dowiedzieć się więcej, przejdź do tematu Omówienie usługi [Azure Pipeline.](/azure/devops/pipelines/get-started/what-is-azure-pipelines)

**Azure Monitor** zbiera i przechowuje metryki i dzienniki, w tym metryki platformy dla usług platformy Azure w rozwiązaniu i telemetrii aplikacji. Te dane są wykorzystywane do monitorowania aplikacji, konfigurowanie alertów i pulpitów nawigacyjnych oraz analizowanie głównych przyczyn błędów. Azure Monitor integruje się z rozwiązaniami Kubernetes w celu zbierania metryk z kontrolerów, węzłów i kontenerów, a także dzienników kontenerów i dzienników węzłów głównych. Przejdź do [Azure Monitor Omówienie,](/azure/azure-monitor/overview) aby dowiedzieć się więcej.

**Azure Traffic Manager** to oparty na systemie DNS równoważenie obciążenia ruchu, który umożliwia optymalną dystrybucję ruchu do usług w różnych regionach platformy Azure lub Azure Stack Hub wdrożeniach. Traffic Manager zapewnia również wysoką dostępność i czas odpowiedzi. Punkty końcowe aplikacji muszą być dostępne z zewnątrz. Dostępne są również inne rozwiązania lokalne.

**Kontroler ruchu przychodzących Kubernetes** uwidacznia trasy HTTP(S) usługom w klastrze Kubernetes. W tym celu można użyć Nginx lub dowolnego odpowiedniego kontrolera ruchu wychodzącego.

**Helm** to menedżer pakietów dla wdrażania na platformie Kubernetes, który zapewnia sposób na wiązanie różnych obiektów Kubernetes, takich jak wdrożenia, usługi i wpisy tajne, w jeden "wykres". Można publikować, wdrażać i kontrolować zarządzanie wersjami oraz aktualizować obiekt wykresu. Azure Container Registry mogą służyć jako repozytorium do przechowywania spakowanego pakietu pakietów Helm.

## <a name="design-considerations"></a>Zagadnienia dotyczące projektowania

Ten wzorzec jest zgodny z kilkoma zagadnieniami wysokiego poziomu, które wyjaśniono bardziej szczegółowo w kolejnych sekcjach tego artykułu:

- Aplikacja używa rozwiązań natywnych dla rozwiązania Kubernetes, aby uniknąć blokady dostawcy.
- Aplikacja używa architektury mikrousług.
- Azure Stack Hub nie wymaga ruchu przychodzącego, ale zezwala na wychodzącą łączność internetową.

Te zalecane rozwiązania będą stosowane również w rzeczywistych obciążeniach i scenariuszach.

## <a name="scalability-considerations"></a>Zagadnienia dotyczące skalowalności

Skalowalność jest ważna, aby zapewnić użytkownikom spójny, niezawodny i wydajny dostęp do aplikacji.

Przykładowy scenariusz obejmuje skalowalność na wielu warstwach stosu aplikacji. Poniżej przedstawiono ogólne omówienie różnych warstw:

| Poziom architektury | Wpływa | Jak to zrobić? |
| --- | --- | ---
| Aplikacja | Aplikacja | Skalowanie w poziomie na podstawie liczby zasobników/replik/Container Instances* |
| Klaster | Klaster Kubernetes | Liczba węzłów (od 1 do 50), rozmiary SKU maszyny wirtualnej i pule węzłów (aparat AKS na platformie Azure Stack Hub obecnie obsługuje tylko jedną pulę węzłów); za pomocą polecenia skalowania aparatu AKS (ręczne) |
| Infrastruktura | Azure Stack Hub | Liczba węzłów, pojemności i jednostek skalowania w ramach Azure Stack Hub wdrożenia |

\* Korzystanie z narzędzia Horizontal Pod Autoscaler (HPA) rozwiązania Kubernetes; automatyczne skalowanie oparte na metrykach lub skalowanie w pionie przez określanie rozmiaru wystąpień kontenera (procesor/pamięć).

**Azure Stack Hub (poziom infrastruktury)**

Infrastruktura Azure Stack Hub jest podstawą tej implementacji, ponieważ Azure Stack Hub działa na sprzęcie fizycznym w centrum danych. Podczas wybierania sprzętu koncentratora należy wybrać procesor CPU, gęstość pamięci, konfigurację magazynu i liczbę serwerów. Aby dowiedzieć się Azure Stack Hub o skalowalności tej firmy, zapoznaj się z następującymi zasobami:

- [Planowanie pojemności na Azure Stack Hub omówienie](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [Dodawanie dodatkowych węzłów jednostki skalowania w Azure Stack Hub](/azure-stack/operator/azure-stack-add-scale-node)

**Klaster Kubernetes (poziom klastra)**

Sam klaster Kubernetes składa się z i jest zbudowany na podstawie składników IaaS platformy Azure (Stack), w tym zasobów obliczeniowych, magazynowych i sieciowych. Rozwiązania Kubernetes obejmują węzły główne i robocze, które są wdrażane jako maszyny wirtualne na platformie Azure (i Azure Stack Hub).

- [Węzły płaszczyzny](/azure/aks/concepts-clusters-workloads#control-plane) sterowania (master) zapewniają podstawowe usługi Kubernetes i aranżację obciążeń aplikacji.
- [Węzły procesu](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) roboczego (proces roboczy) uruchamiają obciążenia aplikacji.

Podczas wybierania rozmiarów maszyn wirtualnych dla początkowego wdrożenia należy wziąć pod uwagę kilka kwestii:  

- **Koszt** — podczas planowania węzłów procesu roboczego należy pamiętać o ogólnym kosztie maszyny wirtualnej, który zostanie naliczony. Jeśli na przykład obciążenia aplikacji wymagają ograniczonych zasobów, należy zaplanować wdrożenie maszyn wirtualnych o mniejszym rozmiarze. Azure Stack Hub, na przykład platforma Azure, jest zwykle rozliczana na podstawie zużycia, dlatego odpowiednie dobór rozmiaru maszyn wirtualnych dla ról kubernetes ma kluczowe znaczenie dla optymalizacji kosztów użycia. 

- **Skalowalność** — skalowalność klastra jest osiągana przez skalowanie do i na zewnątrz liczby węzłów głównych i procesów roboczych lub przez dodanie dodatkowych pul węzłów (niedostępne Azure Stack Hub obecnie). Skalowanie klastra można wykonać na podstawie danych wydajności zebranych przy użyciu usługi Container Szczegółowe informacje (Azure Monitor + Log Analytics). 

    Jeśli aplikacja potrzebuje więcej (lub mniej) zasobów, możesz skalować bieżące węzły w poziomie (od 1 do 50 węzłów). Jeśli potrzebujesz więcej niż 50 węzłów, możesz utworzyć dodatkowy klaster w oddzielnej subskrypcji. Nie można skalować w górę rzeczywistych maszyn wirtualnych w pionie do innego rozmiaru maszyny wirtualnej bez ponownego wdniania klastra.

    Skalowanie odbywa się ręcznie przy użyciu maszyny wirtualnej pomocnika aparatu usługi AKS, która została użyta do początkowego wdrożenia klastra Kubernetes. Aby uzyskać więcej informacji, zobacz [Scaling Kubernetes clusters (Skalowanie klastrów Kubernetes)](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Limity** przydziału — weź pod uwagę [limity przydziału](/azure-stack/operator/azure-stack-quota-types) skonfigurowane podczas planowania wdrożenia usługi AKS na Azure Stack Hub. Upewnij się, [że każda subskrypcja](/azure-stack/operator/service-plan-offer-subscription-overview) ma skonfigurowane odpowiednie plany i przydziały. Subskrypcja musi uwzględniać ilość zasobów obliczeniowych, magazynu i innych usług wymaganych dla klastrów podczas skalowania w poziomie.

- **Obciążenia aplikacji —** zapoznaj się z pojęciami [klastrów](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) i obciążeń w podstawowym dokumencie Kubernetes core concepts for Azure Kubernetes Service document (Podstawowe pojęcia dotyczące klastrów i obciążeń). Ten artykuł pomoże Ci w zakresie odpowiedniego rozmiaru maszyny wirtualnej w zależności od potrzeb obliczeniowych i pamięci aplikacji.  

**Aplikacja (poziom aplikacji)**

W warstwie aplikacji używamy rozwiązania Kubernetes [Horizontal Pod Autoscaler (HPA).](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) Rozwiązanie HPA może zwiększyć lub zmniejszyć liczbę replik (Zasobnik/Container Instances) we wdrożeniu na podstawie różnych metryk (takich jak użycie procesora CPU).

Inną opcją jest skalowanie wystąpień kontenerów w pionie. Można to osiągnąć, zmieniając ilość procesora CPU i pamięci żądaną i dostępną dla określonego wdrożenia. Aby dowiedzieć [się więcej,](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) zobacz Managing Resources for Containers on kubernetes.io (Zarządzanie zasobami dla kontenerów na platformie).

## <a name="networking-and-connectivity-considerations"></a>Zagadnienia dotyczące sieci i łączności

Sieć i łączność mają również wpływ na trzy warstwy wymienione wcześniej dla usługi Kubernetes na platformie Azure Stack Hub. W poniższej tabeli przedstawiono warstwy i usługi, które zawierają:

| Warstwa | Wpływa | Co? |
| --- | --- | ---
| Aplikacja | Aplikacja | W jaki sposób aplikacja jest dostępna? Czy zostanie on ujawniony w Internecie? |
| Klaster | Klaster Kubernetes | Interfejs API kubernetes, maszyna wirtualna aparatu usługi AKS, ściąganie obrazów kontenera (ruch wychodzący), wysyłanie danych monitorowania i telemetrii (ruch wychodzący) |
| Infrastruktura | Azure Stack Hub | Dostępność punktów końcowych Azure Stack Hub zarządzania, takich jak portal i Azure Resource Manager końcowe. |

**Aplikacja**

W przypadku warstwy aplikacji najważniejszą kwestią jest to, czy aplikacja jest dostępna i dostępna z Internetu. Z perspektywy rozwiązania Kubernetes dostępność internetowa oznacza uwidnie wdrożenia lub zasobnika przy użyciu usługi Kubernetes Service lub kontrolera ruchu wychodzącego.

> [!NOTE]
> Zalecamy używanie kontrolerów ruchu przychodzących do uwidocznień usług Kubernetes, ponieważ liczba publicznych ip frontonu na Azure Stack Hub jest ograniczona do 5. Ten projekt ogranicza również liczbę usług Kubernetes (z typem LoadBalancer) do 5, co będzie zbyt małe dla wielu wdrożeń. Przejdź do dokumentacji [aparatu AKS,](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) aby dowiedzieć się więcej.

Udostępnianie aplikacji przy użyciu publicznego adresu IP za pośrednictwem Load Balancer lub kontrolera ruchu wychodzącego nie oznacza, że aplikacja jest teraz dostępna za pośrednictwem Internetu. Istnieje możliwość, Azure Stack Hub publiczny adres IP, który jest widoczny tylko w lokalnym intranecie — nie wszystkie publiczne adresy IP są rzeczywiście dostępne z Internetu.

Poprzedni blok uwzględnia ruch przychodzący do aplikacji. Innym tematem, który należy rozważyć w celu pomyślnego wdrożenia rozwiązania Kubernetes, jest ruch wychodzący/wychodzący. Oto kilka przypadków użycia, które wymagają ruchu wychodzącego:

- Ściąganie obrazów kontenerów przechowywanych w usłudze DockerHub lub Azure Container Registry
- Pobieranie wykresów Helm
- Emitowanie danych Szczegółowe informacje aplikacji (lub innych danych monitorowania)

Niektóre środowiska przedsiębiorstwa mogą wymagać użycia _przezroczystych_ lub _nieprzezroczystych serwerów_ proxy. Te serwery wymagają konkretnej konfiguracji dla różnych składników klastra. Dokumentacja aparatu AKS zawiera różne szczegóły dotyczące sposobu przystosowania sieciowych serwerów proxy. Aby uzyskać więcej informacji, zobacz [AKS Engine and proxy servers (Aparat usługi AKS i serwery proxy)](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Na koniec ruch między klastrami musi przepływać między Azure Stack Hub wystąpieniami. Przykładowe wdrożenie składa się z pojedynczych klastrów Kubernetes uruchomionych w poszczególnych Azure Stack Hub wystąpieniach. Ruch między nimi, taki jak ruch replikacji między dwiema bazami danych, jest "ruchem zewnętrznym". Ruch zewnętrzny musi być przekierowyny przez sieć VPN typu lokacja-lokacja lub Azure Stack Hub publiczne adresy IP, aby połączyć platformę Kubernetes w dwóch Azure Stack Hub wystąpieniach:

![ruch między klastrami i wewnątrz klastra](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Klaster**  

Klaster Kubernetes nie musi być dostępny za pośrednictwem Internetu. Odpowiednią częścią jest interfejs API kubernetes używany do obsługi klastra, na przykład przy użyciu interfejsu `kubectl` . Punkt końcowy interfejsu API kubernetes musi być dostępny dla wszystkich użytkowników, którzy działają w klastrze lub wdrażają na jego podstawie aplikacje i usługi. Ten temat opisano bardziej szczegółowo z perspektywy DevOps w sekcji Zagadnienia dotyczące wdrażania [(CI/CD)](#deployment-cicd-considerations) poniżej.

Na poziomie klastra istnieje również kilka kwestii, które należy wziąć pod uwagę w przypadku ruchu wychodzącego:

- Aktualizacje węzła (dla systemu Ubuntu)
- Dane monitorowania (wysyłane do usługi Azure LogAnalytics)
- Inni agenci wymagający ruchu wychodzącego (specyficznego dla środowiska każdego wdrażającego)

Przed wdrożeniem klastra Kubernetes przy użyciu aparatu usługi AKS zaplanuj ostateczny projekt sieci. Zamiast tworzyć dedykowane Virtual Network, wydajniejsze może być wdrożenie klastra w istniejącej sieci. Można na przykład korzystać z istniejącego połączenia sieci VPN typu lokacja-lokacja już skonfigurowanego w środowisku Azure Stack Hub lokacji.

**Infrastruktura**  

Infrastruktura odnosi się do uzyskiwania dostępu do Azure Stack Hub końcowych zarządzania. Punkty końcowe obejmują dzierżawę i portale administracyjne oraz Azure Resource Manager i punkty końcowe dzierżawy. Te punkty końcowe są wymagane do obsługi Azure Stack Hub i jej podstawowych usług.

## <a name="data-and-storage-considerations"></a>Zagadnienia dotyczące danych i magazynu

Dwa wystąpienia naszej aplikacji zostaną wdrożone w dwóch pojedynczych klastrach Kubernetes w dwóch Azure Stack Hub wystąpieniach. Ten projekt będzie wymagał zastanowienia się nad tym, jak replikować i synchronizować dane między nimi.

Platforma Azure ma wbudowaną funkcję replikowania magazynu w wielu regionach i strefach w chmurze. Obecnie w Azure Stack Hub nie ma natywnych sposobów replikowania magazynu między dwoma różnymi wystąpieniami usługi Azure Stack Hub — tworzą one dwie niezależne chmury bez overarchingu, aby zarządzać nimi jako zestawem. Planowanie odporności aplikacji działających w różnych Azure Stack Hub wymusza rozważenie tej niezależność w projekcie i wdrożeniach aplikacji.

W większości przypadków replikacja magazynu nie będzie konieczna w przypadku odpornej i wysoce dostępnej aplikacji wdrożonej w u usługi AKS. Jednak w projekcie aplikacji należy rozważyć Azure Stack Hub magazynu na wystąpienie. Jeśli ten projekt stanowi problem lub stanowi problem podczas wdrażania rozwiązania na platformie Azure Stack Hub, istnieją możliwe rozwiązania od partnerów firmy Microsoft, które dostarczają załączniki do magazynu. Storage zapewniają rozwiązanie replikacji magazynu w wielu centrach Azure Stack Hub i na platformie Azure. Aby uzyskać więcej informacji, zobacz [Rozwiązania partnerskie.](#partner-solutions)

W naszej architekturze zostały rozważone następujące warstwy:

**Konfiguracja**

Konfiguracja obejmuje konfigurację Azure Stack Hub, aparatu AKS i samego klastra Kubernetes. Konfiguracja powinna być jak najbardziej zautomatyzowana i przechowywana jako infrastruktura jako kod w systemie kontroli wersji opartym na usłudze Git, Azure DevOps lub GitHub. Tych ustawień nie można łatwo zsynchronizować w wielu wdrożeniach. W związku z tym zalecamy przechowywanie i stosowanie konfiguracji z zewnątrz i używanie DevOps potoku.

**Aplikacja**

Aplikacja powinna być przechowywana w repozytorium opartym na usłudze Git. Zawsze, gdy istnieje nowe wdrożenie, zmiany w aplikacji lub odzyskiwanie po awarii, można je łatwo wdrożyć przy użyciu Azure Pipelines.

**Dane**

Dane są najważniejszą kwestią w większości projektów aplikacji. Dane aplikacji muszą być zsynchronizowane między różnymi wystąpieniami aplikacji. W przypadku awarii dane również muszą mieć strategię tworzenia kopii zapasowych i odzyskiwania po awarii.

Osiągnięcie tego projektu w dużym stopniu zależy od wyboru technologii. Oto kilka przykładów rozwiązań do implementowania bazy danych w sposób wysoce dostępny na Azure Stack Hub:

- [Wdrażanie grupy dostępności SQL Server 2016 na platformie Azure i Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Wdrażanie rozwiązania Bazy danych MongoDB o wysokiej Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

Zagadnienia dotyczące pracy z danymi w wielu lokalizacjach są jeszcze bardziej złożonymi zagadnieniami dla rozwiązania o wysokiej dostępnej i odpornej na błędy. Rozważ następujące kwestie:

- Opóźnienie i łączność sieciowa między Azure Stack Hubs.
- Dostępność tożsamości dla usług i uprawnień. Każde Azure Stack Hub integruje się z katalogiem zewnętrznym. Podczas wdrażania można użyć usługi Azure Active Directory (Azure AD) lub Active Directory Federation Services (ADFS). W związku z tym istnieje możliwość użycia jednej tożsamości, która może wchodzić w interakcję z wieloma niezależnymi Azure Stack Hub wystąpieniami.

## <a name="business-continuity-and-disaster-recovery"></a>Ciągłość działania i odzyskiwanie po awarii

Ciągłość działalności biznesowej i odzyskiwanie po awarii (BCDR) to ważny temat zarówno w Azure Stack Hub, jak i na platformie Azure. Główna różnica polega na tym, Azure Stack Hub operator musi zarządzać całym procesem BCDR. Na platformie Azure części bcdr są automatycznie zarządzane przez firmę Microsoft.

BCDR ma wpływ na te same obszary wymienione w poprzedniej sekcji Zagadnienia dotyczące danych [i magazynu:](#data-and-storage-considerations)

- Infrastruktura/konfiguracja
- Dostępność aplikacji
- Dane aplikacji

Jak wspomniano w poprzedniej sekcji, te obszary są obowiązkiem operatora Azure Stack Hub i mogą się różnić między organizacjami. Plan BCDR zgodnie z dostępnymi narzędziami i procesami.

**Infrastruktura i konfiguracja**

Ta sekcja obejmuje infrastrukturę fizyczną i logiczną oraz konfigurację Azure Stack Hub. Obejmuje on akcje w przestrzeni administratora i dzierżawy.

Operator Azure Stack Hub (lub administrator) jest odpowiedzialny za konserwację Azure Stack Hub wystąpień. W tym składniki, takie jak sieć, magazyn, tożsamość i inne tematy, które są poza zakresem tego artykułu. Aby dowiedzieć się więcej na temat Azure Stack Hub operacji, zobacz następujące zasoby:

- [Odzyskiwanie danych w Azure Stack Hub za pomocą usługi Infrastructure Backup Service](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Włączanie tworzenia kopii zapasowej Azure Stack Hub z portalu administratora](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Odzyskiwanie sprawności po utracie danych w wyniku katastrofy](/azure-stack/operator/azure-stack-backup-recover-data)
- [Najlepsze rozwiązania dotyczące usługi Infrastructure Backup](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub to platforma i sieć szkieletowa, na której będą wdrażane aplikacje Kubernetes. Właściciel aplikacji kubernetes będzie użytkownikiem usługi Azure Stack Hub z dostępem do wdrażania infrastruktury aplikacji potrzebnej do rozwiązania. Infrastruktura aplikacji w tym przypadku oznacza klaster Kubernetes wdrożony przy użyciu aparatu AKS i otaczających usług. Te składniki zostaną wdrożone w Azure Stack Hub, ograniczone przez Azure Stack Hub ofertę. Upewnij się, że oferta zaakceptowana przez właściciela aplikacji Kubernetes ma wystarczającą pojemność (wyrażoną w limitach przydziału Azure Stack Hub) do wdrożenia całego rozwiązania. Zgodnie z zaleceniami z poprzedniej sekcji wdrożenie aplikacji powinno zostać zautomatyzowane przy użyciu infrastruktury jako kodu i potoków wdrażania, takich jak Azure DevOps Azure Pipelines.

Aby uzyskać więcej informacji na temat Azure Stack Hub i przydziałów, zobacz [Azure Stack Hub usługi, plany,](/azure-stack/operator/service-plan-offer-subscription-overview) oferty i subskrypcje — omówienie

Ważne jest, aby bezpiecznie zapisywać i przechowywać konfigurację aparatu usługi AKS wraz z jego wyjściami. Te pliki zawierają poufne informacje używane do uzyskiwania dostępu do klastra Kubernetes, dlatego muszą być chronione przed ich ujawnioniem innym niż administratorzy.

**Dostępność aplikacji**

Aplikacja nie powinna polegać na kopiach zapasowych wdrożonego wystąpienia. Standardową praktyką jest ponowne stosowanie aplikacji całkowicie zgodnie z wzorcami infrastruktury jako kodu. Na przykład ponownie wdepniesz przy użyciu Azure DevOps Azure Pipelines. Procedura BCDR powinna obejmować ponowne zastosowanie aplikacji w tym samym lub innym klastrze Kubernetes.

**Dane aplikacji**

Dane aplikacji to kluczowy element minimalizowania utraty danych. W poprzedniej sekcji opisano techniki replikowania i synchronizowania danych między co najmniej dwoma wystąpieniami aplikacji. W zależności od infrastruktury bazy danych (MySQL, MongoDB, MSSQL lub innych) używanej do przechowywania danych do wyboru będą różne techniki dostępności bazy danych i tworzenia kopii zapasowych.

Zalecanymi sposobami osiągnięcia integralności są:
- Natywne rozwiązanie do tworzenia kopii zapasowych dla określonej bazy danych.
- Rozwiązanie do tworzenia kopii zapasowych, które oficjalnie obsługuje tworzenie kopii zapasowych i odzyskiwanie typu bazy danych używanego przez aplikację.

> [!IMPORTANT]
> Nie przechowuj danych kopii zapasowej w tym samym wystąpieniu Azure Stack Hub, w którym znajdują się dane aplikacji. Całkowita Azure Stack Hub również naruszyć kopie zapasowe.

## <a name="availability-considerations"></a>Zagadnienia dotyczące dostępności

Usługa Kubernetes Azure Stack Hub wdrożona za pośrednictwem aparatu usługi AKS nie jest usługą zarządzaną. Jest to zautomatyzowane wdrożenie i konfiguracja klastra Kubernetes przy użyciu infrastruktury jako usługi (IaaS) platformy Azure. W związku z tym zapewnia taką samą dostępność jak podstawowa infrastruktura.

Azure Stack Hub jest już odporna na awarie i oferuje funkcje, takie jak zestawy dostępności, do dystrybucji składników w wielu domenach błędów [i aktualizacji.](/azure-stack/user/azure-stack-vm-considerations#high-availability) Jednak podstawowa technologia (klaster trybu failover) nadal wiąże się z pewnymi przestojami maszyn wirtualnych na serwerze fizycznym, na którym ma to wpływ, w przypadku awarii sprzętowej.

Dobrym rozwiązaniem jest wdrożenie produkcyjnego klastra Kubernetes oraz obciążenia w co najmniej dwóch klastrach. Te klastry powinny być hostowane w różnych lokalizacjach lub centrach danych i używać technologii takich jak Azure Traffic Manager do przekierowania użytkowników na podstawie czasu odpowiedzi klastra lub lokalizacji geograficznej.

![Używanie Traffic Manager do kontrolowania przepływów ruchu](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Klienci, którzy mają pojedynczy klaster Kubernetes, zwykle łączą się z adresem IP usługi lub nazwą DNS danej aplikacji. W przypadku wdrożenia z wieloma klastrami klienci powinni łączyć się z Traffic Manager DNS, która wskazuje usługi/ruch przychodzący w każdym klastrze Kubernetes.

![Używanie Traffic Manager do przekierowania do klastra lokalnego](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Ten wzorzec jest również [najlepszym rozwiązaniem dla (zarządzanych) klastrów usługi AKS na platformie Azure.](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment)

Sam klaster Kubernetes wdrożony za pośrednictwem aparatu usługi AKS powinien składać się z co najmniej trzech węzłów głównych i dwóch węzłów procesu roboczego.

## <a name="identity-and-security-considerations"></a>Zagadnienia dotyczące tożsamości i zabezpieczeń

Tożsamość i zabezpieczenia to ważne tematy. Szczególnie wtedy, gdy rozwiązanie obejmuje niezależne Azure Stack Hub wystąpień. Zarówno platforma Kubernetes, jak i platforma Azure (Azure Stack Hub) mają różne mechanizmy kontroli dostępu opartej na rolach (RBAC):

- Kontrola RBAC platformy Azure kontroluje dostęp do zasobów na platformie Azure (i Azure Stack Hub), w tym możliwość tworzenia nowych zasobów platformy Azure. Uprawnienia można przypisywać użytkownikom, grupom lub jednostkom usługi. (Jednostką usługi jest tożsamość zabezpieczeń używana przez aplikacje).
- Kontrola RBAC platformy Kubernetes kontroluje uprawnienia do interfejsu API kubernetes. Na przykład tworzenie zasobników i wyświetlanie ich listy to akcje, które mogą być autoryzowane (lub odrzucone) dla użytkownika za pośrednictwem kontroli RBAC. Aby przypisać użytkownikom uprawnienia usługi Kubernetes, należy utworzyć role i powiązania ról.

**Azure Stack Hub tożsamości i kontroli RBAC**

Azure Stack Hub udostępnia dwie opcje dostawcy tożsamości. Dostawca, z którym korzystasz, zależy od środowiska i od tego, czy działa on w środowisku połączonym, czy odłączonym:

- Usługa Azure AD — może być używana tylko w połączonym środowisku.
- Usługi ADFS do tradycyjnego lasu usługi Active Directory — mogą być używane zarówno w środowisku połączonym, jak i odłączonym.

Dostawca tożsamości zarządza użytkownikami i grupami, w tym uwierzytelnianiem i autoryzacją w celu uzyskiwania dostępu do zasobów. Dostęp można przyznać do Azure Stack Hub, takich jak subskrypcje, grupy zasobów i poszczególne zasoby, takie jak maszyny wirtualne lub usługi równoważenia obciążenia. Aby mieć spójny model dostępu, należy rozważyć użycie tych samych grup (bezpośrednich lub zagnieżdżonych) dla wszystkich Azure Stack Hubs. Oto przykład konfiguracji:

![zagnieżdżone grupy usługi AAD w usłudze Azure Stack Hub](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

Przykład zawiera dedykowaną grupę (korzystającą z usług AAD lub ADFS) do określonego celu. Na przykład w celu zapewnienia uprawnień współautora dla grupy zasobów zawierającej naszą infrastrukturę klastra Kubernetes w konkretnym wystąpieniu Azure Stack Hub (tutaj "Współautor klastra Seattle K8s"). Te grupy są następnie zagnieżdżone w ogólnej grupie zawierającej "podgrupy" dla każdego Azure Stack Hub.

Nasz przykładowy użytkownik będzie teraz miał uprawnienia "Współautor" do obu grup zasobów, które zawierają cały zestaw zasobów infrastruktury Kubernetes. Użytkownik będzie miał dostęp do zasobów w obu wystąpieniach Azure Stack Hub, ponieważ wystąpienia współużytkuje tego samego dostawcę tożsamości.

> [!IMPORTANT]
> Te uprawnienia mają Azure Stack Hub tylko na Azure Stack Hub i niektóre zasoby wdrożone na jego podstawie. Użytkownik, który ma ten poziom dostępu, może zrobić wiele szkód, ale nie może uzyskać dostępu do maszyn wirtualnych IaaS kubernetes ani interfejsu API Kubernetes bez dodatkowego dostępu do wdrożenia kubernetes.

**Tożsamość Kubernetes i RBAC**

Klaster Kubernetes domyślnie nie używa tego samego dostawcy tożsamości co Azure Stack Hub. Maszyny wirtualne hostują klaster Kubernetes, węzeł główny i węzły procesu roboczego, używają klucza SSH określonego podczas wdrażania klastra. Ten klucz SSH jest wymagany do nawiązania połączenia z tymi węzłami przy użyciu połączenia SSH.

Interfejs API Kubernetes (na przykład dostępny przy użyciu usługi ) jest również chroniony przez konta usług, w tym domyślne `kubectl` konto usługi "administrator klastra". Poświadczenia dla tego konta usługi są początkowo przechowywane w pliku w węzłach głównych `.kube/config` kubernetes.

**Zarządzanie wpisami tajnymi i poświadczenia aplikacji**

Istnieje kilka opcji przechowywania wpisów tajnych, takich jak parametry połączenia lub poświadczenia bazy danych, w tym:

- Azure Key Vault
- Wpisy tajne usługi Kubernetes
- Rozwiązania innych firm, takie jak HashiCorp Vault (działające na kubernetes)

Nie przechowuj wpisów tajnych ani poświadczeń w postaci zwykłego tekstu w plikach konfiguracji, kodzie aplikacji ani w skryptach. Nie należy ich również przechowywać w systemie kontroli wersji. Zamiast tego automatyzacja wdrażania powinna pobierać wpisy tajne zgodnie z potrzebami.

## <a name="patch-and-update"></a>Stosowanie poprawek i aktualizowanie

Proces **poprawek i aktualizacji (PNU)** w Azure Kubernetes Service jest częściowo zautomatyzowany. Uaktualnienia wersji kubernetes są wyzwalane ręcznie, podczas gdy aktualizacje zabezpieczeń są stosowane automatycznie. Te aktualizacje mogą obejmować poprawki zabezpieczeń systemu operacyjnego lub aktualizacje jądra. AKS nie uruchamia automatycznie ponownego uruchomienia tych węzłów systemu Linux w celu ukończenia procesu aktualizacji. 

Proces PNU dla klastra Kubernetes wdrożonego przy użyciu aparatu AKS na platformie Azure Stack Hub jest niezadajny i jest odpowiedzialny za operator klastra. 

Aparat AKS ułatwia wykonywanie dwóch najważniejszych zadań:

- [Uaktualnianie do nowszej wersji systemu Kubernetes i podstawowego obrazu systemu operacyjnego](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Uaktualnianie tylko podstawowego obrazu systemu operacyjnego](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Nowsze podstawowe obrazy systemu operacyjnego zawierają najnowsze poprawki zabezpieczeń systemu operacyjnego i aktualizacje jądra. 

Mechanizm [nienadzorowanych](https://wiki.debian.org/UnattendedUpgrades) uaktualnień automatycznie instaluje aktualizacje zabezpieczeń wydane przed zainstalowaniem nowej wersji obrazu podstawowego systemu operacyjnego w witrynie Azure Stack Hub Marketplace. Uaktualnienie nienadzorowane jest domyślnie włączone i automatycznie instaluje aktualizacje zabezpieczeń, ale nie uruchamia ponownie węzłów klastra Kubernetes. Ponowny rozruch węzłów można zautomatyzować przy użyciu typu open source [ **K** Ubernetes **RE** boot **D** aemon (wytłaczy)).](/azure/aks/node-updates-kured) Węzły systemu Linux wymagają ponownego uruchomienia, a następnie automatycznie obsługują ponowne uruchamianie zasobników i proces ponownego uruchamiania węzłów.

## <a name="deployment-cicd-considerations"></a>Zagadnienia dotyczące wdrażania (CI/CD)

Platforma Azure Azure Stack Hub uwidocznić te same Azure Resource Manager INTERFEJSY API REST. Te interfejsy API są adresowane tak jak każda inna chmura platformy Azure (Azure, Azure (Chiny) — 21Vianet, Azure Government). Mogą wystąpić różnice w wersjach interfejsu API między chmurami, a Azure Stack Hub tylko podzestaw usług. URI punktu końcowego zarządzania jest również inny dla każdej chmury i każdego wystąpienia Azure Stack Hub.

Poza drobnymi różnicami, interfejsy API REST Azure Resource Manager zapewniają spójny sposób interakcji zarówno z platformą Azure, jak i Azure Stack Hub. W tym miejscu można używać tego samego zestawu narzędzi, co w przypadku każdej innej chmury platformy Azure. Do wdrażania i organizowania usług w Azure DevOps można użyć narzędzi, takich Azure Stack Hub jak jenkins lub PowerShell.

**Zagadnienia do rozważenia**

Jedną z głównych różnic w przypadku wdrożeń Azure Stack Hub jest kwestia dostępności Internetu. Dostępność internetowa określa, czy wybrać agenta kompilacji hostowanej przez firmę Microsoft, czy też samodzielnie dla zadań ciasnych/cd.

Samodzielnie hostowany agent może działać na Azure Stack Hub (jako maszyna wirtualna IaaS) lub w podsieci sieciowej, która może uzyskać dostęp do Azure Stack Hub. Przejdź do [Azure Pipelines agentów,](/azure/devops/pipelines/agents/agents) aby dowiedzieć się więcej o różnicach.

Poniższy obraz ułatwia podjęcie decyzji, czy potrzebujesz własnego, czy hostowanej przez firmę Microsoft agenta kompilacji:

![Samodzielnie hostowanych agentów kompilacji Tak lub Nie](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Czy punkty końcowe zarządzania Azure Stack Hub są dostępne za pośrednictwem Internetu?
  - Tak: do nawiązywania połączenia z Azure Pipelines agentami hostowanych przez firmę Microsoft można używać Azure Stack Hub.
  - Nie: Potrzebujemy samodzielnie hostowanych agentów, Azure Stack Hub łączyć się z punktami końcowymi zarządzania usługi .
- Czy nasz klaster Kubernetes jest dostępny za pośrednictwem Internetu?
  - Tak: możemy używać usługi Azure Pipelines agentami hostowanych przez firmę Microsoft do bezpośredniej interakcji z punktem końcowym interfejsu API kubernetes.
  - Nie: Potrzebujemy agentów hostowanych samodzielnie, które mogą łączyć się z punktem końcowym interfejsu API klastra Kubernetes.

W scenariuszach, Azure Stack Hub punkty końcowe zarządzania siecią i interfejs API Kubernetes są dostępne przez Internet, wdrożenie może korzystać z agenta hostowanej przez firmę Microsoft. To wdrożenie spowoduje, że architektura aplikacji będzie następująca:

[![Omówienie architektury publicznej](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Jeśli punkty końcowe Azure Resource Manager, interfejs API Kubernetes lub oba te punkty nie są bezpośrednio dostępne za pośrednictwem Internetu, możemy użyć własnego agenta kompilacji do uruchomienia kroków potoku. Ten projekt wymaga mniejszej łączności i można go wdrożyć tylko z lokalną łącznością sieciową z punktami końcowymi Azure Resource Manager i interfejsem API Kubernetes:

[![Omówienie architektury on-prem](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Co z scenariuszami bez połączenia?** W scenariuszach, Azure Stack Hub lub Kubernetes albo oba nie mają punktów końcowych zarządzania z Internetem, nadal można używać Azure DevOps wdrożeń. Możesz użyć hostowanej samodzielnie puli agentów (która jest agentem programu DevOps uruchomionym lokalnie lub w samym Azure Stack Hub) albo całkowicie samodzielnie hostowanej Azure DevOps Server lokalnie. Własny agent wymaga tylko wychodzącej łączności z Internetem za pośrednictwem protokołu HTTPS (TCP/443).

Wzorzec może używać klastra Kubernetes (wdrożonego i orkiestrowane przy użyciu aparatu AKS) w każdym Azure Stack Hub wystąpienia. Obejmuje ona aplikację składającą się z frontendu, warstwy środkowej, usług zaplecza (na przykład MongoDB) i kontrolera ruchu przychodzących opartego na systemie nginx. Zamiast korzystać z bazy danych hostowanej w klastrze K8s, można korzystać z "zewnętrznych magazynów danych". Opcje bazy danych obejmują program MySQL, SQL Server lub dowolny rodzaj bazy danych hostowanej poza Azure Stack Hub lub WaaS. Takie konfiguracje nie są tutaj objęte zakresem.

## <a name="partner-solutions"></a>Rozwiązania partnerskie

Istnieją rozwiązania partnerskie firmy Microsoft, które mogą rozszerzyć możliwości Azure Stack Hub. Rozwiązania te są przydatne we wdrożeniach aplikacji działających w klastrach Kubernetes.  

## <a name="storage-and-data-solutions"></a>Storage i rozwiązania do przetwarzania danych

Zgodnie z opisem [w teksie](#data-and-storage-considerations)Data and storage considerations (Zagadnienia dotyczące danych i magazynu) program Azure Stack Hub obecnie nie ma natywnego rozwiązania do replikowania magazynu w wielu wystąpieniach. W przeciwieństwie do platformy Azure możliwość replikowania magazynu w wielu regionach nie istnieje. W Azure Stack Hub każde wystąpienie jest własną odrębną chmurą. Jednak dostępne są rozwiązania od partnerów firmy Microsoft, które umożliwiają replikację magazynu Azure Stack Hubs i na platformie Azure. 

**SKALOWALNOŚĆ**

[Skalowalność zapewnia](https://www.scality.com/) magazyn w skali sieci Web, który od 2009 roku napędza firmy cyfrowe. Scality RING, nasz magazyn zdefiniowany programowo, zamienia serwery x86 w nieograniczoną pulę magazynów dla dowolnego typu danych — pliku i obiektu — w skali petabajtów.

**CLOUDIAN**

[Usługa Cloudian](https://www.cloudian.com/) upraszcza magazyn przedsiębiorstwa dzięki nieograniczonemu skalowalnemu magazynowi, który konsoliduje ogromne zestawy danych w jednym, łatwo zarządzanym środowisku.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat pojęć owanych w tym artykule:

- [Wzorce skalowania między chmurami](pattern-cross-cloud-scale.md) i [aplikacji rozproszonych](pattern-geo-distributed.md) geograficznie Azure Stack Hub.
- [Architektura mikrousług w Azure Kubernetes Service (AKS).](/azure/architecture/reference-architectures/microservices/aks)

Gdy wszystko będzie gotowe do przetestowania przykładowego rozwiązania, przejdź do przewodnika wdrażania klastra [Kubernetes](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes)o wysokiej dostępności. Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.