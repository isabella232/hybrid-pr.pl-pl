---
title: Wzorzec DevOps w centrum Azure Stack
description: Dowiedz się więcej o wzorcu DevOps, dzięki czemu możesz zapewnić spójność we wdrożeniach na platformie Azure i w centrum Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: e26056a9507a7467473b009725d4f210d9d59ec8
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477239"
---
# <a name="devops-pattern"></a>Wzorzec DevOps

Kod z jednej lokalizacji i wdrażaj w środowisku deweloperskim, testowym i produkcyjnym, które mogą znajdować się w lokalnym centrum danych, chmurach prywatnych lub w chmurze publicznej.

## <a name="context-and-problem"></a>Kontekst i problem

Ciągłość wdrażania aplikacji, bezpieczeństwo i niezawodność są niezbędne dla organizacji i kluczowych dla zespołów programistycznych.

Aplikacje często wymagają kodu refaktoryzacji do uruchomienia w każdym środowisku docelowym. Oznacza to, że aplikacja nie jest całkowicie przenośna. Musi ona być aktualizowana, przetestowana i sprawdzona w miarę poruszania się w każdym środowisku. Na przykład kod zapisany w środowisku deweloperskim musi zostać ponownie zapisany, aby działać w środowisku testowym i został zapisany, gdy ostatecznie jest w środowisku produkcyjnym. Ponadto ten kod jest odpowiednio powiązany z hostem. Zwiększa to koszty i złożoność utrzymywania aplikacji. Każda wersja aplikacji jest powiązana z każdym środowiskiem. Zwiększona złożoność i duplikacja zwiększają ryzyko związane z bezpieczeństwem i jakością kodu. Ponadto kod nie może być łatwo wdrażany ponownie po usunięciu hostów, które nie powiodły się, lub wdrożyć dodatkowe hosty w celu obsłużenia wzrostu popytu.

## <a name="solution"></a>Rozwiązanie

Wzorzec DevOps umożliwia kompilowanie, testowanie i wdrażanie aplikacji, która działa w wielu chmurach. Ten wzorzec polega na tym, że jest to rozwiązanie ciągłej integracji i ciągłego dostarczania. Dzięki ciągłej integracji kod jest kompilowany i testowany za każdym razem, gdy członek zespołu zatwierdzi zmianę kontroli wersji. Ciągłe dostarczanie automatyzuje każdy krok z kompilacji do środowiska produkcyjnego. Razem te procesy tworzą proces wydania, który obsługuje wdrażanie w różnych środowiskach. Korzystając z tego wzorca, można naszkicować kod, a następnie wdrożyć ten sam kod w środowisku lokalnym, w różnych chmurach prywatnych i chmurach publicznych. Różnice w środowisku wymagają zmiany pliku konfiguracji, a nie zmian w kodzie.

