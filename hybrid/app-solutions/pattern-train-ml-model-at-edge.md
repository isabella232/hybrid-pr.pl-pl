---
title: Uczenie modelu uczenia maszynowego pod wzorcem krawędzi
description: Dowiedz się, jak przeprowadzić szkolenie modelu uczenia maszynowego na brzegu na platformie Azure i w centrum Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911098"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a>Uczenie modelu uczenia maszynowego pod wzorcem krawędzi

Generuj modele przenośnej uczenia maszynowego (ML) na podstawie danych, które istnieją tylko lokalnie.

## <a name="context-and-problem"></a>Kontekst i problem

Wiele organizacji chce odwiedzać szczegółowe informacje z lokalnych lub starszych danych przy użyciu narzędzi, które są zrozumiałe dla analityków danych. [Azure Machine Learning](/azure/machine-learning/) zapewnia natywne narzędzia chmurowe umożliwiające uczenie, dostrajanie i wdrażanie modeli i uczenia głębokiego.  

Jednak niektóre dane są zbyt duże do chmury lub nie można ich wysłać do chmury z przyczyn prawnych. Korzystając z tego wzorca, analityki danych mogą używać Azure Machine Learning do uczenia modeli przy użyciu danych lokalnych i obliczeń.

## <a name="solution"></a>Rozwiązanie

Szkolenia w ramach wzorca brzegowego korzystają z maszyny wirtualnej uruchomionej w centrum Azure Stack. Maszyna wirtualna jest zarejestrowana jako obiekt docelowy obliczeń w usłudze Azure ML, dzięki czemu dostęp do danych jest dostępny tylko lokalnie. W takim przypadku dane są przechowywane w magazynie obiektów Blob Azure Stack Hub.

Po przeszkoleniu modelu jest on rejestrowany w usłudze Azure ML, kontenerze i dodawany do Azure Container Registry wdrożenia. Dla tej iteracji wzorca maszyna wirtualna szkoleń Azure Stack Hub musi być dostępna za pośrednictwem publicznego Internetu.

[![Model uczenia maszynowego na architekturze brzegowej](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)

Oto jak działa wzorzec:

1. Maszyna wirtualna Azure Stack Hub jest wdrażana i rejestrowana jako obiekt docelowy obliczeń przy użyciu usługi Azure ML.
2. Na platformie Azure ML jest tworzony eksperyment, który używa maszyny wirtualnej Azure Stack Hub jako elementu docelowego obliczeń.
3. Po przeszkoleniu modelu jest on zarejestrowany i kontener.
4. Model można teraz wdrożyć do lokalizacji lokalnych lub w chmurze.

## <a name="components"></a>Składniki

To rozwiązanie używa następujących składników:

| Warstwa | Składnik | Opis |
|----------|-----------|-------------|
| Azure | Azure Machine Learning | [Azure Machine Learning](/azure/machine-learning/) organizować szkolenia modelu ml. |
| | Azure Container Registry | Usługa Azure ML pakuje model do kontenera i zapisuje go w [Azure Container Registry](/azure/container-registry/) do wdrożenia.|
| Azure Stack Hub | App Service | [Azure Stack Hub z App Service](/azure-stack/operator/azure-stack-app-service-overview) stanowi podstawę dla składników na krawędzi. |
| | Wystąpienia obliczeniowe | Maszyna wirtualna Azure Stack Hub z systemem Ubuntu z platformą Docker służy do uczenia modelu ML. |
| | Magazyn | Dane prywatne mogą być hostowane w Azure Stack centrum obiektów BLOB Storage. |

## <a name="issues-and-considerations"></a>Problemy i kwestie do rozważenia

Podczas decydowania o sposobie wdrożenia tego rozwiązania należy wziąć pod uwagę następujące kwestie:

### <a name="scalability"></a>Skalowalność

Aby umożliwić skalowanie tego rozwiązania, należy utworzyć maszynę wirtualną o odpowiednim rozmiarze na Azure Stack Hub na potrzeby szkolenia.

### <a name="availability"></a>Dostępność

Upewnij się, że skrypty szkoleniowe i maszyna wirtualna Azure Stack Hub mają dostęp do danych lokalnych używanych do szkoleń.

### <a name="manageability"></a>Możliwości zarządzania

Upewnij się, że modele i eksperymenty są odpowiednio zarejestrowane, w wersji i otagowane, aby uniknąć pomyłek podczas wdrażania modelu.

### <a name="security"></a>Zabezpieczenia

Ten wzorzec umożliwia usłudze Azure ML dostęp do danych poufnych w środowisku lokalnym. Upewnij się, że konto używane do protokołu SSH do maszyny wirtualnej Azure Stack Hub ma silne hasło i że skrypty szkoleniowe nie zachowają ani nie przekazują danych do chmury.

## <a name="next-steps"></a>Następne kroki

Aby dowiedzieć się więcej na temat tematów wprowadzonych w tym artykule:

- Zapoznaj się z [dokumentacją Azure Machine Learning](/azure/machine-learning) , aby zapoznać się z tematem dotyczącym tablic i tematów pokrewnych.
- Zobacz [Azure Container Registry](/azure/container-registry/) , aby dowiedzieć się, jak tworzyć i przechowywać obrazy i zarządzać nimi na potrzeby wdrożeń kontenerów.
- Aby dowiedzieć się więcej na temat dostawcy zasobów i sposobu wdrażania, zapoznaj się z tematem [App Service w centrum Azure Stack](/azure-stack/operator/azure-stack-app-service-overview) .
- Zobacz [zagadnienia dotyczące projektowania aplikacji hybrydowych](overview-app-design-considerations.md) , aby dowiedzieć się więcej o najlepszych rozwiązaniach i uzyskać odpowiedzi na dodatkowe pytania.
- Zapoznaj się z [rodziną Azure Stack produktów i rozwiązań](/azure-stack) , aby dowiedzieć się więcej o całym portfolio produktów i rozwiązań.

Gdy wszystko będzie gotowe do przetestowania przykładowego rozwiązania, przejdź do [modelu uczenia maszynowego w przewodniku wdrażania programu Edge](https://aka.ms/edgetrainingdeploy). Przewodnik wdrażania zawiera instrukcje krok po kroku dotyczące wdrażania i testowania jego składników.
