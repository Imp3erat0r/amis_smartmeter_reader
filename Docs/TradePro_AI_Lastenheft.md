# Lastenheft – Projekt „TradePro AI"

**Version:** 0.3 (AT-Fassung, Desktop-MVP, Free-Tier-First)
**Status:** Konzeptphase
**Sprache:** Deutsch
**Steuerrechtlicher Geltungsbereich:** Österreich (AT)
**Arbeitstitel:** TradePro AI

---

## 1. Einleitung und Zielsetzung

### 1.1 Vision
Entwicklung einer All-in-One-Desktop-Applikation für professionelle Aktien- und ETF-Analyse, automatisiertes Portfolio-Tracking, ein tiefgehendes, datengetriebenes Börsentagebuch und – als zentrales Differenzierungs­merkmal – ein **KI-gestütztes Kaufsignal-Modul**, das neue Aktien analysiert und mit nachvollziehbarer Begründung bewertet. Die App soll bestehende Lösungen (Parqet, Portfolio Performance, JustETF, getquin) in Analysetiefe, Live-Daten, Steuer­korrektheit (AT-spezifisch) und KI-gestützter Entscheidungs­findung übertreffen – optimiert auf den österreichischen Markt und den Broker **Flatex/flatexDEGIRO**.

### 1.2 Mission
„Jeder Trade soll erklärbar, messbar und reproduzierbar sein."

### 1.3 Erfolgskriterien / KPIs
- Import einer Flatex-Wertpapierabrechnung in **< 3 Sekunden** mit ≥ 99,5 % Feld-Trefferquote.
- Dashboard-Ladezeit **< 2 s** bei 500 Positionen.
- Steuerliche Korrektheit reproduzierbar gegen die **Wertpapier-KESt-Bestätigung** des Brokers (Abweichung 0,00 €).
- TTWROR-Berechnung deckungsgleich mit Portfolio Performance (Referenz-Implementierung) auf ±0,01 Prozentpunkte.
- **KI-Kaufsignal:** reproduzierbare, mit Quellen belegte Bewertung pro Aktie in **< 30 Sekunden**.

### 1.4 Abgrenzung (Out of Scope)
- Keine Order-Ausführung / kein Brokerage (App ist **read-only** gegenüber dem Depot).
- Keine Anlageberatung im Sinne **WAG 2018 / MiFID II / § 1 BWG** → reine Informations- und Analyse-Software für den Eigenbedarf.
- Keine Krypto-Wallet-Anbindung in v1 (vorgemerkt für v2).
- **Mobile App** kein MVP-Ziel – Desktop-First.
- **Cloud-Backend** kein MVP-Ziel – Local-First, Cloud-Sync später optional.

---

## 2. Zielgruppe & Personas

| Persona | Charakteristik | Kernbedarf |
|---|---|---|
| **Primär – „Der Auftraggeber"** | Eigenbedarf, in Österreich ansässig, Flatex-Depot, Mix aus Buy & Hold + Swing, will KI-gestützte Kauf­entscheidungen | KI-Kaufsignal für neue Aktien, korrekte AT-KESt, Tagebuch |
| **Sekundär – „Ambitionierter Retail"** | 5–7-stelliges Depot, mehrere Strategien | Performance-Attribution, Risiko­kennzahlen |
| **Tertiär – „Semi-Pro / Werkstudent Finance"** | Backtesting, Screener, eigene Modelle | API-Zugriff, CSV-Export, Reproduzierbarkeit |

---

## 3. Funktionale Anforderungen

