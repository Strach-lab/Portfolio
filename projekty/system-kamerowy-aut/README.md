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
| Urządzenia w pojazdach | Expert Electronics MR9504 (MDVR), protokół JT808-2019 |
| Oprogramowanie serwerowe | CMSV6 (CMSServerV6) — platforma zarządzania flotą |
| Serwer | Windows Server 2022 na sprzęcie rack HPE Gen9 |
| Storage | RAID 6 na macierzy dyskowej HP Smart Array |
| Baza danych | MySQL (bundlowany z CMSV6) |
| Sieć WiFi | Ubiquiti U6 Mesh (tryb standalone AP) |
| Router | MikroTik RouterOS |
| Automatyzacja | PowerShell + Windows Task Scheduler |

---

## Jak to działa

```
[MDVR w aucie]
     │
     ├── 4G ──────────────────► [serwer CMSV6] ← podgląd live przez internet
     │
     └── WiFi (po powrocie) ──► [serwer CMSV6] ← automatyczny upload nagrań
```

1. MDVR nagrywa lokalnie na kartę SD przez całą dobę
2. Po powrocie na teren firmy łączy się z siecią WiFi
3. Serwer (usługa GPSDownSvr) wykrywa połączenie i pobiera nagrania
4. Pliki trafiają na macierz dyskową (~9 TB przestrzeni)
5. Nagrania dostępne przez interfejs webowy CMSV6

---

## Status projektu

### Ukończone ✅

- [x] Montaż i konfiguracja MDVRów w całej flocie (7 pojazdów)
- [x] Konfiguracja sieci WiFi (Ubiquiti U6 Mesh, tryb standalone)
- [x] Instalacja i konfiguracja serwera CMSV6 z bazą MySQL
- [x] Uruchomienie automatycznego pobierania nagrań przez WiFi dla całej floty
- [x] Diagnoza i naprawa błędu **Param:22** (GPSDownSvr zapętlał się na datach bez nagrań)
- [x] Wdrożenie skryptu `CMSV6_Reset_DownRelation.ps1` — codzienny reset stanu pobierania (Task Scheduler, 06:00)
- [x] Wdrożenie skryptu `CMSV6_Czyszczenie_DB.ps1` — usuwanie osieroconych rekordów z bazy
- [x] Wymiana uszkodzonego MDVR w jednym z pojazdów + aktualizacja konfiguracji w CMSV6
- [x] Dokumentacja techniczna całego projektu

### W toku / do zrobienia ⏳

- [ ] Przypisanie statycznych adresów IP dla wszystkich MDVRów (przez DHCP reservations)
- [ ] Weryfikacja nieaktywnej karty SIM w jednym z pojazdów (błąd CME ERROR: 10)
- [ ] Konfiguracja zdalnego dostępu dla zarządzających flotą

---

## Czego się nauczyłem

### Diagnostyka bez dokumentacji
Projekt był przejęty po poprzednim administratorze — bez dokumentacji, bez haseł, bez opisu konfiguracji. Cały stan systemu trzeba było odtworzyć przez analizę logów, bazy danych i zachowania urządzeń.

### Jak działa GPSDownSvr i błąd Param:22
Usługa oblicza datę startu pobierania jako `DZIŚ - Days` i ignoruje pole `BeginTime` z bazy. Jeśli parametr `Days` przekroczy retencję karty SD (~3–4 dni przy 256 GB), serwer zapętla się na datach bez nagrań. Rozwiązanie: trzymać `Days = 3` i codziennie resetować `BeginTime` skryptem PowerShell.

### Dual-ID w urządzeniach MR9504
MDVRy raportują się jednocześnie pod dwoma identyfikatorami (JT808 `118...` i UTID `018...`). Tylko jeden powinien być aktywny w tabeli `down_relation` — dublowanie powoduje konflikty pobierania.

### Stale records blokują rejestrację
Stary rekord w `down_relation` blokuje świeżą rejestrację urządzenia po wymianie sprzętu lub długiej przerwie. Usunięcie rekordu (bez restartu usługi — GPSDownSvr czyta tabelę dynamicznie) przywraca działanie natychmiast.

### RAID 1 ≠ ochrona przed korupcją danych
Doświadczenie z utratą danych przy przenoszeniu serwera. RAID 1 kopiuje błędy 1:1 — nie zastępuje backupu. Obecna konfiguracja: RAID 6.

### PowerShell do automatyzacji zadań serwera Windows
Skrypty do interakcji z MySQL przez PowerShell wymagają ostrożnej obsługi — stderr MySQL (ostrzeżenia o haśle w CLI) trafia do strumienia błędów PowerShella i musi być filtrowany, żeby nie psuć logiki skryptu.

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

Repozytorium zawiera wyłącznie dokumentację i skrypty automatyzacji — nie ma tu kodu źródłowego CMSV6 (oprogramowanie komercyjne). Dane wrażliwe (IP, loginy, hasła, dane firmy) są przechowywane wyłącznie wewnętrznie i nie są częścią tego repo.
