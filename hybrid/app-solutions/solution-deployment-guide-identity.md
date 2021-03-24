---
title: Konfigurowanie tożsamości chmury hybrydowej dla aplikacji platformy Azure i usługi Azure Stack Hub
description: Dowiedz się, jak skonfigurować tożsamość chmury hybrydowej dla platformy Azure i aplikacji Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895351"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Konfigurowanie tożsamości chmury hybrydowej dla aplikacji platformy Azure i usługi Azure Stack Hub

Dowiedz się, jak skonfigurować tożsamość chmury hybrydowej dla aplikacji platformy Azure i usługi Azure Stack Hub.

Dostępne są dwie opcje udzielania dostępu do aplikacji zarówno w globalnych centrach Azure, jak i Azure Stack.

 * Gdy usługa Azure Stack Hub ma ciągłe połączenie z Internetem, możesz użyć Azure Active Directory (Azure AD).
 * Gdy Azure Stack Hub zostanie odłączony od Internetu, możesz użyć usług federacyjnych w usłudze Azure Directory (AD FS).

Nazwy główne usługi są używane do udzielania dostępu do aplikacji Centrum Azure Stack w celu wdrożenia lub konfiguracji przy użyciu Azure Resource Manager w centrum Azure Stack.

W tym rozwiązaniu utworzysz przykładowe środowisko w celu:

> [!div class="checklist"]
> - Ustanów tożsamość hybrydową w globalnych centrach Azure i Azure Stack
> - Pobierz token, aby uzyskać dostęp do interfejsu API usługi Azure Stack Hub.

Aby wykonać kroki opisane w tym rozwiązaniu, musisz mieć uprawnienia operatora centrum Azure Stack.

> [!Tip]  
> ![Diagram filarów hybrydowych](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub to rozszerzenie platformy Azure. Usługa Azure Stack Hub zapewnia elastyczność i innowacje w chmurze obliczeniowej w środowisku lokalnym, umożliwiając jedyną chmurę hybrydową, która umożliwia tworzenie i wdrażanie aplikacji hybrydowych w dowolnym miejscu.  
> 
> [Zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) w artykule przegląd filarów jakości oprogramowania (rozmieszczenia, skalowalności, dostępności, odporności, możliwości zarządzania i zabezpieczeń) do projektowania, wdrażania i obsługi aplikacji hybrydowych. Zagadnienia dotyczące projektowania pomagają zoptymalizować projekt aplikacji hybrydowej i zminimalizować wyzwania w środowiskach produkcyjnych.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Tworzenie nazwy głównej usługi dla usługi Azure AD w portalu

Jeśli usługa Azure Stack Hub została wdrożona przy użyciu usługi Azure AD jako magazynu tożsamości, można utworzyć jednostki usługi tak samo jak w przypadku platformy Azure. [Użyj tożsamości aplikacji, aby uzyskać dostęp do zasobów](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) , pokazuje, jak wykonać kroki w portalu. Przed rozpoczęciem upewnij się, że masz [wymagane uprawnienia usługi Azure AD](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) .

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Tworzenie nazwy głównej usługi dla AD FS przy użyciu programu PowerShell

Jeśli wdrożono Azure Stack Hub z AD FS, można użyć programu PowerShell do utworzenia jednostki usługi, przypisania roli do dostępu i zalogowania się za pomocą programu PowerShell przy użyciu tej tożsamości. [Do uzyskiwania dostępu do zasobów służy tożsamość aplikacji](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) . przedstawiono w nim sposób wykonywania wymaganych kroków przy użyciu programu PowerShell.

## <a name="using-the-azure-stack-hub-api"></a>Korzystanie z interfejsu API Azure Stack Hub

Rozwiązanie [interfejsu api Azure Stack Hub](/azure-stack/user/azure-stack-rest-api-use)  przeprowadzi Cię przez proces pobierania tokenu w celu uzyskania dostępu do interfejsu api usługi Azure Stack Hub.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Nawiązywanie połączenia z centrum Azure Stack przy użyciu programu PowerShell

Przewodnik Szybki Start [do rozpoczęcia pracy z programem PowerShell w usłudze Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) przeprowadzi Cię przez kroki wymagane do zainstalowania Azure PowerShell i nawiązania połączenia z instalacją centrum Azure Stack.

### <a name="prerequisites"></a>Wymagania wstępne

Wymagana jest instalacja centrum Azure Stack podłączona do usługi Azure AD z subskrypcją, do której można uzyskać dostęp. Jeśli nie masz instalacji Azure Stack Hub, możesz użyć tych instrukcji, aby skonfigurować [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).

#### <a name="connect-to-azure-stack-hub-using-code"></a>Nawiązywanie połączenia z centrum Azure Stack przy użyciu kodu

Aby nawiązać połączenie z centrum Azure Stack przy użyciu kodu, użyj interfejsu API Azure Resource Manager punktów końcowych, aby pobrać punkty końcowe uwierzytelniania i grafu dla instalacji centrum Azure Stack. Następnie Uwierzytelnij się przy użyciu żądań REST. Przykładową aplikację kliencką można znaleźć w witrynie [GitHub](https://github.com/shriramnat/HybridARMApplication).

>[!Note]
>Jeśli zestaw Azure SDK dla wybranego języka nie obsługuje profilów interfejsu API platformy Azure, zestaw SDK może nie współpracować z centrum Azure Stack. Aby dowiedzieć się więcej o profilach interfejsu API platformy Azure, zobacz artykuł [Zarządzanie wersjami profilów interfejsu API](/azure-stack/user/azure-stack-version-profiles) .

## <a name="next-steps"></a>Następne kroki

- Aby dowiedzieć się więcej o sposobie obsługi tożsamości w centrum Azure Stack, zobacz temat [Architektura tożsamości dla centrum Azure Stack](/azure-stack/operator/azure-stack-identity-architecture).
- Aby dowiedzieć się więcej o wzorcach chmury platformy Azure, zobacz [wzorce projektowe w chmurze](/azure/architecture/patterns).
