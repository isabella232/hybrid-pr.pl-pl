---
title: Kierowanie ruchu za pomocą aplikacji rozproszonej geograficznie przy użyciu platformy Azure i usługi Azure Stack Hub
description: Dowiedz się, jak skierować ruch do określonych punktów końcowych za pomocą rozwiązania do obsługi aplikacji rozproszonych geograficznie przy użyciu platformy Azure i Azure Stack centrum.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911322"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Kierowanie ruchu za pomocą aplikacji rozproszonej geograficznie przy użyciu platformy Azure i usługi Azure Stack Hub

Dowiedz się, jak skierować ruch do określonych punktów końcowych na podstawie różnych metryk przy użyciu wzorca aplikacji rozproszonych geograficznie. Tworzenie profilu Traffic Manager przy użyciu geograficznej konfiguracji routingu i punktu końcowego zapewnia, że informacje są kierowane do punktów końcowych w oparciu o wymagania regionalne, przepisy firmowe i międzynarodowe oraz Twoje potrzeby dotyczące danych.

W tym rozwiązaniu utworzysz przykładowe środowisko w celu:

> [!div class="checklist"]
> - Tworzenie aplikacji rozproszonej geograficznie.
> - Użyj Traffic Manager, aby określić aplikację jako docelową.

## <a name="use-the-geo-distributed-apps-pattern"></a>Używanie wzorca aplikacji rozproszonych geograficznie

Ze wzorcem rozproszonym geograficznie aplikacja obejmuje regiony. Możesz domyślnie być chmurą publiczną, ale niektórzy użytkownicy mogą wymagać, aby ich dane pozostawały w ich regionie. Możesz kierować użytkowników do najbardziej odpowiedniej chmury na podstawie ich wymagań.

### <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

#### <a name="scalability-considerations"></a>Zagadnienia dotyczące skalowalności

Rozwiązanie, które będziesz kompilować w tym artykule, nie uwzględnia skalowalności. Jednak jeśli są używane w połączeniu z innymi rozwiązaniami Azure i lokalnymi, można obsłużyć wymagania dotyczące skalowalności. Aby uzyskać informacje na temat tworzenia rozwiązania hybrydowego z skalowaniem automatycznym za pomocą usługi Traffic Manager, zobacz [Tworzenie rozwiązań do skalowania między chmurami przy użyciu platformy Azure](solution-deployment-guide-cross-cloud-scaling.md).

#### <a name="availability-considerations"></a>Zagadnienia dotyczące dostępności

Podobnie jak w przypadku problemów z skalowalnością, to rozwiązanie nie jest bezpośrednio związane z dostępnością. Jednak rozwiązania platformy Azure i lokalne można zaimplementować w ramach tego rozwiązania, aby zapewnić wysoką dostępność dla wszystkich składników.

### <a name="when-to-use-this-pattern"></a>Kiedy używać tego wzorca

- Organizacja ma rozgałęzienia międzynarodowe wymagające niestandardowych zasad zabezpieczeń i dystrybucji.

- Każda z biur organizacji pobiera dane pracowników, firm i materiałów, które wymagają działania raportowania na lokalne regulacje i strefy czasowe.

- Wymagania dotyczące wysokiego rozmiaru są spełnione przez skalowanie w poziomie aplikacji z wieloma wdrożeniami aplikacji w jednym regionie i w różnych regionach, aby obsługiwać skrajne wymagania dotyczące obciążenia.

### <a name="planning-the-topology"></a>Planowanie topologii

Przed rozpoczęciem tworzenia rozproszonej aplikacji można znać następujące kwestie:

- **Domena niestandardowa dla aplikacji:** Jaka jest nazwa domeny niestandardowej, która będzie używana przez klientów w celu uzyskania dostępu do aplikacji? W przypadku przykładowej aplikacji niestandardowa nazwa domeny to *www \. scalableasedemo.com.*

- **Traffic Manager domeny:** Nazwa domeny jest wybierana podczas tworzenia [profilu usługi Azure Traffic Manager](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles). Ta nazwa jest połączona z sufiksem *trafficmanager.NET* w celu zarejestrowania wpisu domeny, który jest zarządzany przez Traffic Manager. W przypadku przykładowej aplikacji wybrana nazwa to *skalowalne — Demonstracja*. W związku z tym pełna nazwa domeny, która jest zarządzana przez Traffic Manager, to *Scalable-ASE-demo.trafficmanager.NET*.

- **Strategia skalowania rozmiaru aplikacji:** Zdecyduj, czy rozmiar aplikacji będzie dystrybuowany między wieloma środowiskami App Service w jednym regionie, wielu regionach, czy też z obu obu sposobów. Decyzja powinna być oparta na oczekiwaniach, w których nastąpi ruch klienta i jak również pozostała część aplikacji obsługującej infrastrukturę zaplecza może skalować. Na przykład w przypadku aplikacji bezstanowej 100% aplikacja może być w znacznym stopniu skalowana przy użyciu kombinacji wielu App Serviceych środowisk w regionie świadczenia usługi Azure, pomnożonych przez App Service środowiska wdrożone w wielu regionach świadczenia usługi Azure. Dzięki 15 i globalnym regionom platformy Azure dostępnym do wyboru klienci mogą naprawdę kompilować skalę na całym świecie. W przypadku przykładowej aplikacji używanej w tym miejscu trzy środowiska App Service zostały utworzone w jednym regionie świadczenia usługi Azure (Południowo-środkowe stany USA).

