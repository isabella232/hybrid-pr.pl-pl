---
title: Wdrażanie aplikacji, która skaluje wiele chmur na platformie Azure i w Azure Stack Hub
description: Dowiedz się, jak wdrożyć aplikację, która skaluje wiele chmur na platformie Azure i Azure Stack centrum.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5ae6c4323324fa104cd0e5c7b5198492be14b8eb
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886819"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Wdrażanie aplikacji, która skaluje wiele chmur przy użyciu platformy Azure i usługi Azure Stack Hub

Dowiedz się, jak utworzyć rozwiązanie międzychmurowe, aby zapewnić ręcznie wyzwolony proces przełączania z hostowanej aplikacji sieci Web Azure Stack Hub do hostowanej aplikacji sieci Web platformy Azure z funkcją automatycznego skalowania za pomocą usługi Traffic Manager. Ten proces zapewnia elastyczne i skalowalne narzędzie w chmurze w ramach obciążenia.

Ten wzorzec może nie być gotowy do uruchamiania aplikacji w chmurze publicznej. Niemniej jednak firma może nie być ekonomicznie wykonalna w celu utrzymania pojemności wymaganej w środowisku lokalnym w celu obsłużenia podaży na żądanie dla aplikacji. Twoja dzierżawa może korzystać z elastyczności chmury publicznej za pomocą rozwiązania lokalnego.

W tym rozwiązaniu utworzysz przykładowe środowisko w celu:

