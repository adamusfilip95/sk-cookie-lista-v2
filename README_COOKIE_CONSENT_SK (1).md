# Mojra.sk – Cookie Consent + GA4 (Consent Mode V2)

Implementace cookie banneru pro **mojra.sk** s podporou **Google Consent Mode V2**, **GA4** a **Google Ads** (jediný gtag.js, obojí `config`).

---

## 🎯 Cíl
- Nasadit **jediný balíček** pro mojra.sk: Consent Mode V2 + GA4 (ID: `G-64M04RPHJ9`) + Google Ads (ID: `AW-329379451`).
- **Odstranit** starý **Universal Analytics (UA)** a **duplicitní** načítání `gtag.js`.
- Zajistit správné **pořadí skriptů** – `consent default` musí běžet **před** načtením `gtag.js`.

---

## 📦 Soubory
- `cookie_consent_full.sk.pretty.html` – čitelná verze (pro review, kopírování do šablony).
- `cookie_consent_full.sk.min.html` – minifikovaná verze (doporučená pro produkci, pokud ji používáte).

> V případě, že nasazujete přímo ze `pretty` souboru, **zkopírujte** z něj část do `<head>` a část do `<body>` dle pokynů níže.

---

## 🧩 Identifikátory
- **GA4 Measurement ID (SK):** `G-64M04RPHJ9`
- **Google Ads ID (SK):** `AW-329379451`

*Obě měření běží přes **jediný** `gtag.js` a dvě `config` volání.*

---

## 🔧 Postup nasazení (mojra.sk)

### 1) Odstranit staré/duplicitní kódy
Na webu bývají zbytky historických snippetů. **Smažte**:
- **Universal Analytics (UA)** – nefunguje, Google ho ukončil:
  ```html
  <script async src="https://www.googletagmanager.com/gtag/js?id=UA-203239467-1"></script>
  <!-- a veškerá navazující volání gtag('config','UA-203239467-1', …) nebo ga('create'…) -->
  ```
- **Duplicitní načtení** `gtag.js` (nechte **jen 1×**). Např. tyto řádky pryč (a podobné):
  ```html
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-64M04RPHJ9&cx=c&gtm=..."></script>
  <script async src="https://www.googletagmanager.com/gtag/js?id=UA-203239467-1&cx=c&gtm=..."></script>
  <script async src="https://www.googletagmanager.com/gtag/js?id=AW-329379451"></script> <!-- separate include už nebude potřeba -->
  ```

> Důvod: `gtag.js` se **načítá jen jednou** (ideálně s `G-64M04RPHJ9`) a obě měření se zapnou přes `gtag('config', ...)`. Tím se vyhnete konfliktům a změť duplicít.

---

### 2) Vložení do `<head>` (musí být v tomto pořadí)
Z **`cookie_consent_full.sk.pretty.html`** zkopírujte **tuto část** do `<head>` vaší šablony:

```html
<!-- 1) Consent Mode V2: DEFAULT – musí běžet dřív než se načte gtag.js -->
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){ dataLayer.push(arguments); }
  gtag('consent', 'default', {
    ad_storage: 'denied',
    analytics_storage: 'denied',
    ad_user_data: 'denied',
    ad_personalization: 'denied',
    functionality_storage: 'granted',
    security_storage: 'granted'
  });
</script>

<!-- 2) Jediný include gtag.js (GA4) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-64M04RPHJ9"></script>

<!-- 3) Init + konfigurace GA4 + Ads (respektuje Consent Mode) -->
<script>
  gtag('js', new Date());
  gtag('config', 'G-64M04RPHJ9', { wait_for_update: 500 }); // GA4
  gtag('config', 'AW-329379451');                            // Google Ads
</script>
```

> **Poznámka:** Žádné další `gtag.js` include už v `<head>` ani jinde nesmí být.

---

### 3) Vložení do `<body>` (na konec, před `</body>`)
Z téhož souboru zkopírujte **CSS/HTML/JS lišty** (za `<body>`) úplně **na konec stránky**, těsně před `</body>`.

- Banner + tlačítka: slovensky
- Modal „Nastavenie súhlasu“: slovensky
- Kulaté tlačítko **„Spravovať cookies“** vlevo dole (barva `#49A4EE`)

