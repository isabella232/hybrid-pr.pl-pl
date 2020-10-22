---
title: Wdróż aplikację hybrydową przy użyciu danych lokalnych, które skaluje się z wielu chmur
description: Dowiedz się, jak wdrożyć aplikację korzystającą z danych lokalnych i skalować wiele chmur przy użyciu usług Azure i Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ecc42a94e2c59531b2a2e933772b0d8ce8c58609
ms.sourcegitcommit: 0d5b5336bdb969588d0b92e04393e74b8f682c3b
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 10/22/2020
ms.locfileid: "92353482"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Wdróż aplikację hybrydową przy użyciu danych lokalnych, które skaluje się z wielu chmur

W tym przewodniku po rozwiązaniu pokazano, jak wdrożyć aplikację hybrydową, która obejmuje zarówno platformę Azure, jak i centrum Azure Stack i używa jednego lokalnego źródła danych.

Korzystając z rozwiązania w chmurze hybrydowej, można połączyć zalety chmury prywatnej z skalowalnością chmury publicznej. Deweloperzy mogą również korzystać z ekosystemu deweloperów firmy Microsoft i stosować swoje umiejętności w środowiskach chmurowych i lokalnych.

## <a name="overview-and-assumptions"></a>Przegląd i założenia

Postępuj zgodnie z tym samouczkiem, aby skonfigurować przepływ pracy, który umożliwia deweloperom wdrożenie identycznej aplikacji sieci Web w chmurze publicznej i prywatnej. Ta aplikacja może uzyskiwać dostęp do sieci z obsługą routingu bez Internetu hostowanej w chmurze prywatnej. Te aplikacje sieci Web są monitorowane i w przypadku wzrostu ruchu program modyfikuje rekordy DNS w celu przekierowania ruchu do chmury publicznej. Gdy ruch spadnie do poziomu przed skokiem, ruch jest kierowany z powrotem do chmury prywatnej.

Ten samouczek obejmuje następujące zadania:

> [!div class="checklist"]
> - Wdróż serwer bazy danych SQL Server połączony hybrydowo.
> - Połącz aplikację sieci Web na platformie Azure w sieci hybrydowej.
> - Skonfiguruj system DNS do skalowania między chmurami.
> - Skonfiguruj certyfikaty SSL na potrzeby skalowania między chmurami.
> - Skonfiguruj i Wdróż aplikację internetową.
> - Utwórz profil Traffic Manager i skonfiguruj go do skalowania między chmurami.
> - Skonfiguruj Application Insights monitorowania i generowania alertów w przypadku zwiększonego ruchu.
> - Skonfiguruj automatyczne przełączanie ruchu między globalnymi centrami Azure i Azure Stack.

