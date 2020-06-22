---
title: Wzorzec wielu chmur (dane lokalne) w centrum Azure Stack
description: Dowiedz się, jak utworzyć skalowalną aplikację obejmującą wiele chmur, która używa danych Premium na platformie Azure i w centrum Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: edbb608fbf8e5288f29572bfe4cca98ffb3cb8fc
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911126"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Wzorzec wielu chmur (dane lokalne)

Dowiedz się, jak utworzyć aplikację hybrydową obejmującą platformę Azure i Azure Stack Hub. Ten wzorzec pokazuje również, jak korzystać z jednego lokalnego źródła danych w celu zapewnienia zgodności.

## <a name="context-and-problem"></a>Kontekst i problem

Wiele organizacji zbiera i przechowuje ogromne ilości poufnych danych klientów. Często uniemożliwiają one przechowywanie poufnych danych w chmurze publicznej ze względu na przepisy prawne lub zasady rządowe. Organizacje te chcą również wykorzystać skalowalność chmury publicznej. Chmura publiczna może obsługiwać sezonowe szczytowe natężenie ruchu sieciowego, dzięki czemu klienci mogą uregulować dokładnie ten sprzęt, gdy ich potrzebują.

## <a name="solution"></a>Rozwiązanie

Rozwiązanie wykorzystuje zalety zgodności chmury prywatnej, łącząc je z skalowalnością chmury publicznej. Chmura hybrydowa platformy Azure i Azure Stack centrum zapewnia spójne środowisko dla deweloperów. Taka spójność pozwala im stosować swoje umiejętności zarówno w chmurze publicznej, jak i w środowiskach lokalnych.

Przewodnik wdrażania rozwiązań umożliwia wdrożenie identycznej aplikacji sieci Web w chmurze publicznej i prywatnej. Możesz również uzyskać dostęp do sieci z obsługą routingu spoza Internetu hostowanej w chmurze prywatnej. Aplikacje sieci Web są monitorowane pod kątem obciążenia. W przypadku znacznego wzrostu ruchu program manipuluje rekordami DNS w celu przekierowania ruchu do chmury publicznej. Gdy ruch nie jest już znaczący, rekordy DNS są aktualizowane, aby skierować ruch z powrotem do chmury prywatnej.

