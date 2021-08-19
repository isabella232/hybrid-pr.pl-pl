---
title: Wzorzec aplikacji rozproszonej geograficznie w Azure Stack Hub
description: Dowiedz się więcej o wzorcu aplikacji rozproszonej geograficznie dla inteligentnej krawędzi korzystającej z platformy Azure i Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 3c839d9bf3b6c3e1ff50cc695fd5f1a1127793d2
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281231"
---
# <a name="geo-distributed-app-pattern"></a>Wzorzec aplikacji rozproszonej geograficznie

Dowiedz się, jak zapewnić punkty końcowe aplikacji w wielu regionach i rozsyłać ruch użytkowników na podstawie potrzeb dotyczących lokalizacji i zgodności.

## <a name="context-and-problem"></a>Kontekst i problem

Organizacje z szerokimi obszarami geograficznymi dokładają starań, aby bezpiecznie i dokładnie dystrybuować i zapewniać dostęp do danych przy jednoczesnym zapewnieniu wymaganych poziomów zabezpieczeń, zgodności i wydajności poszczególnych użytkowników, lokalizacji i urządzeń między granicami.

## <a name="solution"></a>Rozwiązanie

Wzorzec Azure Stack Hub geograficznego ruchu lub aplikacje rozproszone geograficznie umożliwiają kierowanie ruchu do określonych punktów końcowych na podstawie różnych metryk. Utworzenie planu Traffic Manager routingiem geograficznym i konfiguracją punktu końcowego kieruje ruch do punktów końcowych na podstawie wymagań regionalnych, przepisów firmowych i międzynarodowych oraz potrzeb dotyczących danych.

![Wzorzec rozproszony geograficznie](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Składniki

### <a name="outside-the-cloud"></a>Poza chmurą

#### <a name="traffic-manager"></a>Traffic Manager

Na diagramie Traffic Manager znajduje się poza chmurą publiczną, ale musi mieć możliwość koordynowania ruchu zarówno w lokalnym centrum danych, jak i w chmurze publicznej. Równoważenie kieruje ruch do lokalizacji geograficznych.

#### <a name="domain-name-system-dns"></a>System nazw domen (DNS)

System nazw domen (DNS) jest odpowiedzialny za tłumaczenie (lub rozpoznawanie) nazwy witryny internetowej lub usługi na jej adres IP.

### <a name="public-cloud"></a>Chmura publiczna

#### <a name="cloud-endpoint"></a>Punkt końcowy w chmurze

Publiczne adresy IP są używane do rozsyłania ruchu przychodzącego za pośrednictwem usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.  

### <a name="local-clouds"></a>Chmury lokalne

#### <a name="local-endpoint"></a>Lokalny punkt końcowy

Publiczne adresy IP są używane do rozsyłania ruchu przychodzącego za pośrednictwem usługi Traffic Manager do punktu końcowego zasobów aplikacji w chmurze publicznej.

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas podejmowania decyzji o sposobie wdrożenia tego wzorca należy rozważyć następujące punkty:

### <a name="scalability"></a>Skalowalność

Wzorzec obsługuje geograficzny routing ruchu, a nie skalowanie w celu zaspokojenia wzrostu ruchu. Można jednak połączyć ten wzorzec z innymi rozwiązaniami platformy Azure i lokalnymi. Ten wzorzec może być na przykład używany ze wzorcem skalowania między chmurami.

### <a name="availability"></a>Dostępność

Upewnij się, że lokalnie wdrożone aplikacje są skonfigurowane do wysokiej dostępności za pośrednictwem lokalnej konfiguracji sprzętu i wdrażania oprogramowania.

### <a name="manageability"></a>Możliwości zarządzania

Wzorzec zapewnia bezproblemowe zarządzanie i znany interfejs między środowiskami.

## <a name="when-to-use-this-pattern"></a>Kiedy używać tego wzorca

- Moja organizacja ma oddziały międzynarodowe wymagające niestandardowych regionalnych zasad zabezpieczeń i dystrybucji.
- Każde z biur mojej organizacji ściąga dane pracowników, firm i obiektów, co wymaga działań raportowania według lokalnych przepisów i strefy czasowej.
- Wymagania dotyczące dużej skali można spełnić przez skalowanie w poziomie aplikacji w poziomie, przy użyciu wielu wdrożeń aplikacji w jednym regionie i w różnych regionach w celu obsługi skrajnych wymagań dotyczących obciążenia.
- Aplikacje muszą być wysoce dostępne i odpowiadać na żądania klientów nawet w przypadku 3000 000 000 000 000 000 000 000 000 000 000 00

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów owanych w tym artykule:

- Zobacz omówienie [Azure Traffic Manager,](/azure/traffic-manager/traffic-manager-overview) aby dowiedzieć się więcej o tym, jak działa ten oparty na systemie DNS równoważenie obciążenia ruchu.
- Zobacz [Zagadnienia dotyczące projektowania aplikacji hybrydowych,](overview-app-design-considerations.md) aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na wszelkie dodatkowe pytania.
- Zobacz Azure Stack [produktów i rozwiązań,](/azure-stack) aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do przetestowania przykładowego rozwiązania, przejdź do przewodnika wdrażania rozwiązania aplikacji rozproszonej [geograficznie.](/azure/architecture/hybrid/deployments/solution-deployment-guide-geo-distributed) Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników. Dowiesz się, jak kierować ruch do określonych punktów końcowych na podstawie różnych metryk przy użyciu wzorca aplikacji rozproszonej geograficznie. Utworzenie profilu Traffic Manager z routingiem geograficznym i konfiguracją punktu końcowego gwarantuje, że informacje są kierowane do punktów końcowych na podstawie wymagań regionalnych, przepisów firmowych i międzynarodowych oraz potrzeb dotyczących danych.