- **Konwencja nazewnictwa dla środowisk App Service:** Każde środowisko App Service wymaga unikatowej nazwy. Poza jednym lub dwoma środowiskami App Service warto mieć konwencję nazewnictwa ułatwiającą identyfikację każdego środowiska App Service. W przypadku przykładowej aplikacji użytej w tym miejscu użyto prostej konwencji nazewnictwa. Nazwy trzech środowisk App Service to *fe1ase*, *fe2ase*i *fe3ase*.

- **Konwencja nazewnictwa dla aplikacji:** Ponieważ zostanie wdrożonych wiele wystąpień aplikacji, wymagana jest nazwa dla każdego wystąpienia wdrożonej aplikacji. Za pomocą App Service Environment dla aplikacji zaawansowanych można używać tej samej nazwy aplikacji w wielu środowiskach. Ponieważ każde środowisko App Service ma unikatowy sufiks domeny, deweloperzy mogą skorzystać z dokładnej nazwy aplikacji w każdym środowisku. Na przykład deweloper może mieć aplikacje o nazwie w następujący sposób: *MyApp.Foo1.p.azurewebsites.NET*, *MyApp.foo2.p.azurewebsites.NET*, *MyApp.Foo3.p.azurewebsites.NET*itd. W przypadku aplikacji użytej w tym miejscu każde wystąpienie aplikacji ma unikatową nazwę. Używane nazwy wystąpień aplikacji to *webfrontend1*, *webfrontend2*i *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub to rozszerzenie platformy Azure. Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.  
> 
> [Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych. Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.

## <a name="part-1-create-a-geo-distributed-app"></a>Część 1. Tworzenie aplikacji rozproszonej geograficznie

W tej części utworzysz aplikację sieci Web.

> [!div class="checklist"]
> - Twórz aplikacje sieci Web i Publikuj.
> - Dodaj kod do Azure Repos.
> - Wskaż kompilację aplikacji w wielu celach w chmurze.
> - Zarządzanie procesem CD i konfigurowanie go.

### <a name="prerequisites"></a>Wymagania wstępne

Wymagana jest subskrypcja platformy Azure i instalacja centrum Azure Stack.

### <a name="geo-distributed-app-steps"></a>Kroki aplikacji rozproszonej geograficznie

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Uzyskaj domenę niestandardową i skonfiguruj system DNS

Zaktualizuj plik strefy DNS dla domeny. Usługa Azure AD może następnie zweryfikować własność niestandardowej nazwy domeny. Użyj [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) dla systemu Azure/usługi Office 365/zewnętrznych rekordów DNS na platformie Azure lub Dodaj wpis DNS w [innym rejestratorze DNS](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Zarejestruj domenę niestandardową z rejestratorem publicznym.

2. Zaloguj się do rejestratora nazw domen dla domeny. Aby można było zaktualizować DNS, może być wymagane zatwierdzenie administratora.

3. Zaktualizuj plik strefy DNS dla domeny przez dodanie wpisu DNS dostarczonego przez usługę Azure AD. Wpis DNS nie zmienia zachowań, takich jak Routing poczty lub hosting internetowy.

### <a name="create-web-apps-and-publish"></a>Tworzenie aplikacji sieci Web i publikowanie

Skonfiguruj hybrydową ciągłą integrację/ciągłe dostarczanie (CI/CD), aby wdrożyć aplikację sieci Web na platformie Azure i w Azure Stack Hub, a następnie przeprowadź autowypychanie zmian w obu chmurach.

> [!Note]  
> Wymagane są Azure Stack centrum z odpowiednimi obrazami do uruchomienia (system Windows Server i SQL) oraz wdrożenie App Service. Aby uzyskać więcej informacji, zobacz [wymagania wstępne dotyczące wdrażania App Service w centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

#### <a name="add-code-to-azure-repos"></a>Dodaj kod do Azure Repos

1. Zaloguj się do programu Visual Studio przy użyciu **konta, które ma prawa do tworzenia projektu** na Azure Repos.

    Ciągłej integracji/ciągłego wdrażania można użyć zarówno w kodzie aplikacji, jak i w kodzie infrastruktury. Używaj [szablonów Azure Resource Manager](https://azure.microsoft.com/resources/templates/) zarówno do programowania w chmurze prywatnej, jak i hostowanej.

    ![Nawiązywanie połączenia z projektem w programie Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. **Sklonuj repozytorium** , tworząc i otwierając domyślną aplikację sieci Web.

    ![Klonowanie repozytorium w programie Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Utwórz wdrożenie aplikacji sieci Web w obu chmurach

1. Edytuj plik **WebApplication. csproj** : Wybierz `Runtimeidentifier` i Dodaj `win10-x64` . (Zobacz dokumentację [wdrożenia prewartą](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) ).

    ![Edytuj plik projektu aplikacji sieci Web w programie Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. **Zaewidencjonuj kod, aby Azure Repos** przy użyciu Team Explorer.

3. Upewnij się, że **kod aplikacji** został zaewidencjonowany do Azure Repos.

### <a name="create-the-build-definition"></a>Tworzenie definicji kompilacji

1. **Zaloguj się do Azure Pipelines** , aby potwierdzić możliwość tworzenia definicji kompilacji.

2. Dodaj `-r win10-x64` kod. Jest to konieczne do wyzwolenia wdrożenia samodzielnego z platformą .NET Core.

    ![Dodawanie kodu do definicji kompilacji w Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. **Uruchom kompilację**. Proces [kompilacji samodzielnego wdrożenia](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) spowoduje opublikowanie artefaktów, które mogą być uruchamiane na platformie Azure i w centrum Azure Stack.

#### <a name="using-an-azure-hosted-agent"></a>Korzystanie z agenta hostowanego na platformie Azure

Korzystanie z hostowanego agenta w Azure Pipelines jest wygodną opcją kompilowania i wdrażania aplikacji sieci Web. Konserwacja i uaktualnienia są wykonywane automatycznie przez Microsoft Azure, co umożliwia nieprzerwane programowanie, testowanie i wdrażanie.

### <a name="manage-and-configure-the-cd-process"></a>Zarządzanie i Konfigurowanie procesu CD

Azure DevOps Services zapewnić wysoce konfigurowalny i zarządzany potok dla wydań w wielu środowiskach, takich jak programowanie, przemieszczanie, zarządzanie PYTANIAmi i środowiska produkcyjne; obejmuje to wymaganie zatwierdzeń na określonych etapach.

## <a name="create-release-definition"></a>Utwórz definicję wydania

1. Wybierz przycisk **Plus** , aby dodać nową wersję na karcie **wersje** w sekcji **kompilacja i wersja** Azure DevOps Services.

    ![Tworzenie definicji wydania w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. Zastosuj szablon wdrażania Azure App Service.

   ![Zastosuj szablon wdrożenia Azure App Service w programie Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. W obszarze **Dodawanie artefaktu**Dodaj artefakt dla aplikacji Azure Cloud Build.

   ![Dodaj artefakt do usługi Azure Cloud Build w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. Na karcie potok wybierz pozycję **faza, link zadania** środowiska i ustaw wartości środowiska chmury platformy Azure.

   ![Ustaw wartości środowiska chmury platformy Azure w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. Ustaw **nazwę środowiska** i wybierz **subskrypcję platformy Azure** dla punktu końcowego w chmurze platformy Azure.

      ![Wybierz subskrypcję platformy Azure dla punktu końcowego w chmurze platformy Azure w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. W obszarze **nazwa usługi App Service**Ustaw wymaganą nazwę usługi Azure App Service.

      ![Ustaw nazwę usługi Azure App Service w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. Wprowadź wartość "hostowana program VS2017" w obszarze **Kolejka agenta** dla środowiska hostowanego w chmurze platformy Azure.

      ![Ustaw kolejkę agentów dla środowiska hostowanego w chmurze platformy Azure w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. W menu Wdróż Azure App Service wybierz prawidłowy **pakiet lub folder** dla środowiska. Wybierz pozycję **OK** , aby określić **lokalizację folderu**.
  
      ![Wybierz pakiet lub folder dla środowiska Azure App Service w programie Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Wybierz pakiet lub folder dla środowiska Azure App Service w programie Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. Zapisz wszystkie zmiany i wróć do **potoku wydania**.

    ![Zapisz zmiany w potoku wydania w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. Dodaj nowy artefakt, wybierając kompilację dla aplikacji Centrum Azure Stack.

    ![Dodawanie nowego artefaktu dla aplikacji Azure Stack Hub w programie Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. Dodaj jeszcze jedno środowisko, stosując wdrożenie Azure App Service.

    ![Dodawanie środowiska do wdrożenia Azure App Service w programie Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. Nazwij nowe środowisko Azure Stack centrum.

    ![Nazwa środowiska w Azure App Service wdrożenia w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. Znajdź środowisko centrum Azure Stack w obszarze Karta **zadań** .

    ![Środowisko Azure Stack Hub w Azure DevOps Services w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. Wybierz subskrypcję dla punktu końcowego Azure Stack centrum.

    ![Wybierz subskrypcję punktu końcowego Azure Stack Hub w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. Określ nazwę aplikacji sieci Web Azure Stack Hub jako nazwę usługi App Service.

    ![Ustaw Azure Stack nazwę aplikacji sieci Web Hub w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. Wybierz agenta Azure Stack Hub.

    ![Wybierz agenta Azure Stack Hub w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. W sekcji Wdróż Azure App Service wybierz prawidłowy **pakiet lub folder** dla środowiska. Wybierz pozycję **OK** , aby określić lokalizację folderu.

    ![Wybierz folder do wdrożenia Azure App Service w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Wybierz folder do wdrożenia Azure App Service w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. W obszarze Karta zmienna Dodaj zmienną o nazwie `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , ustaw jej wartość na **true**i zakres na Azure Stack Hub.

    ![Dodawanie zmiennej do wdrożenia aplikacji platformy Azure w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. Wybierz ikonę wyzwalacza **ciągłego** wdrażania w obu artefaktach i Włącz wyzwalacz wdrażania **Kontynuuj** .

    ![Wybierz wyzwalacz ciągłego wdrażania w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. Wybierz ikonę warunki **przed wdrożeniem** w środowisku centrum Azure Stack i ustaw wyzwalacz na wartość **po wydaniu.**

    ![Wybierz warunki przed wdrożeniem w Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. Zapisz wszystkie zmiany.

> [!Note]  
> Niektóre ustawienia zadań mogą zostać automatycznie zdefiniowane jako [zmienne środowiskowe](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) podczas tworzenia definicji wydania na podstawie szablonu. Tych ustawień nie można modyfikować w ustawieniach zadania; Zamiast tego należy wybrać element środowiska nadrzędnego, aby edytować te ustawienia.

## <a name="part-2-update-web-app-options"></a>Część 2: Aktualizowanie opcji aplikacji sieci Web

[Azure App Service](https://docs.microsoft.com/azure/app-service/overview) zapewnia wysoce skalowalną, samoobsługową usługę hostingu w sieci Web.

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Zamapuj istniejącą niestandardową nazwę DNS na platformę Azure Web Apps.
> - Użyj **rekordu CNAME** i **rekordu a,** Aby zmapować niestandardową nazwę DNS na App Service.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Mapowanie istniejącej niestandardowej nazwy DNS na aplikacje internetowe platformy Azure

> [!Note]  
> Użyj rekordu CNAME dla wszystkich niestandardowych nazw DNS, z wyjątkiem domeny głównej (na przykład northwind.com).

Aby przeprowadzić migrację aktywnej witryny oraz jej nazwy domeny DNS do usługi App Service, zobacz [Migrate an active DNS name to Azure App Service](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain) (Migrowanie aktywnej nazwy DNS do usługi Azure App Service).

### <a name="prerequisites"></a>Wymagania wstępne

Aby ukończyć to rozwiązanie:

- [Utwórz aplikację App Service](https://docs.microsoft.com/azure/app-service/)lub użyj aplikacji utworzonej dla innego rozwiązania.

- Kup nazwę domeny i Zapewnij dostęp do rejestru DNS dostawcy domeny.

Zaktualizuj plik strefy DNS dla domeny. Usługa Azure AD sprawdzi własność niestandardowej nazwy domeny. Użyj [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) dla systemu Azure/usługi Office 365/zewnętrznych rekordów DNS na platformie Azure lub Dodaj wpis DNS w [innym rejestratorze DNS](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

- Zarejestruj domenę niestandardową z rejestratorem publicznym.

- Zaloguj się do rejestratora nazw domen dla domeny. (Do aktualizacji DNS może być wymagana zatwierdzona administrator).

- Zaktualizuj plik strefy DNS dla domeny przez dodanie wpisu DNS dostarczonego przez usługę Azure AD.

Aby na przykład dodać wpisy DNS dla northwindcloud.com i www \. northwindcloud.com, skonfiguruj ustawienia DNS dla domeny katalogu głównego northwindcloud.com.

> [!Note]  
> Nazwę domeny można zakupić przy użyciu [Azure Portal](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain). Aby zamapować niestandardową nazwę DNS na aplikację internetową, dla tej aplikacji internetowej musisz mieć płatną warstwę [planu usługi App Service](https://azure.microsoft.com/pricing/details/app-service/) (**Współdzielona**, **Podstawowa**, **Standardowa** lub ** Premium**).

### <a name="create-and-map-cname-and-a-records"></a>Tworzenie i mapowanie rekordów CNAME i A

#### <a name="access-dns-records-with-domain-provider"></a>Uzyskiwanie dostępu do rekordów DNS u dostawcy domen

> [!Note]  
>  Użyj Azure DNS, aby skonfigurować niestandardową nazwę DNS dla Web Apps platformy Azure. Aby uzyskać więcej informacji, zobacz [Use Azure DNS to provide custom domain settings for an Azure service](https://docs.microsoft.com/azure/dns/dns-custom-domain) (Korzystanie z usługi Azure DNS w celu udostępnienia niestandardowych ustawień domeny dla usługi platformy Azure).

1. Zaloguj się do witryny sieci Web głównego dostawcy.

2. Znajdź stronę służącą do zarządzania rekordami DNS. Każdy dostawca domeny ma własny interfejs rekordów DNS. Poszukaj obszarów witryny z etykietą **Nazwa domeny**, **DNS** lub **Zarządzanie serwerami nazw**.

Strona rekordów DNS może być wyświetlana w obszarze **Moje domeny**. Znajdź łącze o nazwie **plik strefy**, **rekordy DNS**lub **Konfiguracja zaawansowana**.

Poniższy zrzut ekranu przedstawia przykład strony rekordów DNS:

![Przykładowa strona rekordów DNS](media/solution-deployment-guide-geo-distributed/image28.png)

1. W obszarze rejestrator nazw domen wybierz pozycję **Dodaj lub Utwórz** , aby utworzyć rekord. Niektórzy dostawcy udostępniają różne linki na potrzeby dodawania różnych typów rekordów. Zapoznaj się z dokumentacją dostawcy.

2. Dodaj rekord CNAME, aby zmapować poddomenę na domyślną nazwę hosta aplikacji.

   W przypadku \. przykładowej domeny northwindcloud.com www Dodaj rekord CNAME, który mapuje nazwę na `<app_name>.azurewebsites.net` .

Po dodaniu rekordu CNAME Strona rekordów DNS wygląda podobnie do poniższego przykładu:

![Nawigacja w portalu do aplikacji platformy Azure](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Włączanie mapowania rekordów CNAME na platformie Azure

1. Na nowej karcie Zaloguj się do Azure Portal.

2. Wybierz pozycję App Services.

3. Wybierz pozycję aplikacja sieci Web.

4. W lewym obszarze nawigacji na stronie aplikacji w witrynie Azure Portal wybierz pozycję **Domeny niestandardowe**.

5. Wybierz **+** ikonę obok pozycji **Dodaj nazwę hosta**.

6. Wpisz w pełni kwalifikowaną nazwę domeny, na przykład `www.northwindcloud.com` .

7. Wybierz przycisk **Weryfikuj**.

8. Jeśli to wskazane, Dodaj dodatkowe rekordy innych typów ( `A` lub `TXT` ) do rekordów DNS rejestratorów nazw domen. Platforma Azure dostarczy wartości i typy tych rekordów:

   a.  Rekord **A** do zmapowania adresu IP aplikacji.

   b.  Rekord **TXT** do zmapowania domyślnej nazwy hosta aplikacji `<app_name>.azurewebsites.net`. App Service używa tego rekordu tylko w czasie konfiguracji do sprawdzenia własności domeny niestandardowej. Po weryfikacji Usuń rekord TXT.

9. Wykonaj to zadanie na karcie rejestrator domen i ponownie sprawdź poprawność, dopóki nie zostanie aktywowany przycisk **Dodaj nazwę hosta** .

10. Upewnij się, że **Typ rekordu nazwy hosta** jest ustawiony na wartość **CNAME** (www.example.com lub dowolna poddomena).

11. Wybierz przycisk **Dodaj nazwę hosta**.

12. Wpisz w pełni kwalifikowaną nazwę domeny, na przykład `northwindcloud.com` .

13. Wybierz przycisk **Weryfikuj**. **Dodatek** jest aktywowany.

14. Upewnij się, że **Typ rekordu nazwy hosta** jest ustawiony na **rekord** (example.com).

15. **Dodaj nazwę hosta**.

    Może upłynąć trochę czasu, zanim nowe nazwy hostów zostaną odzwierciedlone na stronie **domeny niestandardowe** aplikacji. Spróbuj odświeżyć przeglądarkę, aby zaktualizować dane.
  
    ![Niestandardowe domeny](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    Jeśli wystąpi błąd, w dolnej części strony pojawi się powiadomienie o błędzie weryfikacji. ![Błąd weryfikacji domeny](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  Powyższe kroki można powtórzyć, aby zmapować domenę symboli wieloznacznych ( \* . northwindcloud.com). Pozwala to dodać do tej usługi aplikacji dodatkowe domeny podrzędne bez konieczności tworzenia oddzielnego rekordu CNAME dla każdej z nich. Postępuj zgodnie z instrukcjami rejestratora, aby skonfigurować to ustawienie.

#### <a name="test-in-a-browser"></a>Testowanie w przeglądarce

Przejdź do nazw DNS skonfigurowanych wcześniej (na przykład `northwindcloud.com` lub `www.northwindcloud.com` ).

## <a name="part-3-bind-a-custom-ssl-cert"></a>Część 3: wiązanie niestandardowego certyfikatu SSL

W tej części będziemy:

> [!div class="checklist"]
> - Powiąż niestandardowy certyfikat SSL z App Service.
> - Wymuszanie protokołu HTTPS dla aplikacji.
> - Automatyzacja powiązania certyfikatu SSL ze skryptami.

> [!Note]  
> W razie potrzeby uzyskaj certyfikat SSL klienta w Azure Portal i powiąż go z aplikacją sieci Web. Aby uzyskać więcej informacji, zobacz [samouczek App Service Certificates](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site).

### <a name="prerequisites"></a>Wymagania wstępne

Aby ukończyć to rozwiązanie:

- [Utwórz aplikację App Service.](https://docs.microsoft.com/azure/app-service/)
- [Zamapuj niestandardową nazwę DNS na aplikację sieci Web.](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- Uzyskaj certyfikat SSL z zaufanego urzędu certyfikacji i Użyj klucza do podpisania żądania.

### <a name="requirements-for-your-ssl-certificate"></a>Wymagania dotyczące certyfikatu protokołu SSL

Aby używać certyfikatu w usłudze App Service, musi on spełniać wszystkie następujące wymagania:

- Podpisany przez zaufany urząd certyfikacji.

- Eksportowany jako chroniony hasłem plik PFX.

- Zawiera klucz prywatny o długości co najmniej 2048 bitów.

- Zawiera wszystkie certyfikaty pośrednie w łańcuchu certyfikatów.

> [!Note]  
> **Certyfikaty kryptografii krzywej eliptycznej (ECC)** działają z App Service, ale nie są uwzględnione w tym przewodniku. Zapoznaj się z urzędem certyfikacji, aby uzyskać pomoc w tworzeniu certyfikatów ECC.

#### <a name="prepare-the-web-app"></a>Przygotowywanie aplikacji sieci Web

Aby powiązać niestandardowy certyfikat SSL z aplikacją internetową, [plan App Service](https://azure.microsoft.com/pricing/details/app-service/) musi znajdować się w warstwie **podstawowa**, **standardowa**lub **Premium** .

#### <a name="sign-in-to-azure"></a>Logowanie do platformy Azure

1. Otwórz [Azure Portal](https://portal.azure.com/) i przejdź do aplikacji sieci Web.

2. Z menu po lewej stronie wybierz pozycję **App Services**, a następnie wybierz nazwę aplikacji sieci Web.

![Wybierz aplikację sieci Web w Azure Portal](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>Sprawdzanie warstwy cenowej

1. W okienku nawigacji po lewej stronie aplikacji sieci Web przewiń do sekcji **Ustawienia** i wybierz pozycję **Skaluj w górę (plan App Service)**.

    ![Menu skalowania w aplikacji sieci Web](media/solution-deployment-guide-geo-distributed/image34.png)

1. Upewnij się, że aplikacja sieci Web nie znajduje się w warstwie **bezpłatna** lub **współdzielona** . Bieżąca warstwa aplikacji sieci Web zostanie wyróżniona w ciemnoniebieskim polu.

    ![Sprawdzanie warstwy cenowej w aplikacji sieci Web](media/solution-deployment-guide-geo-distributed/image35.png)

Niestandardowy protokół SSL nie jest obsługiwany w warstwie **bezpłatna** ani **współdzielona** . Aby przeprowadzić skalowanie, wykonaj kroki opisane w następnej sekcji lub stronie **Wybierz warstwę cenową** , a następnie przejdź do [przekazywania i powiązania certyfikatu SSL](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

#### <a name="scale-up-your-app-service-plan"></a>Skalowanie w górę planu usługi App Service

1. Wybierz jedną z następujących warstw: **Podstawowa**, **Standardowa** lub **Premium**.

2. Wybierz pozycję **Wybierz**.

![Wybierz warstwę cenową dla aplikacji sieci Web](media/solution-deployment-guide-geo-distributed/image36.png)

Operacja skalowania zostanie zakończona, gdy zostanie wyświetlone powiadomienie.

![Powiadomienie o skalowaniu w górę](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>Powiązywanie certyfikatu SSL i scalanie certyfikatów pośrednich

Scal wiele certyfikatów w łańcuchu.

1. **Otwórz każdy certyfikat** otrzymany w edytorze tekstu.

2. Utwórz plik dla scalonego certyfikatu o nazwie *mergedcertificate. CRT*. W edytorze tekstów skopiuj zawartość każdego certyfikatu do tego pliku. Kolejność certyfikatów powinna być zgodna z kolejnością w łańcuchu certyfikatów, poczynając od Twojego certyfikatu i kończąc na certyfikacie głównym. Wygląda to następująco:

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>Eksportowanie certyfikatu do pliku PFX

Wyeksportuj scalony certyfikat SSL przy użyciu klucza prywatnego wygenerowanego przez certyfikat.

Plik klucza prywatnego jest tworzony za pośrednictwem OpenSSL. Aby wyeksportować certyfikat do pliku PFX, uruchom następujące polecenie i Zastąp symbole zastępcze `<private-key-file>` oraz `<merged-certificate-file>` ścieżkę klucza prywatnego i scalony plik certyfikatu:

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Po wyświetleniu monitu Zdefiniuj hasło eksportu służące do przekazywania certyfikatu SSL w celu App Service późniejszej.

Gdy usługi IIS lub **Certreq.exe** są używane do generowania żądania certyfikatu, należy zainstalować certyfikat na komputerze lokalnym, a następnie [wyeksportować certyfikat do pliku PFX](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).

#### <a name="upload-the-ssl-certificate"></a>Przekaż certyfikat SSL

1. Wybierz pozycję **Ustawienia protokołu SSL** w lewym obszarze nawigacji aplikacji sieci Web.

2. Wybierz pozycję **Przekaż certyfikat**.

3. W **pliku certyfikatu PFX**wybierz pozycję plik PFX.

4. W polu **hasło certyfikatu**wpisz hasło utworzone podczas EKSPORTOWANIA pliku PFX.

5. Wybierz przycisk **Przekaż**.

    ![Przekaż certyfikat SSL](media/solution-deployment-guide-geo-distributed/image38.png)

Po zakończeniu przekazywania certyfikatu App Service zostanie on wyświetlony na stronie **Ustawienia protokołu SSL** .

![Ustawienia protokołu SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>Wiązanie certyfikatu protokołu SSL

1. W sekcji **powiązania SSL** wybierz pozycję **Dodaj powiązanie**.

    > [!Note]  
    >  Jeśli certyfikat został przekazany, ale nie pojawia się na liście nazwy domen na liście rozwijanej **Nazwa hosta** , spróbuj odświeżyć stronę przeglądarki.

2. Na stronie **Dodawanie powiązania SSL** Użyj listy rozwijanej, aby wybrać nazwę domeny do zabezpieczenia i certyfikat do użycia.

3. W obszarze **Typ protokołu SSL** wybierz, czy ma być używane [**Oznaczanie nazwy serwera (SNI, Server Name Indication)**](https://en.wikipedia.org/wiki/Server_Name_Indication), czy też protokół SSL oparty na protokole IP.

    - **Protokół SSL oparty na SNI**: można dodać wiele powiązań SSL opartych na SNI. Ta opcja umożliwia zabezpieczenie wielu domen na tym samym adresie IP za pomocą wielu certyfikatów protokołu SSL. Większość nowoczesnych przeglądarek (w tym programy Internet Explorer, Chrome, Firefox i Opera) obsługuje funkcję SNI. Bardziej szczegółowe informacje dotyczące obsługi przeglądarek możesz znaleźć w artykule [Server Name Indication (Oznaczanie nazwy serwera)](https://wikipedia.org/wiki/Server_Name_Indication).

    - **Protokół SSL oparty na**protokole IP: można dodać tylko jedno powiązanie SSL oparte na adresie IP. Ta opcja umożliwia zabezpieczenie dedykowanego publicznego adresu IP za pomocą tylko jednego certyfikatu protokołu SSL. Aby zabezpieczyć wiele domen, zabezpiecz je wszystkie przy użyciu tego samego certyfikatu SSL. Protokół SSL oparty na protokole IP jest tradycyjną opcją dla powiązania SSL.

4. Wybierz pozycję **Dodaj powiązanie**.

    ![Dodawanie powiązania SSL](media/solution-deployment-guide-geo-distributed/image40.png)

Po zakończeniu przekazywania certyfikatu App Service zostanie on wyświetlony w sekcjach **powiązania protokołu SSL** .

![Zakończono przekazywanie powiązań SSL](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>Ponowne mapowanie rekordu A dla Połączenie SSL z adresu IP

Jeśli w aplikacji sieci Web nie jest używany protokół SSL oparty na protokole IP, przejdź do [testowania protokołu HTTPS dla domeny niestandardowej](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

Domyślnie aplikacja sieci Web używa udostępnionego publicznego adresu IP. Jeśli certyfikat jest powiązany z protokołem SSL opartym na protokole IP, App Service tworzy nowy i dedykowany adres IP dla aplikacji internetowej.

W przypadku zmapowania rekordu do aplikacji sieci Web należy zaktualizować rejestr domeny za pomocą dedykowanego adresu IP.

Strona **domena niestandardowa** jest aktualizowana przy użyciu nowego, dedykowanego adresu IP. Skopiuj ten [adres IP](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain), a następnie ponownie zamapuj [rekord A](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) na ten nowy adres IP.

#### <a name="test-https"></a>Testowanie protokołu HTTPS

W różnych przeglądarkach przejdź do, `https://<your.custom.domain>` Aby upewnić się, że aplikacja sieci Web jest obsługiwana.

![Przejdź do aplikacji sieci Web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> Jeśli wystąpią błędy walidacji certyfikatu, certyfikat z podpisem własnym może być przyczyną lub certyfikaty pośrednie mogły zostać wyłączone podczas eksportowania do pliku PFX.

#### <a name="enforce-https"></a>Wymuszanie protokołu HTTPS

Domyślnie każdy użytkownik może uzyskać dostęp do aplikacji sieci Web przy użyciu protokołu HTTP. Można przekierować wszystkie żądania HTTP do portu HTTPS.

Na stronie aplikacja sieci Web wybierz pozycję **Ustawienia SL**. Następnie w pozycji **Tylko HTTPS** wybierz opcję **Włączone**.

![Wymuszanie protokołu HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

Po zakończeniu operacji przejdź do dowolnego adresu URL protokołu HTTP, który wskazuje aplikację. Przykład:

- https://<app_name>. azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>Wymuszanie protokołu TLS 1.1/1.2

Aplikacja domyślnie zezwala na [protokół TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0, który nie jest już traktowany jako bezpieczny przez standardy branżowe (takie jak [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)). Aby wymusić nowsze wersje protokołu TLS, wykonaj następujące kroki:

1. Na stronie aplikacja sieci Web w lewym okienku nawigacji wybierz pozycję **Ustawienia protokołu SSL**.

2. W polu **wersja protokołu TLS**wybierz pozycję minimalna wersja protokołu TLS.

    ![Wymuszanie protokołu TLS 1.1 lub 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Tworzenie profilu usługi Traffic Manager

1. Wybierz kolejno pozycje **Utwórz zasób**  >  **Sieć**  >  **Traffic Manager**  >  **Utwórz**profil.

2. W obszarze **Tworzenie profilu usługi Traffic Manager** podaj następujące informacje:

    1. W polu **Nazwa**Podaj nazwę profilu. Ta nazwa musi być unikatowa w obrębie strefy manager.net ruchu i powoduje, że w polu Nazwa DNS trafficmanager.net, która jest używana do uzyskiwania dostępu do profilu Traffic Manager.

    2. W obszarze **Metoda routingu**wybierz **metodę routingu geograficznego**.

    3. W obszarze **subskrypcja**wybierz subskrypcję, w ramach której chcesz utworzyć ten profil.

    4. W obszarze **Grupy zasobów** utwórz nową grupę zasobów, w której zostanie umieszczony ten profil.

    5. W obszarze **Lokalizacja grupy zasobów** wybierz lokalizację grupy zasobów. To ustawienie dotyczy lokalizacji grupy zasobów i nie ma wpływu na profil Traffic Manager wdrożony globalnie.

    6. Wybierz przycisk **Utwórz**.

    7. Po ukończeniu globalnego wdrożenia profilu Traffic Manager jest on wyświetlany w odpowiedniej grupie zasobów jako jeden z zasobów.

        ![Grupy zasobów w profilu tworzenia Traffic Manager](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Dodawanie punktów końcowych usługi Traffic Manager

1. Na pasku wyszukiwania portalu Wyszukaj nazwę **profilu Traffic Manager** utworzoną w poprzedniej sekcji, a następnie wybierz profil usługi Traffic Manager w wyświetlonych wynikach.

2. W obszarze **profil Traffic Manager**w sekcji **Ustawienia** wybierz pozycję **punkty końcowe**.

3. Wybierz pozycję **Dodaj**.

4. Dodawanie punktu końcowego Azure Stack Hub.

5. W obszarze **Typ**wybierz pozycję **zewnętrzny punkt końcowy**.

6. Podaj **nazwę** dla tego punktu końcowego, najlepiej nazwę centrum Azure Stack.

7. Dla w pełni kwalifikowanej nazwy domeny (**FQDN**) Użyj zewnętrznego adresu URL dla aplikacji sieci Web Azure Stack Hub.

8. W obszarze mapowanie geograficzne wybierz region/kontynent, w którym znajduje się zasób. Na przykład **Europa.**

9. W wyświetlonym menu rozwijanym Kraj/region wybierz kraj, który ma zastosowanie do tego punktu końcowego. Na przykład **Niemcy**.

10. Pozycję **Dodaj jako wyłączone** pozostaw niezaznaczoną.

11. Wybierz przycisk **OK**.

12. Dodawanie punkt końcowy platformy Azure:

    1. W obszarze **Typ**wybierz pozycję **punkt końcowy platformy Azure**.

    2. Podaj **nazwę** punktu końcowego.

    3. W obszarze **Typ zasobu docelowego**wybierz pozycję **App Service**.

    4. W polu **zasób docelowy**wybierz pozycję **Wybierz usługę App Service** , aby wyświetlić listę Web Apps w ramach tej samej subskrypcji. W obszarze **zasób**wybierz usługę App Service używaną jako pierwszy punkt końcowy.

13. W obszarze mapowanie geograficzne wybierz region/kontynent, w którym znajduje się zasób. Na przykład **Ameryka Północna/środkowe Ameryki/Karaiby.**

14. W wyświetlonym menu rozwijanym Kraj/region, pozostaw to pole puste, aby wybrać wszystkie powyższe grupowanie regionalne.

15. Pozycję **Dodaj jako wyłączone** pozostaw niezaznaczoną.

16. Wybierz przycisk **OK**.

    > [!Note]  
    >  Utwórz co najmniej jeden punkt końcowy z zakresem geograficznym (World), który będzie używany jako domyślny punkt końcowy dla zasobu.

17. Po zakończeniu dodawania obu punktów końcowych są one wyświetlane w **profilu Traffic Manager** wraz z ich stanem monitorowania w **trybie online**.

    ![Stan punktu końcowego profilu Traffic Manager](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>Globalny Enterprise opiera się na możliwościach dystrybucji geograficznej platformy Azure

Kierowanie ruchu danych za pośrednictwem platformy Azure Traffic Manager i punkty końcowe specyficzne dla lokalizacji geograficznej umożliwiają globalnym przedsiębiorstwom przestrzeganie przepisów regionalnych i utrzymywanie zgodności z danymi, co jest niezwykle ważne dla sukcesu lokalnych i zdalnych lokalizacji roboczych.

## <a name="next-steps"></a>Następne kroki

- Aby dowiedzieć się więcej o wzorcach chmury platformy Azure, zobacz [wzorce projektowe w chmurze](https://docs.microsoft.com/azure/architecture/patterns).
