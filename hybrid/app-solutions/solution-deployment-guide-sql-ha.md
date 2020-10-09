---
title: Wdróż grupę dostępności SQL Server 2016 na platformie Azure i w centrum Azure Stack
description: Dowiedz się, jak wdrożyć grupę dostępności SQL Server 2016 na platformie Azure i w centrum Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852477"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a>Wdróż grupę dostępności SQL Server 2016 na platformie Azure i w centrum Azure Stack

Ten artykuł przeprowadzi Cię przez automatyczne wdrożenie podstawowego klastra o wysokiej dostępności (HA) SQL Server 2016 przedsiębiorstwa z asynchroniczną lokacją odzyskiwania po awarii w dwóch środowiskach Azure Stack Hub. Aby dowiedzieć się więcej o SQL Server 2016 i wysokiej dostępności, zobacz [zawsze włączone grupy dostępności: rozwiązanie wysokiej dostępności i odzyskiwania po awarii](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).

W tym rozwiązaniu utworzysz przykładowe środowisko w celu:

> [!div class="checklist"]
> - Organizuj wdrożenie w dwóch centrach Azure Stack.
> - Użyj platformy Docker, aby zminimalizować problemy zależności z profilami interfejsu API platformy Azure.
> - Wdróż klaster Enterprise o wysokiej dostępności SQL Server 2016 przedsiębiorstwa z lokacją odzyskiwania po awarii.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub to rozszerzenie platformy Azure. Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.  
> 
> [Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych. Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.

## <a name="architecture-for-sql-server-2016"></a>Architektura dla SQL Server 2016

![Centrum Azure Stack SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a>Wymagania wstępne dotyczące SQL Server 2016

- Dwa połączone systemy Azure Stack Hub zintegrowane (Azure Stack Hub). To wdrożenie nie działa na Azure Stack Development Kit (ASDK). Aby dowiedzieć się więcej o centrum Azure Stack, zobacz [omówienie Azure Stack](https://azure.microsoft.com/overview/azure-stack/).
- Subskrypcja dzierżawy w każdym centrum Azure Stack.
  - **Zanotuj każdy identyfikator subskrypcji i punkt końcowy Azure Resource Manager dla poszczególnych centrów Azure Stack.**
- Nazwa główna usługi Azure Active Directory (Azure AD), która ma uprawnienia do subskrypcji dzierżawy w poszczególnych centrach Azure Stack. Jeśli centra Azure Stack są wdrażane na różnych dzierżawach usługi Azure AD, może być konieczne utworzenie dwóch jednostek usługi. Aby dowiedzieć się, jak utworzyć jednostkę usługi dla Azure Stack Hub, zobacz [Tworzenie nazw głównych usług, aby umożliwić aplikacjom dostęp do Azure Stack zasobów centrum](/azure-stack/user/azure-stack-create-service-principals).
  - **Zanotuj każdy identyfikator aplikacji jednostki usługi, klucz tajny klienta i nazwę dzierżawy (xxxxx.onmicrosoft.com).**
- SQL Server 2016 Enterprise jest zespolona z każdym rynkiem Azure Stack centrum. Aby dowiedzieć się więcej na temat zespalania portalu Marketplace, zobacz artykuł [Pobieranie elementów z witryny Marketplace do centrum Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
    **Upewnij się, że Twoja organizacja ma odpowiednie licencje SQL.**
- [Docker for Windows](https://docs.docker.com/docker-for-windows/) zainstalowany na komputerze lokalnym.

## <a name="get-the-docker-image"></a>Pobierz obraz platformy Docker

Obrazy platformy Docker dla każdego wdrożenia eliminują problemy zależności między różnymi wersjami Azure PowerShell.

1. Upewnij się, że Docker for Windows używa kontenerów systemu Windows.
2. Uruchom następujący skrypt w wierszu polecenia z podwyższonym poziomem uprawnień, aby uzyskać kontener platformy Docker ze skryptami wdrażania.

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a>Wdróż grupę dostępności

1. Po pomyślnym ściągnięciu obrazu kontenera Uruchom obraz.

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. Po rozpoczęciu kontenera otrzymasz w kontenerze odpowiedni Terminal programu PowerShell z podwyższonym poziomem uprawnień. Zmień katalogi, aby przejść do skryptu wdrażania.

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. Uruchom wdrożenie. Podaj poświadczenia i nazwy zasobów, jeśli jest to potrzebne. HA odnosi się do centrum Azure Stack, w którym zostanie wdrożony klaster HA. Program DR odwołuje się do centrum Azure Stack, w którym zostanie wdrożony klaster odzyskiwania po awarii.

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. Wpisz `Y` polecenie, aby zezwolić na instalację dostawcy NuGet, co spowoduje rozpoczęcie instalacji profilu interfejsu API "2018-03-01-hybrydowe".

5. Poczekaj na zakończenie wdrażania zasobów.

6. Po zakończeniu wdrażania zasobów DR Zamknij kontener.

      ```powershell
      exit
      ```

7. Zbadaj wdrożenie, wyświetlając zasoby w każdym portalu Centrum Azure Stack. Połącz się z jednym z wystąpień programu SQL Server w środowisku HA i sprawdź grupę dostępności za pomocą SQL Server Management Studio (SSMS).

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a>Następne kroki

- Użyj SQL Server Management Studio, aby ręcznie przełączyć klaster w tryb failover. Zobacz [przeprowadzenie wymuszonej ręcznej pracy awaryjnej grupy dostępności zawsze włączone (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)
- Dowiedz się więcej o aplikacjach hybrydowych w chmurze. Zobacz [hybrydowe rozwiązania w chmurze.](/azure-stack/user/)
- Użyj własnych danych lub zmodyfikuj kod w tym przykładzie w witrynie [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).