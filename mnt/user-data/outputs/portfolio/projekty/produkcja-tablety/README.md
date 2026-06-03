# 🏭 System Śledzenia Produkcji

System do rejestracji i raportowania produkcji w środowisku produkcyjnym małej firmy. Operatorzy wprowadzają dane przez tablety działające w trybie kiosku — bez możliwości wyjścia poza aplikację. Dane trafiają do Google Sheets, skąd automatycznie generowane są raporty dla kierownictwa.

Projekt rozwiązuje realny problem: brak cyfrowej rejestracji produkcji per operator, per maszyna i per zlecenie — zastępując papierowe karty pracy.

---

## 🛠️ Stack technologiczny

- **Frontend:** HTML5, CSS3, JavaScript (Vanilla) — bez frameworków
- **Backend:** Google Apps Script (Web App)
- **Baza danych:** Google Sheets
- **Serwer plików:** NAS (lokalnie, sieć wewnętrzna)
- **Urządzenia:** Tablety Android w trybie kiosku (ADB + Chrome)
- **Automatyzacja:** Google Apps Script triggers (czasowe)
- **Wykresy:** Chart.js
- **Sieć Wi-Fi:** enterprise access point

---

## 🏗️ Architektura

```
[Tablety Android — tryb kiosk]
        │  Chrome (fullscreen, zablokowana nawigacja)
        │  HTTP GET/POST
        ▼
[NAS — serwer plików lokalnych]
        │  serwuje pliki HTML (frontend)
        │
        ├── kiosk.html            ← menu główne tabletu
        ├── okleiniarki.html      ← wybór oklejarki
        ├── pily.html             ← wybór piły
        ├── produkcja-form.html   ← formularz wpisu produkcji
        ├── obecnosc.html         ← rejestracja obecności
        ├── warehouse-view.html   ← widok dla magazynu
        └── index.html            ← dashboard zarządzający
        │
        │  fetch() → HTTP POST / GET
        ▼
[Google Apps Script — Web App]
        │  doPost() — zapis danych
        │  doGet()  — odczyt (lista operatorów, status zleceń)
        ▼
[Google Sheets — baza danych]
        │
        ├── Arkusze per maszyna (11 arkuszy)
        ├── Arkusz Operatorzy
        ├── Arkusz Obecność
        └── Arkusz Pamięć Magazynu
        │
        └── Automatyczne raporty e-mail
            (tygodniowe + miesięczne, HTML + PDF)
```

---

## ✅ Status projektu

### Zrobione
- [x] Formularz wprowadzania produkcji (oklejarki, piły, CNC)
- [x] Dynamiczna lista operatorów z Google Sheets
- [x] Rejestracja obecności z licznikiem dni roboczych
- [x] Automatyczne raporty tygodniowe i miesięczne (e-mail + PDF)
- [x] Raport niestandardowy per operator za dowolny zakres dat
- [x] Dashboard z wykresami (Chart.js) i śledzeniem zleceń
- [x] Widok magazynowy z potwierdzaniem odbioru zleceń
- [x] Tryb kiosk na tabletach Android (ADB + Chrome)
- [x] Blokada wierszy starszych niż 2 dni (ochrona danych historycznych)
- [x] Auto-formatowanie dat wpisywanych jako liczby

### W toku
- [ ] Integracja z wewnętrznym systemem magazynowym (WMS)
- [ ] Automatyczne rozliczanie zużycia materiałów per zlecenie
- [ ] Dashboard porównujący produkcję z wydaniami materiałów
- [ ] Centralne zarządzanie aktualizacjami tabletów

---

## 🧠 Ciekawe problemy techniczne

**Podwójne enkodowanie UTF-8 → Windows-1252** — Apps Script przy niektórych konfiguracjach serwerowych zwracał ciągi enkodowane dwa razy. Efekt: `ðŸ"Š` zamiast `📊`. Naprawione chirurgicznym mapowaniem uszkodzonych sekwencji zamiast szerokiego re-enkodowania.

**gviz API — cicho psujące się zapytania** — Google Sheets Visualization API wymaga czystego URL bez parametrów `tq=`. Dodanie `select * limit 400 offset 1` powodowało całkowite milczące odrzucenie odpowiedzi bez błędu w konsoli.

**appendRow na preformatowanych arkuszach** — preformatowane puste wiersze powodują że `appendRow` wpisuje dane do wiersza 1000+ zamiast pierwszego logicznie pustego. Rozwiązanie: funkcja szukająca pierwszego faktycznie pustego wiersza.

**Tryb kiosk bez MDM** — blokada tabletu bez Mobile Device Management, tylko Android ADB + zabezpieczenia JS po stronie frontendowej. Proste, skuteczne na małą skalę.

---

## 📁 Struktura repozytorium

```
produkcja-tablety/
├── README.md
├── CHANGELOG.md
├── frontend/
│   ├── kiosk.html
│   ├── okleiniarki.html
│   ├── pily.html
│   ├── produkcja-form.html
│   ├── obecnosc.html
│   ├── warehouse-view.html
│   └── index.html
├── scripts/
│   └── apps-script.js
└── docs/
    ├── struktura-sheets.md
    ├── konfiguracja-kiosk.md
    └── instrukcja-operatora.md
```

---

## ⚠️ Uwagi

Przed publicznym udostępnieniem kodu upewnij się, że pliki HTML nie zawierają wrażliwych URL-i (adresy Web App, identyfikatory arkuszy). W środowisku produkcyjnym przechowuj je w osobnym pliku konfiguracyjnym nietrackowanym przez git.
