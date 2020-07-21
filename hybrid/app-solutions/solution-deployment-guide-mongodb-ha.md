---
title: Wdróż rozwiązanie MongoDB o wysokiej dostępności na platformie Azure i w centrum Azure Stack
description: Dowiedz się, jak wdrożyć rozwiązanie MongoDB o wysokiej dostępności na platformie Azure i w centrum Azure Stack
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: f6064aaa1087a3c0cfc26e09371e81752c777edb
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477273"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a>Wdróż rozwiązanie MongoDB o wysokiej dostępności na platformie Azure i w centrum Azure Stack

Ten artykuł przeprowadzi Cię przez automatyczne wdrożenie podstawowego klastra o wysokiej dostępności (HA) MongoDB z lokacją odzyskiwania po awarii w dwóch środowiskach centrum Azure Stack. Aby dowiedzieć się więcej na temat MongoDB i wysokiej dostępności, zobacz [elementy członkowskie zestawu replik](https://docs.mongodb.com/manual/core/replica-set-members/).

W tym rozwiązaniu utworzysz przykładowe środowisko, aby:

> [!div class="checklist"]
> - Organizuj wdrożenie w dwóch centrach Azure Stack.
> - Użyj platformy Docker, aby zminimalizować problemy zależności z profilami interfejsu API platformy Azure.
> - Wdróż klaster MongoDB o wysokiej dostępności z lokacją odzyskiwania po awarii.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub to rozszerzenie platformy Azure. Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.  
> 
> [Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych. Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a>Architektura MongoDB z centrum Azure Stack

![Architektura MongoDB o wysokiej dostępności w centrum Azure Stack](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a>Wymagania wstępne dotyczące usługi MongoDB z centrum Azure Stack

- Dwa połączone systemy Azure Stack Hub zintegrowane (Azure Stack Hub). To wdrożenie nie działa na Azure Stack Development Kit (ASDK). Aby dowiedzieć się więcej na temat Azure Stack Hub, zobacz [co to jest centrum Azure Stack?](https://azure.microsoft.com/products/azure-stack/hub/)
  - Subskrypcja dzierżawy w każdym centrum Azure Stack. 
  - **Zanotuj każdy identyfikator subskrypcji i punkt końcowy Azure Resource Manager dla poszczególnych centrów Azure Stack.**
- Nazwa główna usługi Azure Active Directory (Azure AD), która ma uprawnienia do subskrypcji dzierżawy w poszczególnych centrach Azure Stack. Jeśli centra Azure Stack są wdrażane na różnych dzierżawach usługi Azure AD, może być konieczne utworzenie dwóch jednostek usługi. Aby dowiedzieć się, jak utworzyć jednostkę usługi dla Azure Stack Hub, zobacz [Korzystanie z tożsamości aplikacji w celu uzyskiwania dostępu do zasobów centrum Azure Stack](/azure-stack/user/azure-stack-create-service-principals).
  - **Zanotuj każdy identyfikator aplikacji jednostki usługi, klucz tajny klienta i nazwę dzierżawy (xxxxx.onmicrosoft.com).**
- Ubuntu 16,04 został zespolony z każdym z rynków centrum Azure Stack. Aby dowiedzieć się więcej na temat zespalania portalu Marketplace, zobacz artykuł [Pobieranie elementów z witryny Marketplace do centrum Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).
- [Docker for Windows](https://docs.docker.com/docker-for-windows/) zainstalowany na komputerze lokalnym.

## <a name="get-the-docker-image"></a>Pobierz obraz platformy Docker

Obrazy platformy Docker dla każdego wdrożenia eliminują problemy zależności między różnymi wersjami Azure PowerShell.

1. Upewnij się, że Docker for Windows używa kontenerów systemu Windows.
2. Uruchom następujące polecenie w wierszu polecenia z podwyższonym poziomem uprawnień, aby uzyskać kontener platformy Docker ze skryptami wdrażania.

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a>Wdrażanie klastrów

1. Po pomyślnym ściągnięciu obrazu kontenera Uruchom obraz.

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. Po rozpoczęciu kontenera otrzymasz w kontenerze odpowiedni Terminal programu PowerShell z podwyższonym poziomem uprawnień. Zmień katalogi, aby przejść do skryptu wdrażania.

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. Uruchom wdrożenie. Podaj poświadczenia i nazwy zasobów, jeśli jest to potrzebne. HA odnosi się do centrum Azure Stack, w którym zostanie wdrożony klaster HA. Program DR odwołuje się do centrum Azure Stack, w którym zostanie wdrożony klaster odzyskiwania po awarii.

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

5. Zasoby HA zostaną wdrożone jako pierwsze. Monitoruj wdrożenie i poczekaj na jego zakończenie. Gdy zostanie wyświetlony komunikat z informacją, że wdrożenie HA zostało zakończone, można sprawdzić Portal centrum Azure Stack HA, aby zobaczyć wdrożone zasoby.

6. Kontynuuj wdrażanie zasobów DR i zdecyduj, czy chcesz włączyć pole skoku w centrum Azure Stack DR, aby móc korzystać z klastra.

7. Poczekaj na zakończenie wdrożenia zasobu DR.

8. Po zakończeniu wdrażania zasobów DR Zamknij kontener.

  ```powershell
  exit
  ```

## <a name="next-steps"></a>Następne kroki

- Jeśli włączono maszynę wirtualną pola skoku w centrum Azure Stack DR, możesz nawiązać połączenie za pośrednictwem protokołu SSH i korzystać z klastra MongoDB przez zainstalowanie interfejsu wiersza polecenia Mongo. Aby dowiedzieć się więcej na temat współpracy z usługą MongoDB, zobacz [powłoka Mongo](https://docs.mongodb.com/manual/mongo/).
- Aby dowiedzieć się więcej na temat hybrydowych aplikacji w chmurze, zobacz [hybrydowe rozwiązania w chmurze.](https://aka.ms/azsdevtutorials)
- Zmodyfikuj kod do tego przykładu w serwisie [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).
