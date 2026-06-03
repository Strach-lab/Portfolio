# 👨‍💻 Portfolio IT — Strach-lab

> 🇵🇱 Zbiór projektów IT zrealizowanych w środowisku produkcyjnym małej firmy meblowej. Obejmują infrastrukturę sieciową, automatyzację, aplikacje webowe i systemy monitoringu.  
> 🇬🇧 A collection of IT projects implemented in a small furniture manufacturing company. Covering network infrastructure, automation, web applications and monitoring systems.

---

## 📁 Projekty

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

`MikroTik` `Ubiquiti UniFi` `Windows Server` `PowerShell` `Python` `Google Apps Script` `Cloudflare Workers` `Docker` `Synology NAS` `Tailscale`
