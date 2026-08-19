# 👨‍💻 Portfolio IT — Strach-lab

> 🇵🇱 Zbiór projektów IT zrealizowanych w środowisku produkcyjnym małej firmy meblowej. Obejmują infrastrukturę sieciową, automatyzację, aplikacje webowe i systemy monitoringu.  
> 🇬🇧 A collection of IT projects implemented in a small furniture manufacturing company. Covering network infrastructure, automation, web applications and monitoring systems.

**Legenda:** 🔓 pełne repozytorium z kodem/runbookiem · 📄 opis projektu (repozytorium prywatne, dane zanonimizowane)

---

## 🔓 Gotowe rozwiązania

### 💾 [syno-nfs-esxi-backups — Odzyskiwanie ESXi i backup VM](https://github.com/Strach-lab/syno-nfs-esxi-backups)
Runbook z dwóch powiązanych zadań wykonanych na hoście produkcyjnym bez dokumentacji i bez istniejących backupów: odzyskanie hasła root na ESXi 6.0 (edycja `shadow` na obu partycjach stanu — `sda5` i `sda6`) oraz backup wszystkich VM na Synology NAS przez NFS przy użyciu `vmkfstools`.

**Stack:** VMware ESXi 6.0, Dell PowerEdge R510, Synology RS422+, NFS v3, Kali Live, Bash

---

### 📗 [gsheets-supplier-autoformat — Automatyczne formatowanie arkuszy zamówień](https://github.com/Strach-lab/gsheets-supplier-autoformat)
Google Apps Script utrzymujący czytelność długich arkuszy zamówień u dostawców płyt meblowych: naprzemienne kolorowanie wierszy przy każdej edycji i skok do ostatniego wypełnionego wiersza przy otwarciu pliku. Wdrożone po migracji czterech plików LibreOffice (`.ods`) z dysku sieciowego do jednego arkusza Google z zakładkami per dostawca. W repo: skrypt, procedura migracji i instrukcja dla użytkowników.

**Stack:** Google Apps Script, Google Sheets, JavaScript, simple triggers (`onEdit` / `onOpen`)

---

## 📄 Projekty wewnętrzne

> Repozytoria prywatne ze względu na dane operacyjne — poniżej opisy architektury i rozwiązanych problemów.

### 📷 [Fleet MDVR Management System](./projekty/system-kamerowy-aut/)
Automatyczny system pobierania nagrań z kamer samochodowych przez WiFi. MDVR w każdym pojeździe synchronizuje nagrania na serwer centralny bez ingerencji człowieka po powrocie do bazy.

**Stack:** CMSV6, Windows Server, MySQL, PowerShell, MikroTik RouterOS, Ubiquiti U6 Mesh

---

### 🪵 [e-rozrys-import — Generator arkuszy rozkroju](./projekty/e-rozrys-import/)
Webowa aplikacja do składania zamówień na usługi cięcia i oklejania płyt meblowych. Klient wypełnia formularz online — system generuje plik `.xlsx` kompatybilny z oprogramowaniem do optymalizacji rozkroju i wysyła go automatycznie do odpowiedniego oddziału.

**Stack:** React 18 (CDN), SheetJS, Cloudflare Workers, Resend API, EmailJS

---

### 🏭 [System Śledzenia Produkcji](./projekty/produkcja-tablety/)
Tablety Android w trybie kiosku na hali produkcyjnej zastępujące papierowe karty pracy. Operatorzy rejestrują produkcję per maszyna i zlecenie — dane trafiają do Google Sheets, raporty generowane automatycznie.

**Stack:** HTML/JS (Vanilla), Google Apps Script, Google Sheets, Chart.js, Android kiosk (ADB)

---

## 🔧 Tech stack (ogólnie)

`MikroTik` `Ubiquiti UniFi` `VMware ESXi` `Windows Server` `PowerShell` `Python` `Bash` `Google Apps Script` `Cloudflare Workers` `Docker` `Synology NAS` `NFS` `Tailscale`
