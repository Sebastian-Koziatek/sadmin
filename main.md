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

![Anagram](/pictures/PiHole+HAv2.png)
