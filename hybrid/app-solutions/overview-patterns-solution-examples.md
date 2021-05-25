---
title: Wzorce hybrydowe i przykłady rozwiązań dla platformy Azure i Azure Stack Hub
description: Omówienie hybrydowych wzorców i przykładów rozwiązań do uczenia się i tworzenia rozwiązań hybrydowych na platformie Azure i Azure Stack Hub.
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343862"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>Wzorce i przykłady rozwiązań hybrydowych dla platformy Azure i Azure Stack

Firma Microsoft udostępnia platformę Azure Azure Stack i rozwiązania jako jeden spójny ekosystem platformy Azure. Rodzina Microsoft Azure Stack to rozszerzenie platformy Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Chmura hybrydowa i aplikacje hybrydowe

Azure Stack zapewnia elastyczność przetwarzania w chmurze w środowisku lokalnym i na brzegu sieci, włączając *chmurę hybrydową.* Azure Stack Hub, Azure Stack HCI i Azure Stack Edge platformy Azure z chmury na suwerenne centra danych, oddziały, pola i nie tylko. Ten zróżnicowany zestaw możliwości umożliwia:

- Ponowne używanie kodu i spójne uruchamianie aplikacji natywnych dla chmury na platformie Azure i w środowiskach lokalnych.
- Uruchamiaj tradycyjne zwirtualizowane obciążenia z opcjonalnymi połączeniami z usługami platformy Azure.
- Przesyłaj dane do chmury lub przechowuj je w suwerennych centrach danych, aby zachować zgodność.
- Uruchamiaj sprzętowo przyspieszone obciążenia uczenia maszynowego, konteneryzowane lub zwirtualizowane na inteligentnych urządzeniach brzegowych.

Aplikacje obejmujące chmury są również nazywane aplikacjami *hybrydowymi.* Możesz tworzyć hybrydowe aplikacje w chmurze na platformie Azure i wdrażać je w połączonym lub odłączonym centrum danych, które znajduje się w dowolnym miejscu.

Scenariusze aplikacji hybrydowych różnią się znacznie w zależności od zasobów dostępnych do tworzenia. Obejmują one również zagadnienia, takie jak lokalizacja geograficzna, zabezpieczenia, dostęp do Internetu i inne. Chociaż opisane tutaj wzorce i przykłady rozwiązań mogą nie spełniać wszystkich wymagań, zawierają wskazówki i przykłady do zbadania i ponownego użycia podczas wdrażania rozwiązań hybrydowych.

## <a name="solution-patterns"></a>Wzorce rozwiązań

Wzorce rozwiązań są nakierowane na uogólnione, powtarzalne wskazówki dotyczące projektowania z rzeczywistych scenariuszy i doświadczeń klientów. Wzorzec jest abstrakcyjny, co pozwala na zastosowanie go do różnych typów scenariuszy lub branż pionowych. Każdy wzorzec dokumentuje kontekst i problem oraz zawiera omówienie przykładowego rozwiązania. Przykład rozwiązania jest przeznaczony jako możliwa implementacja wzorca.

Istnieją dwa typy artykułów dotyczących wzorców:

- Pojedynczy wzorzec: zawiera wskazówki dotyczące projektowania dla pojedynczego scenariusza ogólnego przeznaczenia.
- Wiele wzorców: zawiera wskazówki dotyczące projektowania, w których jest używane zastosowanie wielu wzorców. Ten wzorzec jest często wymagany do rozwiązywania bardziej złożonych scenariuszy lub problemów branżowych.

## <a name="solution-deployment-guides"></a>Przewodniki wdrażania rozwiązania

Przewodniki krok po kroku dotyczące wdrażania pomagają we wdrażaniu przykładowego rozwiązania. Przewodnik może również odwoływać się do przykładowego kodu towarzyszącego przechowywanego w repozytorium [przykładowych](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)rozwiązań Usługi GitHub.

## <a name="next-steps"></a>Następne kroki

- Zobacz Azure Stack [produktów i rozwiązań,](/azure-stack) aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.
- Zapoznaj się z sekcjami "Patterns" (Wzorce) i "Solution deployment guides" (Przewodniki wdrażania rozwiązań) w owym przewodniku, aby dowiedzieć się więcej o każdym z nich.
- Przeczytaj o [zagadnieniach dotyczących projektowania aplikacji hybrydowych,](overview-app-design-considerations.md) aby zapoznać się z filarami jakości oprogramowania w zakresie projektowania, wdrażania i obsługi aplikacji hybrydowych.
- [Skonfiguruj środowisko projektowe na komputerze Azure Stack](/azure-stack/user/azure-stack-dev-start) i [wd wdrażaj pierwszą aplikację na](/azure-stack/user/azure-stack-dev-start-deploy-app) Azure Stack.