> Pokud používáte **CSP** s `nonce`/`hash`, zvažte přesun inline `<style>` a `<script>` do samostatných souborů, nebo přidejte odpovídající nonce/hash.

---

### 4) Ověřit pořadí skriptů (důležité)
1. `dataLayer`, `function gtag(){…}` a `gtag('consent','default', …)`
2. **jediný** `gtag.js?id=G-64M04RPHJ9`
3. `gtag('js', new Date());`
4. `gtag('config','G-64M04RPHJ9', { wait_for_update: 500 })`
5. `gtag('config','AW-329379451')`

---

### 5) Google Ads konverze (pokud máte)
Vaše konverzní eventy zůstávají. Vzory (respektuje Consent Mode, odešle se až při `granted`):
```html
<script>
  gtag('event', 'conversion', {
    'send_to': 'AW-329379451/XXXXXXXXXXX',  // doplňte vlastní ID konverze
    'value':  1.0,
    'currency': 'EUR'
  });
</script>
```

---

### 6) Odkaz na cookies stránku
- Banner i modal odkazuje na: `https://mojra.sk/page/cookies`  
- Pokud se URL změní, upravte ji přímo v HTML.

---

### 7) CSP (Content-Security-Policy), pokud ji používáte
Doporučené zdroje:
- `script-src`: `https://www.googletagmanager.com https://www.google-analytics.com 'self'` (+ nonce/hash pro inline)
- `connect-src`: `https://www.google-analytics.com`
- `img-src`: `https://www.google-analytics.com data:`
- `style-src`: povolit inline `<style>` **nebo** přenést CSS do souboru a připojit nonce/hash

---

### 8) Cache / verzování
Pokud vyčleňujete CSS/JS do samostatných souborů, používejte verze (hash/param). Po nasazení vyčistěte cache (server/CDN).

---

## ✅ QA Checklist (SK)
1. Otevřít **mojra.sk** v **Incognito** → banner se zobrazí, GA4/Ads **neposílají** bez souhlasu.  
2. Klik **„Prijať všetko“** → zmizí banner, v `localStorage` vznikne `mojra_consent_v2`, GA4/Ads začnou posílat.  
3. Klik **„Odmietnuť“** → data se **neposílají**.  
4. Klik na kulaté tlačítko vlevo dole → otevře se modal a volby jdou měnit.  
5. DevTools Console: uvidíte `consent/update` v `dataLayer`.  
6. Reset pro testy:
   ```js
   localStorage.removeItem('mojra_consent_v2'); location.reload();
   ```
7. Ověřit v **GA4 DebugView** a **Google Tag Assistant**, že eventy jdou až po souhlasu.

---

## 📌 Poznámky
- Nepoužíváme GTM (Google Tag Manager) – běží přímo `gtag.js`.
- „Nevyhnutné“/funkční a security storage jsou **vždy granted** (kvůli základnímu chodu webu).  
- Souhlas je uložen v `localStorage` po dobu **180 dní**. Pro formální GDPR audit trail je možné doplnit uložení na server (timestamp, IP, volby).

---

## 🔎 Shrnutí změn (diff – co má pryč / co má zůstat)

**PRYČ:**
```diff
- <script async src="https://www.googletagmanager.com/gtag/js?id=UA-203239467-1"></script>
- gtag('config','UA-203239467-1', {...});
- <script async src="https://www.googletagmanager.com/gtag/js?id=G-64M04RPHJ9&cx=c&gtm=..."></script>   (duplicitní include)
- <script async src="https://www.googletagmanager.com/gtag/js?id=AW-329379451"></script>              (samostatný include už netřeba)
```

**ZŮSTAT (z balíčku):**
```html
<!-- preload consent default (v head) -->
<script>window.dataLayer=window.dataLayer||[];function gtag(){dataLayer.push(arguments)};gtag('consent','default',{...});</script>

<!-- jediný gtag.js + configy GA4 i Ads -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-64M04RPHJ9"></script>
<script>
  gtag('js', new Date());
  gtag('config','G-64M04RPHJ9',{'wait_for_update':500});
  gtag('config','AW-329379451');
</script>

<!-- v body: CSS/HTML/JS cookie lišty (slovenské texty) -->
```
