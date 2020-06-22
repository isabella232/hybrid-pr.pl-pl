---
title: FootFall — wzorzec wykrywania przy użyciu platformy Azure i usługi Azure Stack Hub
description: Dowiedz się, jak korzystać z platformy Azure i usługi Azure Stack Hub, aby wdrożyć rozwiązanie wykrywania FootFall oparte na systemie AI do analizowania ruchu w sklepie detalicznym.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911224"
---
# <a name="footfall-detection-pattern"></a>FootFall — wzorzec wykrywania

Ten wzorzec zawiera przegląd dotyczący implementacji rozwiązania wykrywania FootFall opartego na formacie AI do analizowania ruchu odwiedzających w sklepach detalicznych. Rozwiązanie generuje wgląd w dane z rzeczywistych działań świata przy użyciu platformy Azure, Azure Stack Hub oraz zestawu Custom Vision AI.

## <a name="context-and-problem"></a>Kontekst i problem

Magazyny firmy Contoso mają wgląd w informacje dotyczące sposobu, w jaki klienci otrzymują swoje bieżące produkty w odniesieniu do układu sklepu. Nie są w stanie umieścić pracowników w każdej sekcji i nie jest wydajny, aby zespół analityków przeglądał materiał z kamerą całego sklepu. Ponadto żaden z ich magazynów nie ma wystarczającej przepustowości do przesyłania strumieniowego wideo ze wszystkich kamer do chmury w celu przeprowadzenia analizy.

Firma Contoso chce znaleźć niezauważalny, przyjazny dla prywatności sposób określania demograficznych klientów, lojalności i reakcji na przechowywanie wyświetlanych i produktów.

## <a name="solution"></a>Rozwiązanie

Ten wzorzec analizy handlu detalicznego używa podejścia warstwowego do inferencing na brzegu. Korzystając z zestawu Custom Vision AI dev Kit, do analizy są wysyłane tylko obrazy ze ludzkich twarzy do usługi Private Azure Stack Hub, na których działa usługa Azure Cognitive Services. Anonimowe dane zagregowane są wysyłane do platformy Azure w celu agregacji we wszystkich sklepach i wizualizacji w Power BI. Łączenie z usługą Edge i chmurą publiczną pozwala firmie Contoso korzystać z nowoczesnej technologii AI, a także zachować zgodność z zasadami firmowymi i przestrzegać poufności swoich klientów.