### 3.1 Broker-Integration & Datenimport (Fokus Flatex)
- **PDF-Drag & Drop / Auto-Import:** Intelligenter Parser für Flatex-Wertpapier­abrechnungen (Kauf, Verkauf, Dividende, Steuern, Gebühren, Stückzinsen, Bezugsrechte, Splits, Spin-offs).
- **OCR-Fallback:** Tesseract / Azure Document Intelligence für nicht-textuelle PDFs (gescannte Altabrechnungen).
- **Duplikat-Erkennung:** Hashing über Buchungsdatum + ISIN + Stückzahl + Betrag → keine Doppel­buchungen.
- **Reconciliation-Modus:** Soll-Ist-Abgleich gegen Flatex-Kontoauszug (Saldo­abweichung wird im UI markiert).
- **Postfach-Scraping (Phase 2, optional):** Headless-Browser-Automation (Playwright) mit lokal verschlüsseltem Credential-Store. **Disclaimer:** Nutzungsbedingungen Flatex prüfen, ggf. nur als „Anleitung zum manuellen Export".
- **Multi-Broker (Phase 2):** Trade Republic (CSV), Scalable Capital (PDF), Interactive Brokers (Flex Query XML), ING DiBa, Comdirect, DKB.
- **FinTS/HBCI (Phase 3):** Banking-Standard zur Konto­saldenabholung (kein Wertpapier­handel, aber Cash-Tracking).
- **CSV-Import / Generic Importer:** Für historische Daten und unsupportete Broker, mit konfigurierbarem Spalten-Mapping und Vorschau.
- **Manuelle Buchung:** Für Out-of-Band-Events (Steuer­erstattungen, Übertragungen zwischen Depots, Sachausschüttungen).

