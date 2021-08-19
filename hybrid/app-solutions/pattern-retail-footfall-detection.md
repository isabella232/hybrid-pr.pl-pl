---
title: Wzorzec wykrywania stopek przy użyciu platformy Azure i Azure Stack Hub
description: Dowiedz się, jak używać platformy Azure Azure Stack Hub do implementowania opartego na AI rozwiązania do wykrywania stopek do analizowania ruchu w sklepach detalicznych.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 79fb39d418bed53ef6a78980fcd9188bdf6e57ae
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281282"
---
# <a name="footfall-detection-pattern"></a>Wzorzec wykrywania stopek

Ten wzorzec zawiera omówienie implementacji opartego na AI rozwiązania do wykrywania stopek do analizowania ruchu odwiedzającego w sklepach detalicznych. Rozwiązanie generuje szczegółowe informacje na podstawie akcji w świecie rzeczywistym przy użyciu platformy Azure, Azure Stack Hub i zestawu Custom Vision AI Dev Kit.

## <a name="context-and-problem"></a>Kontekst i problem

Sklepy Contoso chcą uzyskać szczegółowe informacje na temat sposobu, w jaki klienci otrzymują bieżące produkty w odniesieniu do układu sklepu. Nie mogą umieszczać pracowników w każdej sekcji i nieefektywnie jest, aby zespół analityków przeglądał nagrania z kamer w całym sklepie. Ponadto żaden z magazynów nie ma wystarczającej przepustowości, aby przesyłać strumieniowo wideo ze wszystkich kamer do chmury w celu analizy.

Firma Contoso chce znaleźć dyskretny, przyjazny dla prywatności sposób określania danych demograficznych, lojalności i reakcji klientów na przechowywanie produktów i wyświetlaczy.

## <a name="solution"></a>Rozwiązanie

Ten wzorzec analizy handlu detalicznego korzysta z podejścia warstwowego do wnioskowania na brzegu sieci. Za pomocą zestawu Custom Vision AI Dev Kit tylko obrazy z twarzami ludzkimi są wysyłane do analizy do prywatnego Azure Stack Hub z Azure Cognitive Services. Anonimowe, zagregowane dane są wysyłane na platformę Azure w celu agregacji we wszystkich magazynach i wizualizacji w Power BI. Połączenie chmury brzegowej i chmury publicznej umożliwia firmie Contoso skorzystanie z nowoczesnej technologii AI przy jednoczesnym zachowaniu zgodności z zasadami firmy i ochronie prywatności klientów.

