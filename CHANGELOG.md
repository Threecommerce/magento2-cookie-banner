# Changelog

Tutte le modifiche rilevanti a questo modulo sono documentate in questo file.

Il formato è basato su [Keep a Changelog](https://keepachangelog.com/it/1.1.0/)
e questo progetto aderisce al [Semantic Versioning](https://semver.org/lang/it/).

## 1.5.2

- Fixed: compatibilità con PHP 8.5. Tolta la chiamata a `curl_close()` nella scansione dei cookie: è deprecata in PHP 8.5 e Magento trasforma l'avviso in errore, quindi la scansione si fermava. Nessun cambiamento sulle versioni di PHP precedenti.

## 1.5.1

- Fixed: nella griglia *Registro cookie* la colonna Azioni era vuota (mancavano *Modifica* e *Cancella*).

## 1.5.0

- Added: **registro dei cookie**, unica fonte per il banner e per la cookie policy. Nuova griglia admin Stores → **Registro cookie** (sostituisce *Cookie rilevati*) con filtri per stato, categoria e origine, modifica in linea, esportazione CSV, azioni di massa *Pubblica* / *Nascondi* / *Cancella*, pulsanti *Aggiungi cookie* ed *Esegui la scansione ora*.
- Added: form del cookie con nome, categoria, stato, **titolare**, durata, finalità, ordinamento e testi propri per ogni store view (titolare, durata, finalità; un campo vuoto usa il testo predefinito, tradotto con i file i18n).
- Added: **widget CMS "Tabella dei cookie"** (`{{widget type="Threecommerce\CookieBanner\Block\Widget\CookieTable"}}`) per la cookie policy: tabella semplice Nome / Titolare / Finalità / Durata con i cookie pubblicati della store view, filtro opzionale per categoria (`category`) e colonna Categoria opzionale (`show_category`). Senza CSS del modulo, usa gli stili delle tabelle del tema.
- Added: cache di banner e tabella col tag `tc_cookie`, pulita (anche su Varnish) a ogni modifica del registro e quando la scansione trova cookie nuovi.
- Added: il dizionario dei cookie comuni indica anche il titolare (Google Ireland Ltd, Meta Platforms Ireland Ltd, Microsoft…); i cookie di prima parte mostrano il dominio dello store (es. `threecommerce.it`). Nuovo cookie `maxmind_redirect`; il cookie `store` dura 1 anno ("Ricorda la lingua / lo store scelto"). Traduzioni inglesi dei nuovi testi.
- Changed: tolto il campo *Elenco cookie* dalle impostazioni. Una data patch sposta il suo contenuto nel registro: le righe dello scope default diventano i testi predefiniti (stato pubblicato, origine manuale), quelle di store view (o website) i testi della store view, creando il cookie se manca; poi la configurazione viene cancellata. I cookie riconosciuti e mai modificati prendono i testi aggiornati del dizionario; lo stato "Ignorato" diventa "Nascosto".
- Changed: tabella `threecommerce_cookie_scan` con le colonne `provider`, `sort_order`, `created_at`, `updated_at`; nuova tabella `threecommerce_cookie_store` per i testi delle store view. Serve `bin/magento setup:upgrade`.
- Changed: nel banner i cookie sono ordinati per categoria, ordinamento e nome; il cookie di consenso resta il primo dei necessari, aggiunto in automatico. I cookie pubblicati compaiono anche con la scansione spenta.
- Changed: la vecchia pagina `tccookie/scan/index` rimanda alla griglia del registro; il permesso ACL resta `Threecommerce_CookieBanner::cookie_scan`.

## 1.4.0

- Added: traduzione inglese `i18n/en_US.csv` di tutti i testi del banner (titolo, testi, pulsanti, pannello preferenze, categorie, durata e finalità del cookie di consenso), del dizionario dei cookie comuni (durate e finalità) e delle pagine admin.
- Changed: nel dettaglio dei cookie del banner durata e finalità dei cookie rilevati dalla scansione vengono tradotte nella lingua dello store (le durate "N anni/mesi/giorni…" passano da frasi con segnaposto). I valori già salvati restano validi; le righe dell'elenco manuale non cambiano (sono già per store).
- Changed: la scansione salva le durate sempre in italiano, indipendentemente dalla lingua dell'admin o del cron.

## 1.3.3

- Chore: aggiunto il file `LICENSE` nella radice del modulo. Nessuna modifica al codice.

## 1.3.2

- Fix: escaping degli id nei nomi dei campi della tabella cookie in admin (conformità standard di codice Magento2).

## 1.3.1

- Fix: gli endpoint pubblici di log consenso e report scansione ignorano valori JSON non stringa (niente 500).
- Fix: rate limit e purge del registro usano l'ora del DB, coerente con `created_at`.

## [1.3.0] - 2026-10-26

### Added
- **Scansione automatica dei cookie**, attiva di default. Un cron (default ogni notte alle 02:30, configurabile) visita la home e le pagine indicate come farebbe un visitatore nuovo e legge i cookie impostati dal server, compresi quelli HttpOnly.
- **Segnalazione dal browser**: il banner invia a Magento i soli nomi dei cookie presenti nel browser (mai i valori), così si vedono anche quelli impostati da JavaScript (per esempio Google Analytics). Endpoint pubblico `POST /cookie-consent/scan/report` con input validato, 20 richieste l'ora per IP e tetto di 400 righe.
- **Dizionario dei cookie comuni** (Magento, Hyvä, Google, Meta, Microsoft, TikTok, LinkedIn…) che assegna in automatico categoria, durata e finalità. I cookie non riconosciuti restano "da classificare".
- Pagina admin **Stores → Cookie rilevati**: corregge categoria, durata e finalità, sceglie cosa pubblicare o ignorare e lancia la scansione al volo. Le righe modificate a mano non vengono più sovrascritte né cancellate.
- Il dettaglio dei cookie nel banner include i cookie pubblicati dalla scansione; l'elenco manuale delle impostazioni ha la precedenza.
- Impostazioni in Cookie Banner → *Scansione cookie*: attivazione, orario (cron), pagine da visitare, segnalazione dal browser, pubblicazione automatica dei cookie riconosciuti, giorni dopo i quali un cookie non più visto viene dimenticato.
- Nuova tabella `threecommerce_cookie_scan`: serve `bin/magento setup:upgrade`.

## [1.2.0] - 2026-10-26

### Added
- **Registro dei consensi** (opzionale, spento di default): ogni scelta fatta nel banner, comprese modifiche e revoche, viene salvata nella tabella `threecommerce_cookie_consent_log` per poter dimostrare il consenso (art. 7.1 GDPR).
- Griglia admin **Stores → Registro consensi cookie** (sola lettura) con filtri, ricerca ed esportazione CSV.
- Impostazioni in Threecommerce Plugins → Cookie Banner → *Registro dei consensi*: attivazione, conservazione in giorni (default 365, pulizia notturna via cron), impronta dell'IP (HMAC, default No), associazione al cliente loggato (default No).
- Ogni browser riceve un ID casuale (UUID) salvato nel cookie di consenso, così le scelte successive della stessa persona restano collegate senza dati personali. L'ID è presente anche in `event.detail` dell'evento `tc:cookie-consent`.
- Endpoint pubblico `POST /cookie-consent/log/save` con input validato e limite di 10 scritture all'ora per ID.

## [1.1.1] - 2026-10-26

### Changed
- Tabella **Elenco cookie** in admin più larga, con campi leggibili (CSS admin del modulo).
- Elenco cookie nel pannello Personalizza: titoli in grassetto ed elenchi puntati anche con i temi che azzerano gli stili delle liste.

## [1.1.0] - 2026-10-26

### Added
- Campo admin **Nome cookie di consenso** (default `tc_cookie_consent`), validato lato server.
- Campi admin **Durata del consenso (giorni)** e **Versione del consenso**, prima fissi nel JavaScript.
- Tabella admin **Elenco cookie** (nome, categoria, durata, finalità), mostrata nel pannello *Personalizza*; il cookie di consenso è incluso in automatico.

## [1.0.2] - 2026-10-12

### Added
- Campo admin **Colore principale** (Threecommerce Plugins → Cookie Banner): colore di titolo, link e pulsanti in esadecimale. Il valore viene validato lato server (`#rgb`/`#rrggbb`); se vuoto o non valido resta il colore predefinito.
- Colori del CSS esposti come variabili (`--tc-cookie-primary`, `--tc-cookie-primary-dark`, `--tc-cookie-tint`, `--tc-cookie-border`): il tema può sovrascriverle senza toccare il modulo. Toni chiari e scuri derivati dal primario con `color-mix()`.

### Changed
- Rimosso il campo `"version"` hardcoded dal `composer.json`: Composer/Satis lo preferiscono al tag git, quindi un nuovo tag poteva essere pubblicato con la versione vecchia e scartato come duplicato. Ora la versione deriva dal tag.

## [1.0.1] - 2026-07-08

### Fixed
- `etc/acl.xml`: corretto l'annidamento della risorsa. `Magento_Config::config` era dichiarato direttamente sotto `Magento_Backend::admin`, saltando `Magento_Backend::stores` → `Magento_Backend::stores_settings`. Nel merge globale degli ACL questo generava un secondo nodo `Magento_Config::config` (privo di `title`) in un percorso diverso da quello del core, violando il vincolo XSD `uniqueResourceId` e impedendo la creazione dell'oggetto ACL (`Could not create an acl object`) → admin inaccessibile. Ora la risorsa è annidata nel percorso standard.

## [1.0.0]

- Prima release del modulo cookie banner.