### 3.2 Das Börsentagebuch (Next-Level Tracking)
- **Trade-Verknüpfung:** Automatische Gruppierung von Käufen und Verkäufen zu geschlossenen Trades inkl. **gleitendem Durchschnittspreis** (Standard AT-Steuer­recht, § 27a EStG). Alternativ FIFO / LIFO für reine Analyse­zwecke umschaltbar – die Steuer­berechnung erfolgt jedoch immer gemäß AT-Logik.
- **Tags & Strategien:** Frei definierbar (Swing, Earnings Play, Buy & Hold, Mean Reversion, Breakout, Dividenden­strategie). Mehrfach-Tagging möglich.
- **Psychologie-Log:** Strukturierte Felder für Emotionen (Gier, Angst, FOMO, Überzeugung 1–10), freier Notiz­text, Markdown-fähig.
- **Pre-Trade-Checkliste:** Konfigurierbar pro Strategie (z. B. „KGV < 25?", „Stop-Loss definiert?", „Position­sgröße ≤ 2 % Risiko?") – Trade wird mit Erfüllungs­quote gespeichert.
- **Chart-Snapshots:** Automatisches Speichern bei Buchung (TradingView Snapshot API oder eigener Renderer).
- **R-Multiple-Tracking:** Risiko (R) bei Eröffnung definieren → Ergebnis in R messen (Van Tharp Methodik). Auswertung: Erwartungswert E[R], Trefferquote, Profitfaktor.
- **Mistake Log & Lessons Learned:** Kategorisierte Fehler­bibliothek („Stop ignoriert", „Position zu groß", „FOMO-Einstieg") – Häufigkeits­analyse pro Quartal.
- **Post-Mortem / Trade Replay:** Visualisierung des Kurs­verlaufs zwischen Ein- und Ausstieg mit eingezeichneten Aktionen.

### 3.3 Live-Portfolio & Performance-Dashboard
- **Echtzeit- & EOD-Kurse:** Provider-Abstraktion (Adapter-Pattern) – Default = **kostenlose Tier** (siehe §4.3.1), bezahlte Tier nachträglich zuschaltbar ohne Code-Änderung.
- **Performance-Metriken:**
  - Zeit­gewichtet: **TTWROR**, MWR (Money-Weighted), **IRR/XIRR**.
  - Risiko: **Max Drawdown**, Drawdown-Dauer, **Sharpe**, **Sortino**, **Calmar**, **Treynor**, Information Ratio.
  - Markt: Portfolio-**Beta**, **Alpha** (Jensen), R² ggü. Benchmark.
  - Risiko­maße: **VaR (95/99 %, historisch + parametrisch)**, **CVaR/Expected Shortfall**, Ulcer Index.
- **Asset Allocation:** Sektor (GICS), Land (Hauptsitz + Umsatz-Geografie), Währung, Anlageklasse, Marktkapitalisierung, Style (Value/Growth).
- **Benchmark-Vergleich:** MSCI World, S&P 500, DAX, STOXX 600, eigene Custom-Benchmark (z. B. 70/30).
- **Dividenden-Modul:** Ex-Dividenden-Kalender, Forward-Yield-Prognose, Dividend Safety Score, historische Steigerungs­raten (DGR 1/3/5/10 J).
- **Korrelations­matrix** & Heatmap zwischen Positionen.
- **Szenario-Analyse:** „Was passiert bei –20 % S&P?", Monte-Carlo (10 000 Pfade) für Portfolio-Projektion.

### 3.4 KI-Analyse & Screener
- **KI-Zusammenfassungen:** LLM-Pipeline (Claude / GPT-4-class) für: Earnings-Calls (Transkripte via Seeking Alpha / Motley Fool), 10-K / 10-Q / 20-F, Ad-hoc-Mitteilungen (DGAP), Geschäftsberichte (DE/EN).
- **Sentiment-Analyse:** Aggregation über Reuters, Bloomberg-Headlines (Lizenz prüfen!), Reddit (r/stocks, r/wallstreetbets, r/Aktien), X/Twitter via Filtered Stream, Stocktwits. Score Bullish/Bearish + Volume + Confidence.
- **„Devil's Advocate"-Modus:** KI generiert gezielt die **Gegenthese** zu einer Long-Position (Bias-Check gegen Confirmation Bias).
- **Pattern Recognition:** Klassische Muster (Schulter-Kopf-Schulter, Doppeltop, Cup&Handle, Flagge, Wedge) + Candlestick-Patterns. Plus Indikatoren: RSI, MACD, Bollinger, Ichimoku, VWAP.
- **Fundamental-Scoring:** Piotroski F-Score, Altman Z-Score, Beneish M-Score, Magic Formula (Greenblatt), Quality Score (eigene Formel).
- **DCF / Fair-Value-Schätzer:** Parametrisierbar (WACC, Growth, Terminal Multiple) mit Sensitivitäts­tabelle.
- **Insider Trading & 13F:** SEC Form 4 (Insider), 13F (Institutionelle) – Aggregation, Veränderungen QoQ, Smart-Money-Tracker.
- **Earnings-Kalender** mit Surprise-History und Whisper-Number.

### 3.5 Steuern & Regulatorik (AT-spezifisch) **[NEU]**

> **Achtung – andere Logik als Deutschland.** Österreich kennt keine FIFO-Pflicht, keinen Soli, keine Spekulations­frist, keinen Verlustvortrag ins Folgejahr. Stattdessen: gleitender Durchschnittspreis, 27,5 % KESt, jahresinterner Verlust­ausgleich.

- **KESt 27,5 %** auf realisierte Kursgewinne, Dividenden, Zinsen aus Wertpapieren (§ 27a EStG). 25 % nur auf Sparbuch-/Giro-Zinsen.
- **Gleitender Durchschnittspreis** als Anschaffungs­wert bei mehrfachen Käufen desselben Wertpapiers (**nicht** FIFO).
- **Keine Spekulationsfrist:** seit 1.4.2012 keine 1-Jahres-Behaltefrist mehr – jeder realisierte Gewinn ist KESt-pflichtig.
- **Verlust­ausgleich nur innerhalb des Kalenderjahres** zwischen Aktien, ETFs, Anleihen, Derivaten **eines** Depots; **kein Verlust­vortrag** ins Folgejahr.
  - Bei mehreren Depots / ausländischem Broker → Veranlagung erforderlich (siehe unten).
- **Inländischer Broker (z. B. flatexDEGIRO AT):** Broker führt KESt automatisch ab → Endbesteuerung. Verlust­ausgleich erfolgt durch Broker (Verlust­ausgleichs­bescheinigung).
- **Ausländischer Broker (z. B. deutsches Flatex, IBKR):** keine KESt-Abfuhr, **Selbstveranlagung** über **Formular E1kv** zur Einkommensteuer­erklärung.
- **Meldefonds vs. Nicht-Meldefonds (OeKB-Liste):**
  - Meldefonds: steuereinfach, ausschüttungs­gleiche Erträge (a.g.E.) werden automatisch besteuert.
  - Nicht-Meldefonds: pauschale **Sicherungs­steuer 27,5 %** auf 90 % des Jahres­ablauf­werts (mindestens 10 % des Werts) → deutlich teurer, App muss warnen.
- **Ausschüttungs­gleiche Erträge (a.g.E.)** bei thesaurierenden Fonds – jährlicher Steuer­abzug, auch ohne Verkauf.
- **Quellen­steuer­anrechnung:** US-DBA 15 %, CH 15 %, FR 15 % – Anrechnung in der Veranlagung möglich.
- **Stückzinsen** bei Anleihen-Kauf/Verkauf korrekt erfassen (Kapitalertrag, nicht Anschaffungskosten).
- **Export Formular E1kv** (Einkünfte aus Kapital­vermögen) als PDF + Excel zur Übernahme in FinanzOnline.
- **Was-wäre-wenn-Steuersimulation:** „Wenn ich heute verkaufe → wieviel KESt fällig, welcher Verlust­topf wird verbraucht, wieviel ist im laufenden Jahr noch ausgleichs­fähig?"
- **Stiftungs- / Privatstiftungs-Modus** (optional v2) – abweichende Besteuerung bei Privatstiftungen.

### 3.6 Risiko­management **[NEU]**
- **Position Sizing Calculator:** Fixed-Risk (z. B. 1 % Depot pro Trade), Kelly, Volatility-Adjusted.
- **Cluster-Risiko:** Warnung bei > X % in einem Sektor / Land / Währung / Einzelwert.
- **Margin-Tracker** (für Wertpapier­kredite).
- **Stop-Loss-Monitor:** Aktive Stops mit Trail-Logik, Alerts bei Annäherung.
- **What-If-Stress­tests:** 2008, 2020-COVID, Zinsschock, EUR/USD-Schock.

### 3.7 Backtesting & Strategie-Lab **[NEU]**
- **Event-driven Backtester** mit Survivorship-Bias-freier Datenbasis.
- **Strategie-DSL:** Definierbar in Python-Subset oder per UI-Builder (z. B. „Kaufe wenn RSI < 30 und KGV < 15").
- **Walk-Forward-Analyse** + Out-of-Sample-Validierung.
- **Realistische Friktionen:** Slippage, Spreads, Flatex-Gebühren­modell, Steuerlast.

### 3.8 Alerts, Notifications & Automation **[NEU]**
- Push (FCM / APNs), E-Mail, optional Telegram/Signal-Bot.
- Trigger: Preis, % -Bewegung, Volumen-Spike, News, Sentiment-Shift, Indikator-Crossover, Ex-Dividende, Earnings (T-1).
- Anti-Spam: Dedup-Window, Quiet-Hours.

### 3.9 Reporting & Export **[NEU]**
- Monats-/Quartals-/Jahres­report (PDF) – druckfähig.
- Export: CSV, JSON, Excel, Portfolio-Performance-XML (für Migration).
- API: Read-only REST + GraphQL für eigene Skripte/BI-Tools.

### 3.10 Watchlist & Screener **[NEU]**
- Multi-Filter (Fundamental + Technisch + Sentiment) auf globaler Aktien-Universe (≥ 30.000 Titel).
- Speicherbare Screens („Dividend Aristocrats EU mit Yield > 3 %").
- Vergleichs­ansicht (Side-by-Side bis zu 5 Titel).

### 3.11 KI-Kaufsignal-Modul ("Stock Discovery & Buy Signal") **[NEU – Kernfeature]**

> **Use Case des Auftraggebers:** „Ich möchte sehen, wie die KI neue Aktien analysiert und als Kaufsignal ausgibt."

#### 3.11.1 Input
- **Manuell:** Ticker / ISIN eingeben → Sofort-Analyse.
- **Aus Screener:** Ergebnisliste übergeben → Batch-Analyse.
- **Auto-Discovery:** Tägliches Scannen vordefinierter Universen (ATX, ATX Prime, DAX, MSCI World, S&P 500, NASDAQ-100, eigene Watchlist).

#### 3.11.2 Analyse-Pipeline (4 Säulen → ein Score)
Die KI bewertet jede Aktie in vier Dimensionen, jede mit Sub-Score 0–100 und Begründung. Der Gesamt-Score ist ein gewichteter Mittelwert (Gewichte konfigurierbar).

| Säule | Bewertet | Datenquellen |
|---|---|---|
| **1. Fundamental** | KGV, KBV, KUV, ROE, ROIC, Net Margin, Verschuldungsgrad, FCF-Yield, Umsatz-/Gewinn­wachstum (CAGR 3/5 J), Piotroski F-Score, Altman Z, Beneish M | SEC EDGAR (US), OeKB / WienerBörse (AT), yfinance, FMP (Free Tier) |
| **2. Technisch** | Trend (50/200-MA), Momentum (RSI, MACD), Volatilität (ATR, BB-Width), Volumen-Profil, Distance-to-52w-High, relative Stärke vs. Sektor/Index, klassische Chart­muster | Kurs­daten Provider (s. §4.3) |
| **3. Sentiment & News** | KI-Zusammen­fassung der letzten 30 Tage Nachrichten, Analystendurchschnitt, Reddit/X-Stimmung, Insider-Käufe (SEC Form 4), 13F-Bewegungen | NewsAPI Free, Finnhub Free, SEC EDGAR, Reddit API |
| **4. Bewertung / Risiko** | Fair-Value via DCF (vereinfacht), PEG, EV/EBITDA vs. Peer-Median, Margin of Safety, Dividenden-Sicherheit, Ergebnis-Volatilität | abgeleitet aus Säule 1 + Konsens-Schätzungen |

#### 3.11.3 Output – Signal-Karte pro Aktie
Standardisiertes UI-Element mit:
- **Gesamt-Score 0–100** + Ampel (🟢 ≥ 75 KAUFEN / 🟡 50–74 BEOBACHTEN / 🔴 < 50 MEIDEN).
- **Confidence-Score** der KI (z. B. „hoch" / „mittel" / „niedrig" basierend auf Datenqualität und Konsistenz der Säulen).
- **Drei stärkste Bull-Punkte** und **drei stärkste Bear-Punkte** mit Quellen­zitat.
- **Devil's-Advocate-Sektion:** explizit das Gegen-Szenario (verpflichtend gegen Confirmation Bias).
- **Vorschlag für Einstieg:** technisches Einstiegs­level, Stop-Loss-Empfehlung (z. B. unter 50-Tage-MA oder ATR-basiert), Position­sgröße bei 1-%-Risiko.
- **Vergleich zu Peers:** Tabelle mit 3–5 Konkurrenten und Score-Delta.
- **Steuer-Hinweis:** „Aktie an Börse XY – KESt-pflichtig 27,5 %, US-Quellensteuer 15 % anrechenbar" / Warnung wenn **Nicht-Meldefonds**.

#### 3.11.4 Wichtige Design-Prinzipien
- **Erklärbarkeit ist Pflicht:** Jeder Score muss zu einer Quelle zurück­führbar sein („Sources"-Drawer mit Links).
- **Kein Black-Box-Output:** Keine reine „Trust me bro"-Empfehlung – immer Zahlen + Zitate.
- **Halluzinations­schutz:** LLM darf Zahlen nur aus strukturierten Inputs zitieren, nicht selbst „erinnern". Pipeline: Retrieval → Numerical Tools → LLM nur für Sprach­zusammenfassung.
- **Backtest pro Signal-Regel:** Wenn ein Signal die Schwelle ≥ 75 erreicht, soll die App zeigen, wie diese Regel die letzten 5 Jahre performt hätte.
- **Disclaimer fix integriert:** „Dies ist keine Anlage­beratung gemäß WAG 2018."

#### 3.11.5 Stretch-Goals (Phase 2+)
- **Vergleichende Watchlist-Bewertung:** „Die KI bevorzugt heute Aktie A gegenüber B weil…"
- **Themen-Screens:** „Finde KI-Profiteure in EU mit KGV < 30 und positivem FCF."
- **Sell-Signal-Komplement:** dieselbe Logik invers für gehaltene Positionen ("Warum jetzt vielleicht verkaufen").
- **Lern-Modus:** Nutzer akzeptiert/verwirft Signale → System lernt persönliche Präferenzen (Conviction-Score-Tuning).

---

## 4. Nicht-Funktionale Anforderungen

### 4.1 Sicherheit & Datenschutz
- **Local-first-Architektur** (SQLite/DuckDB auf Gerät) als Default; Cloud-Sync optional.
- **E2EE bei Cloud-Sync** (libsodium / age) – Schlüssel verlässt Gerät nie im Klartext.
- **Zero-Knowledge gegen LLM-Provider:** Anonymisierungs­schicht – Tickers ja, Stückzahlen/€-Beträge **nie** an Drittanbieter. Wenn nötig: Bucketing („€ 10k–25k") statt Klartext.
- **Credential Storage:** OS-Keychain (Keychain Access / Credential Manager / libsecret), kein Klartext auf Disk.
- **Audit-Log:** Alle Datenänderungen lokal protokolliert (append-only).
- **DSGVO-Konformität:** Auskunft, Löschung, Datenportabilität.
- **WAG 2018 / MiFID II Disclaimer** explizit in der UI („Keine Anlage­beratung").

### 4.2 Performance & Verfügbarkeit
- Dashboard-Render **< 2 s** bei 500 Positionen, 10 Jahren Historie.
- Chart-Interaktion 60 fps (TradingView Lightweight Charts oder uPlot).
- Offline-Modus: Letzter Stand jederzeit verfügbar, Sync bei Reconnect.

### 4.3 Architektur & Tech-Stack (Empfehlung) **[NEU]**

#### Stack
- **Frontend:** **Tauri 2 (Rust + Web-UI)** – kleine Binary (~10 MB), echtes Desktop-Erlebnis, plattformüber­greifend (Win/Mac/Linux). Alternative: Electron + React, falls Rust-Ramp-up zu groß ist.
- **UI-Layer:** React + TypeScript + TanStack Query + Tailwind, Charts via **TradingView Lightweight Charts** (Apache 2.0, kommerziell frei mit Attribution).
- **Backend (lokal, im Tauri-Sidecar oder embedded):** **Python 3.12** mit FastAPI – Python ist der Sweet Spot für Finance-Bibliotheken (pandas, NumPy, pyfolio, vectorbt, yfinance). Performance-kritische Pfade (Backtest-Engine) später nach **Rust** optimierbar.
- **Persistenz:**
  - **SQLite** für transaktionale Daten (Trades, Tagebuch, Konfiguration).
  - **DuckDB** für Analytics & Backtests (Spalten­speicher, sehr schnell auf Zeit­reihen).
  - **Parquet-Files** für historische Kurs­daten-Cache.
- **LLM-Layer:** Provider-agnostisch via einheitlichem Adapter.
  - **Default für MVP:** Claude Haiku (günstig, schnell) oder OpenAI gpt-4o-mini – Free-Credits ausreichend für Testbetrieb.
  - **Phase 2:** lokales Modell via **Ollama** (z. B. Llama 3.1 8B, Qwen 2.5 14B) für Daten­schutz und Null-Kosten.
- **PDF-Parsing:** **pdfplumber** + Regel-Engine für bekannte Flatex-Layouts, **LLM-Fallback** (strukturierter Output mit JSON-Schema) für unbekannte/neue Layouts.
- **Background-Jobs:** APScheduler oder Celery + SQLite-Broker für Auto-Discovery & Alerts.

#### 4.3.1 Datenquellen-Strategie (Free-First) **[NEU]**

Default-Konfiguration: nur kostenlose Quellen. Bezahlte Provider sind im UI später per Klick aktivierbar (API-Key eintragen).

| Zweck | Free-Tier-Quelle (MVP) | Limit | Upgrade-Pfad |
|---|---|---|---|
| EOD-Kurse, Fundamentals | **yfinance** (Yahoo, inoffiziell, privat OK) | praktisch unlimitiert (Rate-Limit ~2k/h) | EOD Historical Data ($20/Mo) |
| Realtime-Quotes (15min delayed) | **Stooq.com** + yfinance | unlimitiert | Polygon.io ($30+/Mo) |
| US-Fundamentals | **SEC EDGAR** (offizielle API) | kostenlos, unbegrenzt | Financial Modeling Prep ($15+/Mo) |
| News-Headlines | **NewsAPI Free** | 100 req/Tag | NewsAPI Pro |
| Finanznachrichten + Earnings | **Finnhub Free** | 60 req/min | Finnhub Pro ($50+/Mo) |
| Marktstimmung Social | **Reddit API** (eigene App) + **Stocktwits Public** | sehr großzügig | – |
| Makro-Daten | **FRED API** (St. Louis Fed) | unbegrenzt | – |
| Wechselkurse | **Frankfurter.app** (ECB-basiert) | unbegrenzt | – |
| AT-Spezifika (Meldefonds) | **OeKB Profitweb** (öffentlich) | unbegrenzt | – |
| LLM | **Claude Free / OpenAI $5 Credit** anfangs | Test-Budget | API-Key Pay-as-you-go ab ~$0,25/Mo bei privater Nutzung |

**Schätzung laufende Kosten MVP-Testbetrieb:** **0–5 € / Monat** (nur gelegentliche LLM-Calls).

**Schätzung Pro-Tier später (alle paid Provider aktiv):** ca. 50–100 € / Monat – aber nur wenn die App das wert ist.

#### 4.3.2 Lizenz-Hinweis (wichtig für Privatnutzung OK, kommerziell nicht!)
yfinance/Stooq sind für **Eigenbedarf** unproblematisch. Sobald die App veröffentlicht wird, müssen lizensierte Provider rein (Polygon/EOD/IEX-Nachfolger).

### 4.4 UX/UI **[NEU]**
- **Keyboard-first** (Command Palette à la Linear/Raycast).
- Dark/Light/System-Theme, hoher Kontrast für Trader-Setups (Multi-Monitor).
- Internationalisierung DE/EN von Tag 1, Währungs­formate korrekt (1.234,56 € vs. $1,234.56).
- **Accessibility:** WCAG 2.1 AA.

### 4.5 Skalierbarkeit & Wartbarkeit **[NEU]**
- Modular-Monolith mit klaren Bounded Contexts (Import, Portfolio, Tagebuch, Analytics, KI).
- Test-Pyramide: Unit ≥ 80 %, Integration für alle Importer, Golden-File-Tests für PDF-Parser, E2E für kritische Flows.
- CI mit Coverage-Gate, Pre-commit-Hooks (Format, Lint, Type-Check).

### 4.6 Rechtliches (AT) **[NEU]**
- **Eigenbedarfs-App** in v1 → keine Konzession nötig.
- Sobald an Dritte verteilt: **WAG 2018** prüfen – Anlage­beratung / Anlage­vermittlung erfordern FMA-Konzession. Daher klare Positionierung als **Informations- und Analyse-Tool ohne Empfehlung**.
- Disclaimer „Keine Anlage­beratung im Sinne des WAG 2018" auf jedem Signal-Output sichtbar.
- **DSGVO:** Local-first → keine personen­bezogenen Daten verlassen das Gerät, daher minimaler DSGVO-Aufwand.
- **Datenlizenzen:** yfinance/Stooq nur für **Privatgebrauch** (Eigenbedarf) – bei späterer Veröffent­lichung Pflicht auf lizensierte Provider.
- **Marken:** „flatexDEGIRO"-Logo nicht im Branding verwenden – Formulierung „kompatibel mit Flatex" ist üblich und unbedenklich.

---

## 5. MVP-Definition & Roadmap **[NEU – angepasst an Auftraggeber-Vorgaben]**

> **Leitidee:** Desktop-First, lokal, **kostenlos zu betreiben**. Zentrales Feature ist das **KI-Kaufsignal**. Steuer- und Tagebuch-Tiefe folgt erst, wenn das Kernerlebnis steht.

### Phase 0 – Spike / Proof of Concept (1–2 Wochen)
**Ziel:** Lauffähiger Prototyp, der für eine Hand voll Aktien einen KI-Buy-Signal-Score ausgibt.
- Python-Skript + simples Tauri-/Streamlit-UI.
- yfinance + Finnhub Free als Datenquelle.
- Claude Haiku / GPT-4o-mini für Zusammenfassung.
- Hardcoded Universum: ATX + S&P 500 Top-50.

### Phase 1 – MVP (6–8 Wochen, baut auf Spike auf)
1. **KI-Kaufsignal-Modul (§3.11)** mit allen 4 Säulen, Signal-Karten-UI, Devil's-Advocate.
2. **Watchlist** (manuell pflegbar, KI-Score pro Eintrag, tägliches Refresh).
3. **Flatex-PDF-Import** (Kauf/Verkauf/Dividende) – Basis für Tagebuch.
4. **Lokales Portfolio** (SQLite) mit Positionen + Tages-P&L.
5. **Basis-Tagebuch** (Notizen, Tags, gleitender Durchschnittspreis).
6. Free-Tier-Datenquellen ausschließlich. **Laufkosten Ziel: 0–5 €/Monat.**

### Phase 2 – Pro Features (8–10 Wochen)
7. Erweiterte Kennzahlen (TTWROR, Sharpe, Sortino, Max DD).
8. **AT-Steuer-Modul (§3.5)** inkl. E1kv-Export, Meldefonds-Check, KESt-Simulation.
9. Auto-Discovery (tägliches Scannen ATX/DAX/S&P 500 → neue Signale).
10. Alerts (Desktop-Notification + optional Telegram-Bot).
11. Sentiment-Modul (Reddit, Stocktwits).
12. Backtest pro Signal-Regel.

### Phase 3 – Advanced (12+ Wochen)
13. Lokales LLM via Ollama (Daten­schutz, Null-Kosten).
14. Multi-Broker-Import (IBKR Flex Query XML, Trade Republic CSV).
15. Vollwertige Backtest-Engine (§3.7).
16. Cloud-Sync mit E2EE (optional, opt-in).
17. „Pro"-Tier-Schalter: bezahlte Datenprovider freischalten.

### Phase 4 – Vision
18. Krypto, Optionen, FX.
19. Mobile-Companion (Read-only).
20. Lern-Modus: KI passt Score-Gewichte an persönliche Trade-Historie an.

---

## 6. Risiken & Abhängigkeiten **[NEU]**

| Risiko | Wahrscheinlichkeit | Impact | Gegenmaßnahme |
|---|---|---|---|
| Flatex ändert PDF-Layout | hoch | mittel | Versionierte Parser, Golden-File-Tests, LLM-Fallback |
| yfinance/Stooq werden eingeschränkt oder geblockt | mittel | mittel | Adapter-Pattern, Fallback auf anderen Free-Provider innerhalb 1 Tag möglich |
| Free-Tier-Rate-Limits beim Auto-Discovery (S&P 500) | hoch | niedrig | Caching in DuckDB, Inkrementelle Updates, Backoff |
| **FMA-Einstufung als Finanzdienst­leister** (bei Weitergabe) | niedrig | sehr hoch | Reine Tool-Positionierung, keine Empfehlungen, Disclaimer; v1 nur Eigenbedarf |
| **LLM-Halluzinationen** in Analysen / Buy-Signalen | hoch | hoch | Retrieval-augmentiert, Zahlen nur aus strukturierten Inputs, Quellen-Zitate Pflicht, „Confidence"-Anzeige, Devil's-Advocate-Sektion |
| Falsche Steuer­berechnung AT (Meldefonds, a.g.E.) | mittel | hoch | OeKB-Liste täglich syncen, Warnung bei Nicht-Meldefonds, klares „Beta"-Label am Steuer-Modul |
| Datenlecks / Credential-Diebstahl | niedrig | sehr hoch | Local-first, kein Klartext, OS-Keychain für API-Keys |
| LLM-Kosten laufen aus dem Ruder | mittel | mittel | Hard-Cap pro Tag konfigurierbar, Caching von Signal-Outputs für 24 h, Wechsel auf Ollama möglich |

---

## 7. Getroffene Entscheidungen & noch offene Punkte

### 7.1 Entschieden (v0.3) ✓
- [x] **Steuerrecht:** Österreich (AT) – KESt 27,5 %, gleitender Durchschnittspreis, kein Verlustvortrag.
- [x] **Zielplattform:** Desktop zuerst (Tauri, Win/Mac/Linux). Mobile und Web vorerst Out-of-Scope.
- [x] **Architektur:** Local-First. Cloud-Sync später optional als Opt-in.
- [x] **Kosten:** MVP komplett mit Free-Tier-Quellen, später wahlweise Paid-Tier zuschaltbar.
- [x] **Kern-Feature:** KI-Kaufsignal-Modul (§3.11) hat oberste Priorität – „neue Aktien analysieren und Kaufsignal ausgeben".

### 7.2 Noch zu klären
- [ ] Single-User-App genügt? (Annahme: ja, da Eigenbedarf.)
- [ ] **LLM-Provider** für den Start: Claude Haiku (~ 0,25 $/Mio Token), GPT-4o-mini, oder direkt Ollama lokal? → Empfehlung: **Cloud-LLM für MVP** (Qualität, einfacher Setup), **Ollama als Phase-3-Upgrade**.
- [ ] **KI-Score-Schwellen:** Sind 75 / 50 die richtigen Grenzwerte für KAUFEN / BEOBACHTEN, oder strenger (80 / 60)?
- [ ] **Aktien-Universum** für Auto-Discovery: nur ATX + S&P 500, oder breiter (MSCI World, NASDAQ, STOXX 600)?
- [ ] **Cutoff** für historische Daten – 5 Jahre, 10 Jahre oder ab Depot-Eröffnung?
- [ ] **Monetarisierung später** – privat bleiben, Open Source mit Pro-Tier, oder kommerziell?