[![Rozwiązanie wzorca wykrywania stopek](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Poniżej podano podsumowanie sposobu działania rozwiązania:

1. Zestaw Custom Vision AI Dev Kit pobiera konfigurację z usługi IoT Hub, która instaluje środowisko IoT Edge Runtime i ML model.
2. Jeśli model widzi osobę, robi zdjęcie i przesyła go do magazynu obiektów blob Azure Stack Hub blob.
3. Usługa blob wyzwala funkcję platformy Azure na Azure Stack Hub.
4. Funkcja platformy Azure wywołuje kontener z interfejsem API rozpoznawania twarzy w celu uzyskania danych demograficznych i dotyczących emocji z obrazu.
5. Dane są anonimizowane i wysyłane do Azure Event Hubs klastra.
6. Klaster Event Hubs wypycha dane do Stream Analytics.
7. Stream Analytics agreguje dane i wypycha je do Power BI.

## <a name="components"></a>Składniki

To rozwiązanie korzysta z następujących składników:

| Warstwa | Składnik | Opis |
|----------|-----------|-------------|
| Sprzęt w sklepie | [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Umożliwia filtrowanie w sklepie przy użyciu lokalnego modelu ML, który przechwytuje tylko obrazy osób do analizy. Bezpiecznie aprowizowana i aktualizowana za pośrednictwem IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs zapewnia skalowalną platformę do pozyskania anonimowych danych, która jest starannie zintegrowana z Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Zadanie Azure Stream Analytics agreguje anonimowe dane i grupuje je w 15-sekundowe okna wizualizacji. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI udostępnia łatwy w użyciu interfejs pulpitu nawigacyjnego do wyświetlania danych wyjściowych Azure Stream Analytics. |
| Azure Stack Hub | [App Service](/azure-stack/operator/azure-stack-app-service-overview) | Dostawca App Service zasobów zapewnia podstawę dla składników brzegowych, w tym funkcje hostingu i zarządzania dla aplikacji internetowych/interfejsów API i usługi Functions. |
| | klaster aparatu [Azure Kubernetes Service (AKS)](https://github.com/Azure/aks-engine) | Usługa AKS RP AKS-Engine klastrem wdrożonym w Azure Stack Hub zapewnia skalowalny, odporny aparat do uruchamiania kontenera interfejsu API rozpoznawania twarzy. |
| | Azure Cognitive Services api [rozpoznawania twarzy](/azure/cognitive-services/face/face-how-to-install-containers)| Aplikacja Azure Cognitive Services z kontenerami interfejsu API rozpoznawania twarzy zapewnia wykrywanie danych demograficznych, emocji i unikatowych gości w sieci prywatnej firmy Contoso. |
| | Blob Storage | Obrazy przechwycone z zestawu AI Dev Kit są przekazywane Azure Stack Hub magazynu obiektów blob firmy. |
| | Azure Functions | Funkcja platformy Azure uruchomiona na platformie Azure Stack Hub odbiera dane wejściowe z magazynu obiektów blob i zarządza interakcjami z interfejsem API rozpoznawania twarzy. Emituje anonimowe dane do klastra Event Hubs na platformie Azure.<br><br>|

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podejmując decyzję o wdrożeniu tego rozwiązania, należy wziąć pod uwagę następujące kwestie:

### <a name="scalability"></a>Skalowalność

Aby umożliwić skalowanie tego rozwiązania na wiele kamer i lokalizacji, należy upewnić się, że wszystkie składniki mogą obsłużyć zwiększone obciążenie. Może być konieczne działanie, takie jak:

- Zwiększ liczbę jednostek przesyłania Stream Analytics przesyłania strumieniowego.
- Skalowanie w zewnątrz wdrożenia interfejsu API rozpoznawania twarzy.
- Zwiększ przepływność Event Hubs klastra.
- W skrajnych przypadkach może być Azure Functions migracji z maszyny wirtualnej do maszyny wirtualnej.

### <a name="availability"></a>Dostępność

Ponieważ to rozwiązanie jest warstwowe, ważne jest, aby zastanowić się, jak radzić sobie z awariami sieci lub zasilania. W zależności od potrzeb biznesowych możesz zaimplementować mechanizm lokalnego buforowania obrazów, a następnie przesyłać dalej do Azure Stack Hub, gdy łączność wróci. Jeśli lokalizacja jest wystarczająco duża, lepszym rozwiązaniem może być urządzenie Data Box Edge z kontenerem interfejsu API rozpoznawania twarzy w tej lokalizacji.

### <a name="manageability"></a>Możliwości zarządzania

To rozwiązanie może obejmować wiele urządzeń i lokalizacji, co może być niewygodne. [Usługi IoT](/azure/iot-fundamentals/) platformy Azure mogą służyć do automatycznego doprowadzenia nowych lokalizacji i urządzeń do trybu online oraz ich aktualnych wersji.

### <a name="security"></a>Zabezpieczenia

To rozwiązanie przechwytuje obrazy klientów, przez co bezpieczeństwo jest najważniejszą kwestią. Upewnij się, że wszystkie konta magazynu są zabezpieczone przy użyciu odpowiednich zasad dostępu, i regularnie obracaj klucze. Upewnij się, że konta magazynu Event Hubs mają zasady przechowywania spełniające firmowe i rządowe przepisy dotyczące prywatności. Upewnij się również, że poziomy dostępu użytkowników są warstwowe. Warstwy zapewniają, że użytkownicy mają dostęp tylko do danych, których potrzebują do swojej roli.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów owanych w tym artykule:

- Zobacz [wzorzec danych warstwowych,](https://aka.ms/tiereddatadeploy)który jest wykorzystywany przez wzorzec wykrywania stopek.
- Zobacz zestaw [Custom Vision AI Dev Kit,](https://azure.github.io/Vision-AI-DevKit-Pages/) aby dowiedzieć się więcej na temat korzystania z usługi Custom Vision. 

Gdy wszystko będzie gotowe do przetestowania przykładowego rozwiązania, przejdź do przewodnika wdrażania wykrywania [stopek](/azure/architecture/hybrid/deployments/solution-deployment-guide-retail-footfall-detection). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.