[![Skalowanie między chmurami przy użyciu wzorca danych Premium](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Składniki

To rozwiązanie używa następujących składników:

| Warstwa | Składnik | Opis |
|----------|-----------|-------------|
| Azure | Azure App Service | [Azure App Service](/azure/app-service/) umożliwia tworzenie i hostowanie aplikacji sieci Web, aplikacji interfejsu API RESTful i Azure Functions. Wszystko to w wybranym języku programowania bez zarządzania infrastrukturą. |
| | Azure Virtual Network| [Azure Virtual Network (VNET)](/azure/virtual-network/virtual-networks-overview) to podstawowy blok konstrukcyjny dla sieci prywatnych na platformie Azure. Sieć wirtualna umożliwia bezpieczne komunikowanie się ze sobą za pomocą wielu typów zasobów platformy Azure, takich jak maszyny wirtualne, z Internetu i sieci lokalnych. Rozwiązanie to również ilustruje użycie dodatkowych składników sieciowych:<br>— podsieci aplikacji i bramy.<br>— lokalna Brama sieci lokalnej.<br>-Brama sieci wirtualnej, która działa jako połączenie bramy sieci VPN typu lokacja-lokacja.<br>— publiczny adres IP.<br>-połączenie sieci VPN typu punkt-lokacja.<br>— Azure DNS do hostowania domen DNS i rozpoznawania nazw. |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) to moduł równoważenia obciążenia opartego na systemie DNS. Pozwala ona na kontrolowanie dystrybucji ruchu użytkowników dla punktów końcowych usługi w różnych centrach danych. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) to rozszerzalna usługa zarządzania wydajnością aplikacji dla deweloperów sieci Web, którzy tworzą aplikacje i zarządzają nimi na wielu platformach.|
| | Azure Functions | [Azure Functions](/azure/azure-functions/) umożliwia wykonywanie kodu w środowisku bezserwerowym bez konieczności uprzedniego tworzenia maszyny wirtualnej lub publikowania aplikacji sieci Web. |
| | Autoskalowanie platformy Azure | [Skalowanie automatyczne](/azure/azure-monitor/platform/autoscale-overview) to wbudowana funkcja Cloud Services, maszyn wirtualnych i aplikacji sieci Web. Funkcja umożliwia aplikacjom optymalne wykonywanie w przypadku zmiany zapotrzebowania. Aplikacje dostosowują się do powiększania ruchu, co powiadamia o zmianie i skalowaniu metryk w miarę potrzeb. |
| Azure Stack Hub | Obliczenia IaaS | Azure Stack Hub umożliwia korzystanie z tego samego modelu aplikacji, portalu samoobsługowego i interfejsów API włączonych przez platformę Azure. Azure Stack Hub IaaS umożliwia szeroką gamę technologii typu "open source" na potrzeby spójnych wdrożeń w chmurze hybrydowej. Przykład rozwiązania używa maszyny wirtualnej z systemem Windows Server do SQL Server, na przykład.|
| | Azure App Service | Podobnie jak w przypadku aplikacji sieci Web platformy Azure, rozwiązanie używa [Azure App Service na Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) do hostowania aplikacji sieci Web. |
| | Networking | Virtual Network centrum Azure Stack działa tak samo jak Virtual Network platformy Azure. Używa wielu z tych samych składników sieciowych, w tym niestandardowych nazw hostów.
| Usługa Azure DevOps Services | Rejestrowanie | Szybko Konfiguruj ciągłą integrację dla kompilowania, testowania i wdrażania. Aby uzyskać więcej informacji, zobacz [Rejestrowanie się, logowanie do usługi Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops). |
| | Azure Pipelines | Użyj [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) na potrzeby ciągłej integracji/ciągłego dostarczania. Azure Pipelines pozwala zarządzać obsługiwanymi agentami kompilacji i wydań oraz definicjami. |
| | Repozytorium kodu | Korzystaj z wielu repozytoriów kodu, aby usprawnić proces tworzenia. Używaj istniejących repozytoriów kodu w serwisach GitHub, BitBucket, Dropbox, OneDrive i Azure Repos. |

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas decydowania o sposobie wdrożenia tego rozwiązania należy wziąć pod uwagę następujące kwestie:

### <a name="scalability"></a>Skalowalność

Usługa Azure i Azure Stack Hub są jednoznacznie dostosowane do potrzeb współczesnych, dystrybuowanych w całości firmowych danych.

#### <a name="hybrid-cloud-without-the-hassle"></a>Chmura hybrydowa bez kłopotów

Firma Microsoft oferuje niezrównanąą integrację zasobów lokalnych z centrum Azure Stack i platformą Azure w jednym, ujednoliconym rozwiązaniu. Ta integracja eliminuje problemy związane z zarządzaniem wieloma rozwiązaniami punktowymi i różnymi dostawcami chmury. Dzięki skalowaniu między chmurami potęgą platformy Azure jest zaledwie kilka kliknięć. Po prostu Połącz centrum Azure Stack z platformą Azure z funkcją przenoszenia w chmurze, a Twoje dane i aplikacje będą dostępne na platformie Azure w razie potrzeby.

