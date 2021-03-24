---
title: Wzorce hybrydowe i przykłady rozwiązań dla platformy Azure i centrum Azure Stack
description: Omówienie wzorców hybrydowych i przykładów rozwiązań służących do uczenia się i tworzenia rozwiązań hybrydowych na platformie Azure i Azure Stack centrum.
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4f86e5ae4b8b9bd7693617b07419b67dfcf05dc1
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895316"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a>Wzorce hybrydowe i przykłady rozwiązań dla platformy Azure i Azure Stack

Firma Microsoft udostępnia produkty i rozwiązania platformy Azure i Azure Stack jako jeden spójny ekosystem platformy Azure. Rodzina Microsoft Azure Stack to rozszerzenie platformy Azure.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Chmura hybrydowa i aplikacje hybrydowe

Azure Stack zapewnia elastyczność obliczeń w chmurze w środowisku lokalnym i na krawędzi przez włączenie *chmury hybrydowej*. Azure Stack Hub, Azure Stack HCL i Azure Stack Edge rozszerzenie platformy Azure z chmury do swoich suwerennych centrów danych, biur oddziałów, pól i poza nią. Dzięki temu różnorodnemu zestawowi możliwości można:

- Używaj ponownie kodu i Uruchamiaj aplikacje natywne w chmurze na platformie Azure i w środowiskach lokalnych.
- Uruchamiaj tradycyjne Zwirtualizowane obciążenia z opcjonalnymi połączeniami z usługami platformy Azure.
- Przenieś dane do chmury lub Zachowaj je w suwerennym centrum danych, aby zachować zgodność.
- Uruchamiaj przyspieszane sprzętowo Uczenie maszynowe, kontenery lub zwirtualizowane obciążenia, a wszystko to w inteligentnej krawędzi.

Aplikacje obejmujące chmury są również nazywane *aplikacjami hybrydowymi*. Możesz tworzyć hybrydowe aplikacje w chmurze na platformie Azure i wdrażać je w połączonym lub rozłączonym centrum danych znajdującym się w dowolnym miejscu.

Scenariusze aplikacji hybrydowych różnią się w zależności od zasobów, które są dostępne do programowania. Obejmują one również kwestie, takie jak Geografia, zabezpieczenia, dostęp do Internetu i inne. Chociaż opisane tutaj wzorce i rozwiązania mogą nie dotyczyć wszystkich wymagań, udostępniają wskazówki i przykłady umożliwiające eksplorowanie i ponowne użycie podczas implementowania rozwiązań hybrydowych.

## <a name="design-patterns"></a>Wzorce projektowe

Wzorce projektowe wycofane wskazówki dotyczące projektowania, które zostały uogólnione z rzeczywistymi scenariuszami i środowiskami klientów. Wzorzec jest abstrakcyjny, co pozwala na zastosowanie go do różnych typów scenariuszy lub branż w pionie. Każdy wzorzec dokumentuje kontekst i problem i zawiera omówienie przykładu rozwiązania. Przykładem rozwiązania jest to możliwa implementacja wzorca.

Istnieją dwa typy artykułów wzorca:

- Pojedynczy wzorzec: zawiera wskazówki dotyczące projektowania dla jednego scenariusza ogólnego przeznaczenia.
- Wiele wzorców: zawiera wskazówki dotyczące projektowania, w których używane są aplikacje wielu wzorców. Ten wzorzec jest często wymagany do rozwiązywania bardziej złożonych scenariuszy lub problemów specyficznych dla branż.

## <a name="solution-deployment-guides"></a>Przewodniki wdrażania rozwiązania

Wskazówki krok po kroku ułatwiają wdrożenie przykładowego rozwiązania. Przewodnik może również odnosić się do przykładowego kodu pomocnika, który jest przechowywany w [przykładowym repozytorium rozwiązań](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)usługi GitHub.

## <a name="next-steps"></a>Następne kroki

- Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.
- Zapoznaj się z sekcją "wzorce" i "przewodniki wdrażania rozwiązania" spisu treści, aby dowiedzieć się więcej o każdej z nich.
- Przeczytaj o [zagadnieniach dotyczących projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby przejrzeć filary jakości oprogramowania do projektowania, wdrażania i obsługi aplikacji hybrydowych.
- [Skonfiguruj środowisko programistyczne na Azure Stack](/azure-stack/user/azure-stack-dev-start) i [Wdróż swoją pierwszą aplikację](/azure-stack/user/azure-stack-dev-start-deploy-app) na Azure Stack.
