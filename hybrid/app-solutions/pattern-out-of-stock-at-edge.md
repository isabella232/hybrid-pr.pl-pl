---
title: Wykrywanie poza magazynem przy użyciu platformy Azure i Azure Stack Edge
description: Dowiedz się, jak używać platformy Azure i Azure Stack Edge do implementowania wykrywania poza magazynem.
author: BryanLa
ms.topic: article
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: b25a6391c4e64fa7018031bac4fb7d098c56b529
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343879"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Wykrywanie poza magazynem we wzorcu brzegowym

Ten wzorzec ilustruje sposób określania, czy na półce nie ma elementów magazynowych, przy użyciu Azure Stack Edge lub Azure IoT Edge i kamer sieciowych.

## <a name="context-and-problem"></a>Kontekst i problem

Fizyczne sklepy detaliczne tracą sprzedaż, ponieważ gdy klienci poszukają produktu, nie jest on na półce. Jednak element mógł być na zasypce sklepu i nie został ponownie wsadowy. Sklepy będą chciały wydajniej korzystać ze swoich pracowników i automatycznie powiadamiać o konieczności ponownego dokańczania zapasów.

## <a name="solution"></a>Rozwiązanie

W przykładzie rozwiązania używane jest urządzenie brzegowe, takie Azure Stack Edge w każdym magazynie, które efektywnie przetwarza dane z kamer w sklepie. Ten zoptymalizowany projekt umożliwia magazyny tylko odpowiednich zdarzeń i obrazów do chmury. Projekt oszczędza przepustowość i miejsce do magazynowania oraz zapewnia prywatność klientów. Gdy ramki są odczytywane z poszczególnych aparatów, model uczenia maszynowego przetwarza obraz i zwraca wszystkie obszary poza magazynem. Obraz i obszary magazynowe są wyświetlane w lokalnej aplikacji internetowej. Te dane można wysłać do środowiska usługi Time Series Insights w celu pokazania szczegółowych informacji Power BI.

![Architektura rozwiązania out of stock at edge](media/pattern-out-of-stock-at-edge/solution-architecture.png)

Oto jak działa to rozwiązanie:

1. Obrazy są przechwytywane z kamery sieciowej za pośrednictwem protokołu HTTP lub RTSP.
2. Rozmiar obrazu jest zmieniany i wysyłany do sterownika wnioskowania, który komunikuje się z modelem uczenia maszynowego w celu określenia, czy istnieją jakieś obrazy chybne.
3. Model uczenia maszynowego zwraca wszystkie obszary magazynowe.
4. Sterownik wnioskowania przekazuje nieprzetworzony obraz do obiektu blob (jeśli został określony) i wysyła wyniki z modelu do Azure IoT Hub i procesora pola granicznego na urządzeniu.
5. Procesor pól granicznych dodaje pola granic do obrazu i buforuje ścieżkę obrazu w bazie danych w pamięci.
6. Aplikacja internetowa wykonuje zapytanie o obrazy i wyświetla je w otrzymanej kolejności.
7. Komunikaty z IoT Hub są agregowane w Time Series Insights.
8. Power BI interaktywny raport z 100 000 000 000 000 000 000 000 000 000 000 000 000 000 000 Time Series Insights.


## <a name="components"></a>Składniki

To rozwiązanie używa następujących składników:

| Warstwa | Składnik | Opis |
|----------|-----------|-------------|
| Sprzęt w środowisku lokalnym | Aparat sieciowy | Wymagana jest kamera sieciowa ze źródłami danych HTTP lub RTSP w celu zapewnienia obrazów do wnioskowania. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) obsługuje aprowizowanie urządzeń i obsługę komunikatów dla urządzeń brzegowych. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) przechowuje komunikaty z IoT Hub wizualizacji. |
|  | Power BI | [Firma Microsoft Power BI](https://powerbi.microsoft.com/) udostępnia raporty dla firmy dotyczące zdarzeń poza magazynem. Power BI udostępnia łatwy w użyciu interfejs pulpitu nawigacyjnego do wyświetlania danych wyjściowych Azure Stream Analytics. |
| Azure Stack Edge lub<br>Azure IoT Edge urządzenia | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) orkiestruje środowisko uruchomieniowe dla kontenerów lokalnych oraz obsługuje zarządzanie urządzeniami i aktualizacje.|
| | Projekt platformy Azure brainwave | Na urządzeniu Azure Stack Edge projekt [Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) używa układów Field-Programmable Gate Arrays (FPGA) do przyspieszania wnioskowania uczenia maszynowego.|

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podejmując decyzję o wdrożeniu tego rozwiązania, należy wziąć pod uwagę następujące kwestie:

### <a name="scalability"></a>Skalowalność

Większość modeli uczenia maszynowego może działać tylko z określoną liczbą ramek na sekundę, w zależności od dostarczonego sprzętu. Określ optymalną szybkość próbkowania z aparatów, aby upewnić się, że potok uczenia maszynowego nie będzie back up. Różne typy sprzętu będą obsługiwać różne liczby aparatów i klatek.

### <a name="availability"></a>Dostępność

Ważne jest, aby rozważyć, co może się zdarzyć, jeśli urządzenie brzegowe utraci łączność. Zastanów się, jakie dane mogą zostać utracone z Time Series Insights i Power BI nawigacyjnego. Podane przykładowe rozwiązanie nie zostało zaprojektowane z myślą o wysokiej dostępie.

### <a name="manageability"></a>Możliwości zarządzania

To rozwiązanie może obejmować wiele urządzeń i lokalizacji, co może być niewygodne. Usługi IoT platformy Azure mogą automatycznie wprowadzać nowe lokalizacje i urządzenia w tryb online oraz być na bieżąco z nimi aktualizowane. Należy również przestrzegać odpowiednich procedur zarządzania danymi.

### <a name="security"></a>Zabezpieczenia

Ten wzorzec obsługuje potencjalnie poufne dane. Upewnij się, że klucze są regularnie obracane, a uprawnienia na koncie usługi Azure Storage i udziałach lokalnych są prawidłowo ustawione.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów owanych w tym artykule:
- W tym wzorcu jest używanych wiele usług powiązanych z IoT, w tym [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/)i [Azure Time Series Insights](/azure/time-series-insights/).
- Aby dowiedzieć się więcej o projekcie Firmy Microsoft Brainwave, zobacz wpis w [blogu](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) i wyewidencjojnij film wideo Azure Accelerated Machine Learning with Project Brainwave (Usługa [Azure Accelerated Machine Learning z projektem Brainwave).](https://www.youtube.com/watch?v=DJfMobMjCX0)
- Zobacz [Uwagi dotyczące projektowania aplikacji hybrydowych,](overview-app-design-considerations.md) aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na wszelkie dodatkowe pytania.
- Zobacz Azure Stack [produktów i rozwiązań,](/azure-stack) aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do przetestowania przykładowego rozwiązania, przejdź do przewodnika wdrażania rozwiązania do wnioskowania usługi [Edge ML.](https://aka.ms/edgeinferencingdeploy) Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.