- Eliminuje konieczność kompilowania i konserwowania dodatkowej lokacji programu DR.
- Oszczędzaj czas i pieniądze, eliminując tworzenie kopii zapasowej na taśmach i przechowywanie danych kopii zapasowych na platformie Azure do 99 lat.
- Łatwo Migruj uruchomione funkcje Hyper-V, fizyczne (w wersji zapoznawczej) i VMware (w wersji zapoznawczej) do platformy Azure, aby wykorzystać ekonomię i elastyczność chmury.
- Uruchamiaj raporty intensywnie korzystające z obliczeń lub analizy dotyczące zreplikowanej kopii zasobów lokalnych na platformie Azure bez wpływu na obciążenia produkcyjne.
- Przetwarzanie zadań w chmurze i uruchamianie lokalnych obciążeń na platformie Azure przy użyciu większych szablonów obliczeniowych w razie konieczności. Hybrydowe zapewnia Ci potrzebną moc, gdy będzie potrzebna.
- Twórz wielowarstwowe środowiska deweloperskie na platformie Azure za pomocą kilku kliknięć — nawet Replikuj dane produkcyjne na żywo w środowisku deweloperskim/testowym, aby zachować je niemal w czasie rzeczywistym.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Ekonomia skalowania między chmurami za pomocą Centrum Azure Stack

Najważniejsze zalety korzystania z usługi w chmurze to ekonomiczne oszczędności. Płacisz tylko za dodatkowe zasoby, gdy istnieje zapotrzebowanie na te zasoby. Nie ma więcej wydatków na niezbędną dodatkową pojemność lub podjęcie próby przewidywania szczytów i wahań popytu.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Redukcja obciążeń o dużym zapotrzebowaniu do chmury

Skalowanie między chmurami może służyć do obciążeń przetwarzania barków. Obciążenie jest dystrybuowane przez przeniesienie podstawowych aplikacji do chmury publicznej i zwolnienie zasobów lokalnych dla aplikacji o krytycznym znaczeniu dla firmy. Aplikacja może zostać zastosowana do chmury prywatnej, a następnie do chmury publicznej tylko wtedy, gdy jest to konieczne do spełnienia wymagań.

### <a name="availability"></a>Dostępność

Globalne wdrożenie ma własne wyzwania, takie jak łączność zmienna i różne regulacje rządowe według regionów. Deweloperzy mogą opracowywać tylko jedną aplikację, a następnie wdrażać ją w różnych sytuacjach z różnymi wymaganiami. Wdróż aplikację w chmurze publicznej platformy Azure, a następnie wdróż dodatkowe wystąpienia lub składniki lokalnie. Ruch między wszystkimi wystąpieniami można zarządzać przy użyciu platformy Azure.

### <a name="manageability"></a>Możliwości zarządzania

#### <a name="a-single-consistent-development-approach"></a>Jedno spójne podejście programistyczne

Platforma Azure i usługa Azure Stack Hub umożliwiają używanie spójnego zestawu narzędzi programistycznych w całej organizacji. Ta spójność ułatwia wdrażanie rozwiązań ciągłej integracji i ciągłego tworzenia oprogramowania (CI/CD). Wiele aplikacji i usług wdrożonych na platformie Azure lub w centrum Azure Stack są zamienne i mogą być uruchamiane w jednej lokalizacji bezproblemowo.

Hybrydowy potok ciągłej integracji/ciągłego wdrażania może pomóc:

- Zainicjuj nową kompilację na podstawie zatwierdzeń kodu do repozytorium kodu.
- Automatycznie Wdróż nowo utworzony kod na platformie Azure na potrzeby testowania akceptacji użytkownika.
- Gdy kod przeszedł test, automatycznie Wdróż go w centrum Azure Stack.

### <a name="a-single-consistent-identity-management-solution"></a>Jedno spójne rozwiązanie do zarządzania tożsamościami

Azure Stack Hub współpracuje z zarówno Azure Active Directory (Azure AD), jak i Active Directory Federation Services (AD FS). Azure Stack Hub współpracuje z usługą Azure AD w połączonych scenariuszach. W przypadku środowisk, które nie mają łączności, możesz użyć usług AD FS jako rozłączonego rozwiązania. Nazwy główne usług są używane do udzielania dostępu do aplikacji, co umożliwia ich wdrażanie lub Konfigurowanie zasobów za Azure Resource Manager.

### <a name="security"></a>Zabezpieczenia

