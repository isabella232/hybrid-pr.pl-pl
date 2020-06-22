---
title: Wzorzec skalowania między chmurami w centrum Azure Stack
description: Dowiedz się, jak utworzyć skalowalną aplikację międzychmurową na platformie Azure i w centrum Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911135"
---
# <a name="cross-cloud-scaling-pattern"></a>Wzorzec skalowania między chmurami

Automatycznie Dodaj zasoby do istniejącej aplikacji, aby zwiększyć obciążenie.

## <a name="context-and-problem"></a>Kontekst i problem

Twoja aplikacja nie może zwiększyć wydajności, aby sprostać nieoczekiwanym wzrostom popytu. Ten brak skalowalności powoduje, że użytkownicy nie docierają do aplikacji w czasie szczytowego użycia. Aplikacja może obsłużyć określoną liczbę użytkowników.

Przedsiębiorstwa globalne wymagają bezpiecznych, niezawodnych i dostępnych aplikacji opartych na chmurze. Zwiększenie popytu i użycie odpowiedniej infrastruktury do obsługi tego żądania ma kluczowe znaczenie. Firmy nie mogą zrównoważyć kosztów i konserwacji dzięki zabezpieczeniom danych biznesowych, magazynowaniu i dostępności w czasie rzeczywistym.

Uruchomienie aplikacji w chmurze publicznej może nie być możliwe. Niemniej jednak firma może nie być ekonomicznie wykonalna w celu utrzymania pojemności wymaganej w środowisku lokalnym w celu obsłużenia podaży na żądanie dla aplikacji. Korzystając z tego wzorca, można wykorzystać elastyczność chmury publicznej z rozwiązaniem lokalnym.

## <a name="solution"></a>Rozwiązanie

Wzorzec skalowania między chmurami rozszerza aplikację znajdującą się w chmurze lokalnej z zasobami chmury publicznej. Wzorzec jest wyzwalany przez zwiększenie lub zmniejszenie zapotrzebowania, a odpowiednio dodaje lub usuwa zasoby w chmurze. Te zasoby zapewniają nadmiarowość, szybką dostępność i Routing zgodny ze geograficzną.

![Wzorzec skalowania między chmurami](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Ten wzorzec dotyczy tylko bezstanowych składników aplikacji.

## <a name="components"></a>Składniki

Wzorzec skalowania między chmurami składa się z następujących składników.

### <a name="outside-the-cloud"></a>Poza chmurą

#### <a name="traffic-manager"></a>Traffic Manager

Na diagramie znajduje się poza grupą chmury publicznej, ale musi być w stanie koordynować ruch zarówno w lokalnym centrum danych, jak i w chmurze publicznej. Moduł równoważenia obciążenia zapewnia wysoką dostępność aplikacji przez monitorowanie punktów końcowych i udostępnianie ponownej dystrybucji trybu failover, jeśli jest to wymagane.

#### <a name="domain-name-system-dns"></a>System nazw domen (DNS)

System nazw domen (DNS) jest odpowiedzialny za tłumaczenie (lub rozwiązanie) nazwy witryny sieci Web lub usługi na adres IP.

### <a name="cloud"></a>Chmura

#### <a name="hosted-build-server"></a>Hostowany serwer kompilacji

Środowisko do hostowania potoku kompilacji.

#### <a name="app-resources"></a>Zasoby aplikacji

Zasoby aplikacji muszą mieć możliwość skalowania w poziomie i skalowalności, tak jak w przypadku zestawów skalowania maszyn wirtualnych i kontenerów.

#### <a name="custom-domain-name"></a>Niestandardowa nazwa domeny

Użyj niestandardowej nazwy domeny dla żądań routingu globalizowania.

#### <a name="public-ip-addresses"></a>Publiczne adresy IP

Publiczne adresy IP są używane do kierowania ruchu przychodzącego za pomocą usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.  

### <a name="local-cloud"></a>Chmura lokalna

#### <a name="hosted-build-server"></a>Hostowany serwer kompilacji

Środowisko do hostowania potoku kompilacji.

#### <a name="app-resources"></a>Zasoby aplikacji

Zasoby aplikacji muszą mieć możliwość skalowania w poziomie i skalowania, takich jak zestawy skalowania maszyn wirtualnych i kontenery.

#### <a name="custom-domain-name"></a>Niestandardowa nazwa domeny

Użyj niestandardowej nazwy domeny dla żądań routingu globalizowania.

#### <a name="public-ip-addresses"></a>Publiczne adresy IP

Publiczne adresy IP są używane do kierowania ruchu przychodzącego za pomocą usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas podejmowania decyzji o sposobie wdrożenia tego wzorca należy rozważyć następujące punkty:

### <a name="scalability"></a>Skalowalność

Kluczowym elementem skalowania między chmurami jest możliwość dostarczania skalowania na żądanie. Skalowanie musi nastąpić między chmurą publiczną a lokalną i zapewnić spójną, niezawodną usługę na żądanie.

### <a name="availability"></a>Dostępność

Upewnij się, że aplikacje wdrożone lokalnie są skonfigurowane pod kątem wysokiej dostępności za poorednictwem konfiguracji sprzętu lokalnego i wdrożenia oprogramowania.

### <a name="manageability"></a>Możliwości zarządzania

Wzorzec między chmurami zapewnia bezproblemowe zarządzanie i przyjazny interfejs między środowiskami.

## <a name="when-to-use-this-pattern"></a>Kiedy używać tego wzorca

Użyj tego wzorca, aby:

- Gdy trzeba zwiększyć pojemność aplikacji z nieoczekiwanymi wymaganiami lub okresowe zapotrzebowanie na żądanie.
- Gdy nie chcesz inwestować w zasoby, które będą używane tylko podczas szczytów. Płacisz za to, czego używasz.

Ten wzorzec nie jest zalecany w przypadku:

- Twoje rozwiązanie wymaga od użytkowników nawiązywania połączenia przez Internet.
- Twoja firma ma lokalne regulacje, które wymagają, aby połączenie pochodzące z wywołania Onsite było dostępne.
- Twoja sieć ma stałe wąskie gardła, które ograniczają wydajność skalowania.
- Twoje środowisko zostało odłączone od Internetu i nie może nawiązać połączenia z chmurą publiczną.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:

- Zobacz [Omówienie usługi Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) , aby dowiedzieć się więcej na temat działania tego modułu równoważenia obciążenia opartego na systemie DNS.
- Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na dodatkowe pytania.
- Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika wdrażania rozwiązań do skalowania między chmurami](solution-deployment-guide-cross-cloud-scaling.md). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników. Dowiesz się, jak utworzyć rozwiązanie międzychmurowe, aby zapewnić ręcznie wyzwolony proces przełączania z hostowanej aplikacji sieci Web Azure Stack Hub do aplikacji sieci Web hostowanej na platformie Azure. Dowiesz się również, jak używać skalowania automatycznego za pomocą usługi Traffic Manager, zapewniając elastyczne i skalowalne narzędzie w chmurze w ramach obciążenia.