> [!Tip]  
> ![Diagram filarów hybrydowych](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub to rozszerzenie platformy Azure. Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.  
> 
> [Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych. Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.

### <a name="assumptions"></a>Założenia

W tym samouczku założono, że masz podstawową wiedzę na temat globalnego systemu Azure i centrum Azure Stack. Jeśli chcesz dowiedzieć się więcej przed rozpoczęciem pracy z samouczkiem, zapoznaj się z następującymi artykułami:

- [Wprowadzenie do platformy Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Podstawowe pojęcia Azure Stack centrum](/azure-stack/operator/azure-stack-overview.md)

W tym samouczku przyjęto również założenie, że masz subskrypcję platformy Azure. Jeśli nie masz subskrypcji, przed rozpoczęciem [Utwórz bezpłatne konto](https://azure.microsoft.com/free/) .

## <a name="prerequisites"></a>Wymagania wstępne

Przed rozpoczęciem tego rozwiązania upewnij się, że spełniasz następujące wymagania:

- Azure Stack Development Kit (ASDK) lub subskrypcja zintegrowanego systemu Azure Stack centrum. Aby wdrożyć ASDK, postępuj zgodnie z instrukcjami podanymi w temacie [wdrażanie ASDK przy użyciu Instalatora](/azure-stack/asdk/asdk-install.md).
- Instalacja centrum Azure Stack powinna mieć zainstalowane następujące elementy:
  - Azure App Service. Współpracuj z operatorem Azure Stack Hub, aby wdrażać i konfigurować Azure App Service w danym środowisku. Ten samouczek wymaga, aby App Service mieć co najmniej jedną dostępną rolę procesu roboczego (1).
  - Obraz systemu Windows Server 2016.
  - Windows Server 2016 z obrazem Microsoft SQL Server.
  - Odpowiednie plany i oferty.
  - Nazwa domeny dla aplikacji sieci Web. Jeśli nie masz nazwy domeny, możesz zakupić ją od dostawcy domeny, np. GoDaddy, Bluehost i unmotion.
- Certyfikat SSL dla domeny z zaufanego urzędu certyfikacji, takiego jak LetsEncrypt.
- Aplikacja sieci Web, która komunikuje się z SQL Server bazą danych i obsługuje Application Insights. Przykładową aplikację [dotnetcore-SQLDB-samouczka](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) można pobrać z witryny GitHub.
- Sieć hybrydowa między siecią wirtualną platformy Azure a siecią wirtualną centrum Azure Stack. Aby uzyskać szczegółowe instrukcje, zobacz [Konfigurowanie łączności chmury hybrydowej za pomocą platformy Azure i centrum Azure Stack](solution-deployment-guide-connectivity.md).

- Potok hybrydowej ciągłej integracji/ciągłego wdrażania (CI/CD) z prywatnym agentem kompilacji na Azure Stack centrum. Aby uzyskać szczegółowe instrukcje, zobacz [Konfigurowanie tożsamości chmury hybrydowej przy użyciu platformy Azure i aplikacji centrum Azure Stack](solution-deployment-guide-identity.md).

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Wdrażanie serwera bazy danych połączonego hybrydowo SQL Server

1. Zaloguj się do portalu użytkowników centrum Azure Stack.

2. Na **pulpicie nawigacyjnym**wybierz pozycję **Marketplace**.

    ![Azure Stack centrum Marketplace](media/solution-deployment-guide-hybrid/image1.png)

3. W obszarze **Marketplace**wybierz pozycję **obliczenia**, a następnie wybierz pozycję **więcej**. W obszarze **więcej**wybierz **licencję bezpłatna SQL Server: SQL Server 2017 Developer w obrazie systemu Windows Server** .

    ![Wybieranie obrazu maszyny wirtualnej w portalu użytkowników Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. **Bezpłatna SQL Server Licencja: SQL Server 2017 Developer w systemie Windows Server**, wybierz pozycję **Utwórz**.

5. Na stronie **podstawowe > Skonfiguruj ustawienia podstawowe**, podaj **nazwę** maszyny wirtualnej (VM), **nazwę użytkownika** dla SQL Server sa i **hasło** dla tego skojarzenia.  Z listy rozwijanej **subskrypcja** wybierz subskrypcję, która ma zostać wdrożona. W obszarze **Grupa zasobów**Użyj **opcji wybierz istniejącą** i umieść maszynę wirtualną w tej samej grupie zasobów, co aplikacja sieci Web Centrum Azure Stack.

    ![Konfigurowanie podstawowych ustawień maszyny wirtualnej w portalu użytkownika Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. W obszarze **rozmiar**wybierz rozmiar maszyny wirtualnej. W tym samouczku zalecamy A2_Standard lub DS2_V2_Standard.

7. W obszarze **ustawienia > Skonfiguruj funkcje opcjonalne**skonfiguruj następujące ustawienia:

   - **Konto magazynu**: Utwórz nowe konto, jeśli będzie potrzebne.
   - **Sieć wirtualna**:

     > [!Important]  
     > Upewnij się, że maszyna wirtualna SQL Server jest wdrożona w tej samej sieci wirtualnej co bramy sieci VPN.

   - **Publiczny adres IP**: Użyj ustawień domyślnych.
   - **Sieciowa Grupa zabezpieczeń**: (sieciowej grupy zabezpieczeń). Utwórz nowy sieciowej grupy zabezpieczeń.
   - **Rozszerzenia i monitorowanie**: Zachowaj ustawienia domyślne.
   - **Konto magazynu diagnostyki**: Utwórz nowe konto, jeśli będzie potrzebne.
   - Wybierz **przycisk OK** , aby zapisać konfigurację.

     ![Skonfiguruj opcjonalne funkcje maszyny wirtualnej w portalu użytkowników w Azure Stack Hub](media/solution-deployment-guide-hybrid/image4.png)

8. W obszarze **ustawienia SQL Server**skonfiguruj następujące ustawienia:

   - W przypadku **połączeń SQL**wybierz opcję **publiczny (Internet)**.
   - W polu **port**pozostaw wartość domyślną **1433**.
   - W obszarze **uwierzytelnianie SQL**wybierz pozycję **Włącz**.

     > [!Note]  
     > Po włączeniu uwierzytelniania SQL należy automatycznie wypełnić przy użyciu informacji "sqladmin", które zostały skonfigurowane w **podstawie**.

   - W pozostałych ustawieniach Zachowaj ustawienia domyślne. Wybierz przycisk **OK**.

     ![Konfigurowanie ustawień SQL Server w portalu użytkowników Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. Na stronie **Podsumowanie**Przejrzyj konfigurację maszyny wirtualnej, a następnie wybierz pozycję **OK** , aby rozpocząć wdrażanie.

    ![Podsumowanie konfiguracji w portalu użytkowników w Azure Stack Hub](media/solution-deployment-guide-hybrid/image6.png)

10. Utworzenie nowej maszyny wirtualnej zajmuje trochę czasu. STAN maszyn wirtualnych można wyświetlić na **maszynach wirtualnych**.

    ![Stan maszyn wirtualnych w portalu użytkowników w Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Tworzenie aplikacji sieci Web na platformie Azure i w centrum Azure Stack

Azure App Service upraszcza uruchamianie aplikacji sieci Web i zarządzanie nią. Ponieważ Azure Stack Hub jest spójna z platformą Azure, App Service może działać w obu środowiskach. Będziesz używać App Service do hostowania aplikacji.

### <a name="create-web-apps"></a>Tworzenie aplikacji sieci Web

1. Utwórz aplikację sieci Web na platformie Azure, postępując zgodnie z instrukcjami w temacie [Zarządzanie planem App Service na platformie Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan). Upewnij się, że aplikacja sieci Web została umieszczona w tej samej subskrypcji i grupie zasobów, co w sieci hybrydowej.

2. Powtórz poprzedni krok (1) w Azure Stack centrum.

### <a name="add-route-for-azure-stack-hub"></a>Dodawanie trasy dla centrum Azure Stack

Aby umożliwić użytkownikom dostęp do aplikacji, App Service na Azure Stack Hub muszą być trasowane z publicznej sieci Internet. Jeśli centrum Azure Stack jest dostępne z Internetu, zanotuj publiczny adres IP lub adres URL dla aplikacji sieci Web Centrum Azure Stack.

Jeśli używasz ASDK, możesz [skonfigurować statyczne mapowanie NAT](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) , aby uwidocznić App Service poza środowiskiem wirtualnym.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Łączenie aplikacji sieci Web na platformie Azure z siecią hybrydową

Aby zapewnić łączność między frontonem sieci Web na platformie Azure a SQL Server bazą danych w centrum Azure Stack, aplikacja sieci Web musi być połączona z siecią hybrydową między platformą Azure i usługą Azure Stack Hub. Aby włączyć łączność, należy:

- Konfigurowanie połączenia punkt-lokacja.
- Skonfiguruj aplikację internetową.
- Zmodyfikuj bramę sieci lokalnej w centrum Azure Stack.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Konfigurowanie sieci wirtualnej platformy Azure na potrzeby połączeń punkt-lokacja

Brama sieci wirtualnej na stronie platformy Azure w sieci hybrydowej musi zezwalać na połączenia typu punkt-lokacja z usługą Azure App Service.

1. W Azure Portal przejdź do strony bramy sieci wirtualnej. W obszarze **Ustawienia**wybierz pozycję **Konfiguracja punktu do lokacji**.

    ![Opcja punkt-lokacja w bramie sieci wirtualnej platformy Azure](media/solution-deployment-guide-hybrid/image8.png)

2. Wybierz pozycję **Konfiguruj teraz** , aby skonfigurować pozycję punkt-lokacja.

    ![Konfiguracja punktu do lokacji w bramie sieci wirtualnej platformy Azure](media/solution-deployment-guide-hybrid/image9.png)

3. Na stronie Konfiguracja **punktu do lokacji** Wprowadź zakres prywatnych adresów IP, który ma być używany w **puli adresów**.

   > [!Note]  
   > Upewnij się, że określony zakres nie pokrywa się z żadnym z zakresów adresów używanych już przez podsieci w globalnych składnikach platformy Azure lub Azure Stack Centrum sieci hybrydowej.

   W obszarze **Typ tunelu**Usuń zaznaczenie **sieci VPN protokołu IKEv2**. Wybierz pozycję **Zapisz** , aby zakończyć konfigurowanie punktu do lokacji.

   ![Ustawienia punkt-lokacja w bramie sieci wirtualnej platformy Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Integrowanie aplikacji Azure App Service z siecią hybrydową

1. Aby połączyć aplikację z siecią wirtualną platformy Azure, postępuj zgodnie z instrukcjami w temacie [wymagana integracja](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)z siecią wirtualną.

2. Przejdź do pozycji **Ustawienia** dla planu App Service hostowania aplikacji sieci Web. W obszarze **Ustawienia**wybierz pozycję **Sieć**.

    ![Konfigurowanie sieci dla planu App Service](media/solution-deployment-guide-hybrid/image11.png)

3. W **integracja z siecią wirtualną**wybierz **pozycję kliknij tutaj, aby zarządzać**.

    ![Zarządzanie integracją sieci wirtualnej w ramach planu App Service](media/solution-deployment-guide-hybrid/image12.png)

4. Wybierz sieć wirtualną, którą chcesz skonfigurować. W obszarze **adresy IP kierowane do sieci wirtualnej**Wprowadź zakres adresów IP dla sieci wirtualnej platformy Azure, sieć wirtualną Azure Stack Hub i przestrzenie adresowe punkt-lokacja. Wybierz pozycję **Zapisz** , aby sprawdzić poprawność i zapisać te ustawienia.

    ![Zakresy adresów IP do rozesłania w ramach integracji Virtual Network](media/solution-deployment-guide-hybrid/image13.png)

Aby dowiedzieć się więcej o tym, jak App Service integrują się z usługą Azure sieci wirtualnych, zobacz [Integrowanie aplikacji z usługą azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Konfigurowanie sieci wirtualnej Azure Stack Hub

Bramę sieci lokalnej w sieci wirtualnej Azure Stack Hub należy skonfigurować do kierowania ruchem z zakresu adresów App Service punkt-lokacja.

1. W portalu Azure Stack Hub przejdź do pozycji **Brama sieci lokalnej**. W obszarze **Ustawienia** wybierz pozycję **Konfiguracja**.

    ![Opcja konfiguracji bramy w bramie sieci lokalnej centrum Azure Stack](media/solution-deployment-guide-hybrid/image14.png)

2. W **obszarze przestrzeń adresowa**Wprowadź zakres adresów punkt-lokacja dla bramy sieci wirtualnej na platformie Azure.

    ![Przestrzeń adresowa punkt-lokacja w bramie sieci lokalnej centrum Azure Stack](media/solution-deployment-guide-hybrid/image15.png)

3. Wybierz pozycję **Zapisz** , aby sprawdzić poprawność i zapisać konfigurację.

## <a name="configure-dns-for-cross-cloud-scaling"></a>Konfigurowanie systemu DNS do skalowania między chmurami

Po poprawnym skonfigurowaniu usługi DNS dla aplikacji w chmurze użytkownicy mogą uzyskiwać dostęp do globalnych wystąpień platformy Azure i Azure Stack w aplikacji sieci Web. Konfiguracja systemu DNS w tym samouczku umożliwia również usłudze Azure Traffic Manager kierowanie ruchu w przypadku zwiększenia lub zmniejszenia obciążenia.

Ten samouczek używa Azure DNS do zarządzania systemem DNS, ponieważ domeny App Service nie będą działały.

### <a name="create-subdomains"></a>Tworzenie poddomen

Ponieważ Traffic Manager opiera się na rekordach CNAME systemu DNS, poddomena jest wymagana do prawidłowego kierowania ruchu do punktów końcowych. Aby uzyskać więcej informacji na temat rekordów DNS i mapowania domen, zobacz [Mapowanie domen za pomocą Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).

W przypadku punktu końcowego platformy Azure utworzysz poddomenę, za pomocą której użytkownicy mogą uzyskiwać dostęp do aplikacji sieci Web. W tym samouczku można używać **App.Northwind.com**, ale należy dostosować tę wartość na podstawie własnej domeny.

Należy również utworzyć poddomenę z rekordem dla punktu końcowego Azure Stack Hub. Możesz użyć **azurestack.Northwind.com**.

### <a name="configure-a-custom-domain-in-azure"></a>Konfigurowanie domeny niestandardowej na platformie Azure

1. Dodaj nazwę hosta **App.Northwind.com** do aplikacji sieci Web platformy Azure, [mapując rekord CNAME na Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Konfigurowanie domen niestandardowych w centrum Azure Stack

1. Dodaj nazwę hosta **azurestack.Northwind.com** do aplikacji sieci web centrum Azure Stack, [mapując rekord A na Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Użyj adresu IP z obsługą routingu internetowego dla aplikacji App Service.

2. Dodaj nazwę hosta **App.Northwind.com** do aplikacji sieci web centrum Azure Stack, [mapując rekord CNAME na Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Użyj nazwy hosta skonfigurowanej w poprzednim kroku (1) jako elementu docelowego rekordu CNAME.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Konfigurowanie certyfikatów SSL na potrzeby skalowania między chmurami

Ważne jest, aby zapewnić, że poufne dane zbierane przez aplikację sieci Web są zabezpieczane podczas przesyłania do i przechowywane w bazie danych SQL.

Skonfigurujesz aplikacje sieci Web na platformie Azure i Azure Stack, aby używać certyfikatów SSL dla całego ruchu przychodzącego.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Dodawanie protokołu SSL do platformy Azure i usługi Azure Stack Hub

Aby dodać protokół SSL do platformy Azure:

1. Upewnij się, że pobrany certyfikat SSL jest prawidłowy dla utworzonej domeny podrzędnej. (Można używać certyfikatów symboli wieloznacznych).

2. W Azure Portal postępuj zgodnie z instrukcjami podanymi w sekcji **przygotowanie aplikacji sieci Web** i **POwiązaniu certyfikatu SSL** w artykule [Powiązywanie istniejącego niestandardowego certyfikatu protokołu SSL z usługą Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) . Wybierz **protokół SSL oparty na SNI** jako **Typ protokołu SSL**.

3. Przekieruj cały ruch do portu HTTPS. Postępuj zgodnie z instrukcjami w sekcji   **Wymuszanie protokołu HTTPS** w artykule [Powiązywanie istniejącego niestandardowego certyfikatu protokołu SSL z usługą Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) .

Aby dodać protokół SSL do Azure Stack Hub:

1. Powtórz kroki 1-3, które były używane w przypadku platformy Azure, przy użyciu portalu Centrum Azure Stack.

## <a name="configure-and-deploy-the-web-app"></a>Konfigurowanie i wdrażanie aplikacji internetowej

Skonfigurujesz kod aplikacji w taki sposób, aby zgłaszał dane telemetryczne do poprawnego wystąpienia Application Insights i skonfigurować aplikacje sieci Web przy użyciu właściwych parametrów połączenia. Aby dowiedzieć się więcej na temat Application Insights, zobacz [co to jest Application Insights?](/azure/application-insights/app-insights-overview)

### <a name="add-application-insights"></a>Dodaj Application Insights

1. Otwórz aplikację sieci Web w Microsoft Visual Studio.

2. [Dodaj Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) do projektu, aby przesłać dane telemetryczne, których Application Insights używa do tworzenia alertów w przypadku wzrostu lub zmniejszenia ruchu w sieci Web.

### <a name="configure-dynamic-connection-strings"></a>Konfigurowanie dynamicznych parametrów połączenia

Każde wystąpienie aplikacji sieci Web będzie używać innej metody do łączenia się z bazą danych SQL. Aplikacja na platformie Azure używa prywatnego adresu IP maszyny wirtualnej SQL Server, a aplikacja w centrum Azure Stack używa publicznego adresu IP SQL Server maszyny wirtualnej.

> [!Note]  
> W systemie zintegrowanym z centrum Azure Stack publiczny adres IP nie powinien być obsługiwany przez Internet. Na ASDK publiczny adres IP nie jest w trakcie routingu poza ASDK.

Za pomocą zmiennych środowiskowych App Service można przekazać inne parametry połączenia do każdego wystąpienia aplikacji.

1. Otwórz aplikację w programie Visual Studio.

2. Otwórz Startup.cs i znajdź następujący blok kodu:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Zastąp poprzedni blok kodu poniższym kodem, który używa parametrów połączenia zdefiniowanych w *appsettings.jsw* pliku:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>Konfigurowanie ustawień aplikacji App Service

1. Utwórz parametry połączenia dla systemu Azure i centrum Azure Stack. Ciągi powinny być takie same, z wyjątkiem adresów IP, które są używane.

2. Na platformie Azure i Azure Stack Hub Dodaj odpowiednie parametry połączenia [jako ustawienia aplikacji](/azure/app-service/web-sites-configure) w aplikacji sieci Web, używając `SQLCONNSTR\_` jako prefiksu w nazwie.

3. **Zapisz** ustawienia aplikacji sieci Web i ponownie uruchom aplikację.

## <a name="enable-automatic-scaling-in-global-azure"></a>Włącz automatyczne skalowanie na globalnym platformie Azure

Gdy tworzysz aplikację sieci Web w środowisku App Service, rozpocznie się ono z jednym wystąpieniem. Możesz automatycznie skalować w poziomie, aby dodać wystąpienia w celu zapewnienia większej ilości zasobów obliczeniowych dla aplikacji. Podobnie można automatycznie skalować i zmniejszać liczbę wystąpień potrzebnych aplikacji.

> [!Note]  
> Musisz mieć App Service plan, aby skonfigurować skalowanie w poziomie i w poziomie. Jeśli nie masz planu, utwórz go przed rozpoczęciem następnych kroków.

### <a name="enable-automatic-scale-out"></a>Włącz automatyczne skalowanie w poziomie

1. W Azure Portal Znajdź plan App Service dla witryn, które mają być skalowane w poziomie, a następnie wybierz pozycję **skalowanie w poziomie (plan App Service)**.

    ![Skalowanie w poziomie Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. Wybierz pozycję **Włącz automatyczne skalowanie**.

    ![Włącz automatyczne skalowanie w Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. Wprowadź nazwę **nazwy ustawienia skalowania automatycznego**. Dla **domyślnej** reguły automatycznego skalowania wybierz pozycję **skalowanie na podstawie metryki**. Ustaw **limity wystąpienia** na wartość **minimum: 1**, **maksimum: 10**, a **wartość domyślna: 1**.

    ![Konfigurowanie automatycznego skalowania w Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. Wybierz pozycję **+ Dodaj regułę**.

5. W polu **Źródło metryk**wybierz pozycję **bieżący zasób**. Użyj następujących kryteriów i akcji dla reguły.

#### <a name="criteria"></a>Kryteria

1. W obszarze **agregacja czasu** wybierz pozycję **średnia**.

2. W obszarze **Nazwa metryki**wybierz opcję **procent procesora CPU**.

3. W obszarze **operator**wybierz pozycję **większe niż**.

   - Ustaw **próg** na **50**.
   - Ustaw **czas trwania** na **10**.

#### <a name="action"></a>Akcja

1. W obszarze **operacja**wybierz pozycję **Zwiększ liczbę według**.

2. Ustaw **liczbę wystąpień** na **2**.

3. Ustaw **chłodną** wartość na **5**.

4. Wybierz pozycję **Dodaj**.

5. Wybierz pozycję **+ Dodaj regułę**.

6. W polu **Źródło metryk**wybierz pozycję **bieżący zasób.**

   > [!Note]  
   > Bieżący zasób będzie zawierać nazwę/identyfikator GUID planu App Service i lista rozwijana **Typ zasobu** i **zasób** będzie niedostępna.

### <a name="enable-automatic-scale-in"></a>Włącz automatyczne skalowanie w

Po zmniejszeniu natężenia ruchu aplikacja internetowa platformy Azure może automatycznie zmniejszyć liczbę aktywnych wystąpień, aby obniżyć koszty. Ta akcja jest mniej agresywna niż skalowanie w poziomie i minimalizuje wpływ na użytkowników aplikacji.

1. Przejdź do **domyślnego** warunku skalowania w poziomie, a następnie wybierz pozycję **+ Dodaj regułę**. Użyj następujących kryteriów i akcji dla reguły.

#### <a name="criteria"></a>Kryteria

1. W obszarze **agregacja czasu** wybierz pozycję **średnia**.

2. W obszarze **Nazwa metryki**wybierz opcję **procent procesora CPU**.

3. W obszarze **operator**wybierz pozycję **mniejsze niż**.

   - Ustaw **próg** na wartość **30**.
   - Ustaw **czas trwania** na **10**.

#### <a name="action"></a>Akcja

1. W obszarze **operacja**wybierz pozycję **Zmniejsz liczbę według**.

   - Ustaw **liczbę wystąpień** na **1**.
   - Ustaw **chłodną** wartość na **5**.

2. Wybierz pozycję **Dodaj**.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Utwórz profil Traffic Manager i skonfiguruj skalowanie między chmurami

Utwórz profil Traffic Manager przy użyciu Azure Portal, a następnie skonfiguruj punkty końcowe, aby umożliwić skalowanie między chmurami.

### <a name="create-traffic-manager-profile"></a>Utwórz profil Traffic Manager

1. Wybierz pozycję **Utwórz zasób**.
2. Wybierz pozycję **Sieć**.
3. Wybierz **profil Traffic Manager** i skonfiguruj następujące ustawienia:

   - W polu **Nazwa**wprowadź nazwę profilu. Ta nazwa **musi** być unikatowa w strefie trafficmanager.NET i służy do tworzenia nowej nazwy DNS (na przykład northwindstore.trafficmanager.NET).
   - W polu **Metoda routingu**wybierz opcję **ważone**.
   - W obszarze **subskrypcja**wybierz subskrypcję, w której chcesz utworzyć ten profil.
   - W obszarze **Grupa zasobów**Utwórz nową grupę zasobów dla tego profilu.
   - W obszarze **Lokalizacja grupy zasobów** wybierz lokalizację grupy zasobów. To ustawienie dotyczy lokalizacji grupy zasobów i nie ma wpływu na profil Traffic Manager, który został wdrożony globalnie.

4. Wybierz pozycję **Utwórz**.

    ![Utwórz profil Traffic Manager](media/solution-deployment-guide-hybrid/image19.png)

   Po ukończeniu globalnego wdrożenia profilu Traffic Manager zostanie on wyświetlony na liście zasobów dla grupy zasobów, w której został utworzony.

### <a name="add-traffic-manager-endpoints"></a>Dodawanie punktów końcowych usługi Traffic Manager

1. Wyszukaj utworzony profil Traffic Manager. Jeśli przejdziesz do grupy zasobów profilu, wybierz profil.

2. W oknie **profil Traffic Manager**w obszarze **Ustawienia**wybierz pozycję **punkty końcowe**.

3. Wybierz pozycję **Dodaj**.

4. W obszarze **Dodaj punkt końcowy**Użyj następujących ustawień centrum Azure Stack:

   - W obszarze **Typ**wybierz pozycję **zewnętrzny punkt końcowy**.
   - Wprowadź **nazwę** punktu końcowego.
   - W polu w **pełni kwalifikowana nazwa domeny (FQDN) lub adres IP**wprowadź zewnętrzny adres URL aplikacji sieci Web Azure Stack Hub.
   - W polu **waga**pozostaw wartość domyślną **1**. Ta waga powoduje, że cały ruch przechodzi do tego punktu końcowego, jeśli jest w dobrej kondycji.
   - Pozostaw pole wyboru **Dodaj jako wyłączone wyłączona** .

5. Wybierz **przycisk OK** , aby zapisać punkt końcowy Azure Stack centrum.

Następnie skonfigurujesz punkt końcowy platformy Azure.

1. W obszarze **profil Traffic Manager**wybierz pozycję **punkty końcowe**.
2. Wybierz pozycję **+Dodaj**.
3. Na stronie **Dodawanie punktu końcowego**Użyj następujących ustawień platformy Azure:

   - W obszarze **Typ**wybierz pozycję **punkt końcowy platformy Azure**.
   - Wprowadź **nazwę** punktu końcowego.
   - W obszarze **Typ zasobu docelowego**wybierz pozycję **App Service**.
   - W polu **zasób docelowy**wybierz pozycję **Wybierz usługę App Service** , aby wyświetlić listę Web Apps w tej samej subskrypcji.
   - W obszarze **Zasoby** wybierz usługę aplikacji, którą chcesz dodać jako pierwszy punkt końcowy.
   - W obszarze **waga**wybierz pozycję **2**. To ustawienie powoduje, że cały ruch przechodzi do tego punktu końcowego, jeśli podstawowy punkt końcowy ma złą kondycję lub jeśli masz regułę/alert przekierowuje ruch po wyzwoleniu.
   - Pozostaw pole wyboru **Dodaj jako wyłączone wyłączona** .

4. Wybierz **przycisk OK** , aby zapisać punkt końcowy platformy Azure.

Po skonfigurowaniu obu punktów końcowych są one wyświetlane w **Traffic Manager profilu** w przypadku wybrania **punktów końcowych**. W przykładzie na poniższym zrzucie ekranu przedstawiono dwa punkty końcowe, z informacjami o stanie i konfiguracją każdego z nich.

![Punkty końcowe w profilu Traffic Manager](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a>Konfigurowanie Application Insights monitorowania i alertów na platformie Azure

Usługa Azure Application Insights umożliwia monitorowanie aplikacji i wysyłanie alertów na podstawie skonfigurowanych warunków. Oto kilka przykładów: aplikacja jest niedostępna, występuje błędy lub pokazuje problemy z wydajnością.

Do tworzenia alertów służą metryki usługi Azure Application Insights. Gdy te alerty wyzwalają, wystąpienie aplikacji sieci Web automatycznie przejdzie z centrum Azure Stack na platformę Azure w celu skalowania w poziomie, a następnie z powrotem do centrum Azure Stack w celu skalowania w poziomie.

### <a name="create-an-alert-from-metrics"></a>Tworzenie alertu na podstawie metryki

W Azure Portal przejdź do grupy zasobów w tym samouczku i wybierz wystąpienie Application Insights, aby otworzyć **Application Insights**.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

Ten widok służy do tworzenia alertu skalowania w poziomie i alertu dotyczącego skalowania w poziomie.

### <a name="create-the-scale-out-alert"></a>Tworzenie alertu skalowania w poziomie

1. W obszarze **Konfiguracja**wybierz pozycję **alerty (klasyczne)**.
2. Wybierz pozycję **Dodaj alert dotyczący metryki (klasyczny)**.
3. W obszarze **Dodaj regułę**skonfiguruj następujące ustawienia:

   - W obszarze **Nazwa**wprowadź polecenie przebicie **na chmurę Azure**.
   - **Opis** jest opcjonalny.
   - W obszarze alert **źródłowy**  >  **na**wybierz pozycję **metryki**.
   - W obszarze **kryteria**wybierz swoją subskrypcję, grupę zasobów dla profilu Traffic Manager i nazwę profilu Traffic Manager dla zasobu.

4. W obszarze **Metryka**wybierz pozycję **Liczba żądań**.
5. W obszarze **warunek**wybierz opcję **większe niż**.
6. W obszarze **próg wprowadź wartość** **2**.
7. W polu **okres**wybierz pozycję **w ciągu ostatnich 5 minut**.
8. W obszarze **Powiadamiaj za pośrednictwem**:
   - Zaznacz pole wyboru dla **właścicieli, współautorów i czytelników poczty e-mail**.
   - Wprowadź swój adres e-mail, aby uzyskać **dodatkowe adresy e-mail administratora**.

9. Na pasku menu wybierz pozycję **Zapisz**.

### <a name="create-the-scale-in-alert"></a>Tworzenie alertu dotyczącego skalowania w poziomie

1. W obszarze **Konfiguracja**wybierz pozycję **alerty (klasyczne)**.
2. Wybierz pozycję **Dodaj alert dotyczący metryki (klasyczny)**.
3. W obszarze **Dodaj regułę**skonfiguruj następujące ustawienia:

   - W obszarze **Nazwa**wprowadź **skalowanie z powrotem do Azure Stack Hub**.
   - **Opis** jest opcjonalny.
   - W obszarze alert **źródłowy**  >  **na**wybierz pozycję **metryki**.
   - W obszarze **kryteria**wybierz swoją subskrypcję, grupę zasobów dla profilu Traffic Manager i nazwę profilu Traffic Manager dla zasobu.

4. W obszarze **Metryka**wybierz pozycję **Liczba żądań**.
5. W obszarze **warunek**wybierz pozycję **mniejsze niż**.
6. W obszarze **próg wprowadź wartość** **2**.
7. W polu **okres**wybierz pozycję **w ciągu ostatnich 5 minut**.
8. W obszarze **Powiadamiaj za pośrednictwem**:
   - Zaznacz pole wyboru dla **właścicieli, współautorów i czytelników poczty e-mail**.
   - Wprowadź swój adres e-mail, aby uzyskać **dodatkowe adresy e-mail administratora**.

9. Na pasku menu wybierz pozycję **Zapisz**.

Poniższy zrzut ekranu przedstawia alerty dotyczące skalowania w poziomie i skalowania w poziomie.

   ![Alerty Application Insights (klasyczne)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Przekierowywanie ruchu między platformą Azure i usługą Azure Stack Hub

Można skonfigurować ręczne lub automatyczne przełączanie ruchu aplikacji sieci Web między platformą Azure i Azure Stack centrum.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Konfigurowanie ręcznego przełączania między platformą Azure i usługą Azure Stack Hub

Gdy witryna sieci Web osiągnie skonfigurowane progi, otrzymasz alert. Wykonaj następujące kroki, aby ręcznie przekierować ruch do platformy Azure.

1. W Azure Portal wybierz profil Traffic Manager.

    ![Traffic Manager punkty końcowe w Azure Portal](media/solution-deployment-guide-hybrid/image20.png)

2. Wybierz **punkty końcowe**.
3. Wybierz **punkt końcowy platformy Azure**.
4. W obszarze **stan**wybierz pozycję **włączone**, a następnie wybierz pozycję **Zapisz**.

    ![Włącz punkt końcowy platformy Azure w Azure Portal](media/solution-deployment-guide-hybrid/image23.png)

5. W obszarze **punkty końcowe** dla profilu Traffic Manager wybierz pozycję **zewnętrzny punkt końcowy**.
6. W obszarze **stan**wybierz pozycję **wyłączone**, a następnie wybierz pozycję **Zapisz**.

    ![Wyłącz punkt końcowy Azure Stack Hub w programie Azure Portal](media/solution-deployment-guide-hybrid/image24.png)

Po skonfigurowaniu punktów końcowych ruch aplikacji przechodzi do aplikacji sieci Web platformy Azure w poziomie, a nie aplikacji sieci Web Centrum Azure Stack.

 ![Punkty końcowe zostały zmienione w ruchu aplikacji sieci Web platformy Azure](media/solution-deployment-guide-hybrid/image25.png)

Aby odwrócić przepływ z powrotem do Azure Stack Hub, wykonaj poprzednie kroki, aby:

- Włącz punkt końcowy Azure Stack Hub.
- Wyłącz punkt końcowy platformy Azure.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Konfigurowanie automatycznego przełączania między platformą Azure i usługą Azure Stack Hub

Można również użyć monitorowania Application Insights, jeśli aplikacja działa w środowisku [bezserwerowym](https://azure.microsoft.com/overview/serverless-computing/) udostępnianym przez Azure Functions.

W tym scenariuszu można skonfigurować Application Insights do używania elementu webhook, który wywołuje aplikację funkcji. Ta aplikacja automatycznie włącza lub wyłącza punkt końcowy w odpowiedzi na alert.

Poniższe kroki służą do konfigurowania automatycznego przełączania ruchu.

1. Tworzenie aplikacji funkcji platformy Azure.
2. Utwórz funkcję wyzwalaną przez protokół HTTP.
3. Zaimportuj zestawy SDK platformy Azure dla Menedżer zasobów, Web Apps i Traffic Manager.
4. Rozwijaj kod, aby:

   - Uwierzytelnianie w ramach subskrypcji platformy Azure.
   - Użyj parametru, który przełącza Traffic Manager punkty końcowe, aby skierować ruch do platformy Azure lub Azure Stack Hub.

5. Zapisz swój kod i Dodaj adres URL aplikacji funkcji z odpowiednimi parametrami do sekcji **elementu webhook** w ustawieniach reguły alertu Application Insights.
6. Ruch jest automatycznie przekierowywany po uruchomieniu alertu Application Insights.

## <a name="next-steps"></a>Następne kroki

- Aby dowiedzieć się więcej o wzorcach chmury platformy Azure, zobacz [wzorce projektowe w chmurze](/azure/architecture/patterns).
