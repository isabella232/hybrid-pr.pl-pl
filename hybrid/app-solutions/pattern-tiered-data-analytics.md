---
title: Dane warstwowe dla wzorca analizy przy użyciu platformy Azure i usługi Azure Stack Hub
description: Dowiedz się, jak wdrożyć rozwiązanie danych warstwowych w chmurze hybrydowej za pomocą platformy Azure i usługi Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911247"
---
# <a name="tiered-data-for-analytics-pattern"></a>Dane warstwowe dla wzorca analizy

Ten wzorzec ilustruje sposób używania centrum Azure Stack i platformy Azure do przygotowywania, analizowania, przetwarzania, oczyszczania i przechowywania danych w wielu lokalizacjach lokalnych i w chmurze.

## <a name="context-and-problem"></a>Kontekst i problem

Jednym z problemów związanych z organizacjami przedsiębiorstwa w nowoczesnej technologii jest zapewnienie bezpiecznego przechowywania, przetwarzania i analizowania danych. Zagadnienia obejmują:

- zawartość danych
- location
- wymagania dotyczące zabezpieczeń i ochrony prywatności
- uprawnienia dostępu
- konserwacji
- magazynowanie magazynu

Platforma Azure, w połączeniu z centrum Azure Stack, rozwiązuje problemy z danymi i oferuje rozwiązania o niskich kosztach. To rozwiązanie jest najlepiej wyrażone w ramach rozproszonej firmy produkcyjnej lub logistycznej.

Rozwiązanie jest oparte na następującym scenariuszu:

- Duża organizacja produkcyjna z obsługą wielu gałęzi.
- Wymagane są szybkie i bezpieczne przechowywanie danych, przetwarzanie i dystrybucja między globalnymi lokalizacjami zdalnymi a centralnym siedzibą firmy.
- Działania pracowników i maszyn, informacje o funkcji i dane raportowania biznesowe, które muszą pozostać bezpieczne. Dane muszą być odpowiednio dystrybuowane i zgodne z regionalnymi zasadami zgodności i przepisami branżowymi.

## <a name="solution"></a>Rozwiązanie

Korzystanie z środowisk lokalnych i chmurowych w chmurze jest zgodne z wymaganiami firmy dla wielu firm. Azure Stack Hub oferuje szybkie, bezpieczne i elastyczne rozwiązanie do zbierania, przetwarzania, przechowywania i dystrybuowania danych lokalnych i zdalnych. Ten wzorzec jest szczególnie przydatny, gdy wymagania dotyczące zabezpieczeń, poufności, zasad firmowych i przepisów prawnych mogą się różnić między lokalizacjami i użytkownikami.

