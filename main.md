# Ultimate PiHole Tutorial

### PiHole

Oprogramowanie `Pi-hole` to rodzaj serwera proxy DNS, który działa jako  instancja filtrująca ruch sieciowy poprzez blokowanie reklam i innych niechcianych treści internetowych. Jest to narzędzie typu open-source, które działa głównie jako system blokujący zapytania o adresy internetowe wykorzystywane przez serwisy reklamowe i śledzące użytkowników.

Główną funkcjonalnością Pi-hole jest wykorzystanie listy czarnych adresów (blacklist) w celu zapobiegania dostępowi do adresów URL i serwerów, które są znane z dostarczania reklam, śledzenia użytkowników lub zawierania potencjalnie szkodliwych treści. Jako serwer proxy DNS, Pi-hole działa na etapie translacji nazw domenowych na adresy IP, co umożliwia mu wykrycie i zablokowanie żądanych adresów internetowych zawierających treści reklamowe, śledzące lub potencjalnie szkodliwe. Centralnym elementu systemu jest `Gravity` które jest mechanizmem używanym przez Pi-hole do tworzenia, aktualizowania i zarządzania black listą.
W przedstawionej konfiguracji Gravity będzie jednym z elementów które będziemy sychronizować między instacjami PiHole w konfiugracji HA. Będziemy do tego używac oprogramowania Gravity sync. 

W ramach naszej konfiguracji będziemy również korzystać z fukcji która nazywa się `unbound`. Jest to funkcja rozwiązywania nazw (resolver) działającym jako serwer DNS, jest ona jednym z wyborów do implementacji funkcji serwera DNS, który można zintegrować z Pi-hole w celu poprawy prywatności, bezpieczeństwa i wydajności sieci. Na czym to polega? W skrócie, Unbound jest serwerem DNS zapewniającym lokalne rozwiązywanie nazw, zabezpieczenia komunikacji DNS, buforowanie oraz wysoką wydajność. Co taka konfiguracja nam daje?

    1. Lokalne rozwiązywanie nazw: Unbound może działać jako lokalny resolver DNS, co oznacza, że może być skonfigurowany do obsługi zapytań DNS bez konieczności polegania na zewnętrznych serwerach DNS dostarczanych przez dostawców usług internetowych.

    2. Bezpieczeństwo: Jest w stanie obsługiwać i przetwarzać zabezpieczone protokoły takie jak DNS over TLS (DoT) oraz DNS over HTTPS (DoH), co pozwala na zabezpieczenie komunikacji między klientami a serwerem DNS.

    3. Buforowanie: Unbound może buforować odpowiedzi DNS, czyli przechowywać lokalnie wyniki wcześniej rozwiązanych zapytań. Dzięki temu, gdy jest ponownie wysyłane to samo zapytanie, może szybko zwrócić odpowiedź, bez konieczności ponownego kierowania się do zewnętrznych serwerów DNS.

    4. Wydajność: Jest projektowany z myślą o wydajności i efektywności. Może obsługiwać duże ilości zapytań DNS równocześnie, co jest istotne w sieciach o dużej aktywności.
___
### Konfiguracja HA (High availability)
High availability (wysoka dostępność) jest terminem odnoszącym się do architektury lub konfiguracji systemów, które są zaprojektowane w taki sposób, aby zapewnić nieprzerwane działanie usług lub aplikacji nawet w przypadku awarii sprzętu, oprogramowania lub innego rodzaju zakłóceń.

W przypadku konkretnie oprogramowania Pi-hole konfiguracja HA może być osiągnięta poprzez zastosowanie redundancji w postaci zapasowych serwerów lub klastrów, które mogą przejąć obsługę w razie awarii głównego serwera Pi-hole. Można również skonfigurować mechanizmy failover, które automatycznie przeniosą ruch sieciowy na inne serwery w przypadku awarii głównego.

W prezentowanym rozwiązaniu zostanie użyta wirtualna maszyna hostowana na domowym serwerze z hypervisorem typu-1 Proxmox oraz druga instacja hostowana na maszynie RaspberyPI. 

### Schemat struktury:

![Schemat](/pictures/PiHole+HAv2.png)

___

