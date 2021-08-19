---
title: Wzorzec skalowania między chmurami (dane lokalne) w Azure Stack Hub
description: Dowiedz się, jak utworzyć skalowalną aplikację międzychmurową, która korzysta z danych na platformie Azure i Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281248"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Wzorzec skalowania między chmurami (dane lokalne)

Dowiedz się, jak utworzyć aplikację hybrydową, która obejmuje platformę Azure i Azure Stack Hub. Ten wzorzec pokazuje również, jak używać jednego lokalnego źródła danych w celu zapewnienia zgodności.

## <a name="context-and-problem"></a>Kontekst i problem

Wiele organizacji zbiera i przechowuje ogromne ilości poufnych danych klientów. Często nie mogą przechowywać poufnych danych w chmurze publicznej ze względu na przepisy firmowe lub zasady rządowe. Te organizacje chcą również korzystać ze skalowalności chmury publicznej. Chmura publiczna może obsługiwać sezonowe wzrosty ruchu, umożliwiając klientom płacenie dokładnie za potrzebny sprzęt, gdy tego potrzebują.

## <a name="solution"></a>Rozwiązanie

Rozwiązanie korzysta z zalet zgodności chmury prywatnej, łącząc je ze skalowalnością chmury publicznej. Chmura hybrydowa Azure Stack Hub azure i chmura hybrydowa zapewniają spójne środowisko dla deweloperów. Dzięki tej spójności mogą stosować swoje umiejętności zarówno w chmurze publicznej, jak i w środowiskach lokalnych.

Przewodnik wdrażania rozwiązania umożliwia wdrożenie identycznej aplikacji internetowej w chmurze publicznej i prywatnej. Możesz również uzyskać dostęp do sieci routowalnej dla Internetu hostowanej w chmurze prywatnej. Aplikacje internetowe są monitorowane pod kątem obciążenia. Po znacznym zwiększeniu ruchu program manipuluje rekordami DNS w celu przekierowania ruchu do chmury publicznej. Gdy ruch nie jest już znaczący, rekordy DNS są aktualizowane w celu kierowania ruchu z powrotem do chmury prywatnej.