#### <a name="ensure-compliance-and-data-sovereignty"></a>Zapewnienie zgodności i suwerenności danych

Usługa Azure Stack Hub umożliwia uruchamianie tej samej usługi w wielu krajach, podobnie jak w przypadku korzystania z chmury publicznej. Wdrożenie tej samej aplikacji w centrach danych w poszczególnych krajach pozwala na spełnienie wymagań związanych z jurysdykcją. Ta funkcja zapewnia, że dane osobowe są przechowywane w granicach poszczególnych krajów.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub — zabezpieczenia stan

Nie ma zabezpieczeń stan bez pełnego ciągłego procesu obsługi. Z tego powodu firma Microsoft zainwestowali w aparat aranżacji, który bezproblemowo stosuje poprawki i aktualizacje w całej infrastrukturze.

Dzięki partnerstwie z partnerami OEM Azure Stack Hub firma Microsoft rozszerza ten sam stan zabezpieczeń do składników określonych przez producenta OEM, takich jak Host cyklu życia sprzętu i oprogramowanie uruchomione na tym komputerze. Dzięki temu partnerstwu Azure Stack centrum ma jednolity, pełny stan zabezpieczeń w całej infrastrukturze. Z kolei klienci mogą tworzyć i zabezpieczać obciążenia aplikacji.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Korzystanie z jednostek usługi za pośrednictwem programu PowerShell, interfejsu wiersza polecenia i Azure Portal

Aby zapewnić dostęp do zasobu do skryptu lub aplikacji, skonfiguruj tożsamość aplikacji i Uwierzytelnij aplikację przy użyciu własnych poświadczeń. Ta tożsamość jest znana jako nazwa główna usługi i umożliwia:

- Przypisywanie uprawnień do tożsamości aplikacji, które różnią się od własnych uprawnień i są ograniczone do potrzeb aplikacji.
- Używanie certyfikatu do uwierzytelnienia podczas wykonywania skryptu nienadzorowanego.

Aby uzyskać więcej informacji na temat tworzenia jednostki usługi i używania certyfikatu na potrzeby poświadczeń, zobacz [Korzystanie z tożsamości aplikacji w celu uzyskania dostępu do zasobów](/azure-stack/operator/azure-stack-create-service-principals).

## <a name="when-to-use-this-pattern"></a>Kiedy używać tego wzorca

- Moja organizacja używa podejścia DevOps lub jest planowane dla najbliższej przyszłości.
- Chcę zaimplementować praktyki ciągłej integracji/ciągłego wdrażania w ramach implementacji centrum Azure Stack i chmury publicznej.
- Chcę skonsolidować potok ciągłej integracji/ciągłego wdrażania w środowiskach w chmurze i lokalnych.
- Chcę bezproblemowo opracowywać aplikacje przy użyciu usług w chmurze lub lokalnych.
- Chcę korzystać z spójnych umiejętności programistycznych w aplikacjach w chmurze i lokalnych.
- Korzystam z platformy Azure, ale mam deweloperów, którzy pracują w lokalnej chmurze Azure Stack Hub.
- Moje aplikacje lokalne mają wpływ na zapotrzebowanie podczas sezonowych, cyklicznych lub nieprzewidzianych wahań.
- Mam składniki lokalne i chcę używać chmury do bezproblemowego skalowania.
- Chcę korzystać z skalowalności chmury, ale chcę, aby moja aplikacja działała lokalnie, jak to możliwe.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:

- Obejrzyj [dynamiczne skalowanie aplikacji między centrami danych i chmurą publiczną](https://www.youtube.com/watch?v=2lw8zOpJTn0) , aby zapoznać się z omówieniem sposobu korzystania z tego wzorca.
- Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i odpowiedzieć na dodatkowe pytania.
- Ten wzorzec używa rodziny produktów Azure Stack, w tym Azure Stack Hub. Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko jest gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika wdrażania rozwiązań między chmurami (dane lokalne)](solution-deployment-guide-cross-cloud-scaling-onprem-data.md). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.