![Wzorzec DevOps](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

Mając spójny zestaw narzędzi programistycznych w środowiskach lokalnych, chmur prywatnych i chmur publicznych, można zaimplementować praktykę ciągłej integracji i ciągłego dostarczania. Aplikacje i usługi wdrożone przy użyciu wzorca DevOps są zamienne i mogą być uruchamiane w dowolnej z tych lokalizacji, korzystając z funkcji i możliwości chmury lokalnej oraz publicznej.

Użycie potoku wydania DevOps umożliwia:

- Zainicjuj nową kompilację na podstawie zatwierdzeń kodu do jednego repozytorium.
- Automatycznie Wdróż nowo utworzony kod w chmurze publicznej na potrzeby testowania akceptacji użytkownika.
- Automatycznie Wdrażaj w chmurze prywatnej, gdy kod przeszedł testy.

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Wzorzec DevOps jest przeznaczony do zapewnienia spójności między wdrożeniami niezależnie od środowiska docelowego. Jednak możliwości różnią się w środowiskach w chmurze i lokalnych. Rozważ następujące kwestie:

- Czy funkcje, punkty końcowe, usługi i inne zasoby we wdrożeniu są dostępne w docelowych lokalizacjach wdrożenia?
- Czy artefakty konfiguracji są przechowywane w lokalizacjach, które są dostępne w chmurach?
- Czy parametry wdrożenia będą działały we wszystkich środowiskach docelowych?
- Czy właściwości specyficzne dla zasobów są dostępne we wszystkich chmurach docelowych?

Aby uzyskać więcej informacji, zobacz [opracowywanie szablonów Azure Resource Manager na potrzeby spójności w chmurze](/azure/azure-resource-manager/templates-cloud-consistency).

Ponadto należy wziąć pod uwagę następujące kwestie podczas decydowania o sposobie wdrożenia tego wzorca:

### <a name="scalability"></a>Skalowalność

Systemy automatyzacji wdrażania to kluczowy punkt kontrolny w wzorcach DevOps. Implementacje mogą się różnić. Wybór prawidłowego rozmiaru serwera zależy od rozmiaru oczekiwanego obciążenia. Skalowanie maszyn wirtualnych jest droższe niż w przypadku kontenerów. Jednak aby można było skalować przy użyciu kontenerów, procesy kompilacji muszą być uruchamiane za pomocą kontenerów.

### <a name="availability"></a>Dostępność

Dostępność w kontekście DevPattern oznacza możliwość odzyskania wszelkich informacji o stanie skojarzonych z przepływem pracy, takich jak wyniki testów, zależności kodu lub inne artefakty. Aby ocenić wymagania dotyczące dostępności, weź pod uwagę dwie typowe metryki:

- Cel czasu odzyskiwania (RTO) określa, jak długo można wyjść bez systemu.

- Cel punktu odzyskiwania (RPO) wskazuje, ile danych można stracić, jeśli zakłócenia w działaniu usługi ma wpływ na system.

W RTO i cel punktu odzyskiwania implikuje nadmiarowość i kopię zapasową. W globalnej chmurze platformy Azure dostępność nie jest kwestią odzyskiwania sprzętowego, która jest częścią platformy Azure, ale nie zapewnia utrzymania stanu systemów DevOps. W centrum Azure Stack można rozważyć odzyskiwanie sprzętu.

Innym ważnym zagadnieniem podczas projektowania systemu służącego do automatyzacji wdrażania jest kontrola dostępu oraz odpowiednie zarządzanie prawami wymaganymi do wdrożenia usług w środowiskach chmurowych. Jakie prawa są potrzebne do tworzenia, usuwania i modyfikowania wdrożeń? Na przykład jeden zestaw praw jest zwykle wymagany do utworzenia grupy zasobów na platformie Azure i innej do wdrożenia usług w grupie zasobów.

### <a name="manageability"></a>Możliwości zarządzania

Projektowanie dowolnego systemu opartego na wzorcu DevOps musi uwzględniać automatyzację, rejestrowanie i alerty dla każdej usługi w ramach portfolio. Używaj usług udostępnionych, zespołów aplikacji lub obu, a także Śledź zasady zabezpieczeń i zarządzanie.

Wdrażaj środowiska produkcyjne i środowiska deweloperskie/testowe w oddzielnych grupach zasobów na platformie Azure lub w centrum Azure Stack. Następnie możesz monitorować zasoby każdego środowiska i rzutować koszty rozliczeń według grupy zasobów. Możesz również usunąć zasoby jako zestaw, który jest przydatny w przypadku wdrożeń testowych.

## <a name="when-to-use-this-pattern"></a>Kiedy używać tego wzorca

Użyj tego wzorca, jeśli:

- Można opracowywać kod w jednym środowisku, który spełnia potrzeby deweloperów, i wdrożyć go w środowisku specyficznym dla Twojego rozwiązania, gdzie może być trudne do opracowania nowego kodu.
- Możesz użyć kodu i narzędzi, dla których deweloperzy chcą, o ile mogą postępować zgodnie z procesem ciągłej integracji i ciągłego dostarczania we wzorcu DevOps.

Ten wzorzec nie jest zalecany:

- Jeśli nie możesz zautomatyzować infrastruktury, zasobów aprowizacji, konfiguracji, tożsamości i zabezpieczeń.
- Jeśli zespoły nie mają dostępu do zasobów chmury hybrydowej w celu zaimplementowania podejścia ciągłej integracji/ciągłego wdrażania (CI/CD).

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:

- Zapoznaj się z [dokumentacją usługi Azure DevOps](/azure/devops) , aby dowiedzieć się więcej o usłudze Azure DevOps i powiązanych narzędziach, w tym Azure Repos i Azure Pipelines.
- Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika wdrażania hybrydowej integracji DevOps/CD](https://aka.ms/hybriddevopsdeploy). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników. Dowiesz się, jak wdrożyć aplikację na platformie Azure i Azure Stack Hub przy użyciu potoku hybrydowej ciągłej integracji/ciągłego dostarczania (CI/CD).
