---
title: Zagadnienia dotyczące projektowania aplikacji hybrydowych na platformie Azure i w centrum Azure Stack
description: Poznaj zagadnienia dotyczące projektowania w przypadku kompilowania aplikacji hybrydowej dla inteligentnej chmury i inteligentnej krawędzi, w tym umieszczania, skalowalności, dostępności i odporności.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4fd52f76baad8059e130adfc01cdd0152b40a510
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911112"
---
# <a name="hybrid-app-design-considerations"></a>Zagadnienia dotyczące projektowania aplikacji hybrydowych

Microsoft Azure jest jedyną spójną chmurą hybrydową. Umożliwia ponowne wykorzystanie inwestycji programistycznych i umożliwia korzystanie z aplikacji, które mogą obejmować globalne platformy Azure, suwerenne chmury platformy Azure i Azure Stack, które są rozszerzeniami platformy Azure w centrum danych. Aplikacje obejmujące chmury są również nazywane *aplikacjami hybrydowymi*.

[*Przewodnik po architekturze aplikacji platformy Azure*](https://docs.microsoft.com/azure/architecture/guide) zawiera opis podejścia strukturalnego do projektowania aplikacji, które są skalowalne, odporne i o wysokiej dostępności. Zagadnienia opisane w [*przewodniku po architekturze aplikacji platformy Azure*](https://docs.microsoft.com/azure/architecture/guide) są równie stosowane dla aplikacji, które są przeznaczone dla jednej chmury i dla aplikacji, które obejmują chmury.

Ten artykuł rozszerza [*filary jakości oprogramowania*](https://docs.microsoft.com/azure/architecture/guide/pillars) omówione w [ *przewodniku po architekturze*](https://docs.microsoft.com/azure/architecture/guide/) [*aplikacji platformy Azure*](https://docs.microsoft.com/azure/architecture/guide/) , koncentrując się na projektowaniu aplikacji hybrydowych. Ponadto dodamy filar *umieszczania* , ponieważ aplikacje hybrydowe nie są wyłącznie w jednej chmurze lub jednym lokalnym centrum danych.

Scenariusze hybrydowe różnią się w zależności od zasobów, które są dostępne do programowania, i obejmują zagadnienia, takie jak lokalizacja geograficzna, zabezpieczenia, dostęp do Internetu i inne zagadnienia. Mimo że ten przewodnik nie może wyliczyć określonych zagadnień, może on dostarczyć pewnych najważniejszych wytycznych i najlepszych rozwiązań, które należy wykonać. Pomyślne projektowanie, konfigurowanie, wdrażanie i konserwowanie architektury aplikacji hybrydowej obejmuje wiele zagadnień związanych z projektowaniem, które mogą być nieznane.

Ten dokument ma na celu agregowanie ewentualnych pytań, które mogą wystąpić podczas implementowania aplikacji hybrydowych i zawiera informacje (te filary) oraz najlepsze rozwiązania dotyczące pracy z nimi. Odnosząc się do tych pytań podczas fazy projektowania, można uniknąć problemów, które mogą wystąpić w środowisku produkcyjnym.

Zasadniczo są to pytania, które należy wziąć pod uwagę przed utworzeniem aplikacji hybrydowej. Aby rozpocząć, należy wykonać następujące czynności:

- Zidentyfikuj i Oceń składniki aplikacji.
- Oceń składniki aplikacji na filarach.

## <a name="evaluate-the-app-components"></a>Oceń składniki aplikacji

Każdy składnik aplikacji ma własną konkretną rolę w większej aplikacji i powinien być przeglądany ze wszystkimi zagadnieniami dotyczącymi projektowania. Wymagania i funkcje każdego składnika powinny być mapowane na te zagadnienia, aby pomóc w ustaleniu architektury aplikacji.

Rozłożyć aplikację na jej składniki, badając architekturę aplikacji i określając, co obejmuje. Składniki mogą również obejmować inne aplikacje, z którymi współdziała aplikacja. Podczas identyfikowania składników należy oszacować zamierzone operacje hybrydowe zgodnie z ich charakterystykami, zadając te pytania:

- Jaki jest cel składnika?
- Jakie są współzależności między składnikami?

Na przykład aplikacja może mieć zdefiniowany element frontonu i zaplecza jako dwa składniki. W scenariuszu hybrydowym fronton znajduje się w jednej chmurze, a zaplecze znajduje się w drugiej. Aplikacja zawiera kanały komunikacji między frontonem a użytkownikiem, a także między frontonem a zapleczem.

Składnik aplikacji jest definiowany przez wiele formularzy i scenariuszy. Najważniejsze zadanie służy do identyfikowania ich oraz w chmurze lub lokalizacji lokalnej.

Typowe składniki aplikacji do uwzględnienia w spisie są wymienione w tabeli 1.

### <a name="table-1-common-app-components"></a>Tabela 1. Popularne składniki aplikacji

| **Składnik** | **Wskazówki dotyczące aplikacji hybrydowej** |
| ---- | ---- |
| Połączenia klienta | Aplikacja (na dowolnym urządzeniu) może uzyskiwać dostęp do użytkowników na różne sposoby, z jednego punktu wejścia, w tym na następujące sposoby:<br>-Model klient-serwer, który wymaga od użytkownika zainstalowania klienta programu do pracy z aplikacją. Aplikacja oparta na serwerze, do której można uzyskać dostęp za pomocą przeglądarki.<br>-Połączenia klienckie mogą zawierać powiadomienia w przypadku przerwania połączenia lub alerty, gdy mogą być naliczane opłaty za roaming. |
| Authentication  | Uwierzytelnianie może być wymagane dla użytkownika łączącego się z aplikacją lub z jednego składnika łączącego się z innym. |
| Interfejsy API  | Możesz udostępnić deweloperom dostęp programistyczny do aplikacji z zestawami interfejsów API i bibliotekami klas i udostępnić interfejs połączenia na podstawie standardów internetowych. Możesz również użyć interfejsów API, aby rozłożyć aplikację na niezależne działania jednostki logicznej. |
| Usługi  | Możesz użyć zwięzłych usług, aby udostępnić funkcje aplikacji. Usługa może być aparatem, na którym działa aplikacja. |
| Kolejki | Za pomocą kolejek można organizować stan cykli życia i Stanów składników aplikacji. Kolejki te mogą zapewniać obsługę komunikatów, powiadomień i buforowania w celu subskrybowania stron. |
| Magazyn danych | Aplikacja może być bezstanowa lub bezstanowa. Aplikacje stanowe potrzebują magazynów danych, które mogą być spełnione przez wiele formatów i woluminów. |
| Buforowanie danych  | Składnik buforowania danych w projekcie może strategicznie zwiększać przyczyny problemów z opóźnieniami i odgrywać rolę w wyzwalaniu tworzenia dużych obciążeń w chmurze. |
| Wprowadzanie danych | Dane można przesyłać do aplikacji na wiele sposobów, od wartości przesłanych przez użytkownika w formularzu sieci Web w celu ciągłego przepływu danych o dużym natężeniu. |
| Przetwarzanie danych | Zadania związane z przetwarzaniem danych (takie jak raporty, analizy, eksporty wsadowe i przekształcenia danych) mogą być przetwarzane w źródle lub Odciążone w oddzielnym składniku przy użyciu kopii danych. |

## <a name="assess-app-components-for-pillars"></a>Oceń składniki aplikacji dla filarów

Dla każdego składnika Oceń jego cechy charakterystyczne dla każdego filaru. Podczas obliczania każdego składnika ze wszystkimi filarami, pytania, które mogą nie być uważane, mogą być znane, które mają wpływ na projekt aplikacji hybrydowej. W ramach tych zagadnień można dodać wartość w obszarze optymalizacja aplikacji. Tabela 2 zawiera opis każdego filaru, który odnosi się do aplikacji hybrydowych.

### <a name="table-2-pillars"></a>Tabela 2. Filarów

| **Filar** | **Opis** |
| ----------- | --------------------------------------------------------- |
| Umieszczanie  | Strategiczne pozycjonowanie składników w aplikacjach hybrydowych. |
| Skalowalność  | Zdolność systemu do obsługi zwiększonego obciążenia. |
| Dostępność  | Czas, przez jaki aplikacja hybrydowa działa i działa. |
| Odporność | Możliwość odzyskania aplikacji hybrydowej. |
| Możliwości zarządzania | Procesy operacji, które utrzymują działanie systemu w środowisku produkcyjnym. |
| Zabezpieczenia | Ochrona hybrydowych aplikacji i danych przed zagrożeniami. |

## <a name="placement"></a>Umieszczanie

Aplikacja hybrydowa z założenia ma rozmieszczenie, na przykład w centrum danych.

Umieszczenie jest ważnym zadaniem do pozycjonowania składników, dzięki czemu mogą one najlepiej obsłużyć aplikację hybrydową. Według definicji aplikacje hybrydowe obejmują lokalizacje, takie jak od lokalnego do chmury i między różnymi chmurami. Składniki aplikacji można umieszczać w chmurach na dwa sposoby:

- **Aplikacje hybrydowe w pionie**  
    Składniki aplikacji są dystrybuowane między lokalizacjami. Każdy pojedynczy składnik może mieć wiele wystąpień, które znajdują się tylko w jednej lokalizacji.

- **Poziome aplikacje hybrydowe**  
    Składniki aplikacji są dystrybuowane między lokalizacjami. Każdy pojedynczy składnik może mieć wiele wystąpień obejmujących wiele lokalizacji.

    Niektóre składniki mogą być świadome ich lokalizacji, podczas gdy inne nie znają ich lokalizacji i umieszczania. Ten virtuousness można osiągnąć za pomocą warstwy abstrakcji. Ta warstwa z nowoczesnej struktury aplikacji, takiej jak mikrousługi, może definiować sposób obsługi aplikacji przez rozmieszczenie składników aplikacji działających na węzłach w różnych chmurach.

### <a name="placement-checklist"></a>Lista kontrolna umieszczania

**Sprawdź wymagane lokalizacje.** Upewnij się, że aplikacja lub jej składniki są wymagane do działania w programie lub wymagają certyfikacji w odniesieniu do określonej chmury. Może to obejmować wymagania dotyczące suwerenności firmy lub podyktowanie przez prawo. Sprawdź również, czy dla określonej lokalizacji lub ustawień regionalnych są wymagane wszystkie operacje lokalne.

**Ustalanie zależności łączności.** Wymagane lokalizacje i inne czynniki mogą dyktować zależności między składnikami. Podczas umieszczania składników należy określić optymalne połączenia i zabezpieczenia komunikacji między nimi. Dostępne opcje to: [ *VPN*,](https://docs.microsoft.com/azure/vpn-gateway/) [ *ExpressRoute*](https://docs.microsoft.com/azure/expressroute/) i [ *połączenia hybrydowe*.](https://docs.microsoft.com/azure/app-service/app-service-hybrid-connections)

**Oceń możliwości platformy.** W przypadku każdego składnika aplikacji Sprawdź, czy wymagany dostawca zasobów dla składnika aplikacji jest dostępny w chmurze i czy przepustowość może uwzględniać oczekiwane wymagania dotyczące przepływności i opóźnienia.

**Zaplanuj przenośność.** Używaj nowoczesnych platform aplikacji, takich jak kontenery lub mikrousługi, aby planować przechodzenie operacji i zapobiegać zależnościom usługi.

**Określ wymagania dotyczące suwerenności danych.** Aplikacje hybrydowe są dostosowane do rozdzielania izolacji danych, na przykład w lokalnym centrum. Zapoznaj się z rozmieszczeniem zasobów, aby zoptymalizować sukces dla spełnienia tego wymagania.

**Zaplanuj opóźnienia.** Operacje między chmurami mogą wprowadzić fizyczną odległość między składnikami aplikacji. Należy upewnić się, że wymagania dotyczące dowolnych opóźnień.

**Sterowanie przepływem ruchu.** Obsługuj użycie szczytowe i odpowiednią i bezpieczną komunikację na potrzeby danych osobowych, które są dostępne w ramach frontonu w chmurze publicznej.

## <a name="scalability"></a>Skalowalność

Skalowalność to zdolność systemu do obsługi zwiększonego obciążenia aplikacji, która może się różnić w miarę upływu czasu, ponieważ inne czynniki i siły wpływają na rozmiar odbiorców (oprócz rozmiaru i zakresu aplikacji).

W przypadku podstawowej dyskusji tego filaru zobacz [*skalowalność*](https://docs.microsoft.com/azure/architecture/guide/pillars#scalability) w pięciu filarach doskonałości architektury.

Podejście skalowania w poziomie dla aplikacji hybrydowych umożliwia dodanie większej liczby wystąpień w celu spełnienia wymagań, a następnie wyłączenie ich w okresach ciszy.

W scenariuszach hybrydowych skalowanie pojedynczych składników wymaga dodatkowej uwagi, gdy składniki są rozłożone między chmurami. Skalowanie jednej części aplikacji może wymagać skalowania innych. Jeśli na przykład liczba połączeń klientów rośnie, ale usługi sieci Web aplikacji nie są odpowiednio skalowane, obciążenie bazy danych może spowodować nasycenie aplikacji.

Niektóre składniki aplikacji można skalować w poziomie liniowo, podczas gdy inne mają skalowanie zależności i mogą być ograniczone do zakresu, w którym można skalować. Na przykład tunel VPN zapewniający łączność hybrydową dla lokalizacji składników aplikacji ma limit przepustowości i opóźnień, do których można skalować. Jak są skalowane składniki aplikacji, aby upewnić się, że te wymagania są spełnione?

### <a name="scalability-checklist"></a>Lista kontrolna dotycząca skalowalności

**Ustalanie progów skalowania.** Aby obsłużyć różne zależności w aplikacji, ustal, w jakim stopniu składniki aplikacji w różnych chmurach mogą być skalowane niezależnie od siebie, a nadal spełniają wymagania dotyczące uruchamiania aplikacji. Aplikacje hybrydowe często wymagają skalowania określonych obszarów w aplikacji, aby obsługiwały funkcję w miarę ich działania i wpływać na resztę aplikacji. Na przykład przekroczenie liczby wystąpień frontonu może wymagać skalowania zaplecza.

**Zdefiniuj harmonogramy skalowania.** Większość aplikacji ma zajęte okresy, dlatego należy agregować ich czasy szczytu do harmonogramów, aby koordynować optymalne skalowanie.

**Użyj scentralizowanego systemu monitorowania.** Możliwości monitorowania platform umożliwiają Skalowanie automatyczne, ale aplikacje hybrydowe wymagają scentralizowanego systemu monitorowania, który agreguje kondycję i obciążenie systemu. Scentralizowany system monitorowania może inicjować skalowanie zasobu w jednej lokalizacji i skalowanie w zależności od zasobu w innej lokalizacji. Ponadto centralny system monitorowania może śledzić, w których chmurach zasoby skalowania automatycznego i które chmury nie są.

**Skorzystaj z funkcji skalowania automatycznego (jak to możliwe).** Jeśli możliwości skalowania automatycznego są częścią architektury, należy zaimplementować Skalowanie automatyczne przez ustawienie progów, które definiują, kiedy składnik aplikacji ma być skalowany w górę, w dół, w dół czy w. Przykładem automatycznego skalowania jest połączenie klienta, które jest automatycznie skalowane w jednej chmurze w celu obsługi zwiększonej pojemności, ale powoduje, że inne zależności aplikacji są rozłożone w różnych chmurach. Należy upewnić się, że funkcje skalowania automatycznego w tych składnikach zależnych muszą zostać ustalone.

Jeśli automatyczne skalowanie nie jest dostępne, rozważ zaimplementowanie skryptów i innych zasobów, aby umożliwić ręczne skalowanie, wyzwalane przez progi w scentralizowanym systemie monitorowania.

**Określ oczekiwane obciążenie według lokalizacji.** Aplikacje hybrydowe obsługujące żądania klientów mogą opierać się głównie na pojedynczej lokalizacji. Gdy obciążenie żądań klientów przekracza wartość progową, można dodać dodatkowe zasoby w innej lokalizacji w celu rozdzielenia obciążenia żądaniami przychodzącymi. Upewnij się, że połączenia klienckie mogą obsługiwać zwiększone obciążenia, a także określić wszelkie zautomatyzowane procedury dla połączeń klientów, aby obsłużyć obciążenie.

## <a name="availability"></a>Dostępność

Dostępność to czas, jaki system jest funkcjonalny i działa. Dostępność jest mierzona jako procent czasu pracy. Błędy aplikacji, problemy z infrastrukturą i obciążenie systemu mogą zmniejszać dostępność.

Aby zapoznać się z podstawową dyskusją tego filaru, zobacz [*dostępność*](/azure/architecture/framework/) w pięciu filarach doskonałości architektury.

### <a name="availability-checklist"></a>Lista kontrolna dotycząca dostępności

**Zapewnianie nadmiarowości łączności.** Aplikacje hybrydowe wymagają łączności między chmurami, w których aplikacja jest rozłożona. Istnieje możliwość wybrania technologii łączności hybrydowej, a także do wyboru podstawowej technologii, korzystania z innej technologii w celu zapewnienia nadmiarowości przy użyciu funkcji automatycznej pracy awaryjnej, jeśli podstawowa technologia nie powiedzie się.

**Klasyfikowanie domen błędów.** Aplikacje odporne na uszkodzenia wymagają wielu domen błędów. Domeny błędów ułatwiają odizolowanie punktu awarii, na przykład jeśli pojedynczy dysk twardy ulegnie awarii, jeśli przestanie działać, lub jeśli pełne centrum danych jest niedostępne. W aplikacji hybrydowej lokalizacja może być sklasyfikowana jako domena błędów. Im więcej wymagań dotyczących dostępności, tym więcej potrzeba do oszacowania, jak należy klasyfikować pojedynczą domenę błędów.

**Klasyfikowanie domen uaktualnienia.** Domeny uaktualnień są używane do zapewnienia dostępności wystąpień składników aplikacji, podczas gdy inne wystąpienia tego samego składnika są obsługują aktualizacje lub uaktualnienia funkcji. Podobnie jak w przypadku domen błędów domeny uaktualnienia mogą być klasyfikowane według ich rozmieszczenia w różnych lokalizacjach. Należy określić, czy składnik aplikacji może obsłużyć uaktualnienie w jednej lokalizacji, zanim zostanie uaktualniony w innej lokalizacji lub jeśli są wymagane inne konfiguracje domeny. Pojedyncza lokalizacja może mieć wiele domen uaktualnienia.

**Śledzenie wystąpień i dostępności.** Składniki aplikacji o wysokiej dostępności mogą być dostępne za poorednictwem równoważenia obciążenia i synchronicznej replikacji danych. Należy określić liczbę wystąpień, które mogą być w trybie offline, zanim usługa zostanie przerwana.

**Zaimplementuj Samonaprawianie.** W przypadku wystąpienia problemu powoduje przerwanie dostępności aplikacji, wykrywanie przez system monitorowania może inicjować działania samonaprawiania aplikacji, takie jak opróżnianie wystąpienia uszkodzonego i jego ponowne wdrożenie. Najprawdopodobniej wymaga to centralne rozwiązanie do monitorowania zintegrowane z hybrydową ciągłą integracją oraz potokiem ciągłego dostarczania (CI/CD). Aplikacja jest zintegrowana z systemem monitorowania, aby identyfikować problemy, które mogą wymagać ponownego wdrożenia składnika aplikacji. System monitorowania może również wyzwalać hybrydową ciągłość/dysk CD, aby ponownie wdrożyć składnik aplikacji i potencjalnie inne zależne składniki w tej samej lub innej lokalizacji.

**Obsługa umów dotyczących poziomu usług (umowy SLA).** Dostępność ma kluczowe znaczenie dla wszelkich umów dotyczących zapewnienia łączności z usługami i aplikacjami, które są dostępne dla klientów. Każda lokalizacja, na której bazuje aplikacja hybrydowa, może mieć własną umowę SLA. Te różne umowy SLA mogą mieć wpływ na ogólną umowę SLA aplikacji hybrydowej.

## <a name="resiliency"></a>Odporność

Odporność na aplikacje hybrydowe i system mają na celu odzyskiwanie z błędów i kontynuowanie działania. Celem odporności jest przywrócenie aplikacji do stanu w pełni funkcjonalnym po wystąpieniu błędu. Strategie odporności obejmują rozwiązania, takie jak tworzenie kopii zapasowych, replikacja i odzyskiwanie po awarii.

W przypadku podstawowej dyskusji tego filaru zapoznaj się z [*odpornością*](https://docs.microsoft.com/azure/architecture/guide/pillars#resiliency) w pięciu filarach doskonałości architektury.

### <a name="resiliency-checklist"></a>Lista kontrolna dotycząca odporności

**Odkrywaj zależności odzyskiwania po awarii.** Odzyskiwanie po awarii w jednej chmurze może wymagać zmian w składnikach aplikacji w innej chmurze. Jeśli co najmniej jeden składnik z jednej chmury zostanie przełączona w tryb failover do innej lokalizacji w tej samej chmurze lub w innej chmurze, składniki zależne muszą być świadome tych zmian. Obejmuje to również zależności łączności. Odporność wymaga w pełni przetestowanego planu odzyskiwania aplikacji dla każdej chmury.

**Ustanów przepływ odzyskiwania.** Efektywny projekt przepływu odzyskiwania ocenia składniki aplikacji, aby można było obsłużyć bufory, ponawianie prób, ponawianie nieudanych transferów danych i, w razie potrzeby, powracać do innej usługi lub przepływu pracy. Należy określić, jaki mechanizm tworzenia kopii zapasowej ma być używany, co obejmuje procedura przywracania oraz częstotliwość jej testowania. Należy również określić częstotliwość przyrostowych i pełnych kopii zapasowych.

**Przetestować częściowe operacje odzyskiwania.** Częściowe odzyskiwanie części aplikacji może zapewnić zagwarantowanie użytkownikom, że wszystkie nie są dostępne. Ta część planu powinna mieć pewność, że częściowe przywracanie nie ma żadnych efektów ubocznych, takich jak usługa tworzenia kopii zapasowej i przywracania, która współdziała z aplikacją, aby bezpiecznie ją zamknąć przed utworzeniem kopii zapasowej.

**Określenie i przypisanie odpowiedzialności za odzyskiwanie po awarii.** W planie odzyskiwania należy opisać, kto i jakie role mogą inicjować akcje tworzenia kopii zapasowej i odzyskiwania, a także tworzyć i przywracać kopie zapasowe.

**Porównaj progi samonaprawiania z odzyskiwaniem po awarii.** Określ możliwości samonaprawiania aplikacji w celu automatycznego inicjowania odzyskiwania oraz czas wymagany do samonaprawiania aplikacji, które mają być uznawane za Niepowodzenie lub powodzenie. Określanie progów dla każdej chmury.

**Sprawdź dostępność funkcji odporności.** Określ dostępność funkcji i możliwości odporności dla każdej lokalizacji. Jeśli lokalizacja nie zapewnia wymaganych możliwości, rozważ integrację tej lokalizacji w scentralizowaną usługę, która zapewnia funkcje odporności.

**Określanie przestojów.** Określ oczekiwany czas przestoju spowodowany konserwacją aplikacji jako całościową i składnikiem aplikacji.

**Procedury rozwiązywania problemów z dokumentem.** Zdefiniuj procedury rozwiązywania problemów dotyczące ponownego wdrażania zasobów i składników aplikacji.

## <a name="manageability"></a>Możliwości zarządzania

Zagadnienia związane z zarządzaniem aplikacjami hybrydowymi mają kluczowe znaczenie dla projektowania architektury. Dobrze zarządzana aplikacja hybrydowa zapewnia infrastrukturę jako kod, który umożliwia integrację spójnego kodu aplikacji w typowym potoku programistycznym. Implementując spójne systemy i indywidualne testowanie zmian w infrastrukturze, można zapewnić wdrożenie zintegrowane, jeśli zmiany przechodzą testy, umożliwiając ich scalenie z kodem źródłowym.

Aby zapoznać się z podstawową dyskusją tego filaru, zobacz [*DevOps*](/azure/architecture/framework/#devops) w pięciu filarach dotyczących architektury.

### <a name="manageability-checklist"></a>Lista kontrolna zarządzania

**Implementuj monitorowanie.** Aby zapewnić zagregowany wgląd w kondycję i wydajność, należy użyć scentralizowanego systemu monitorowania składników aplikacji w chmurze. Ten system obejmuje monitorowanie zarówno składników aplikacji, jak i pokrewnych możliwości platformy.

Określ części aplikacji, które wymagają monitorowania.

**Zasady współrzędnych.** Każda lokalizacja, w której znajdują się aplikacje hybrydowe, może dysponować własnymi zasadami obejmującymi dozwolone typy zasobów, konwencje nazewnictwa, Tagi i inne kryteria.

**Zdefiniuj role i używaj ich.** Jako administrator bazy danych należy określić uprawnienia wymagane do różnych osób (na przykład właściciela aplikacji, administratora bazy danych i użytkownika końcowego), które muszą uzyskiwać dostęp do zasobów aplikacji. Te uprawnienia muszą zostać skonfigurowane w zasobach i wewnątrz aplikacji. System kontroli dostępu opartej na rolach (RBAC) umożliwia ustawianie tych uprawnień dla zasobów aplikacji. Te prawa dostępu są trudne, gdy wszystkie zasoby są wdrażane w jednej chmurze, ale wymagają jeszcze większej uwagi, gdy zasoby są rozkładane między chmurami. Uprawnienia do zasobów ustawionych w jednej chmurze nie mają zastosowania do zasobów ustawionych w innej chmurze.

**Użyj potoków ciągłej integracji/ciągłego wdrażania.** Potok ciągłej integracji i ciągłego programowania (CI/CD) może zapewnić spójny proces tworzenia i wdrażania aplikacji obejmujących wiele chmur oraz zapewniający gwarancję jakości infrastruktury i aplikacji. Ten potok umożliwia testowanie infrastruktury i aplikacji w jednej chmurze i wdrażanie jej w innej chmurze. Potok pozwala nawet wdrożyć pewne składniki aplikacji hybrydowej w jednej chmurze i innych składnikach w innej chmurze, głównie tworząc podstawę wdrożenia aplikacji hybrydowej. System ciągłej integracji/ciągłego wdrażania ma kluczowe znaczenie dla obsługi składników aplikacji zależnych od siebie podczas instalacji, takich jak aplikacja sieci Web, która wymaga parametrów połączenia z bazą danych.

**Zarządzanie cyklem życia.** Ponieważ zasoby aplikacji hybrydowej mogą obejmować lokalizacje, każda możliwość zarządzania cyklem życia jednej lokalizacji musi być zagregowana w jednej jednostce zarządzania cyklem życia. Należy rozważyć sposób ich tworzenia, aktualizowania i usuwania.

**Sprawdzanie strategii rozwiązywania problemów.** Rozwiązywanie problemów z aplikacją hybrydową obejmuje więcej składników aplikacji niż ta sama aplikacja, która jest uruchomiona w pojedynczej chmurze. Oprócz łączności między chmurami aplikacja jest uruchamiana na dwóch platformach zamiast jednej. Ważnym zadaniem w rozwiązywaniu problemów z aplikacjami hybrydowymi jest sprawdzenie zagregowanego monitorowania kondycji i wydajności składników aplikacji.

## <a name="security"></a>Zabezpieczenia

Zabezpieczenia są jednym z głównych zagadnień związanych z każdą aplikacją w chmurze i coraz bardziej krytyczne dla hybrydowych aplikacji w chmurze.

W przypadku podstawowej dyskusji tego filaru zapoznaj się z tematem [*zabezpieczenia*](https://docs.microsoft.com/azure/architecture/guide/pillars#security) w pięciu filarach dotyczących architektury.

### <a name="security-checklist"></a>Lista kontrolna zabezpieczeń

**Przyjmij naruszenie.** Jeśli jedna część aplikacji zostanie naruszona, upewnij się, że istnieją rozwiązania, aby zminimalizować rozmieszczenie naruszenia, nie tylko w tej samej lokalizacji, ale również w różnych lokalizacjach.

**Monitoruj dozwolony dostęp do sieci.** Określ zasady dostępu do sieci dla aplikacji, takie jak dostęp do aplikacji tylko z określonej podsieci i Zezwalaj na prawidłowe działanie aplikacji tylko dla minimalnych portów i protokołów między składnikami wymaganymi przez aplikację.

**Korzystanie z niezawodnego uwierzytelniania.** Niezawodny schemat uwierzytelniania ma kluczowe znaczenie dla bezpieczeństwa aplikacji. Rozważ użycie dostawcy tożsamości federacyjnych, który udostępnia funkcje logowania jednokrotnego i wykorzystuje co najmniej jeden z następujących schematów: Logowanie przy użyciu nazwy użytkownika i hasła, klucze publiczne i prywatne, uwierzytelnianie dwuskładnikowe i usługi uwierzytelniania wieloskładnikowego oraz zaufane grupy zabezpieczeń. Określ odpowiednie zasoby do przechowywania danych poufnych i innych wpisów tajnych w celu uwierzytelniania aplikacji, a także typy certyfikatów i ich wymagania.

**Użyj szyfrowania.** Określ, które obszary aplikacji używają szyfrowania, na przykład w przypadku przechowywania danych lub komunikacji z klientem i dostępu.

**Użyj bezpiecznych kanałów.** Bezpieczny kanał między chmurami ma kluczowe znaczenie dla zapewnienia bezpieczeństwa i kontroli uwierzytelniania, ochrony w czasie rzeczywistym, kwarantanny i innych usług w chmurach.

**Zdefiniuj role i używaj ich.** Implementowanie ról dla konfiguracji zasobów i dostępu z jedną tożsamością w chmurach. Określ wymagania kontroli dostępu opartej na rolach (RBAC) dla aplikacji i jej zasobów platformy.

**Inspekcja systemu.** Monitorowanie systemu umożliwia rejestrowanie i agregowanie danych zarówno ze składników aplikacji, jak i związanych z nimi operacji platformy w chmurze.

## <a name="summary"></a>Podsumowanie

Ten artykuł zawiera listę kontrolną rzeczy, które są ważne do uwzględnienia podczas tworzenia i projektowania aplikacji hybrydowych. Przeglądanie tych filarów przed wdrożeniem aplikacji uniemożliwi wykonywanie tych pytań w przestoju produkcyjnym i potencjalnie wymaga ponownego odwiedzania projektu.

Może to wyglądać jak czasochłonne zadanie wcześniej, ale w przypadku projektowania aplikacji w oparciu o te filary możesz łatwo uzyskać zwrot z inwestycji.

## <a name="next-steps"></a>Następne kroki

Więcej informacji zawierają następujące zasoby:

- [Chmura hybrydowa](https://azure.microsoft.com/overview/hybrid-cloud/)
- [Hybrydowe aplikacje w chmurze](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [Opracowywanie szablonów usługi Azure Resource Manager pozwalających zachować spójność w chmurze](https://aka.ms/consistency)