> [!div class="checklist"]
> - Utwórz wielowęzłową aplikację sieci Web.
> - Skonfiguruj proces ciągłego wdrażania (CD) i Zarządzaj nim.
> - Opublikuj aplikację sieci Web w centrum Azure Stack.
> - Utwórz wydanie.
> - Dowiedz się, jak monitorować i śledzić wdrożenia.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub to rozszerzenie platformy Azure. Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.  
> 
> [Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych. Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.

## <a name="prerequisites"></a>Wymagania wstępne

- Subskrypcja platformy Azure. W razie konieczności przed rozpoczęciem Utwórz [bezpłatne konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) .
- Zintegrowany system Azure Stack Hub lub wdrożenie Azure Stack Development Kit (ASDK).
  - Aby uzyskać instrukcje dotyczące instalowania centrum Azure Stack, zobacz [Install the ASDK](/azure-stack/asdk/asdk-install.md).
  - W przypadku skryptu automatyzacji po wdrożeniu ASDK przejdź do: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - Instalacja może wymagać kilku godzin.
- Wdróż [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) usługi PaaS Services w usłudze Azure Stack Hub.
- [Utwórz plany/oferty](/azure-stack/operator/service-plan-offer-subscription-overview.md) w ramach środowiska Azure Stack Hub.
- [Utwórz subskrypcję dzierżawy](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) w ramach środowiska Azure Stack Hub.
- Utwórz aplikację internetową w ramach subskrypcji dzierżawy. Zanotuj nowy adres URL aplikacji sieci Web do późniejszego użycia.
- Wdróż Azure Pipelines maszynę wirtualną w ramach subskrypcji dzierżawy.
- Wymagana jest maszyna wirtualna z systemem Windows Server 2016 z platformą .NET 3,5. Ta maszyna wirtualna zostanie utworzona w ramach subskrypcji dzierżawy na Azure Stack Hub jako prywatny agent kompilacji.
- [System Windows Server 2016 z obrazem maszyny wirtualnej SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) jest dostępny w witrynie Centrum Azure Stack Hub. Jeśli ten obraz nie jest dostępny, należy skontaktować się z operatorem centrum Azure Stack, aby upewnić się, że został on dodany do środowiska.

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

### <a name="scalability"></a>Skalowalność

Kluczowym elementem skalowania między chmurami jest możliwość dostarczania natychmiastowych i na żądanie skalowania między publiczną i lokalną infrastrukturą chmurową, zapewniając spójną i niezawodną usługę.

### <a name="availability"></a>Dostępność

Upewnij się, że aplikacje wdrożone lokalnie są skonfigurowane pod kątem wysokiej dostępności za poorednictwem konfiguracji sprzętu lokalnego i wdrożenia oprogramowania.

### <a name="manageability"></a>Możliwości zarządzania

Rozwiązanie międzychmurowe zapewnia bezproblemowe zarządzanie i przyjazny interfejs między środowiskami. Program PowerShell jest zalecany w przypadku zarządzania na wielu platformach.

## <a name="cross-cloud-scaling"></a>Skalowanie między chmurami

### <a name="get-a-custom-domain-and-configure-dns"></a>Pobierz domenę niestandardową i skonfiguruj system DNS

Zaktualizuj plik strefy DNS dla domeny. Usługa Azure AD sprawdzi własność niestandardowej nazwy domeny. Użyj [Azure DNS](/azure/dns/dns-getstarted-portal) dla systemu azure/Microsoft 365/zewnętrznych rekordów DNS na platformie Azure lub Dodaj wpis DNS w [innym rejestratorze DNS](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).

1. Zarejestruj domenę niestandardową z rejestratorem publicznym.
2. Zaloguj się do rejestratora nazw domen dla domeny. Aby można było zaktualizować DNS, może być wymagane zatwierdzenie administratora.
3. Zaktualizuj plik strefy DNS dla domeny przez dodanie wpisu DNS dostarczonego przez usługę Azure AD. (Wpis DNS nie dotyczy zachowań routingu e-mail ani usług hostingu sieci Web).

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Tworzenie domyślnej wielowęzłowej aplikacji sieci Web w centrum Azure Stack

Skonfiguruj hybrydową ciągłą integrację i ciągłe wdrażanie (CI/CD), aby wdrażać aplikacje sieci Web na platformie Azure i w Azure Stack Hub oraz w celu samowypychania zmian w obu chmurach.

> [!Note]  
> Wymagane są Azure Stack centrum z odpowiednimi obrazami do uruchomienia (system Windows Server i SQL) oraz wdrożenie App Service. Aby uzyskać więcej informacji, zapoznaj się z dokumentacją App Service [wymagania wstępne dotyczące wdrażania App Service w centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

### <a name="add-code-to-azure-repos"></a>Dodaj kod do Azure Repos

Azure Repos

1. Zaloguj się do Azure Repos przy użyciu konta, które ma prawa do tworzenia projektu na Azure Repos.

    Hybrydowej ciągłej integracji/ciągłego dostarczania można użyć zarówno w kodzie aplikacji, jak i w kodzie infrastruktury. Używaj [szablonów Azure Resource Manager](https://azure.microsoft.com/resources/templates/) zarówno do programowania w chmurze prywatnej, jak i hostowanej.

    ![Nawiązywanie połączenia z projektem w Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. **Sklonuj repozytorium** , tworząc i otwierając domyślną aplikację sieci Web.

    ![Klonowanie repozytorium w usłudze Azure Web App](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Utwórz samodzielne wdrożenie aplikacji sieci Web dla App Services w obu chmurach

1. Edytuj plik **WebApplication. csproj** . Wybierz `Runtimeidentifier` i Dodaj `win10-x64` . (Zobacz dokumentację [wdrożenia prewartą](/dotnet/core/deploying/deploy-with-vs#simpleSelf) ).

    ![Edytuj plik projektu aplikacji sieci Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. Zaewidencjonuj kod, aby Azure Repos przy użyciu Team Explorer.

3. Upewnij się, że kod aplikacji został zaewidencjonowany do Azure Repos.

## <a name="create-the-build-definition"></a>Tworzenie definicji kompilacji

1. Zaloguj się do Azure Pipelines, aby potwierdzić możliwość tworzenia definicji kompilacji.

2. Kod Add **-r Win10-x64** . Jest to konieczne do wyzwolenia wdrożenia samodzielnego z platformą .NET Core.

    ![Dodawanie kodu do aplikacji sieci Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. Uruchom kompilację. Proces [kompilacji samodzielnego wdrażania](/dotnet/core/deploying/deploy-with-vs#simpleSelf) spowoduje opublikowanie artefaktów działających na platformie Azure i w centrum Azure Stack.

## <a name="use-an-azure-hosted-agent"></a>Korzystanie z agenta hostowanego na platformie Azure

Korzystanie z hostowanego agenta kompilacji w Azure Pipelines jest wygodną opcją kompilowania i wdrażania aplikacji sieci Web. Konserwacja i uaktualnienia są wykonywane automatycznie przez Microsoft Azure, co umożliwia ciągły i nieprzerwany cykl programowania.

### <a name="manage-and-configure-the-cd-process"></a>Zarządzanie i Konfigurowanie procesu CD

Azure Pipelines i Azure DevOps Services zapewniają wysoce konfigurowalny i zarządzany potok dla wydań w wielu środowiskach, takich jak programowanie, przemieszczanie, pytania i odpowiedzi oraz środowiska produkcyjne; obejmuje to wymaganie zatwierdzeń na określonych etapach.

## <a name="create-release-definition"></a>Utwórz definicję wydania

1. Wybierz przycisk **Plus** , aby dodać nową wersję na karcie **wersje** w sekcji **kompilacja i wersja** Azure DevOps Services.

    ![Tworzenie definicji wydania](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Zastosuj szablon wdrażania Azure App Service.

   ![Zastosuj szablon wdrożenia Azure App Service](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. W obszarze **Dodawanie artefaktu**Dodaj artefakt dla aplikacji Azure Cloud Build.

   ![Dodawanie artefaktu do kompilacji w chmurze platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. Na karcie potok wybierz pozycję **faza, link zadania** środowiska i ustaw wartości środowiska chmury platformy Azure.

   ![Ustawianie wartości środowiska chmury platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. Ustaw **nazwę środowiska** i wybierz **subskrypcję platformy Azure** dla punktu końcowego w chmurze platformy Azure.

      ![Wybierz subskrypcję platformy Azure dla punktu końcowego w chmurze platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. W obszarze **nazwa usługi App Service**Ustaw wymaganą nazwę usługi Azure App Service.

      ![Ustawianie nazwy usługi Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Wprowadź wartość "hostowana program VS2017" w obszarze **Kolejka agenta** dla środowiska hostowanego w chmurze platformy Azure.

      ![Ustawianie kolejki agentów dla środowiska hostowanego w chmurze platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. W menu Wdróż Azure App Service wybierz prawidłowy **pakiet lub folder** dla środowiska. Wybierz pozycję **OK** , aby określić **lokalizację folderu**.
  
      ![Wybierz pakiet lub folder dla środowiska Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Wybierz pakiet lub folder dla środowiska Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. Zapisz wszystkie zmiany i wróć do **potoku wydania**.

    ![Zapisz zmiany w potoku wydania](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Dodaj nowy artefakt, wybierając kompilację dla aplikacji Centrum Azure Stack.

    ![Dodawanie nowego artefaktu dla aplikacji Azure Stack Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Dodaj jeszcze jedno środowisko, stosując wdrożenie Azure App Service.

    ![Dodawanie środowiska do wdrożenia Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. Nazwij nowe środowisko "Azure Stack".

    ![Nazwa środowiska w Azure App Service wdrożenia](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. Znajdź środowisko Azure Stack na karcie **zadanie** .

    ![Środowisko Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Wybierz subskrypcję dla punktu końcowego Azure Stack.

    ![Wybierz subskrypcję dla punktu końcowego Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Jako nazwę usługi App Service Ustaw nazwę aplikacji sieci Web Azure Stack.
    ![Ustaw Azure Stack nazwę aplikacji sieci Web](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Wybierz agenta Azure Stack.

    ![Wybierz agenta Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. W sekcji Wdróż Azure App Service wybierz prawidłowy **pakiet lub folder** dla środowiska. Wybierz pozycję **OK** , aby określić lokalizację folderu.

    ![Wybierz folder do wdrożenia Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Wybierz folder do wdrożenia Azure App Service](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. W obszarze Karta zmienna Dodaj zmienną o nazwie `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , ustaw jej wartość na **true**i zakres na Azure Stack.

    ![Dodawanie zmiennej do wdrożenia aplikacji platformy Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. Wybierz ikonę wyzwalacza **ciągłego** wdrażania w obu artefaktach i Włącz wyzwalacz wdrażania **Kontynuuj** .

    ![Wybierz wyzwalacz ciągłego wdrażania](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Wybierz ikonę warunki **przed wdrożeniem** w środowisku Azure Stack i ustaw wyzwalacz na **po wydaniu.**

    ![Wybieranie warunków przed wdrożeniem](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. Zapisz wszystkie zmiany.

> [!Note]  
> Niektóre ustawienia zadań mogą zostać automatycznie zdefiniowane jako [zmienne środowiskowe](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) podczas tworzenia definicji wydania na podstawie szablonu. Tych ustawień nie można modyfikować w ustawieniach zadania; Zamiast tego należy wybrać element środowiska nadrzędnego, aby edytować te ustawienia.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Publikowanie w centrum Azure Stack przy użyciu programu Visual Studio

Tworząc punkty końcowe, kompilacja Azure DevOps Services może wdrożyć aplikacje usługi platformy Azure w centrum Azure Stack. Azure Pipelines nawiązuje połączenie z agentem kompilacji, który łączy się z centrum Azure Stack.

1. Zaloguj się do Azure DevOps Services i przejdź do strony Ustawienia aplikacji.

2. W obszarze **Ustawienia**wybierz pozycję **zabezpieczenia**.

3. W obszarze **grupy VSTS**wybierz pozycję **twórcy punktów końcowych**.

4. Na karcie **Członkowie** wybierz pozycję **Dodaj**.

5. W obszarze **Dodawanie użytkowników i grup**wprowadź nazwę użytkownika i wybierz tego użytkownika z listy użytkowników.

6. Wybierz pozycję **Save changes** (Zapisz zmiany).

7. Na liście **grupy VSTS** wybierz pozycję **administratorzy punktów końcowych**.

8. Na karcie **Członkowie** wybierz pozycję **Dodaj**.

9. W obszarze **Dodawanie użytkowników i grup**wprowadź nazwę użytkownika i wybierz tego użytkownika z listy użytkowników.

10. Wybierz pozycję **Save changes** (Zapisz zmiany).

Teraz, gdy istnieją informacje o punkcie końcowym, Azure Pipelines do połączenia z centrum Azure Stack jest gotowy do użycia. Agent kompilacji w Azure Stack Hub otrzymuje instrukcje od Azure Pipelines, a następnie Agent przekazuje informacje o punkcie końcowym komunikacji z centrum Azure Stack.

## <a name="develop-the-app-build"></a>Opracowywanie kompilacji aplikacji

> [!Note]  
> Wymagane są Azure Stack centrum z odpowiednimi obrazami do uruchomienia (system Windows Server i SQL) oraz wdrożenie App Service. Aby uzyskać więcej informacji, zobacz [wymagania wstępne dotyczące wdrażania App Service w centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

Użyj [szablonów Azure Resource Manager](https://azure.microsoft.com/resources/templates/) , takich jak kod aplikacji sieci web z Azure Repos do wdrożenia w obu chmurach.

### <a name="add-code-to-an-azure-repos-project"></a>Dodawanie kodu do projektu Azure Repos

1. Zaloguj się do Azure Repos przy użyciu konta, które ma prawa do tworzenia projektu w centrum Azure Stack.

2. **Sklonuj repozytorium** , tworząc i otwierając domyślną aplikację sieci Web.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Utwórz samodzielne wdrożenie aplikacji sieci Web dla App Services w obu chmurach

1. Edytuj plik **WebApplication. csproj** : Wybierz `Runtimeidentifier` , a następnie Dodaj `win10-x64` . Aby uzyskać więcej informacji, zobacz dokumentację dotyczącą [wdrażania samodzielnego](/dotnet/core/deploying/deploy-with-vs#simpleSelf) .

2. Użyj Team Explorer, aby sprawdzić kod w Azure Repos.

3. Upewnij się, że kod aplikacji został zaewidencjonowany w Azure Repos.

### <a name="create-the-build-definition"></a>Tworzenie definicji kompilacji

1. Zaloguj się do Azure Pipelines przy użyciu konta, które może utworzyć definicję kompilacji.

2. Przejdź do strony **Kompilowanie aplikacji sieci Web** dla projektu.

3. W **argumentach**Add **-r Win10-x64** Code. To dodanie jest wymagane do wyzwolenia wdrożenia samodzielnego z platformą .NET Core.

4. Uruchom kompilację. Proces [kompilacji samodzielnego wdrożenia](/dotnet/core/deploying/deploy-with-vs#simpleSelf) spowoduje opublikowanie artefaktów, które mogą być uruchamiane na platformie Azure i w centrum Azure Stack.

#### <a name="use-an-azure-hosted-build-agent"></a>Korzystanie z agenta kompilacji hostowanej na platformie Azure

Korzystanie z hostowanego agenta kompilacji w Azure Pipelines jest wygodną opcją kompilowania i wdrażania aplikacji sieci Web. Konserwacja i uaktualnienia są wykonywane automatycznie przez Microsoft Azure, co umożliwia ciągły i nieprzerwany cykl programowania.

### <a name="configure-the-continuous-deployment-cd-process"></a>Konfigurowanie procesu ciągłego wdrażania (CD)

Azure Pipelines i Azure DevOps Services zapewniają wysoce konfigurowalny i zarządzany potok dla wersji w wielu środowiskach, takich jak programowanie, przemieszczanie, gwarancja jakości i środowisko produkcyjne. Ten proces może obejmować wymaganie zatwierdzeń na określonych etapach cyklu życia aplikacji.

#### <a name="create-release-definition"></a>Utwórz definicję wydania

Tworzenie definicji wydania jest ostatnim krokiem w procesie kompilacji aplikacji. Ta definicja wydania służy do tworzenia wydania i wdrożenia kompilacji.

1. Zaloguj się do Azure Pipelines i przejdź do pozycji **kompilacja i wydanie** dla projektu.

2. Na karcie **wersje** wybierz opcję **[+]** , a następnie wybierz pozycję **Utwórz definicję wydania**.

3. W obszarze **Wybierz szablon**wybierz pozycję **wdrożenie Azure App Service**, a następnie wybierz pozycję **Zastosuj**.

4. Na stronie **Dodawanie artefaktu**ze **źródła (definicji kompilacji)** wybierz aplikację Azure Cloud Build.

5. Na karcie **potok** wybierz link **1 faza**, **1 zadanie** , aby **wyświetlić zadania środowiska**.

6. Na karcie **zadania** wprowadź wartość Azure jako **nazwę środowiska** , a następnie wybierz pozycję AzureCloud Traders — Web EP z listy **subskrypcji platformy Azure** .

7. Wprowadź **nazwę usługi Azure App Service**, która znajduje się `northwindtraders` w następnym przechwyceniu ekranu.

8. W przypadku fazy agenta wybierz pozycję **hostowana program VS2017** z listy **kolejek agentów** .

9. W obszarze **wdróż Azure App Service**wybierz prawidłowy **pakiet lub folder** dla środowiska.

10. W obszarze **Wybierz plik lub folder**wybierz pozycję **OK** , aby określić **lokalizację**.

11. Zapisz wszystkie zmiany i wróć do **potoku**.

12. Na karcie **potok** wybierz pozycję **Dodaj artefakt**i wybierz pozycję **NorthwindCloud Traders-statek** ze **źródła (definicji kompilacji)** .

13. Po **wybraniu szablonu**Dodaj inne środowisko. Wybierz **wdrożenie Azure App Service** a następnie wybierz pozycję **Zastosuj**.

14. Wprowadź `Azure Stack Hub` jako **nazwę środowiska**.

15. Na karcie **zadania** Znajdź i wybierz pozycję Azure Stack Hub.

16. Z listy **subskrypcja platformy Azure** wybierz pozycję **AzureStack Traders — statek EP** dla punktu końcowego centrum Azure Stack.

17. Wprowadź nazwę aplikacji sieci Web Azure Stack Hub jako **nazwę usługi App Service**.

18. W obszarze **wybór agenta**wybierz pozycję **AzureStack-b Douglas Fir** z listy **kolejek agentów** .

19. W polu **wdróż Azure App Service**wybierz prawidłowy **pakiet lub folder** dla środowiska. Na stronie **Wybieranie pliku lub folderu**wybierz pozycję **OK** dla **lokalizacji**folderu.

20. Na karcie **zmienna** Znajdź zmienną o nazwie `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` . Ustaw wartość zmiennej na **true**, a następnie ustaw jej zakres na **Azure Stack Hub**.

21. Na karcie **potok** wybierz ikonę **wyzwalacza ciągłego wdrażania** dla artefaktu sieci Web NorthwindCloud Traders i ustaw **wyzwalacz ciągłego wdrażania** na **włączone**. Wykonaj tę samą czynność dla artefaktu **NorthwindCloud Traders-zbiornika** .

22. W przypadku środowiska Azure Stack Hub wybierz ikonę **warunki przed wdrożeniem** Ustaw wyzwalacz na **po wydaniu**.

23. Zapisz wszystkie zmiany.

> [!Note]  
> Niektóre ustawienia zadań zlecenia są automatycznie definiowane jako [zmienne środowiskowe](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) podczas tworzenia definicji wydania na podstawie szablonu. Tych ustawień nie można modyfikować w ustawieniach zadania, ale można je modyfikować w elementach środowiska nadrzędnego.

## <a name="create-a-release"></a>Tworzenie wersji

1. Na karcie **potok** Otwórz listę **wersja** i wybierz pozycję **Utwórz wydanie**.

2. Wprowadź opis wydania, sprawdź, czy wybrano poprawne artefakty, a następnie wybierz pozycję **Utwórz**. Po kilku chwilach pojawi się transparent wskazujący, że nowa wersja została utworzona, a nazwa wydania jest wyświetlana jako link. Wybierz łącze, aby wyświetlić stronę podsumowania wydania.

3. Na stronie Podsumowanie wydania są wyświetlane szczegółowe informacje o wersji. W poniższym przechwyceniu ekranu dla wersji "Release-2" sekcja **środowiska** pokazuje **stan wdrożenia** platformy Azure jako "w toku", a stan centrum Azure Stack to "powodzenie". Gdy stan wdrożenia środowiska platformy Azure zmieni się na "powodzenie", zostanie wyświetlony transparent wskazujący, że wydanie jest gotowe do zatwierdzenia. Gdy wdrożenie oczekuje lub nie powiodło się, wyświetlana jest ikona informacji o niebieskie **(i)** . Umieść kursor nad ikoną, aby wyświetlić okno podręczne zawierające przyczynę opóźnienia lub niepowodzenia.

4. Inne widoki, takie jak lista wydań, wyświetlają również ikonę wskazującą, że zatwierdzenie jest w stanie oczekiwania. W oknie podręcznym dla tej ikony zostanie wyświetlona nazwa środowiska i więcej szczegółów dotyczących wdrożenia. W łatwy sposób administrator widzi ogólny postęp wydań i widzi, które wydania oczekują na zatwierdzenie.

## <a name="monitor-and-track-deployments"></a>Monitorowanie i śledzenie wdrożeń

1. Na stronie Podsumowanie **wersji 2** wybierz pozycję **dzienniki**. Podczas wdrażania na tej stronie jest wyświetlany dziennik na żywo od agenta. W okienku po lewej stronie jest wyświetlany stan każdej operacji we wdrożeniu dla każdego środowiska.

2. Wybierz ikonę osoby w kolumnie **Akcja** dla zatwierdzenia przed wdrożeniem lub po wdrożeniu, aby zobaczyć, kto zatwierdził (lub odrzucił) wdrożenie i podaną przez nie wiadomość.

3. Po zakończeniu wdrożenia cały plik dziennika zostanie wyświetlony w okienku po prawej stronie. Wybierz dowolny **krok** w okienku po lewej stronie, aby wyświetlić plik dziennika dla pojedynczego kroku, na przykład **zainicjuj zadanie**. Możliwość wyświetlania pojedynczych dzienników ułatwia śledzenie i debugowanie części ogólnego wdrożenia. **Zapisz** plik dziennika dla kroku lub **Pobierz wszystkie dzienniki w formacie ZIP**.

4. Otwórz kartę **Podsumowanie** , aby wyświetlić ogólne informacje o wersji. Ten widok przedstawia szczegółowe informacje o kompilacji, środowiskach, w których zostały wdrożone, o stanie wdrożenia i innych informacjach o wersji.

5. Wybierz łącze środowiska (**Azure** lub **Azure Stack Hub**), aby wyświetlić informacje o istniejących i oczekujących wdrożeniach w określonym środowisku. Użyj tych widoków, aby szybko sprawdzić, czy ta sama kompilacja została wdrożona w obu środowiskach.

6. Otwórz **wdrożoną aplikację produkcyjną** w przeglądarce. Na przykład dla witryny sieci Web usługi Azure App Services Otwórz adres URL `https://[your-app-name\].azurewebsites.net` .

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>Integracja z platformą Azure i usługą Azure Stack Hub zapewnia skalowalne rozwiązanie dla wielu chmur

Elastyczna i niezawodna usługa w chmurze zapewnia bezpieczeństwo danych, tworzenie kopii zapasowych, spójność i szybka dostępność, skalowalność magazynu i dystrybucji oraz Routing zgodny ze geograficzną. Ten proces jest wyzwalany ręcznie, zapewniając niezawodne i wydajne przełączanie między hostowanymi aplikacjami sieci Web i natychmiastową dostępnością kluczowych danych.

## <a name="next-steps"></a>Następne kroki

- Aby dowiedzieć się więcej o wzorcach chmury platformy Azure, zobacz [wzorce projektowe w chmurze](/azure/architecture/patterns).