[![Skalowanie między chmurami ze wzorcem danych w chmurze](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Składniki

To rozwiązanie używa następujących składników:

| Warstwa | Składnik | Opis |
|----------|-----------|-------------|
| Azure | Azure App Service | [Azure App Service](/azure/app-service/) umożliwia tworzenie i hostować aplikacje internetowe, aplikacje interfejsu API RESTful i Azure Functions. Wszystko w języku programowania, który należy wybrać, bez zarządzania infrastrukturą. |
| | Azure Virtual Network| [Usługa Azure Virtual Network (VNet)](/azure/virtual-network/virtual-networks-overview) to podstawowy blok blokowy dla sieci prywatnych na platformie Azure. Sieć wirtualna umożliwia wielu typom zasobów platformy Azure, takim jak maszyny wirtualne, bezpieczne komunikowanie się ze sobą, z Internetem i sieciami lokalnymi. W rozwiązaniu pokazano również użycie dodatkowych składników sieciowych:<br>— podsieci aplikacji i bramy.<br>— lokalna brama sieci lokalnej.<br>— brama sieci wirtualnej, która działa jako połączenie bramy sieci VPN typu lokacja-lokacja.<br>— publiczny adres IP.<br>— połączenie sieci VPN typu punkt-lokacja.<br>- Azure DNS do hostowania domen DNS i zapewniania rozpoznawania nazw. |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) jest opartym na systemie DNS równoważeniem obciążenia ruchu. Umożliwia kontrolowanie dystrybucji ruchu użytkowników dla punktów końcowych usługi w różnych centrach danych. |
| | Azure Application Insights | [Application Szczegółowe informacje](/azure/azure-monitor/app/app-insights-overview) to rozszerzalna usługa zarządzania wydajnością aplikacji dla deweloperów internetowych, którzy budować aplikacje i zarządzać nimi na wielu platformach.|
| | Azure Functions | [Azure Functions](/azure/azure-functions/) umożliwia wykonywanie kodu w środowisku bez serwera bez konieczności wcześniejszego tworzenia maszyny wirtualnej ani publikowania aplikacji internetowej. |
| | Autoskalowanie platformy Azure | [Autoskalowanie](/azure/azure-monitor/platform/autoscale-overview) to wbudowana funkcja Cloud Services, maszyn wirtualnych i aplikacji internetowych. Ta funkcja umożliwia aplikacjom najlepsze wykonywanie zadań po zmianie zapotrzebowania. Aplikacje dostosują się pod potrzeby skoków ruchu, powiadamiając o zmianie metryk i skalowaniu zgodnie z potrzebami. |
| Azure Stack Hub | IaaS Compute | Azure Stack Hub umożliwia korzystanie z tego samego modelu aplikacji, portalu samoobsługowego i interfejsów API włączonych przez platformę Azure. Azure Stack Hub IaaS umożliwia korzystanie z szerokiej gamy technologii typu open source dla spójnych wdrożeń chmury hybrydowej. Przykład rozwiązania używa maszyny wirtualnej Windows Server do SQL Server, na przykład.|
| | Azure App Service | Podobnie jak aplikacja internetowa platformy Azure, rozwiązanie używa Azure App Service [na Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) do hostować aplikację internetową. |
| | Sieć | Ta Azure Stack Hub Virtual Network działa dokładnie tak samo jak usługa Azure Virtual Network. Używa ona wielu tych samych składników sieciowych, w tym niestandardowych nazw hostów.
| Usługa Azure DevOps Services | Rejestrowanie | Szybkie konfigurowanie ciągłej integracji dla kompilacji, testowania i wdrażania. Aby uzyskać więcej informacji, zobacz [Rejestracja i logowanie do Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops). |
| | Azure Pipelines | Użyj [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) dla ciągłej integracji/ciągłego dostarczania. Azure Pipelines umożliwia zarządzanie hostowaną kompilacją i wydaniami agentów i definicji. |
| | Repozytorium kodu | Korzystaj z wielu repozytoriów kodu, aby usprawnić potok projektowania. Użyj istniejących repozytoriów kodu w GitHub, Bitbucket, Dropbox, OneDrive i Azure Repos. |

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podejmując decyzję o wdrożeniu tego rozwiązania, należy wziąć pod uwagę następujące kwestie:

### <a name="scalability"></a>Skalowalność

Platforma Azure i Azure Stack Hub doskonale nadają się do obsługi potrzeb współczesnych globalnie rozproszonych firm.

#### <a name="hybrid-cloud-without-the-hassle"></a>Chmura hybrydowa bez problemów

Firma Microsoft oferuje niezrównaną integrację zasobów lokalnych z platformą Azure Stack Hub Azure w jednym ujednoliconym rozwiązaniu. Ta integracja eliminuje problemy z zarządzaniem rozwiązaniami z wieloma punktami i kombinacją dostawców usług w chmurze. Dzięki skalowaniu między chmurami możliwości platformy Azure są zaledwie kilka kliknięć. Po prostu połącz swoją Azure Stack Hub platformą Azure z chmurą, a Twoje dane i aplikacje będą dostępne na platformie Azure w razie potrzeby.

- Eliminowanie konieczności kompilowania i obsługi dodatkowej lokacji dr.
- Oszczędzaj czas i pieniądze, eliminując tworzenie kopii zapasowej na taśmie i zapisuj dane kopii zapasowej na platformie Azure przez maksymalnie 99 lat.
- Łatwe migrowanie uruchomionych obciążeń funkcji Hyper-V, fizycznych (w wersji zapoznawczej) i oprogramowania VMware (w wersji zapoznawczej) na platformę Azure w celu wykorzystania ekonomii i elastyczności chmury.
- Uruchamiaj raporty lub analizy intensywnie obciążające zasoby obliczeniowe na zreplikowanej kopii zasobu lokalnego na platformie Azure bez wpływu na obciążenia produkcyjne.
- W razie potrzeby przeszukaj do chmury i uruchamiaj obciążenia lokalne na platformie Azure przy użyciu większych szablonów obliczeniowych. Rozwiązanie hybrydowe zapewnia potrzebne możliwości, gdy są potrzebne.
- Twórz wielowarstwowe środowiska projektowe na platformie Azure za pomocą kilku kliknięć— nawet replikuj dane produkcyjne na żywo do środowiska deweloperskich/testowych, aby zachować je niemal w czasie rzeczywistym.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Ekonomia skalowania między chmurami przy Azure Stack Hub

Kluczową zaletą rozwoju chmury są oszczędności ekonomiczne. Za dodatkowe zasoby płacisz tylko wtedy, gdy istnieje zapotrzebowanie na te zasoby. Koniec z wydatkami na niepotrzebną dodatkową pojemność lub próbą przewidywania szczytów i wahań zapotrzebowania.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Zmniejszanie obciążeń wysokiego zapotrzebowania do chmury

Skalowanie między chmurami może służyć do zmniejszenia obciążeń przetwarzania. Obciążenie jest dystrybuowane przez przeniesienie podstawowych aplikacji do chmury publicznej, co uwolni zasoby lokalne dla aplikacji o krytycznym znaczeniu dla firmy. Aplikację można zastosować do chmury prywatnej, a następnie przesunąć do chmury publicznej tylko wtedy, gdy jest to konieczne, aby spełnić wymagania.

### <a name="availability"></a>Dostępność

Wdrożenie globalne ma swoje własne wyzwania, takie jak zmienna łączność i różne przepisy rządowe w poszczególnych regionach. Deweloperzy mogą tworzyć tylko jedną aplikację, a następnie wdrażać ją z różnych powodów z różnymi wymaganiami. Wdrażanie aplikacji w chmurze publicznej platformy Azure, a następnie wdrażanie dodatkowych wystąpień lub składników lokalnie. Ruchem między wszystkimi wystąpieniami można zarządzać przy użyciu platformy Azure.

### <a name="manageability"></a>Możliwości zarządzania

#### <a name="a-single-consistent-development-approach"></a>Jedno spójne podejście programistyczne

Platforma Azure i Azure Stack Hub umożliwiają korzystanie ze spójnego zestawu narzędzi deweloperskie w całej organizacji. Ta spójność ułatwia implementowanie praktyki ciągłej integracji i ciągłego tworzenia (CI/CD). Wiele aplikacji i usług wdrożonych na platformie Azure lub Azure Stack Hub jest wymiennych i można je bezproblemowo uruchamiać w obu lokalizacjach.

Hybrydowy potok cicha/cd może pomóc w:

- Zainicjuj nową kompilację na podstawie zatwierdzeń kodu w repozytorium kodu.
- Automatyczne wdrażanie nowo utworzonego kodu na platformie Azure w celu testowania akceptacyjnie przez użytkowników.
- Po zakończeniu testowania kodu możesz automatycznie wdrożyć go w Azure Stack Hub.

### <a name="a-single-consistent-identity-management-solution"></a>Pojedyncze, spójne rozwiązanie do zarządzania tożsamościami

Azure Stack Hub działa z usługami Azure Active Directory (Azure AD) i Active Directory Federation Services (ADFS). Azure Stack Hub z usługą Azure AD w scenariuszach połączonych. W środowiskach, które nie mają łączności, można użyć usług AD FS jako rozwiązania odłączonego. Jednostki usługi służą do udzielania dostępu do aplikacji, umożliwiając im wdrażanie lub konfigurowanie zasobów za pośrednictwem Azure Resource Manager.

### <a name="security"></a>Zabezpieczenia

#### <a name="ensure-compliance-and-data-sovereignty"></a>Zapewnianie zgodności i niezależności danych

Azure Stack Hub umożliwia uruchamianie tej samej usługi w wielu krajach, co w przypadku korzystania z chmury publicznej. Wdrożenie tej samej aplikacji w centrach danych w każdym kraju umożliwia spełnianie wymagań dotyczących niezależności danych. Ta funkcja zapewnia, że dane osobowe są przechowywane w granicach każdego kraju.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub — poziom zabezpieczeń

Nie ma żadnego zabezpieczenia bez stałego, ciągłego procesu obsługi. Z tego powodu firma Microsoft zainwestowała w aparat aranżacji, który bezproblemowo stosuje poprawki i aktualizacje w całej infrastrukturze.

Dzięki współpracy z Azure Stack Hub OEM firma Microsoft rozszerza ten sam poziom zabezpieczeń na składniki specyficzne dla producenta OEM, takie jak host cyklu życia sprzętu i oprogramowanie działające na jego podstawie. To partnerstwo gwarantuje Azure Stack Hub ma jednolity, solidny poziom zabezpieczeń w całej infrastrukturze. Z kolei klienci mogą tworzyć i zabezpieczać obciążenia aplikacji.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Korzystanie z jednostki usługi za pośrednictwem programu PowerShell, interfejsu wiersza polecenia i Azure Portal

Aby udzielić dostępu do zasobu skryptowi lub aplikacji, skonfiguruj tożsamość aplikacji i uwierzytelnij aplikację przy użyciu własnych poświadczeń. Ta tożsamość jest znana jako główna usługa i umożliwia:

- Przypisz uprawnienia do tożsamości aplikacji, które są inne niż Twoje uprawnienia i są ograniczone do dokładnie potrzeb aplikacji.
- Używanie certyfikatu do uwierzytelnienia podczas wykonywania skryptu nienadzorowanego.

Aby uzyskać więcej informacji na temat tworzenia jednostki usługi i używania certyfikatu dla poświadczeń, zobacz Use an app identity to access resources (Używanie tożsamości aplikacji [do uzyskiwania dostępu do zasobów).](/azure-stack/operator/azure-stack-create-service-principals)

## <a name="when-to-use-this-pattern"></a>Kiedy używać tego wzorca

- Moja organizacja korzysta z podejścia DevOps lub ma takie, które zostało zaplanowane na najbliższą przyszłość.
- Chcę wdrożyć praktyki ci/CD w mojej implementacji Azure Stack Hub i chmurze publicznej.
- Chcę skonsolidować potok CI/CD w środowiskach w chmurze i lokalnych.
- Chcę mieć możliwość bezproblemowego tworzenia aplikacji przy użyciu usług w chmurze lub lokalnych.
- Chcę wykorzystać spójne umiejętności deweloperów w aplikacjach w chmurze i lokalnych.
- Korzystam z platformy Azure, ale mam deweloperów, którzy pracują w środowisku lokalnym Azure Stack Hub chmurze.
- Moje aplikacje lokalne odczuwają skoki zapotrzebowania podczas sezonowych, cyklicznych lub nieprzewidywalnych fluktuacji.
- Mam składniki lokalne i chcę bezproblemowo skalować je za pomocą chmury.
- Chcę mieć skalowalność w chmurze, ale chcę, aby moja aplikacja działała w środowisku lokalnym w jak najciększej perspektywie.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów owanych w tym artykule:

- Obejrzyj [dynamiczne skalowanie aplikacji między centrami danych](https://www.youtube.com/watch?v=2lw8zOpJTn0) i chmurą publiczną, aby uzyskać omówienie sposobu, w jaki ten wzorzec jest używany.
- Zobacz [Zagadnienia dotyczące projektowania aplikacji hybrydowych,](overview-app-design-considerations.md) aby dowiedzieć się więcej na temat najlepszych rozwiązań i odpowiedzieć na dodatkowe pytania.
- Ten wzorzec używa Azure Stack produktów, w tym Azure Stack Hub. Zobacz Azure Stack [produktów i rozwiązań,](/azure-stack) aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do przetestowania przykładowego rozwiązania, przejdź do przewodnika wdrażania rozwiązania skalowania między chmurami [(dane lokalne).](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data) Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.