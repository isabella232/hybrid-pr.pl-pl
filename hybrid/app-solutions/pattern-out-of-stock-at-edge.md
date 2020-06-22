---
title: Wykrywanie zapasów przy użyciu platformy Azure i Azure Stack Edge
description: Dowiedz się, w jaki sposób używać usług Azure i Azure Stack Edge w celu zaimplementowania wykrywania magazynu.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911191"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Wykrywanie zapasów na wzorcu krawędzi

Ten wzorzec ilustruje, jak ustalić, czy półki zawierają elementy podstawowe przy użyciu Azure Stack krawędzi lub urządzenia Azure IoT Edge i kamer sieciowych.

## <a name="context-and-problem"></a>Kontekst i problem

Fizyczne magazyny detaliczne powodują utratę sprzedaży, ponieważ klienci szukają elementu, ale nie ma go na półce. Jednak element może znajdować się w tylnej części sklepu i nie został odnotowany. Sklepy chcą wydajniej korzystać z swoich pracowników i uzyskiwać automatyczne powiadomienia, gdy elementy wymagają uzupełnienia.

## <a name="solution"></a>Rozwiązanie

Przykładem rozwiązania jest użycie urządzenia brzegowego, takiego jak Azure Stack Edge w każdym sklepie, co skutecznie przetwarza dane z kamer w sklepie. Ten zoptymalizowany projekt pozwala przechowywać tylko odpowiednie zdarzenia i obrazy do chmury. Projekt oszczędza przepustowość, miejsce do magazynowania i zapewnia prywatność klientów. Ponieważ ramki są odczytywane z każdego aparatu, model ML przetwarza obraz i zwraca wszystkie obszary giełdowe. Obraz i obszary podstawowe są wyświetlane w lokalnej aplikacji sieci Web. Te dane mogą być wysyłane do środowiska usługi Time Series Insights w celu wyświetlenia szczegółowych informacji w Power BI.

![Poza zapasami w architekturze rozwiązania brzegowego](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Oto jak działa Rozwiązanie:

1. Obrazy są przechwytywane z aparatu sieciowego za pośrednictwem protokołu HTTP lub RTSP.
2. Rozmiar obrazu jest zmieniany i wysyłany do sterownika wnioskowania, który komunikuje się z modelem ML, aby określić, czy istnieją obrazy poza zapasami.
3. Model ML zwraca wszystkie obszary giełdowe.
4. Sterownik inferencing przekazuje Nieprzetworzony obraz do obiektu BLOB (Jeśli zostanie określony) i wysyła wyniki z modelu do usługi Azure IoT Hub i procesora pola ograniczenia na urządzeniu.
5. Procesor pola ograniczenia dodaje do obrazu pola związane z ograniczeniami i buforuje ścieżkę obrazu w bazie danych w pamięci.
6. Aplikacja sieci Web wyszukuje obrazy i wyświetla je w kolejności odbierania.
7. Komunikaty z IoT Hub są agregowane w Time Series Insights.
8. Power BI wyświetla interaktywny raport z elementów giełdowych z upływem czasu przy użyciu danych z Time Series Insights.


## <a name="components"></a>Składniki

To rozwiązanie używa następujących składników:

| Warstwa | Składnik | Opis |
|----------|-----------|-------------|
| Sprzęt lokalny | Aparat sieciowy | Wymagana jest kamera sieciowa z kanałem informacyjnym HTTP lub RTSP, aby zapewnić obrazy do wnioskowania. |
| Azure | Azure IoT Hub | [Usługa Azure IoT Hub](/azure/iot-hub/) obsługuje obsługę administracyjną urządzeń i obsługę komunikatów na urządzeniach brzegowych. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) przechowuje komunikaty z IoT Hub do wizualizacji. |
|  | Power BI | [Firma Microsoft Power BI](https://powerbi.microsoft.com/) zapewnia firmowe raporty dotyczące braku zdarzeń giełdowych. Power BI zapewnia łatwy w użyciu interfejs nawigacyjny do wyświetlania danych wyjściowych z Azure Stream Analytics. |
| Azure Stack Edge lub<br>Urządzenie Azure IoT Edge | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) organizuje środowisko uruchomieniowe kontenerów lokalnych i obsługuje zarządzanie urządzeniami i ich aktualizacje.|
| | Brainwave projektu platformy Azure | Na urządzeniu brzegowym w Azure Stack [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) używa opartych na programowalnych tablic bram (FPGA) do przyspieszania inferencing ml.|

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas decydowania o sposobie wdrożenia tego rozwiązania należy wziąć pod uwagę następujące kwestie:

### <a name="scalability"></a>Skalowalność

Większość modeli uczenia maszynowego może działać tylko na określonej liczbie klatek na sekundę, w zależności od podanego sprzętu. Określ optymalną częstotliwość próbkowania z kamer, aby upewnić się, że potok ML nie ma kopii zapasowej. Różne typy sprzętu będą obsługiwać różne liczby kamer i szybkości klatek.

### <a name="availability"></a>Dostępność

Ważne jest, aby wziąć pod uwagę, co się dzieje w przypadku utraty łączności przez urządzenie brzegowe. Zastanów się, jakie dane mogą zostać utracone z Time Series Insights i Power BI pulpitu nawigacyjnego. Przykładowe rozwiązanie jako dostarczone nie jest przeznaczone do wysokiej dostępności.

### <a name="manageability"></a>Możliwości zarządzania

To rozwiązanie może obejmować wiele urządzeń i lokalizacji, które mogą uzyskać nieporęczny. Usługi IoT platformy Azure mogą automatycznie przenosić nowe lokalizacje i urządzenia w tryb online i aktualizować je na bieżąco. Należy również przestrzegać odpowiednich procedur zarządzania danymi.

### <a name="security"></a>Zabezpieczenia

Ten wzorzec obsługuje potencjalnie poufne dane. Upewnij się, że klucze są regularnie obracane, a uprawnienia na koncie usługi Azure Storage i lokalnych udziałach zostały prawidłowo ustawione.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:
- W tym wzorcu są używane wiele usług związanych z usługą IoT, w tym [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/)i [Azure Time Series Insights](/azure/time-series-insights/).
- Aby dowiedzieć się więcej o programie Microsoft Project Brainwave, zapoznaj [się z ogłoszeniem na blogu](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) i wyewidencjonuj [Machine Learning przyspieszonej platformy Azure za pomocą programu Project Brainwave wideo](https://www.youtube.com/watch?v=DJfMobMjCX0).
- Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na wszelkie dodatkowe pytania.
- Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika wdrażania rozwiązań analitycznych dla danych warstwowych](https://aka.ms/edgeinferencingdeploy). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.
