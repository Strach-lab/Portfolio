# SOLpio Formatki — generator arkuszy rozkroju płyt meblowych

Webowa aplikacja do składania zamówień na usługi cięcia, oklejania i frezowania płyt meblowych. Klient wypełnia formularz online — system generuje arkusz `.xlsx` kompatybilny z oprogramowaniem do optymalizacji rozkroju (e-rozrys) i automatycznie wysyła go do odpowiedniego oddziału firmy.

Projekt rozwiązuje realny problem: zamiast zbierać zamówienia telefonicznie lub mailem z ręcznie spisanymi wymiarami, klienci (stolarze, firmy meblowe) wypełniają ustrukturyzowany formularz — bez możliwości pominięcia kluczowych danych.

---

## ⚙️ Stack techniczny

| Warstwa | Technologia |
|---|---|
| Frontend | HTML + React 18 (CDN, bez build step) |
| Eksport XLSX | SheetJS |
| E-mail do klienta | EmailJS |
| E-mail z załącznikiem XLSX | Cloudflare Worker + Resend API |
| Hosting | Cloudflare Workers |
| Domena | własna, przekierowanie HTTP |

Całość działa jako **jeden plik HTML** bez backendu i systemu buildowania. Można otworzyć lokalnie lub wdrożyć na dowolnym hostingu statycznym.

---

## 🏗️ Architektura

```
Klient (przeglądarka)
    │
    ├─ wypełnia formularz (grupy kolorów, formatki, blaty)
    │
    ├─ [Wyślij zamówienie]
    │       │
    │       ├─ EmailJS ──────────────────► e-mail HTML do klienta
    │       │                              (czytelne podsumowanie + BCC do firmy)
    │       │
    │       └─ fetch → Cloudflare Worker ► Resend API ► e-mail z XLSX do oddziału
    │
    └─ [Pobierz .xlsx] ── lokalny download (SheetJS, bez serwera)
```

### Dlaczego dwa kanały e-mail?

- **EmailJS** — wysyła czytelny HTML bezpośrednio z przeglądarki; nie obsługuje załączników
- **Cloudflare Worker + Resend** — wysyła plik `.xlsx` jako załącznik; serwer SMTP dostawcy hostingu blokuje ruch z zakresów IP Cloudflare, stąd Resend jako relay

### Routing oddziałów

Jeden deployment obsługuje wiele oddziałów przez parametr URL:

```
https://domena/?branch=uslugi
https://domena/?branch=brodnica
https://domena/?branch=gdansk
https://domena/?branch=bydgoszcz
```

Zamówienie trafia automatycznie na właściwy adres e-mail — klient nie wie o istnieniu innych oddziałów.

---

## ✅ Status projektu

### Zrobione
- [x] Formularz formatek — grupy kolorów, wymiary, okleina, frezowanie
- [x] Obsługa płyt HDF (brak okleiny, frezowanie FR / FR LED)
- [x] Walidacja wymiarów (60–2760 mm)
- [x] Obsługa blatów postformingowych (osobna zakładka, osobny plik XLSX)
- [x] Eksport do `.xlsx` kompatybilnego z e-rozrys
- [x] Wysyłka e-mail do klienta (podsumowanie HTML)
- [x] Wysyłka pliku XLSX do firmy (Cloudflare Worker + Resend API)
- [x] Routing wielooddziałowy przez parametr `?branch=`
- [x] Motyw jasny / ciemny
- [x] Responsywność (mobile)
- [x] Pusty szablon XLSX z legendą (dla dużych klientów)

### W toku / planowane
- [ ] **Wizualny selektor kolorów** — zamiast ręcznego wpisywania kodu, klient wybiera dekor klikając na miniaturkę rzeczywistej próbki
  - Zbudowane scrapery katalogów (Kronospan, Egger, Woodeco, Swiss Krono)
  - Panele selekcji dla pracownika działu usług
  - Czeka na ostateczny wybór ~70 dekorów z ~1000 dostępnych
- [ ] **Rozpoznawanie szkicu ze zdjęcia** — AI (model wizyjny) rozpoznaje elementy i wymiary z rysunku technicznego lub szkicu, automatycznie wypełnia formularz
- [ ] Workflow aktualizacji katalogów dekorów (diff nowego vs. poprzedniego, alerty o usuniętych)

---

## 🔧 Wybrane wyzwania techniczne

**SMTP z Cloudflare Workers** — dostawca hostingu pocztowego blokuje egress IP Cloudflare. Rozwiązanie: Resend API jako relay, weryfikacja domeny przez rekordy DNS (DKIM, SPF, DMARC).

**Brak systemu buildowania** — cała aplikacja działa jako jeden plik HTML z React przez CDN. Ograniczenie: brak JSX (używane `React.createElement`), brak hot reload, wszystko inline. Zaleta: zero zależności przy wdrożeniu, działa na każdym hostingu statycznym.

**Scraping katalogów dekorów** — każdy producent ma inną strukturę:
- Kronospan: pliki ZIP z listingiem produktów
- Egger: wewnętrzne API + token CSRF wymagający Playwright do ekstrakcji
- Woodeco: paginacja przez kilka kolekcji
- Swiss Krono: agresywna ochrona Cloudflare (403), deferred — obejście przez PDF katalogu

**Routing wielooddziałowy bez backendu** — jeden deployment, wiele miejsc docelowych przez `?branch=` w URL. Klient nie ma możliwości samodzielnego wyboru odbiorcy zamówienia.

---

## 📁 Struktura repozytorium

```
e-rozrys-import/
├── index.html              # Cała aplikacja frontendowa
├── solpio-api-worker.js    # Kod Cloudflare Workera (API do wysyłki XLSX)
├── assets/
│   └── logo_transparent.png
└── README.md
```

> `solpio-api-worker.js` wdrażany jest ręcznie przez Cloudflare Dashboard — nie jest automatycznie deployowany z repozytorium.

---

## 📚 Czego się nauczyłem / tech notes

- Cloudflare Workers jako lekki backend bez serwera (fetch API, env secrets)
- Resend API do transakcyjnych e-maili z załącznikami
- SheetJS — generowanie `.xlsx` po stronie klienta bez backendu
- Playwright do ekstrakcji tokenów z Single Page Applications (Egger)
- React bez JSX i bez buildu — zaskakująco praktyczne dla małych projektów
- DNS: DKIM, SPF, DMARC — weryfikacja domeny nadawcy e-mail
- Routing wielooddziałowy przez URL params zamiast osobnych deploymentów
