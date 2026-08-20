# 📷 Fleet MDVR Management System

System automatycznego pobierania nagrań z kamer samochodowych (MDVR) przez WiFi — wdrożony w środowisku produkcyjnym małej firmy.

---

## O projekcie

Firma eksploatuje flotę pojazdów wyposażonych w mobilne rejestratory wideo (MDVR). Każde auto nagrywa lokalnie na kartę SD. Problem: ręczne zbieranie nagrań z kart było czasochłonne i nieefektywne.

**Rozwiązanie:** Po powrocie pojazdu na teren firmy MDVR automatycznie łączy się z siecią WiFi i synchronizuje nagrania na serwer centralny — bez żadnej ingerencji człowieka.

Projekt obejmuje wdrożenie, diagnostykę i automatyzację całego stosu: od konfiguracji urządzeń w pojazdach, przez sieć, po serwer i bazę danych.

---

## Stack technologiczny

| Warstwa | Technologia |
|---------|-------------|
| Urządzenia w pojazdach | MDVR Expert Electronics MR9504, protokół JT808-2019 |
| Oprogramowanie serwerowe | CMSV6 (CMSServerV6) — platforma zarządzania flotą |
| Serwer | Windows Server 2022 na sprzęcie rack HPE Gen9 |
| Storage | RAID 6 na macierzy dyskowej HP Smart Array (~9 TB) |
| Baza danych | MySQL (bundlowany z CMSV6) |
| Sieć WiFi | Ubiquiti U6 Mesh (tryb standalone AP) |
| Router | MikroTik RouterOS |
| Automatyzacja | PowerShell + Windows Task Scheduler |
| Dostęp zdalny | Tailscale VPN + RDP, DWService jako backup out-of-band |

---

## Jak to działa

```
[MDVR w aucie]
     │
     │  (w trasie: brak łączności — nagrywanie lokalne na kartę SD)
     │
     └── WiFi (po powrocie) ──► [serwer CMSV6] ── automatyczny upload nagrań
```

1. MDVR nagrywa lokalnie na kartę SD przez całą dobę
2. Po powrocie na teren firmy łączy się z firmową siecią WiFi
3. Urządzenie rejestruje się na serwerze, deklarując typ sieci i wersję protokołu
4. Usługa download serwera wykrywa połączenie i zaciąga nagrania
5. Pliki trafiają na macierz dyskową, indeksowane w bazie i dostępne przez interfejs webowy

> Architektura jest świadomie **WiFi-only** — w pojazdach nie ma kart SIM. Ma to kluczowe implikacje
> diagnostyczne: brak danych z pojazdu prawie zawsze oznacza, że auta po prostu nie ma na terenie firmy,
> a nie że urządzenie jest uszkodzone. Nauczenie się odróżniania tych dwóch sytuacji zajęło mi
> więcej czasu niż jakikolwiek pojedynczy błąd techniczny w tym projekcie.

---

## Status projektu

### Ukończone ✅

