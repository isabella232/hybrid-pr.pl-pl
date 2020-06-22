---
title: Wdrażanie rozwiązania wykrywania FootFall opartego na AI na platformie Azure i w centrum Azure Stack
description: Dowiedz się, jak wdrożyć rozwiązanie FootFall Detection (AI) do analizowania ruchu gościa w sklepach detalicznych przy użyciu platformy Azure i usługi Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6913cc522da447092dad0af24e148a3b2576495c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910897"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Wdrażanie rozwiązania wykrywania FootFall opartego na formacie AI przy użyciu platformy Azure i usługi Azure Stack Hub

W tym artykule opisano sposób wdrażania rozwiązań opartych na formacie AI, które generują szczegółowe informacje z rzeczywistych działań na świecie przy użyciu platformy Azure, Azure Stack Hub i zestawu Custom Vision AI.

W tym rozwiązaniu dowiesz się, jak:

> [!div class="checklist"]
> - Wdróż zbiory natywnych aplikacji w chmurze (CNAB) na krawędzi. 
> - Wdróż aplikację, która obejmuje granice chmury.
> - Użyj zestawu SDK Custom Vision AI do wnioskowania na brzegu.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub to rozszerzenie platformy Azure. Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.  
> 
> [Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych. Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.

## <a name="prerequisites"></a>Wymagania wstępne

Przed rozpoczęciem pracy z tym przewodnikiem wdrażania upewnij się, że:

- Zapoznaj się z tematem dotyczącym [wzorca wykrywania FootFall](pattern-retail-footfall-detection.md) .
- Uzyskaj dostęp użytkowników do Azure Stack Development Kit (ASDK) lub zintegrowanego wystąpienia systemowego centrum Azure Stack z:
  - Azure App Service na zainstalowaniu [dostawcy zasobów centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-overview.md) . Potrzebujesz dostępu operatora do wystąpienia centrum Azure Stack lub skontaktuj się z administratorem, aby zainstalować program.
  - Subskrypcja oferty zapewniającej przydział App Service i magazynu. Aby utworzyć ofertę, musisz mieć dostęp do operatora.
- Uzyskaj dostęp do subskrypcji platformy Azure.
  - Jeśli nie masz subskrypcji platformy Azure, przed rozpoczęciem Utwórz [konto bezpłatnej wersji próbnej](https://azure.microsoft.com/free/) .
- Utwórz dwie jednostki usługi w katalogu:
  - Jeden skonfigurowany do użycia z zasobami platformy Azure z dostępem w zakresie subskrypcji platformy Azure.
  - Jeden skonfigurowany do użycia z zasobami centrum Azure Stack, z dostępem w zakresie subskrypcji centrum Azure Stack.
  - Aby dowiedzieć się więcej na temat tworzenia jednostek usługi i autoryzowania dostępu, zobacz [Korzystanie z tożsamości aplikacji w celu uzyskania dostępu do zasobów](/azure-stack/operator/azure-stack-create-service-principals.md). Jeśli wolisz używać interfejsu wiersza polecenia platformy Azure, zobacz [Tworzenie jednostki usługi platformy Azure przy użyciu interfejsu wiersza polecenia platformy Azure](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).
- Wdróż usługę Azure Cognitive Services na platformie Azure lub w centrum Azure Stack.
  - Najpierw [Dowiedz się więcej o Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).
  - Następnie odwiedź stronę [wdrażanie usługi Azure Cognitive Services w usłudze Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) , aby wdrożyć Cognitive Services na Azure Stack centrum. Najpierw musisz zarejestrować się w celu uzyskania dostępu do wersji zapoznawczej.
- Klonowanie lub pobieranie nieskonfigurowanego zestawu Azure Custom Vision AI dev Kit. Aby uzyskać szczegółowe informacje, zobacz [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).
- Utwórz konto Power BI.
- Klucz subskrypcji usługi Azure Cognitive Services interfejs API rozpoznawania twarzy i adres URL punktu końcowego. Możesz skorzystać z usługi [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) bezpłatna wersja próbna. Lub postępuj zgodnie z instrukcjami w temacie [Tworzenie konta Cognitive Services](/azure/cognitive-services/cognitive-services-apis-create-account).
- Zainstaluj następujące zasoby programistyczne:
  - [Interfejs wiersza polecenia platformy Azure 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/). Za pomocą Porter można wdrażać aplikacje w chmurze przy użyciu dodanych manifestów pakietu CNAB.
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Narzędzia usługi Azure IoT dla Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Rozszerzenie języka Python dla Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>Wdrażanie aplikacji w chmurze hybrydowej

Najpierw użyj interfejsu wiersza polecenia Porter, aby wygenerować zestaw poświadczeń, a następnie wdróż aplikację w chmurze.  

1. Klonowanie lub Pobieranie przykładowego kodu rozwiązania z https://github.com/azure-samples/azure-intelligent-edge-patterns . 

1. Porter wygeneruje zestaw poświadczeń, które będą automatyzować wdrażanie aplikacji. Przed uruchomieniem polecenia generowania poświadczeń upewnij się, że dostępne są następujące elementy:

    - Jednostka usługi do uzyskiwania dostępu do zasobów platformy Azure, w tym identyfikatora jednostki usługi, klucza i usługi DNS dzierżawy.
    - Identyfikator subskrypcji subskrypcji platformy Azure.
    - Jednostka usługi do uzyskiwania dostępu do zasobów centrum Azure Stack, w tym identyfikatora podmiotu zabezpieczeń, klucza i usługi DNS dzierżawy.
    - Identyfikator subskrypcji dla subskrypcji centrum Azure Stack.
    - Adres URL klucza i punktu końcowego usługi Azure Cognitive Services interfejs API rozpoznawania twarzy.

1. Uruchom proces generowania poświadczeń Porter i postępuj zgodnie z monitami:

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Porter wymaga również zestawu parametrów do uruchomienia. Utwórz plik tekstowy parametrów i wprowadź następujące pary nazwa/wartość. Skontaktuj się z administratorem centrum Azure Stack, jeśli potrzebujesz pomocy przy dowolnych wymaganych wartościach.

   > [!NOTE] 
   > Ta `resource suffix` wartość służy do upewnienia się, że zasoby wdrożenia mają unikatowe nazwy na platformie Azure. Musi to być unikatowy ciąg liter i cyfr, nie dłuższa niż 8 znaków.

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   Zapisz plik tekstowy i zanotuj jego ścieżkę.

1. Teraz możesz przystąpić do wdrażania aplikacji w chmurze hybrydowej za pomocą Porter. Uruchom polecenie instalacji i obejrzyj, jak zasoby są wdrażane na platformie Azure i w centrum Azure Stack:

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. Po zakończeniu wdrażania należy pamiętać o następujących wartościach:
    - Parametry połączenia aparatu.
    - Parametry połączenia konta magazynu obrazu.
    - Nazwy grup zasobów.

## <a name="prepare-the-custom-vision-ai-devkit"></a>Przygotuj Custom Vision AI DevKit

Następnie skonfiguruj zestaw Custom Vision AI dev Kit, jak pokazano w [przewodniku szybki start dla programu Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/). Należy również skonfigurować i przetestować aparat przy użyciu parametrów połączenia podanych w poprzednim kroku.

## <a name="deploy-the-camera-app"></a>Wdrażanie aplikacji aparatu

Użyj interfejsu wiersza polecenia Porter, aby wygenerować zestaw poświadczeń, a następnie wdróż aplikację aparatu.

1. Porter wygeneruje zestaw poświadczeń, które będą automatyzować wdrażanie aplikacji. Przed uruchomieniem polecenia generowania poświadczeń upewnij się, że dostępne są następujące elementy:

    - Jednostka usługi do uzyskiwania dostępu do zasobów platformy Azure, w tym identyfikatora jednostki usługi, klucza i usługi DNS dzierżawy.
    - Identyfikator subskrypcji subskrypcji platformy Azure.
    - Parametry połączenia konta magazynu obrazu podane podczas wdrażania aplikacji w chmurze.

1. Uruchom proces generowania poświadczeń Porter i postępuj zgodnie z monitami:

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Porter wymaga również zestawu parametrów do uruchomienia. Utwórz plik tekstowy parametrów i wprowadź następujący tekst. Skontaktuj się z administratorem centrum Azure Stack, jeśli nie znasz pewnych wymaganych wartości.

    > [!NOTE]
    > Ta `deployment suffix` wartość służy do upewnienia się, że zasoby wdrożenia mają unikatowe nazwy na platformie Azure. Musi to być unikatowy ciąg liter i cyfr, nie dłuższa niż 8 znaków.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    Zapisz plik tekstowy i zanotuj jego ścieżkę.

4. Teraz możesz przystąpić do wdrażania aplikacji aparatu fotograficznego za pomocą Porter. Uruchom polecenie install i obejrzyj, jak zostanie utworzone wdrożenie IoT Edge.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. Sprawdź, czy wdrożenie aparatu fotograficznego zostało ukończone przez wyświetlenie kanału informacyjnego aparatu w lokalizacji `https://<camera-ip>:3000/` , gdzie `<camara-ip>` jest adresem IP aparatu. Ten krok może potrwać do 10 minut.

## <a name="configure-azure-stream-analytics"></a>Konfigurowanie Azure Stream Analytics

Teraz, gdy dane przepływają do Azure Stream Analytics z aparatu, musimy ręcznie autoryzować ją do komunikowania się z Power BI.

1. W Azure Portal Otwórz **wszystkie zasoby**i zadanie *Process-FootFall \[ \] yoursuffix* .

2. W sekcji **Topologia zadania** okienka zadania usługi Stream Analytics wybierz opcję **Dane wyjściowe**.

3. Wybierz **docelowy przepływ** danych wyjściowych.

4. Wybierz pozycję **Odnów autoryzację** i zaloguj się na swoim koncie Power BI.
  
    ![Monit odnawiania autoryzacji w Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. Zapisz ustawienia danych wyjściowych.

6. Przejdź do okienka **Przegląd** i wybierz pozycję **Rozpocznij** , aby rozpocząć wysyłanie danych do Power BI.

7. Wybierz wartość **Teraz** jako godzinę rozpoczęcia generowania danych wyjściowych zadania, a następnie wybierz pozycję **Uruchom**. Stan zadania możesz wyświetlić na pasku powiadomień.

## <a name="create-a-power-bi-dashboard"></a>Tworzenie pulpitu nawigacyjnego Power BI

1. Gdy zadanie zakończy się pomyślnie, przejdź do [Power BI](https://powerbi.com/) i zaloguj się przy użyciu konta służbowego. Jeśli wyniki zapytania zadania Stream Analytics są wyprowadzane, utworzony zestaw danych *FootFall-DataSet* istnieje na karcie **zestawy** danych.

2. W obszarze roboczym Power BI wybierz pozycję **+ Utwórz** , aby utworzyć nowy pulpit nawigacyjny o nazwie *FootFall Analysis.*

3. W górnej części okna wybierz pozycję **Dodaj kafelek**. Następnie wybierz pozycje **Niestandardowe dane przesyłane strumieniowo** i **Dalej**. Wybierz **zestaw danych FootFall-DataSet** w **zestawach**danych. Wybierz **kartę** z listy rozwijanej **typ wizualizacji** , a następnie Dodaj opcję **wiek** do **pól**. Wybierz pozycję **Dalej**, aby wprowadzić nazwę kafelka, a następnie wybierz pozycję **Zastosuj**, aby utworzyć kafelek.

4. W razie potrzeby możesz dodać dodatkowe pola i karty.

## <a name="test-your-solution"></a>Testowanie rozwiązania

Obserwuj, jak dane w kartach utworzonych w Power BI zmieniają się w zależności od aparatu. Po zarejestrowaniu wnioskowania może upłynąć do 20 sekund.

## <a name="remove-your-solution"></a>Usuwanie rozwiązania

Jeśli chcesz usunąć swoje rozwiązanie, uruchom następujące polecenia przy użyciu Porter, używając tych samych plików parametrów, które zostały utworzone dla wdrożenia:

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>Następne kroki

- Dowiedz się więcej o [zagadnienia dotyczące projektowania aplikacji hybrydowych]. (overview-app-design-considerations.md)
- Przejrzyj i Zaproponuj ulepszenia [kodu dla tego przykładu w witrynie GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).
