---
title: Konfigurowanie łączności w chmurze hybrydowej na platformie Azure i w centrum Azure Stack
description: Dowiedz się, jak skonfigurować łączność z chmurą hybrydową przy użyciu platformy Azure i usługi Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 16c5d7820e8c865a9f88cb00da5cc7c854379414
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477290"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Konfigurowanie łączności w chmurze hybrydowej przy użyciu platformy Azure i usługi Azure Stack Hub

Dostęp do zasobów można uzyskać, korzystając ze wzorca łączności hybrydowej na platformie Azure i w centrum Azure Stack.

W tym rozwiązaniu utworzysz przykładowe środowisko w celu:

> [!div class="checklist"]
> - Utrzymywanie lokalnych danych w celu spełnienia wymagań dotyczących ochrony prywatności lub przepisów, ale utrzymywanie dostępu do globalnych zasobów platformy Azure.
> - Obsługa starszego systemu podczas korzystania ze skalowania aplikacji i zasobów w skali chmury na globalnym platformie Azure.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub to rozszerzenie platformy Azure. Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.  
> 
> [Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych. Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.

## <a name="prerequisites"></a>Wymagania wstępne

Do skompilowania wdrożenia połączenia hybrydowego wymagane są kilka składników. Niektóre z tych składników pobierają czas przygotowania, dlatego należy odpowiednio zaplanować.

### <a name="azure"></a>Azure

- Jeśli nie masz subskrypcji platformy Azure, przed rozpoczęciem utwórz [bezpłatne konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Utwórz [aplikację internetową](/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?tabs=vsts&view=vsts) na platformie Azure. Zanotuj adres URL aplikacji sieci Web, ponieważ będzie on potrzebny w rozwiązaniu.

### <a name="azure-stack-hub"></a>Azure Stack Hub

Partner OEM/producent sprzętu może wdrożyć centrum Azure Stack produkcyjnego, a wszyscy użytkownicy mogą wdrożyć Azure Stack Development Kit (ASDK).

- Użyj centrum produkcyjnego Azure Stack lub Wdróż ASDK.
   >[!Note]
   >Wdrożenie ASDK może potrwać do 7 godzin, dlatego należy odpowiednio zaplanować.

- Wdróż [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) usługi PaaS Services w usłudze Azure Stack Hub.
- [Utwórz plany i oferty](/azure-stack/operator/service-plan-offer-subscription-overview.md) w środowisku centrum Azure Stack.
- [Utwórz subskrypcję dzierżawy](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) w ramach środowiska Azure Stack Hub.

### <a name="azure-stack-hub-components"></a>Składniki Azure Stack Hub

Azure Stack operator centrum musi wdrożyć App Service, utworzyć plany i oferty, utworzyć subskrypcję dzierżawy oraz dodać obraz systemu Windows Server 2016. Jeśli masz już te składniki, przed rozpoczęciem tego rozwiązania upewnij się, że spełniają one wymagania.

W tym przykładowym rozwiązaniu założono, że masz pewną podstawową wiedzę na temat platformy Azure i centrum Azure Stack. Aby dowiedzieć się więcej przed rozpoczęciem rozwiązania, przeczytaj następujące artykuły:

- [Wprowadzenie do platformy Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Podstawowe pojęcia Azure Stack centrum](/azure-stack/operator/azure-stack-overview.md)

### <a name="before-you-begin"></a>Przed rozpoczęciem

Przed rozpoczęciem konfigurowania łączności w chmurze hybrydowej Sprawdź, czy zostały spełnione następujące kryteria:

- Potrzebny jest zewnętrzny adres IPv4 urządzenia sieci VPN. Ten adres IP nie może znajdować się za translatorem adresów sieciowych (NAT).
- Wszystkie zasoby są wdrażane w tym samym regionie/lokalizacji.

#### <a name="solution-example-values"></a>Przykładowe wartości rozwiązania

W przykładach w tym rozwiązaniu użyto następujących wartości. Możesz użyć tych wartości do utworzenia środowiska testowego lub odwoływać się do nich, aby lepiej zrozumieć przykłady. Aby uzyskać więcej informacji na temat ustawień bramy sieci VPN, zobacz [Informacje o ustawieniach VPN Gateway](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).

Specyfikacje połączenia:

- **Typ sieci VPN**: oparta na trasach
- **Typ połączenia**: lokacja-lokacja (IPSec)
- **Typ bramy**: VPN
- **Nazwa połączenia platformy Azure**: Azure-Gateway-AzureStack-S2SGateway (Portal spowoduje wypełnienie tej wartości)
- **Nazwa połączenia centrum Azure Stack**: AzureStack-Gateway-Azure-S2SGateway (Portal spowoduje wypełnienie tej wartości)
- **Klucz współużytkowany**: wszystkie zgodne z sprzętem sieci VPN, ze zgodnymi wartościami po obu stronach połączenia
- **Subskrypcja**: dowolna preferowana subskrypcja
- **Grupa zasobów**: test-infrastruktura

Adresy IP sieci i podsieci:

| Połączenie z usługą Azure/Azure Stack Hub | Nazwa | Podsieć | Adres IP |
|---|---|---|---|
| Sieć wirtualna platformy Azure | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| Sieć wirtualna Azure Stack Hub | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Brama usługi Azure Virtual Network | Azure-Gateway |  |  |
| Brama Virtual Network Azure Stack Hub | AzureStack — Brama |  |  |
| Publiczny adres IP platformy Azure | Azure — GatewayPublicIP |  | Określone podczas tworzenia |
| Publiczny adres IP centrum Azure Stack | AzureStack — GatewayPublicIP |  | Określone podczas tworzenia |
| Brama sieci lokalnej platformy Azure | AzureStack — S2SGateway<br>   10.100.100.0/23 |  | Wartość publicznego adresu IP centrum Azure Stack |
| Brama sieci lokalnej centrum Azure Stack | Azure — S2SGateway<br>10.100.102.0/23 |  | Wartość publicznego adresu IP platformy Azure |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>Tworzenie sieci wirtualnej na platformie Azure i w centrum Azure Stack

Wykonaj następujące kroki, aby utworzyć sieć wirtualną przy użyciu portalu. Tych [przykładowych wartości](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) można użyć, jeśli ten artykuł jest używany tylko jako rozwiązanie. Jeśli ten artykuł jest używany do konfigurowania środowiska produkcyjnego, Zastąp Przykładowe ustawienia własnymi wartościami.

> [!IMPORTANT]
> Musisz się upewnić, że nie ma nakładania się adresów IP na platformie Azure ani w przestrzeni adresowej sieci wirtualnej centrum Azure Stack Hub.

Aby utworzyć sieć wirtualną na platformie Azure:

1. Użyj przeglądarki, aby nawiązać połączenie z [Azure Portal](https://portal.azure.com/) i zalogować się przy użyciu konta platformy Azure.
2. Wybierz pozycję **Utwórz zasób**. W polu **Wyszukaj w witrynie Marketplace** wpisz "Sieć wirtualna". Z wyników wybierz pozycję **Sieć wirtualna** .
3. Z listy **Wybierz model wdrożenia** wybierz pozycję **Menedżer zasobów**, a następnie wybierz pozycję **Utwórz**.
4. W obszarze **Utwórz sieć wirtualną**Skonfiguruj ustawienia sieci wirtualnej. Nazwy wymaganych pól są poprzedzone czerwoną gwiazdką.  Po wprowadzeniu prawidłowej wartości gwiazdka zmieni się na zielony znacznik wyboru.

Aby utworzyć sieć wirtualną w centrum Azure Stack:

1. Powtórz powyższe kroki (1-4) przy użyciu **portalu dzierżawy**centrum Azure Stack.

## <a name="add-a-gateway-subnet"></a>Dodawanie podsieci bramy

Przed połączeniem sieci wirtualnej z bramą należy utworzyć podsieć bramy dla sieci wirtualnej, z którą chcesz nawiązać połączenie. Usługi bramy używają adresów IP określonych w podsieci bramy.

W [Azure Portal](https://portal.azure.com/)przejdź do Menedżer zasobów sieci wirtualnej, w której chcesz utworzyć bramę sieci wirtualnej.

1. Wybierz sieć wirtualną, aby otworzyć stronę **sieci wirtualnej** .
2. W obszarze **Ustawienia**wybierz pozycję **podsieci**.
3. Na stronie **podsieci** wybierz pozycję **+ podsieć bramy** , aby otworzyć stronę **Dodaj podsieć** .

    ![Dodaj podsieć bramy](media/solution-deployment-guide-connectivity/image4.png)

4. **Nazwa** podsieci jest wypełniana automatycznie wartością "GatewaySubnet". Ta wartość jest wymagana, aby platforma Azure mogła rozpoznać podsieć jako podsieć bramy.
5. Zmień wartości **zakresu adresów** podane w celu dopasowania do wymagań konfiguracji, a następnie wybierz przycisk **OK**.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Utwórz bramę Virtual Network na platformie Azure i Azure Stack

Wykonaj następujące kroki, aby utworzyć bramę sieci wirtualnej na platformie Azure.

1. W lewej części strony portalu wybierz **+** i wprowadź "Brama sieci wirtualnej" w polu wyszukiwania.
2. W obszarze **wyniki**wybierz pozycję **Brama sieci wirtualnej**.
3. W obszarze **Brama sieci wirtualnej**wybierz pozycję **Utwórz** , aby otworzyć stronę **Tworzenie bramy sieci wirtualnej** .
4. Na stronie **Tworzenie bramy sieci wirtualnej**Określ wartości dla bramy sieci za pomocą naszego **przykładowych wartości samouczka**. Uwzględnij następujące wartości dodatkowe:

   - **Jednostka SKU**: podstawowa
   - **Virtual Network**: Wybierz utworzoną wcześniej sieć wirtualną. Utworzona podsieć bramy zostanie automatycznie wybrana.
   - **Pierwsza konfiguracja protokołu IP**: publiczny adres IP bramy.
     - Wybierz pozycję **Utwórz konfigurację adresu IP bramy**, która spowoduje przejście do strony **Wybierz publiczny adres IP** .
     - Wybierz pozycję **+ Utwórz nowy** , aby otworzyć stronę **Tworzenie publicznego adresu IP** .
     - Wprowadź **nazwę** publicznego adresu IP. Pozostaw jednostkę SKU jako **podstawową**, a następnie wybierz pozycję **OK** , aby zapisać zmiany.

       > [!Note]
       > Obecnie VPN Gateway obsługuje tylko dynamiczną alokację publicznego adresu IP. Nie oznacza to jednak, że adres IP zmienia się po przypisaniu go do bramy sieci VPN. Jedyną zmianą publicznego adresu IP jest usunięcie bramy i jej ponowne utworzenie. Zmiana rozmiarów, Resetowanie lub przeprowadzenie wewnętrznej konserwacji/uaktualnień do bramy sieci VPN nie powoduje zmiany adresu IP.

5. Sprawdź ustawienia bramy.
6. Wybierz pozycję **Utwórz** , aby utworzyć bramę sieci VPN. Ustawienia bramy są weryfikowane i na pulpicie nawigacyjnym jest wyświetlany kafelek "wdrażanie bramy sieci wirtualnej".

   >[!Note]
   >Tworzenie bramy może potrwać do 45 minut. Być może będzie trzeba odświeżyć stronę portalu, aby zobaczyć, czy tworzenie zostało ukończone.

    Po utworzeniu bramy można zobaczyć przypisany do niej adres IP, przeglądając sieć wirtualną w portalu. Brama jest widoczna jako urządzenie podłączone. Aby wyświetlić więcej informacji na temat bramy, wybierz urządzenie.

7. Powtórz poprzednie kroki (1-5) w ramach wdrożenia centrum Azure Stack.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Tworzenie bramy sieci lokalnej na platformie Azure i w centrum Azure Stack

Brama sieci lokalnej zazwyczaj odwołuje się do lokalizacji lokalnej. Należy podać nazwę witryny, do której może odwoływać się usługa Azure lub Azure Stack Hub, a następnie określić:

- Adres IP lokalnego urządzenia sieci VPN, dla którego tworzysz połączenie.
- Prefiksy adresów IP, które będą kierowane za pośrednictwem bramy sieci VPN do urządzenia sieci VPN. Określone prefiksy adresów są prefiksami znajdującymi się w Twojej sieci lokalnej.

  >[!Note]
  >Jeśli sieć lokalna ulegnie zmianie lub musisz zmienić publiczny adres IP dla urządzenia sieci VPN, możesz później zaktualizować te wartości.

1. W portalu wybierz pozycję **+ Utwórz zasób**.
2. W polu wyszukiwania wprowadź ciąg **Brama sieci lokalnej**, a następnie wybierz **klawisz ENTER** , aby wyszukać. Zostanie wyświetlona lista wyników.
3. Wybierz pozycję **Brama sieci lokalnej**, a następnie wybierz pozycję **Utwórz** , aby otworzyć stronę **Tworzenie bramy sieci lokalnej** .
4. Na stronie **Tworzenie bramy sieci lokalnej**Określ wartości dla bramy sieci lokalnej przy użyciu naszego **przykładowych wartości samouczka**. Uwzględnij następujące wartości dodatkowe:

    - **Adres IP**: publiczny adres IP urządzenia sieci VPN, z którym ma zostać nawiązane połączenie z platformą Azure lub Azure Stack. Określ prawidłowy publiczny adres IP, który nie znajduje się za translatorem adresów sieciowych, aby platforma Azure mogła uzyskać dostęp do tego adresu. Jeśli nie masz teraz adresu IP, możesz użyć wartości z przykładu jako symbolu zastępczego. Musisz wrócić i zastąpić symbol zastępczy publicznym adresem IP urządzenia sieci VPN. Platforma Azure nie może nawiązać połączenia z urządzeniem do momentu podania prawidłowego adresu.
    - **Przestrzeń adresowa**: zakres adresów dla sieci, którą reprezentuje ta sieć lokalna. Można dodać wiele zakresów przestrzeni adresów. Upewnij się, że określone zakresy nie pokrywają się z zakresami innych sieci, z którymi chcesz nawiązać połączenie. Platforma Azure będzie kierować określony zakres adresów pod adres IP lokalnego urządzenia sieci VPN. Użyj własnych wartości, jeśli chcesz nawiązać połączenie z lokacją lokalną, a nie z przykładową wartością.
    - **Skonfiguruj ustawienia protokołu BGP**: Użyj tylko podczas KONFIGUROWANIA protokołu BGP. W przeciwnym razie nie zaznaczaj tej opcji.
    - **Subskrypcja**: Sprawdź, czy wyświetlana jest prawidłowa subskrypcja.
    - **Grupa zasobów**: Wybierz grupę zasobów, której chcesz użyć. Można utworzyć nową grupę zasobów lub wybrać już utworzoną.
    - **Lokalizacja**: Wybierz lokalizację, w której zostanie utworzony ten obiekt. Możesz wybrać tę samą lokalizację, w której znajduje się sieć wirtualna, ale nie jest to konieczne.
5. Po zakończeniu określania wymaganych wartości wybierz pozycję **Utwórz** , aby utworzyć bramę sieci lokalnej.
6. Powtórz te kroki (1-5) w ramach wdrożenia centrum Azure Stack.

## <a name="configure-your-connection"></a>Skonfiguruj połączenie

Połączenia typu lokacja-lokacja z siecią lokalną wymagają urządzenia sieci VPN. Konfigurowane urządzenie sieci VPN nazywa się połączeniem. Aby skonfigurować połączenie, potrzebne są:

- Klucz współużytkowany. Ten klucz jest tym samym kluczem współdzielonym, który jest określany podczas tworzenia połączenia sieci VPN typu lokacja-lokacja. W naszych przykładach używamy podstawowego klucza współużytkowanego. Zalecamy, aby do użycia wygenerować bardziej złożony klucz.
- Publiczny adres IP bramy sieci wirtualnej. Publiczny adres IP można wyświetlić za pomocą witryny Azure Portal, programu PowerShell lub interfejsu wiersza polecenia. Aby znaleźć publiczny adres IP bramy sieci VPN przy użyciu Azure Portal, przejdź do pozycji bramy sieci wirtualnej, a następnie wybierz nazwę bramy.

Wykonaj następujące kroki, aby utworzyć połączenie sieci VPN typu lokacja-lokacja między bramą sieci wirtualnej i lokalnym urządzeniem sieci VPN.

1. W Azure Portal wybierz pozycję **+ Utwórz zasób**.
2. Wyszukaj **połączenia**.
3. W obszarze **wyniki**wybierz pozycję **połączenia**.
4. W obszarze **połączenie**wybierz pozycję **Utwórz**.
5. Na stronie **Tworzenie połączenia**skonfiguruj następujące ustawienia:

    - **Typ połączenia**: wybierz pozycję lokacja-lokacja (IPSec).
    - **Grupa zasobów**: Wybierz testową grupę zasobów.
    - **Virtual Network Gateway**: Wybierz utworzoną bramę sieci wirtualnej.
    - **Brama sieci lokalnej**: Wybierz utworzoną bramę sieci lokalnej.
    - **Nazwa połączenia**: Ta nazwa jest automatycznie wypełniana przy użyciu wartości z dwóch bram.
    - **Klucz współużytkowany**: Ta wartość musi być zgodna z wartością używaną dla lokalnego urządzenia sieci VPN. W przykładzie użyto elementu "abc123", ale należy użyć bardziej złożonej. Ważną kwestią jest to, że ta wartość *musi* być taka sama jak wartość określona podczas konfigurowania urządzenia sieci VPN.
    - Wartości dla **subskrypcji**, **grupy zasobów**i **lokalizacji** są rozwiązane.

6. Wybierz **przycisk OK** , aby utworzyć połączenie.

Połączenie można zobaczyć na stronie **połączenia** bramy sieci wirtualnej. Stan przejdzie z *nieznanego* na *połączenie*, a następnie *zakończyło się pomyślnie*.

## <a name="next-steps"></a>Następne kroki

- Aby dowiedzieć się więcej o wzorcach chmury platformy Azure, zobacz [wzorce projektowe w chmurze](/azure/architecture/patterns).
