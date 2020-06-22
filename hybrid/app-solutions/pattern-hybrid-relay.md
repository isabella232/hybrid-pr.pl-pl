---
title: Wzorzec przekazywania hybrydowego na platformie Azure i w centrum Azure Stack
description: Użyj wzorca przekaźnika hybrydowego na platformie Azure i Azure Stack Hub, aby nawiązać połączenie z zasobami brzegowymi chronionymi przez zapory.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911149"
---
# <a name="hybrid-relay-pattern"></a>Wzorzec przekaźnika hybrydowego

Dowiedz się, jak łączyć się z zasobami lub urządzeniami brzegowymi chronionymi przez zapory przy użyciu wzorca przekaźnika hybrydowego i Azure Relay.

## <a name="context-and-problem"></a>Kontekst i problem

Urządzenia brzegowe często znajdują się za zaporą firmową lub urządzeniem NAT. Mimo że są one bezpieczne, mogą nie być w stanie komunikować się z chmurą publiczną lub urządzeniami brzegowymi w innych sieciach firmowych. Może być konieczne udostępnienie określonych portów i funkcji użytkownikom w chmurze publicznej w bezpieczny sposób.

## <a name="solution"></a>Rozwiązanie

Wzorzec przekaźnika hybrydowego używa Azure Relay do ustanowienia tunelu WebSockets między dwoma punktami końcowymi, które nie mogą się bezpośrednio komunikować. Urządzenia, które nie są lokalne, ale muszą połączyć się z lokalnym punktem końcowym, będą łączyć się z punktem końcowym w chmurze publicznej. Ten punkt końcowy przekierowuje ruch ze wstępnie zdefiniowanych tras za pośrednictwem bezpiecznego kanału. Punkt końcowy wewnątrz środowiska lokalnego odbiera ruch i kieruje go do poprawnego miejsca docelowego.

![Architektura rozwiązania hybrydowego wzorca przekaźnika](media/pattern-hybrid-relay/solution-architecture.png)

Oto jak działa wzorzec przekaźnika hybrydowego:

1. Urządzenie nawiązuje połączenie z maszyną wirtualną na platformie Azure przy użyciu wstępnie zdefiniowanego portu.
2. Ruch jest przekazywany do Azure Relay na platformie Azure.
3. Maszyna wirtualna w Azure Stack Hub, która ustanowiła już połączenie o długim czasie życia Azure Relay, odbiera ruch i przekazuje go do miejsca docelowego.
4. Usługa lokalna lub punkt końcowy przetwarza żądanie.

## <a name="components"></a>Składniki

To rozwiązanie używa następujących składników:

| Warstwa | Składnik | Opis |
|----------|-----------|-------------|
| Azure | Maszyna wirtualna platformy Azure | Maszyna wirtualna platformy Azure udostępnia publicznie dostępny punkt końcowy dla zasobu lokalnego. |
| | Azure Relay | [Azure Relay](/azure/azure-relay/) zapewnia infrastrukturę do obsługi tunelu i połączenia między maszyną wirtualną platformy Azure i Azure Stack węzłem maszyny wirtualnej.|
| Azure Stack Hub | Wystąpienia obliczeniowe | Maszyna wirtualna w Azure Stack Hub udostępnia po stronie serwera tunelu przekaźnika hybrydowego. |
| | Magazyn | Klaster aparatu AKS wdrożony w centrum Azure Stack udostępnia skalowalny, odporny na awarię aparat do uruchomienia kontenera interfejs API rozpoznawania twarzy.|

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas decydowania o sposobie wdrożenia tego rozwiązania należy wziąć pod uwagę następujące kwestie:

### <a name="scalability"></a>Skalowalność

Ten wzorzec zezwala tylko na mapowania portów 1:1 na kliencie i serwerze. Na przykład jeśli port 80 jest tunelowany dla jednej usługi w punkcie końcowym platformy Azure, nie można go użyć dla innej usługi. Mapowania portów powinny być odpowiednio planowane. Azure Relay i maszyny wirtualne powinny być odpowiednio skalowane w celu obsługi ruchu.

### <a name="availability"></a>Dostępność

Te tunele i połączenia nie są nadmiarowe. Aby zapewnić wysoką dostępność, można zaimplementować błąd sprawdzania kodu. Inną opcją jest posiadanie puli maszyn wirtualnych połączonych z Azure Relay za modułem równoważenia obciążenia.

### <a name="manageability"></a>Możliwości zarządzania

To rozwiązanie może obejmować wiele urządzeń i lokalizacji, które mogą uzyskać nieporęczny. Usługi IoT platformy Azure mogą automatycznie przenosić nowe lokalizacje i urządzenia w tryb online i aktualizować je na bieżąco.

### <a name="security"></a>Zabezpieczenia

Ten wzorzec, jak pokazano, umożliwia swobodnego dostęp do portu na urządzeniu wewnętrznym z krawędzi. Rozważ dodanie mechanizmu uwierzytelniania do usługi na urządzeniu wewnętrznym lub przed punktem końcowym przekaźnika hybrydowego.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:

- Ten wzorzec używa Azure Relay. Aby uzyskać więcej informacji, zobacz [dokumentację Azure Relay](/azure/azure-relay/).
- Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na wszelkie dodatkowe pytania.
- Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika wdrażania rozwiązań przekazywania hybrydowego](https://aka.ms/hybridrelaydeployment). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.