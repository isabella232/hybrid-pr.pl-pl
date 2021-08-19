---
title: Wzorzec skalowania między chmurami w Azure Stack Hub
description: Dowiedz się, jak utworzyć skalowalną aplikację międzychmurową na platformie Azure i Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 90e0c177b5eaee4d223b4613e0b2ddf385fa799c
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281265"
---
# <a name="cross-cloud-scaling-pattern"></a>Wzorzec skalowania między chmurami

Automatyczne dodawanie zasobów do istniejącej aplikacji w celu uwzględnienia wzrostu obciążenia.

## <a name="context-and-problem"></a>Kontekst i problem

Twoja aplikacja nie może zwiększyć pojemności, aby sprostać nieoczekiwanym wzrostom zapotrzebowania. Ten brak skalowalności powoduje, że użytkownicy nie docierają do aplikacji w godzinach szczytowego użycia. Aplikacja może korzystać ze stałej liczby użytkowników.

Globalne przedsiębiorstwa wymagają bezpiecznych, niezawodnych i dostępnych aplikacji opartych na chmurze. Obsługa wzrostu zapotrzebowania i użycie odpowiedniej infrastruktury do obsługi tego zapotrzebowania ma kluczowe znaczenie. Firmy mają trudności z zrównoważeniem kosztów i konserwacji z zabezpieczeniami danych biznesowych, magazynem i dostępnością w czasie rzeczywistym.

Uruchomienie aplikacji w chmurze publicznej może nie być możliwe. Jednak ekonomicznie ekonomicznie może nie być zachowanie pojemności wymaganej w środowisku lokalnym do obsługi skokowego zapotrzebowania na aplikację. Dzięki temu wzorcowi można używać elastyczności chmury publicznej z rozwiązaniem lokalnym.

## <a name="solution"></a>Rozwiązanie

Wzorzec skalowania między chmurami rozszerza aplikację znajdującą się w chmurze lokalnej z zasobami chmury publicznej. Wzorzec jest wyzwalany przez wzrost lub spadek zapotrzebowania i odpowiednio dodaje lub usuwa zasoby w chmurze. Te zasoby zapewniają nadmiarowość, szybką dostępność i routing zgodny z obszarem geograficznym.

![Wzorzec skalowania między chmurami](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Ten wzorzec dotyczy tylko bez stanowych składników aplikacji.

## <a name="components"></a>Składniki

Wzorzec skalowania między chmurami składa się z następujących składników.

### <a name="outside-the-cloud"></a>Poza chmurą

#### <a name="traffic-manager"></a>Traffic Manager

Na diagramie znajduje się on poza grupą chmury publicznej, ale musi być w stanie skoordynować ruch zarówno w lokalnym centrum danych, jak i w chmurze publicznej. Równoważenie zapewnia wysoką dostępność aplikacji dzięki monitorowaniu punktów końcowych i redystrybucji trybu failover, gdy jest to wymagane.

#### <a name="domain-name-system-dns"></a>System nazw domen (DNS)

System nazw domen (DNS) jest odpowiedzialny za tłumaczenie (lub rozpoznawanie) nazwy witryny internetowej lub usługi na jej adres IP.

### <a name="cloud"></a>Chmura

#### <a name="hosted-build-server"></a>Hostowany serwer kompilacji

Środowisko do hostowania potoku kompilacji.

#### <a name="app-resources"></a>Zasoby aplikacji

Zasoby aplikacji muszą mieć możliwość skalowania w zewnątrz i do zewnątrz, takich jak zestawy skalowania maszyn wirtualnych i kontenery.

#### <a name="custom-domain-name"></a>Niestandardowa nazwa domeny

Użyj niestandardowej nazwy domeny dla globu żądań routingu.

#### <a name="public-ip-addresses"></a>Publiczne adresy IP

Publiczne adresy IP są używane do rozsyłania ruchu przychodzącego za pośrednictwem usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.  

### <a name="local-cloud"></a>Chmura lokalna

#### <a name="hosted-build-server"></a>Hostowany serwer kompilacji

Środowisko do hostowania potoku kompilacji.

#### <a name="app-resources"></a>Zasoby aplikacji

Zasoby aplikacji muszą mieć możliwość skalowania w zewnątrz i do zewnątrz, takich jak zestawy skalowania maszyn wirtualnych i kontenery.

#### <a name="custom-domain-name"></a>Niestandardowa nazwa domeny

Użyj niestandardowej nazwy domeny dla globu żądań routingu.

#### <a name="public-ip-addresses"></a>Publiczne adresy IP

Publiczne adresy IP są używane do rozsyłania ruchu przychodzącego za pośrednictwem usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas podejmowania decyzji o sposobie wdrożenia tego wzorca należy rozważyć następujące punkty:

### <a name="scalability"></a>Skalowalność

Kluczowym składnikiem skalowania między chmurami jest możliwość skalowania na żądanie. Skalowanie musi odbywać się między infrastrukturą chmury publicznej i lokalnej i zapewniać spójną, niezawodną usługę na żądanie.

### <a name="availability"></a>Dostępność

Upewnij się, że lokalnie wdrożone aplikacje są skonfigurowane pod celu zapewnienia wysokiej dostępności za pośrednictwem konfiguracji sprzętu lokalnego i wdrażania oprogramowania.

### <a name="manageability"></a>Możliwości zarządzania

Wzorzec międzychmurowy zapewnia bezproblemowe zarządzanie i znany interfejs między środowiskami.

## <a name="when-to-use-this-pattern"></a>Kiedy używać tego wzorca

Użyj tego wzorca, aby:

- Gdy musisz zwiększyć pojemność aplikacji z nieoczekiwanymi wymaganiami lub okresowymi zapotrzebowaniem.
- Jeśli nie chcesz inwestować w zasoby, które będą używane tylko w szczytowych godzinach pracy. Płać za to, czego używasz.

Ten wzorzec nie jest zalecany w przypadku:

- Twoje rozwiązanie wymaga, aby użytkownicy łączyli się przez Internet.
- Twoja firma ma lokalne przepisy, które wymagają, aby połączenie pochodzące pochodziło z rozmowy na miejscu.
- Sieć ma zwykłe wąskie gardła, które ograniczałyby wydajność skalowania.
- Środowisko jest odłączone od Internetu i nie może uzyskać dostępu do chmury publicznej.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów owanych w tym artykule:

- Zobacz omówienie [Azure Traffic Manager,](/azure/traffic-manager/traffic-manager-overview) aby dowiedzieć się więcej na temat sposobu działania tego opartego na systemie DNS równoważenia obciążenia ruchu.
- Zobacz [Zagadnienia dotyczące projektowania aplikacji hybrydowych,](overview-app-design-considerations.md) aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na wszelkie dodatkowe pytania.
- Zobacz Azure Stack [produktów i rozwiązań,](/azure-stack) aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do przetestowania przykładowego rozwiązania, przejdź do przewodnika wdrażania rozwiązania skalowania [między chmurami](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników. Dowiesz się, jak utworzyć rozwiązanie międzychmurowe, aby zapewnić ręcznie wyzwalany proces przełączania z aplikacji internetowej hostowanej Azure Stack Hub do aplikacji internetowej hostowanej na platformie Azure. Dowiesz się również, jak używać skalowania automatycznego za pośrednictwem usługi Traffic Manager, zapewniając elastyczne i skalowalne narzędzie w chmurze pod obciążeniem.