### Instalacja 

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git -y
curl -sSL https://install.pi-hole.net | bash
```

```bash
  [i] Root user check
  [i] Script called with non-root privileges
      The Pi-hole requires elevated privileges to install and run
      Please check the installer for any concerns regarding this requirement
      Make sure to download this script from a trusted source

  [✓] Sudo utility check

  [✓] Root user check

        .;;,.
        .ccccc:,.
         :cccclll:.      ..,,
          :ccccclll.   ;ooodc
           'ccll:;ll .oooodc
             .;cll.;;looo:.
                 .. ','.
                .',,,,,,'.
              .',,,,,,,,,,.
            .',,,,,,,,,,,,....
          ....''',,,,,,,'.......
        .........  ....  .........
        ..........      ..........
        ..........      ..........
        .........  ....  .........
          ........,,,,,,,'......
            ....',,,,,,,,,,,,.
               .',,,,,,,,,'.
                .',,,,,,'.
                  ..'''.

  [i] SELinux not detected
  [✓] Update local cache of available packages

  [✓] Checking apt-get for upgraded packages... up to date!

  [i] Checking for / installing Required dependencies for OS Check...
  [✓] Checking for grep
  [i] Checking for dnsutils (will be installed)
  [i] Waiting for package manager to finish (up to 30 seconds)
  [i] Processing apt-get install(s) for: dnsutils, please wait...
----------------------------------------------------------------------
Selecting previously unselected package bind9-dnsutils.
(Reading database ... 44916 files and directories currently installed.)
Preparing to unpack .../bind9-dnsutils_1%3a9.16.44-1~deb11u1_armhf.deb ...
Unpacking bind9-dnsutils (1:9.16.44-1~deb11u1) ...
Selecting previously unselected package dnsutils.
Preparing to unpack .../dnsutils_1%3a9.16.44-1~deb11u1_all.deb ...
Unpacking dnsutils (1:9.16.44-1~deb11u1) ...
Setting up bind9-dnsutils (1:9.16.44-1~deb11u1) ...
Setting up dnsutils (1:9.16.44-1~deb11u1) ...
Processing triggers for man-db (2.9.4-2) ...
----------------------------------------------------------------------
  [✓] Supported OS detected
  [i] Checking for / installing Required dependencies for this install script...
  [✓] Checking for git
  [✓] Checking for iproute2
  [i] Checking for dialog (will be installed)
  [✓] Checking for ca-certificates
  [i] Waiting for package manager to finish (up to 30 seconds)
  [i] Processing apt-get install(s) for: dialog, please wait...
----------------------------------------------------------------------
Selecting previously unselected package dialog.
(Reading database ... 44936 files and directories currently installed.)
Preparing to unpack .../dialog_1.3-20201126-1_armhf.deb ...
Unpacking dialog (1.3-20201126-1) ...
Setting up dialog (1.3-20201126-1) ...
Processing triggers for man-db (2.9.4-2) ...
  [i] Root user check
  [i] Script called with non-root privileges
      The Pi-hole requires elevated privileges to install and run
      Please check the installer for any concerns regarding this requirement
      Make sure to download this script from a trusted source

  [✓] Sudo utility check

  [✓] Root user check

        .;;,.
        .ccccc:,.
         :cccclll:.      ..,,
          :ccccclll.   ;ooodc
           'ccll:;ll .oooodc
             .;cll.;;looo:.
                 .. ','.
                .',,,,,,'.
              .',,,,,,,,,,.
            .',,,,,,,,,,,,....
          ....''',,,,,,,'.......
        .........  ....  .........
        ..........      ..........
        ..........      ..........
        .........  ....  .........
          ........,,,,,,,'......
            ....',,,,,,,,,,,,.
               .',,,,,,,,,'.
                .',,,,,,'.
                  ..'''.

  [i] SELinux not detected
  [✓] Update local cache of available packages

  [✓] Checking apt-get for upgraded packages... up to date!

  [i] Checking for / installing Required dependencies for OS Check...
  [✓] Checking for grep
  [i] Checking for dnsutils (will be installed)
  [i] Waiting for package manager to finish (up to 30 seconds)
  [i] Processing apt-get install(s) for: dnsutils, please wait...
----------------------------------------------------------------------
Selecting previously unselected package bind9-dnsutils.
(Reading database ... 44916 files and directories currently installed.)
Preparing to unpack .../bind9-dnsutils_1%3a9.16.44-1~deb11u1_armhf.deb ...
Unpacking bind9-dnsutils (1:9.16.44-1~deb11u1) ...
Selecting previously unselected package dnsutils.
Preparing to unpack .../dnsutils_1%3a9.16.44-1~deb11u1_all.deb ...
Unpacking dnsutils (1:9.16.44-1~deb11u1) ...
Setting up bind9-dnsutils (1:9.16.44-1~deb11u1) ...
Setting up dnsutils (1:9.16.44-1~deb11u1) ...
Processing triggers for man-db (2.9.4-2) ...
----------------------------------------------------------------------
  [✓] Supported OS detected
  [i] Checking for / installing Required dependencies for this install script...
  [✓] Checking for git
  [✓] Checking for iproute2
  [i] Checking for dialog (will be installed)
  [✓] Checking for ca-certificates
  [i] Waiting for package manager to finish (up to 30 seconds)
  [i] Processing apt-get install(s) for: dialog, please wait...