![Wzorzec danych warstwowych dla architektury rozwiązań analitycznych](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>Składniki

Ten wzorzec używa następujących składników:

| Warstwa | Składnik | Opis |
|----------|-----------|-------------|
| Azure | Magazyn | Konto [usługi Azure Storage](/azure/storage/) zapewnia sterylny punkt końcowy zużycia danych. Usługa Azure Storage to rozwiązanie do magazynowania w chmurze firmy Microsoft dla nowoczesnych scenariuszy magazynowania danych. Usługa Azure Storage oferuje wysoce skalowalny magazyn obiektów dla obiektów danych i usługi systemu plików dla chmury. Udostępnia również magazyn komunikatów dla niezawodnej obsługi komunikatów i magazynu NoSQL. |
| Azure Stack Hub | Magazyn | Konto [magazynu Azure Stack Hub](/azure-stack/user/azure-stack-storage-overview) jest używane w wielu usługach:<br><br>- **Magazyn obiektów BLOB** dla magazynu nieprzetworzonych danych. Magazyn obiektów BLOB może przechowywać dowolny typ danych tekstowych lub binarnych, takich jak dokument, plik multimedialny lub Instalator aplikacji. Każdy obiekt BLOB jest zorganizowany w kontenerze. Kontenery zapewniają przydatny sposób przypisywania zasad zabezpieczeń do grup obiektów. Konto magazynu może zawierać dowolną liczbę kontenerów, a kontener może zawierać dowolną liczbę obiektów blob z limitem pojemności 500 TB dla konta magazynu.<br>- **Magazyn obiektów BLOB** na potrzeby archiwum danych. W przypadku archiwizowania danych chłodnych istnieją zalety niskich kosztów magazynu. Przykładami chłodnych danych są kopie zapasowe, zawartość multimedialna, dane naukowe, zgodność i dane archiwalne. Ogólnie rzecz biorąc, wszystkie dane, do których często uzyskuje się dostęp, są traktowane jako chłodny magazyn. Warstwowe dane na podstawie atrybutów, takich jak częstotliwość dostępu i okres przechowywania. Dane klientów są rzadko używane, ale wymagają podobnych opóźnień i wydajności w celu uzyskania dostępu do danych.<br>- **Kolejkowanie magazynu** dla przetworzonych magazynów danych. Usługa queue storage zapewnia obsługę komunikatów w chmurze między składnikami aplikacji. Podczas projektowania aplikacji do skalowania składniki aplikacji są często rozłączane, aby można je było skalować niezależnie. Usługa queue storage zapewnia asynchroniczne przesyłanie komunikatów na potrzeby komunikacji między składnikami aplikacji, niezależnie od tego, czy działają w chmurze, na komputerze, na serwerze lokalnym, czy na urządzeniu przenośnym. Magazyn kolejek obsługuje również zarządzanie asynchronicznymi zadaniami oraz przepływy pracy procesu kompilacji. |
| | Azure Functions | Usługa [Azure Functions](/azure/azure-functions/) jest udostępniana przez Azure App Service dostawcy zasobów [centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-overview) . Azure Functions umożliwia wykonywanie kodu w prostym środowisku bezserwerowym w odpowiedzi na różne zdarzenia. Azure Functions skalowanie do zaspokajania popytu bez konieczności tworzenia maszyny wirtualnej lub publikowania aplikacji sieci Web przy użyciu wybranego języka programowania. Funkcja jest używana przez rozwiązanie dla:<br><br>- **Pobieranie danych**<br>- **Sterylizacja danych.** Funkcje wyzwalane ręcznie mogą wykonywać zaplanowane przetwarzanie danych, czyścić i archiwizowanie. Przykłady mogą obejmować nocne i jednodniowe szybkie tworzenie raportów.|

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas decydowania o sposobie wdrożenia tego rozwiązania należy wziąć pod uwagę następujące kwestie:

### <a name="scalability"></a>Skalowalność

Skalowanie rozwiązań Azure Functions i magazynu w celu spełnienia wymagań dotyczących ilości danych i przetwarzania. Aby uzyskać informacje dotyczące skalowalności i elementów docelowych platformy Azure, zobacz [dokumentację dotyczącą skalowalności usługi Azure Storage](/azure/storage/common/storage-scalability-targets).

### <a name="availability"></a>Dostępność

Magazyn jest podstawową kwestią dostępności dla tego wzorca. Połączenie przez szybkie linki jest wymagane do przetwarzania i rozpowszechniania dużych ilości danych.

### <a name="manageability"></a>Możliwości zarządzania

Łatwość zarządzania tego rozwiązania zależy od narzędzi autorskich używanych i zaangażowania kontroli źródła.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:

- Zobacz dokumentację [usługi Azure Storage](/azure/storage/) i [Azure Functions](/azure/azure-functions/) . Ten wzorzec sprawia, że duże wykorzystanie kont usługi Azure Storage i Azure Functions na platformie Azure i w centrum Azure Stack.
- Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na dodatkowe pytania.
- Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika wdrażania rozwiązań analitycznych dla danych warstwowych](https://aka.ms/tiereddatadeploy). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.
