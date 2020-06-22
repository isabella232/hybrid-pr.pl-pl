---
title: Wzorzec aplikacji rozproszonej geograficznie w centrum Azure Stack
description: Dowiedz się więcej na temat wzorca aplikacji rozproszonej geograficznej dla inteligentnej krawędzi za pomocą platformy Azure i usługi Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910883"
---
# <a name="geo-distributed-app-pattern"></a>Wzorzec aplikacji rozproszonej geograficznie

Dowiedz się, jak dostarczać punkty końcowe aplikacji w wielu regionach i kierować ruchem użytkowników w oparciu o lokalizacje i wymagania dotyczące zgodności.

## <a name="context-and-problem"></a>Kontekst i problem

Organizacje o szerokim zasięgu lokalizacje geograficzne dążą do bezpiecznego i dokładnego rozpowszechniania i zapewniania dostępu do danych przy jednoczesnym zapewnieniu wymaganych poziomów zabezpieczeń, zgodności i wydajności dla poszczególnych użytkowników, lokalizacji i urządzeń.

## <a name="solution"></a>Rozwiązanie

Wzorzec routingu ruchu geograficznego centrum Azure Stack lub aplikacje rozproszone geograficznie umożliwiają kierowanie ruchu do określonych punktów końcowych na podstawie różnych metryk. Tworzenie Traffic Manager przy użyciu geograficznej konfiguracji routingu i punktu końcowego kieruje ruch do punktów końcowych na podstawie wymagań regionalnych, regulacji korporacyjnych i międzynarodowych oraz potrzeb dotyczących danych.

![Wzorzec rozproszony geograficznie](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Składniki

### <a name="outside-the-cloud"></a>Poza chmurą

#### <a name="traffic-manager"></a>Traffic Manager

Na diagramie Traffic Manager znajduje się poza chmurą publiczną, ale musi być w stanie koordynować ruch zarówno w lokalnym centrum danych, jak i w chmurze publicznej. Moduł równoważenia obciążenia kieruje ruch do lokalizacji geograficznych.

#### <a name="domain-name-system-dns"></a>System nazw domen (DNS)

System nazw domen (DNS) jest odpowiedzialny za tłumaczenie (lub rozwiązanie) nazwy witryny sieci Web lub usługi na adres IP.

### <a name="public-cloud"></a>Chmura publiczna

#### <a name="cloud-endpoint"></a>Punkt końcowy w chmurze

Publiczne adresy IP są używane do kierowania ruchu przychodzącego za pomocą usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.  

### <a name="local-clouds"></a>Chmury lokalne

#### <a name="local-endpoint"></a>Lokalny punkt końcowy

Publiczne adresy IP są używane do kierowania ruchu przychodzącego za pomocą usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas podejmowania decyzji o sposobie wdrożenia tego wzorca należy rozważyć następujące punkty:

### <a name="scalability"></a>Skalowalność

Wzorzec obsługuje routing ruchu geograficznego zamiast skalowania w celu osiągnięcia wzrostu ruchu. Można jednak połączyć ten wzorzec z innymi rozwiązaniami platformy Azure i lokalnymi. Na przykład ten wzorzec może być używany z wzorcem skalowania między chmurami.

### <a name="availability"></a>Dostępność

Upewnij się, że aplikacje wdrożone lokalnie są skonfigurowane pod kątem wysokiej dostępności za poorednictwem konfiguracji sprzętu lokalnego i wdrożenia oprogramowania.

### <a name="manageability"></a>Możliwości zarządzania

Wzorzec zapewnia bezproblemowe zarządzanie i przyjazny interfejs między środowiskami.

## <a name="when-to-use-this-pattern"></a>Kiedy używać tego wzorca

- Moja organizacja ma rozgałęzienia międzynarodowe wymagające niestandardowych zasad zabezpieczeń i dystrybucji.
- Każda z biur organizacji pobiera dane pracowników, działalności i możliwości, wymagając działań związanych z raportowaniem na lokalne regulacje i strefę czasową.
- Wymagania dotyczące dużej skali mogą być spełnione przez skalowanie w poziomie aplikacji, z wieloma wdrożeniami aplikacji wykonywanych w jednym regionie i między regionami w celu obsługi skrajnych wymagań dotyczących obciążenia.
- Aplikacje muszą mieć wysoką dostępność i odpowiadać na żądania klientów nawet w przypadku awarii w jednym regionie.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:

- Zobacz [Omówienie usługi Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) , aby dowiedzieć się więcej na temat działania tego modułu równoważenia obciążenia opartego na systemie DNS.
- Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na dodatkowe pytania.
- Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika po wdrożeniu rozwiązania aplikacji rozproszonej geograficznie](solution-deployment-guide-geo-distributed.md). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników. Dowiesz się, jak skierować ruch do określonych punktów końcowych na podstawie różnych metryk przy użyciu wzorca aplikacji rozproszonej geograficznie. Tworzenie profilu Traffic Manager przy użyciu geograficznej konfiguracji routingu i punktu końcowego zapewnia, że informacje są kierowane do punktów końcowych w oparciu o wymagania regionalne, przepisy firmowe i międzynarodowe oraz Twoje potrzeby dotyczące danych.