[![Rozwiązanie do tworzenia wzorców wykrywania FootFall](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Oto podsumowanie sposobu działania rozwiązania:

1. Custom Vision AI dev Kit Pobiera konfigurację z IoT Hub, która instaluje IoT Edge środowiska uruchomieniowego i modelu ML.
2. Jeśli model widzi osobę, pobiera obraz i przekazuje go do Azure Stack centrum obiektów BLOB Storage.
3. Usługa BLOB Service wyzwala funkcję platformy Azure w centrum Azure Stack.
4. Funkcja platformy Azure wywołuje kontener z interfejs API rozpoznawania twarzy, aby uzyskać dane demograficzne i rozpoznawania emocji z obrazu.
5. Dane są anonimowe i wysyłane do klastra Event Hubs platformy Azure.
6. Klaster Event Hubs wypycha dane, aby Stream Analytics.
7. Stream Analytics agreguje dane i wypycha je do Power BI.

## <a name="components"></a>Składniki

To rozwiązanie używa następujących składników:

| Warstwa | Składnik | Opis |
|----------|-----------|-------------|
| Sprzęt w sklepie | [Custom Vision AI](https://azure.github.io/Vision-AI-DevKit-Pages/) | Zapewnia filtrowanie w sklepie przy użyciu lokalnego modelu ML, który przechwytuje tylko obrazy osób do analizy. Bezpiecznie obsługiwane i aktualizowane za IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Usługa Azure Event Hubs zapewnia skalowalną platformę do pozyskiwania danych anonimowe, które są zintegrowane z Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Zadanie Azure Stream Analytics agreguje dane anonimowe i grupuje je do 15-sekundowego systemu Windows w celu wizualizacji. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI zapewnia łatwy w użyciu interfejs nawigacyjny do wyświetlania danych wyjściowych z Azure Stream Analytics. |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview.md) | Dostawca zasobów App Service (RP) stanowi podstawę dla składników brzegowych, w tym funkcje hostingu i zarządzania dla aplikacji sieci Web/interfejsów API i funkcji. |
| | Klaster aparatu usługi Azure Kubernetes Service [(AKS)](https://github.com/Azure/aks-engine) | AKS RP z klastrem AKS-Engine wdrożonym w usłudze Azure Stack Hub udostępnia skalowalny, odporny na uruchomienie aparat interfejs API rozpoznawania twarzy. |
| | [Kontenery interfejs API rozpoznawania twarzy](/azure/cognitive-services/face/face-how-to-install-containers) Cognitive Services platformy Azure| Usługa Azure Cognitive Services RP z kontenerami interfejs API rozpoznawania twarzy oferuje funkcje demograficzne, rozpoznawania emocji i unikatowe wykrywanie użytkowników w sieci prywatnej firmy Contoso. |
| | Blob Storage | Obrazy przechwycone z zestawu AI, są przekazywane do magazynu obiektów Blob Azure Stack Hub. |
| | Azure Functions | Funkcja platformy Azure działająca w systemie Azure Stack Hub otrzymuje dane wejściowe z usługi BLOB Storage i zarządza interakcjami z interfejs API rozpoznawania twarzy. Emituje dane anonimowe do klastra Event Hubs znajdującego się na platformie Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas decydowania o sposobie wdrożenia tego rozwiązania należy wziąć pod uwagę następujące kwestie:

### <a name="scalability"></a>Skalowalność

Aby umożliwić skalowanie tego rozwiązania między wieloma aparatami i lokalizacjami, należy upewnić się, że wszystkie składniki mogą obsłużyć zwiększone obciążenie. Może być konieczne wykonanie działań takich jak:

- Zwiększ liczbę Stream Analytics jednostek przesyłania strumieniowego.
- Skaluj w poziomie wdrożenie interfejs API rozpoznawania twarzy.
- Zwiększ przepływność klastra Event Hubs.
- W skrajnych przypadkach może być konieczne przeprowadzenie migracji z Azure Functions na maszynę wirtualną.

### <a name="availability"></a>Dostępność

Ponieważ to rozwiązanie jest warstwowe, należy wziąć pod uwagę sposób postępowania z awariami sieci lub zasilaczem. W zależności od potrzeb firmy można zaimplementować mechanizm buforowania obrazów lokalnie, a następnie przesłać do Azure Stack Hub, gdy połączenie zwróci wartość. Jeśli lokalizacja jest wystarczająco duża, wdrożenie Data Box Edge z kontenerem interfejs API rozpoznawania twarzy do tej lokalizacji może być lepszym rozwiązaniem.

### <a name="manageability"></a>Możliwości zarządzania

To rozwiązanie może obejmować wiele urządzeń i lokalizacji, które mogą uzyskać nieporęczny. [Usługi IoT platformy Azure](/azure/iot-fundamentals/) mogą służyć do automatycznego przenoszenia nowych lokalizacji i urządzeń w tryb online oraz do ich Aktualności.

### <a name="security"></a>Zabezpieczenia

To rozwiązanie służy do przechwytywania obrazów klientów, co sprawia, że zabezpieczenia są najważniejsze. Upewnij się, że wszystkie konta magazynu są zabezpieczone przy użyciu odpowiednich zasad dostępu i regularnie Obróć klucze. Upewnij się, że konta magazynu i Event Hubs mają zasady przechowywania spełniające przepisy w zakresie ochrony prywatności dla firm i instytucji rządowych. Należy również pamiętać o warstwach poziomów dostępu użytkowników. Obsługa warstw gwarantuje, że użytkownicy będą mieli dostęp tylko do danych, które są im potrzebne do ich roli.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:

- Zobacz [wzorzec danych warstwowych](https://aka.ms/tiereddatadeploy), który jest używany przez wzorzec wykrywania FootFall.
- Aby dowiedzieć się więcej o używaniu niestandardowej wizji, zobacz temat [Custom Vision AI dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) . 

Gdy wszystko będzie gotowe do testowania przykładowego rozwiązania, przejdź do [przewodnika po wdrożeniu wykrywania FootFall](solution-deployment-guide-retail-footfall-detection.md). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.