----------------------------------------------------------------------
Selecting previously unselected package dialog.
(Reading database ... 44936 files and directories currently installed.)
Preparing to unpack .../dialog_1.3-20201126-1_armhf.deb ...
Unpacking dialog (1.3-20201126-1) ...
Setting up dialog (1.3-20201126-1) ...
----------------------------------------------------------------------
```
## Interfejs webowy
Wejdźmy do naszego interfejsu webowego: 

Przykładowy adres IP
```IP
192.168.10.5/web
```

Zwróć uwagę na wyszczególnione elementy: 
![Domains on Adlist](/pictures/web_1.png)

**Domains on Adlist** pokazuję nam liczbę akutalnych filtrów domenowych systemu gravity.
Im większa wartość tego parametru tym więcej domen jest blokowanych. Na dzień pisania artykułu wartośc tego parametru po instalacji PiHole to **145 893**. Jednak za chwilę mocno to zwiększymy za pomocą przygotowanych black list domenowych. 
<br>
<br>

## Query logs
![Query logs](/pictures/web_query_log.png)
Query logs (dzienniki zapytań) w Pi-hole to rejestr zawierający szczegółowe informacje na temat wszystkich zapytań DNS, które są przechodzone przez serwer DNS Pi-hole. Te dzienniki zawierają rejestr wszystkich zapytań DNS dokonywanych przez urządzenia w sieci, które korzystają z Pi-hole jako serwera DNS.

Informacje zawarte w query logs obejmują:

    1. Czas i data zapytania: Timestamp (znacznik czasowy) określający dokładną datę i godzinę każdego zapytania DNS.

    2. Adres IP źródła zapytania: Adres IP urządzenia, które wysłało zapytanie DNS do serwera Pi-hole.

    3. Nazwa domenowa (URL) zapytania: Adres URL lub nazwa domenowa, którą próbowano rozwiązać za pomocą zapytania DNS.

    4. Typ zapytania DNS: Rodzaj zapytania DNS, na przykład A (zwraca adres IPv4), AAAA (zwraca adres IPv6), PTR (odwrotne odwzorowanie adresu IP na nazwę domenową) itp.

    5. Status zapytania: Informacja czy zapytanie zostało rozwiązane poprawnie (zwrócono odpowiedź) lub czy zostało zablokowane przez Pi-hole (zwrócono odpowiedź zablokowaną).

    6. Odpowiedź DNS: Odpowiedź zwrócona przez serwer DNS w przypadku rozwiązania zapytania.

Query logs pozwalaja na śledzenie aktywności sieciowej, identyfikowanie zapytań DNS pochodzących z różnych urządzeń, monitorowanie blokowanych reklam i niechcianych treści oraz analizowanie ruchu sieciowego. Te dzienniki są przydatne do zrozumienia, jakie zasoby są pobierane przez urządzenia w sieci oraz jak Pi-hole zarządza ruchem sieciowym i blokuje niechciane treści.

To co nas najbardziej interesuje to sekcja `Status` w której znajdują się aktualnie dwie wartości:
![Query logs](/pictures/web_2.png)
+ **Blocked (gravity)** -  oznacza to że system Gravity dokonał zablokowania domeny, co oznacza że znajdowałą się ona na naszej black liscie. System gravity odczytał próbę połączenia z tą domeną, zweryfikował ją i potwierdził, że znajduje się ona na naszej black liscie co poskutkowało jej zablokowablokowaniem, powodując tym samym że nawiązanie połączenia z domeną nie zostało ustanowione, ergo potencjalna reklama się nie wyświetliła. 
+ **OK(answered by dns.google#53)** - Oznacza że translacja adresu domeny została wykonana za pośrednictwem serera DNS google. Korzystanie z publicznych serwerów DNS, może mieć zarówno zalety, jak i wady z punktu widzenia prywatności, bezpieczeństwa i wydajności. Aby nie korzystać z publicznych sererwów DNS w ramach naszej konfiguracji będziemy korzstać ze wspomnianej opcji `unbound`, którą przygotujemy w dalszej części artykułu. Istnieje kilka powodów, dla których korzystanie z własnej instancji serwera DNS jest bardziej korzystne w porównaniu do użytkowania publicznych serwerów DNS:
    
    1. ***Prywatność:*** Publiczne serwery DNS, nawet te znane i zaufane, często gromadzą dane o zapytaniach DNS, co może potencjalnie naruszać prywatność użytkowników. Dostawcy publicznych serwerów DNS mogą analizować te dane dla celów statystycznych lub reklamowych. Posiadanie własnej instancji serwera DNS daje kontrolę nad danymi i ogranicza możliwość śledzenia.

    2. ***Bezpieczeństwo:*** Publiczne serwery DNS mogą być podatne na ataki typu cache poisoning lub DNS spoofing, które mogą prowadzić do przekierowania ruchu internetowego na fałszywe strony lub do ataków typu man-in-the-middle. Samodzielnie zarządzany serwer DNS może być lepiej zabezpieczony i mniej podatny na tego rodzaju ataki.

    3. ***Filtrowanie i kontrola:*** Korzystając z własnej instancji serwera DNS, można łatwiej zaimplementować filtrowanie treści, blokowanie reklam, niechcianych treści, a także kontrolować, które zapytania DNS są przekierowywane i jakie odpowiedzi są zwracane.

    4. ***Szybkość i wydajność:*** Lokalny serwer DNS może być szybszy od zdalnych serwerów DNS, ponieważ redukuje opóźnienia wynikające z transmisji danych przez internet. <br>

Z tego menu mamy w sekcji `Actions` mamy możliwość również skorzstać z dodania domeny do `Blacklist` lub do `Whitelist` zależnie od potrzeb. 


## Long term data
![Long term data](/pictures/web_3.png)

Sekcja "Long-term data" odnosi się do funkcji, w której przechowywane są długoterminowe dane dotyczące statystyk dotyczących ruchu sieciowego oraz blokowania reklam.
Ta sekcja oferuje możliwość przeglądania danych z dłuższego okresu czasu, co pozwala na analizę i zrozumienie trendów dotyczących aktywności sieciowej, ilości blokowanych zapytań DNS oraz innych istotnych informacji związanych z ruchem w sieci.

W ramach sekcji "Long-term data" w Pi-hole można oczekiwać następujących informacji:

    1. Statystyki ruchu sieciowego: Dane na temat ogólnej ilości zapytań DNS oraz ilości zapytań, które zostały zablokowane. Te statystyki mogą być wyświetlane w formie wykresów lub tabel, umożliwiając użytkownikowi analizę częstotliwości zapytań DNS w czasie.

    2. Statystyki dotyczące blokowania reklam: Informacje o ilości reklam i innych niechcianych zasobów zablokowanych przez Pi-hole. Można sprawdzać, jak wiele reklam zostało zatrzymanych w określonym okresie czasu, co może być pomocne w ocenie skuteczności blokowania treści reklamowych.

    3. Wykresy i raporty: Sekcja "Long-term data" może zawierać wykresy i raporty, które wizualizują te dane, umożliwiając użytkownikowi łatwiejsze zrozumienie i analizę informacji dotyczących ruchu sieciowego.

Długoterminowe dane w Pi-hole są przydatne do monitorowania trendów, identyfikowania wzorców zachowania sieciowego oraz oceny efektywności blokowania reklam i niechcianych treści. Umożliwiają one użytkownikowi podejmowanie decyzji dotyczących optymalizacji konfiguracji Pi-hole, jak również śledzenie zmian w aktywności sieciowej w dłuższej perspektywie czasowej.

## Groups
![Long term data](/pictures/groups.png)


W Pi-hole, sekcja "Groups" odnosi się do funkcji umożliwiającej grupowanie urządzeń lub adresów IP w logiczne zbiory dla bardziej skoncentrowanego zarządzania ruchem sieciowym i aplikowaniu reguł blokowania lub innych ustawień.

Funkcja "Groups" umożliwia użytkownikom tworzenie różnych grup, do których można przypisywać urządzenia na podstawie ich adresów IP lub nazw hostów. Dzięki temu można łatwiej zarządzać regułami blokowania reklam, filtrowania ruchu lub innymi ustawieniami specyficznymi dla tych grup.

W sekcji "Groups" w Pi-hole możesz się spodziewać następujących funkcji:

    Tworzenie grup: Użytkownicy mogą tworzyć różne grupy na podstawie potrzeb lub kategorii urządzeń, np. "Rodzina", "Praca", "Goście" itp.

    Przypisywanie urządzeń: Można przypisywać urządzenia do konkretnych grup na podstawie ich adresów IP lub nazw hostów.

    Zastosowanie reguł dla grup: Użytkownicy mogą stosować różne reguły blokowania reklam lub filtrowania ruchu do określonych grup, co pozwala na bardziej precyzyjne zarządzanie tym, jakie zasoby sieciowe są dostępne dla danej grupy urządzeń.

    Analiza statystyk grup: Sekcja "Groups" może również oferować statystyki związane z aktywnością sieciową konkretnych grup, takie jak ilość zapytań DNS, ilość zablokowanych zapytań itp.

Dzięki tej funkcji administrator lub użytkownik Pi-hole może lepiej kontrolować i zarządzać ruchem sieciowym w zależności od potrzeb różnych grup urządzeń. Jest to szczególnie przydatne w domowych sieciach, gdzie różne urządzenia mogą wymagać różnych ustawień filtracji reklam lub reguł dostępu do zasobów sieciowych.