- [x] Montaż i konfiguracja MDVRów w całej flocie
- [x] Konfiguracja sieci WiFi (Ubiquiti U6 Mesh, tryb standalone)
- [x] Instalacja i konfiguracja serwera CMSV6 z bazą MySQL
- [x] Uruchomienie automatycznego pobierania nagrań przez WiFi
- [x] Rozpracowanie i udokumentowanie **czterech różnych przyczyn** tego samego objawu `Param:22`
- [x] Ustalenie roli slotu konfiguracyjnego w rejestracji urządzenia (naprawa „nieuleczalnego" pojazdu)
- [x] Ustalenie różnicy między dwiema wersjami protokołu rejestracji i ich wpływu na pobieranie
- [x] Wdrożenie skryptów PowerShell do codziennej konserwacji stanu pobierania i czyszczenia bazy
- [x] Wdrożenie harmonogramu automatycznego restartu usług (przeciwdziałanie degradacji)
- [x] Inwentaryzacja kanałów całej floty — wykrycie kanałów bez kamery marnujących transfer
- [x] Rewizja dokumentacji przez konfrontację z faktycznym stanem systemu
- [x] Dokumentacja techniczna całego projektu

### W toku / do zrobienia ⏳

- [ ] Statyczne adresy IP dla wszystkich MDVRów (DHCP reservations na routerze)
- [ ] Wyłączenie martwych kanałów z planu pobierania (oszczędność transferu i miejsca)
- [ ] Oględziny jednego urwanego kabla kamery
- [ ] Domknięcie licencjonowania serwera (przejście z wersji Evaluation na perpetual)
- [ ] Uzyskanie dostępu do kontrolera sieci bezprzewodowej

---

## Czego się nauczyłem

### Diagnostyka bez dokumentacji

Projekt był przejęty po poprzednim administratorze — bez dokumentacji, bez haseł, bez opisu konfiguracji.
Cały stan systemu trzeba było odtworzyć przez analizę logów, bazy danych i zachowania urządzeń.

### Jeden kod błędu, cztery różne przyczyny

Najważniejsza lekcja całego projektu. `Param:22` w logach wyglądał jak konkretny kod błędu.
W rzeczywistości to **generyczny timeout „brak danych z urządzenia"** — i cztery zupełnie różne
usterki dają identyczny objaw:

1. **Degradacja usług serwera** — usługi po 2–3 dobach pracy ciągłej przestają obsługiwać
   wyszukiwanie plików. Objaw: cała flota pada naraz. Rozwiązanie: harmonogram auto-restartu
2. **Stan „limbo" w bazie** — urządzenie wypada z obu kubełków selekcji harmonogramu i staje się
   dla niego trwale niewidoczne
3. **Niezgodność zakresu dat z zawartością karty SD**
4. **Faktyczna usterka firmware urządzenia**

Praktyczny wniosek: **nigdy nie wysyłaj sprzętu do serwisu, dopóki nie zweryfikujesz go przy
świeżo zrestartowanym serwerze.** Zdarzenie, w którym padła cała flota, wyglądało dokładnie
jak seria awarii sprzętowych. Nie było nią.

### Numer slotu konfiguracyjnego zmieniał zadeklarowany typ sieci

Jeden pojazd miesiącami nie chciał się zarejestrować, mimo potwierdzonego połączenia z WiFi
i osiągalnego portu serwera. Przyczyną okazało się to, że adres serwera był wpisany w **drugim
slocie konfiguracji platformy zamiast pierwszym**.

Numer slotu determinuje profil sieciowy, jaki urządzenie **deklaruje** platformie —
slot 1 zgłasza WiFi, slot 2 zgłasza 4G. Usługa download filtruje urządzenia po **zadeklarowanym**
typie sieci, nie po faktycznym połączeniu. Urządzenie deklarujące 4G jest dla harmonogramu
niewidoczne, choćby siedziało na firmowym WiFi.

Przeniesienie wpisu o jeden slot wyżej naprawiło problem natychmiast.

### Dwa pola adresu serwera = dwie wersje protokołu

Urządzenia MR9504 mają dwa osobne pola adresu serwera, które rejestrują je pod różnymi
wersjami protokołu. Starsza wersja zwraca uchwyty plików **bez sufiksu numeru kanału**,
przez co serwer odrzuca rejestrację odtwarzania i pobieranie nigdy się nie udaje.

Wcześniejsza dokumentacja (moja własna) traktowała urządzenie zarejestrowane pod nowszym
protokołem jako „anomalię do naprawienia". Było odwrotnie — to był jedyny prawidłowo
skonfigurowany egzemplarz w całej flocie.

Lekcja miękka, ale ważniejsza od technicznej: **prefiks identyfikatora był korelacją, nie przyczyną.**
Zbudowałem regułę na wzorcu, który wyglądał na diagnostyczny, i przez kilka miesięcy naprawiałem
w złą stronę.

### Dokumentacja, której nikt nie konfrontuje z systemem, cicho gnije

Najbardziej pouczający dzień projektu. Weryfikacja statusu floty wykazała, że
**połowa moich własnych wpisów była nieprawdziwa**: trzy pojazdy opisane jako
uszkodzone działały bez zarzutu, dwa opisane jako nieobecne pobierały codziennie
od tygodni, a jeden „przewlekle uszkodzony" ustąpił sam po naprawach serwerowych.
Gotów byłem wysłać sprawny sprzęt do serwisu na podstawie własnej notatki.

Przyczyna była strukturalna: dokumentacja mieszała fakty trwałe z ulotnymi.
„Slot 1 zgłasza WiFi, slot 2 zgłasza 4G" jest prawdą mechaniczną i będzie
aktualne za rok. „Pojazd X jest w innym mieście" było prawdą przez trzy tygodnie.
Oba siedziały w jednym pliku, sformatowane tak samo, bez daty obserwacji — więc
jedno starzało się cicho, a drugie nie.

Wniosek wdrożeniowy: stan systemu ma być **generowany z systemu**, nie opisywany
ręcznie. Daty modyfikacji katalogów na dysku leżały tam przez cały czas i
odpowiadały na pytanie „kto bywa i kiedy" w dwie sekundy — dokładniej niż
narzędzie diagnostyczne, które w tym celu zbudowałem.

### Cisza urządzenia nie jest objawem technicznym

Reguła wyprowadzona z powyższego. Zanim ruszy jakakolwiek diagnostyka, musi być
potwierdzone, że urządzenie **było w zasięgu i mimo to nie zadziałało**. Bez tego
każdy log opisuje wyłącznie nieobecność. Ta sama pomyłka wystąpiła u mnie w obie
strony: uznałem obecne pojazdy za nieobecne, a chwilę później o mało nie wysłałem
do diagnostyki auta, które po prostu rzadko jeździ.

### Pusty plik, który waży 88 MB

Kanał bez podłączonej kamery nie produkuje pliku pustego — nagrywa normalnie
zakodowany strumień czarnej planszy z napisem VIDEO LOSS. Jedno z urządzeń robi
to przy 85 KB/s, co przy 17-minutowym nagraniu daje ok. 88 MB nieodróżnialne
rozmiarem od prawdziwego materiału.

Napisany przeze mnie filtr progu rozmiaru był tu bezużyteczny. Zadziałało dopiero
kryterium **bitrate'u** — kanał z obrazem 100–280 KB/s, martwy 1–3 KB/s — plus
porównanie w obrębie tego samego urządzenia dla przypadków skrajnych.

Drugi wniosek: takich plików nie wolno kasować automatem. Urządzenie odtworzy je
przy następnym transferze i powstanie pętla kasowania. Rozwiązanie jest
konfiguracyjne — wyłączyć kanał z planu pobierania — a nie porządkowe.

### PowerShell rozpakowuje jednoelementowe tablice

Błąd, który sam wprowadziłem i który wykryło dopiero narzędzie diagnostyczne
raportujące bezsensowną wartość. `return @(...)` z funkcji oddaje **skalar**, gdy
element jest jeden. Wtedy `$wynik[0]` indeksuje pierwszy **znak** stringa:
`COUNT(*) = 6669` odczytywało się jako `6`, a `ROW_COUNT() = 45` jako `5`.

Konstrukcja siedziała w trzech skryptach naraz, bo skopiowałem ją bez namysłu.
Poprawka to `return ,@(...)`. Lekcja szersza: liczniki w logach same są danymi
i też wymagają weryfikacji — sukces raportowany przez skrypt nie jest dowodem
sukcesu.

### `DELETE` ≠ `UPDATE`

Tabela stanu pobierania ma wartość statusu, która wygląda na „neutralną" (`0`), a w rzeczywistości
oznacza limbo — rekord wypada z obu warunków selekcji harmonogramu. Reset pola przez `UPDATE`
nie pomaga; wpisanie tam zera aktywnie **wprowadza** urządzenie w stan zablokowany.
Odblokowuje wyłącznie usunięcie rekordu, po którym urządzenie jest traktowane jak nowe.

### Sukces w logu nie zawsze oznacza sukces

Ok. 2% pobrań kończy się urwanym plikiem <1 MB — i jest logowane jako **sukces**.
Harmonogram nigdy ich nie ponawia, bo nie ma powodu. Wykryłem to dopiero porównując
liczbę rekordów w bazie z rzeczywistymi rozmiarami plików na dysku.
Bez ground-truth po stronie systemu plików ten problem byłby niewidoczny.

### Zadania Task Schedulera cicho umierają po zmianie hasła

Zadanie z `LogonType = Password` przestaje działać po resecie hasła konta — bez żadnego alertu.
Codzienna konserwacja nie działała tygodniami, zanim to zauważyłem.
Odtąd sprawdzanie „Last Run Result" jest na stałe w checkliście po każdej zmianie poświadczeń.

### RAID 1 ≠ ochrona przed korupcją danych

Doświadczenie z utratą danych przy przenoszeniu serwera. RAID 1 kopiuje błędy 1:1 —
nie zastępuje backupu. Obecna konfiguracja: RAID 6, ale wniosek jest ten sam.

### PowerShell do automatyzacji zadań serwera Windows

Skrypty do interakcji z MySQL przez PowerShell wymagają ostrożnej obsługi — stderr MySQL
(ostrzeżenia o haśle w CLI) trafia do strumienia błędów PowerShella i musi być filtrowany,
żeby nie psuć logiki skryptu. Podobnie ścieżki do logów wymagają `-LiteralPath`,
bo nazwy plików zawierają znaki specjalne.

---

## Struktura repozytorium

```
├── README.md
├── docs/
│   ├── konfiguracja-cmsv6.md          ← konfiguracja serwera i usług
│   ├── checklist-pojazdy.md           ← status każdego pojazdu
│   ├── mikrotik-dhcp-reservations.md  ← przypisania IP urządzeń
│   └── troubleshooting.md             ← znane problemy i rozwiązania
└── scripts/
    ├── CMSV6_Reset_DownRelation.ps1   ← codzienny reset stanu pobierania
    └── CMSV6_Czyszczenie_DB.ps1       ← czyszczenie bazy z osieroconych rekordów
```

---

## Uwagi

Repozytorium zawiera wyłącznie dokumentację i skrypty automatyzacji — nie ma tu kodu źródłowego CMSV6 (oprogramowanie komercyjne). Dane wrażliwe (nazwa firmy, adresy IP, loginy, hasła, numery rejestracyjne i identyfikatory urządzeń) są przechowywane wyłącznie wewnętrznie i nie są częścią tego repo.
