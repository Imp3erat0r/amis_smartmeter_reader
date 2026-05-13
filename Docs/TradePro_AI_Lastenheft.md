# Lastenheft – Projekt „TradePro AI"

**Version:** 0.2 (Entwurf, erweitert)
**Status:** Konzeptphase
**Sprache:** Deutsch
**Arbeitstitel:** TradePro AI

---

## 1. Einleitung und Zielsetzung

### 1.1 Vision
Entwicklung einer All-in-One-Applikation für professionelle Aktien- und ETF-Analyse, automatisiertes Portfolio-Tracking sowie ein tiefgehendes, datengetriebenes Börsentagebuch. Die App soll bestehende Lösungen (Parqet, Portfolio Performance, JustETF, getquin) in Analysetiefe, Live-Daten, Steuer­korrektheit und KI-gestützter Entscheidungsfindung übertreffen – speziell optimiert auf den deutschen Markt und den Broker **Flatex**.

### 1.2 Mission
„Jeder Trade soll erklärbar, messbar und reproduzierbar sein."

### 1.3 Erfolgskriterien / KPIs
- Import einer Flatex-Wertpapierabrechnung in **< 3 Sekunden** mit ≥ 99,5 % Feld-Trefferquote.
- Dashboard-Ladezeit **< 2 s** bei 500 Positionen.
- Steuerliche Korrektheit reproduzierbar gegen die Original-Jahres­steuerbescheinigung von Flatex (Abweichung 0,00 €).
- TTWROR-Berechnung deckungsgleich mit Portfolio Performance (Referenz-Implementierung) auf ±0,01 Prozentpunkte.

### 1.4 Abgrenzung (Out of Scope)
- Keine Order-Ausführung / kein Brokerage (App ist **read-only** gegenüber dem Depot).
- Keine Anlageberatung im Sinne § 1 KWG / MiFID II → reine Informations- und Analyse-Software.
- Keine Krypto-Wallet-Anbindung in v1 (vorgemerkt für v2).

---

## 2. Zielgruppe & Personas

| Persona | Charakteristik | Kernbedarf |
|---|---|---|
| **Primär – „Der Auftraggeber"** | Eigenbedarf, Flatex-Depot, Mix aus Buy & Hold + Swing | Wahrheits­getreues Tagebuch, Steuer, KI-Insights |
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
- **Trade-Verknüpfung:** Automatische Gruppierung von Käufen und Verkäufen zu geschlossenen Trades inkl. **FIFO** (Standard DE-Steuer­recht). Alternativ LIFO / Average-Cost für Analyse­zwecke umschaltbar.
- **Tags & Strategien:** Frei definierbar (Swing, Earnings Play, Buy & Hold, Mean Reversion, Breakout, Dividenden­strategie). Mehrfach-Tagging möglich.
- **Psychologie-Log:** Strukturierte Felder für Emotionen (Gier, Angst, FOMO, Überzeugung 1–10), freier Notiz­text, Markdown-fähig.
- **Pre-Trade-Checkliste:** Konfigurierbar pro Strategie (z. B. „KGV < 25?", „Stop-Loss definiert?", „Position­sgröße ≤ 2 % Risiko?") – Trade wird mit Erfüllungs­quote gespeichert.
- **Chart-Snapshots:** Automatisches Speichern bei Buchung (TradingView Snapshot API oder eigener Renderer).
- **R-Multiple-Tracking:** Risiko (R) bei Eröffnung definieren → Ergebnis in R messen (Van Tharp Methodik). Auswertung: Erwartungswert E[R], Trefferquote, Profitfaktor.
- **Mistake Log & Lessons Learned:** Kategorisierte Fehler­bibliothek („Stop ignoriert", „Position zu groß", „FOMO-Einstieg") – Häufigkeits­analyse pro Quartal.
- **Post-Mortem / Trade Replay:** Visualisierung des Kurs­verlaufs zwischen Ein- und Ausstieg mit eingezeichneten Aktionen.

### 3.3 Live-Portfolio & Performance-Dashboard
- **Echtzeit-Kurse:** Provider-Abstraktion (Adapter-Pattern) für: Polygon.io, Alpha Vantage, Twelve Data, EOD Historical Data, Yahoo Finance (Fallback). Konfigurierbar wegen Kosten & Rate Limits.
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

### 3.5 Steuern & Regulatorik (DE-spezifisch) **[NEU]**
- **FIFO-Konformität** gemäß § 23 EStG.
- **Abgeltungs­steuer 25 % + Soli + ggf. KiSt** mit Quellen­steuer­anrechnung (US: 15 %).
- **Verlust­verrechnungs­töpfe:** getrennt Aktien / Sonstige / Termingeschäfte (§ 20 (6) EStG inkl. 20.000-€-Cap-Berücksichtigung).
- **Teilfreistellung Fonds** (30 % Aktien-, 15 % Misch-, 60 % Immobilien-Fonds nach InvStG).
- **Vorab­pauschale** auf thesaurierende ETFs (Basiszins × 70 % × Fondswert).
- **Sparer-Pauschbetrag** (1.000 € / 2.000 €) tracking.
- **Export Anlage KAP / KAP-INV** als PDF & CSV.
- **Was-wäre-wenn-Steuersimulation:** „Wenn ich heute verkaufe → wie viel Steuer fällig?" inkl. 12-Monats-Haltefrist­hinweis.

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

---

## 4. Nicht-Funktionale Anforderungen

### 4.1 Sicherheit & Datenschutz
- **Local-first-Architektur** (SQLite/DuckDB auf Gerät) als Default; Cloud-Sync optional.
- **E2EE bei Cloud-Sync** (libsodium / age) – Schlüssel verlässt Gerät nie im Klartext.
- **Zero-Knowledge gegen LLM-Provider:** Anonymisierungs­schicht – Tickers ja, Stückzahlen/€-Beträge **nie** an Drittanbieter. Wenn nötig: Bucketing („€ 10k–25k") statt Klartext.
- **Credential Storage:** OS-Keychain (Keychain Access / Credential Manager / libsecret), kein Klartext auf Disk.
- **Audit-Log:** Alle Datenänderungen lokal protokolliert (append-only).
- **DSGVO-Konformität:** Auskunft, Löschung, Datenportabilität.
- **§ 1 KWG / MiFID II Disclaimer** explizit in der UI.

### 4.2 Performance & Verfügbarkeit
- Dashboard-Render **< 2 s** bei 500 Positionen, 10 Jahren Historie.
- Chart-Interaktion 60 fps (TradingView Lightweight Charts oder uPlot).
- Offline-Modus: Letzter Stand jederzeit verfügbar, Sync bei Reconnect.

### 4.3 Architektur & Tech-Stack (Empfehlung) **[NEU]**
- **Frontend:** Tauri (Rust + Web) oder Flutter Desktop für Native-Feel bei kleiner Binary. Web-Fallback mit React/Next.js + TanStack Query.
- **Backend (lokal):** Rust oder Go (Performance-kritisch für Backtests), Python-Microservice für ML/LLM-Pipeline.
- **Persistenz:** SQLite + DuckDB (Spalten­speicher für Analytics), Migrationen via Sqlx/Alembic.
- **Charts:** TradingView Lightweight Charts (kostenlos, kommerziell nutzbar mit Attribution).
- **LLM-Layer:** Provider-agnostisch (Claude API, OpenAI, lokales Llama via Ollama für Daten­schutz-Premium-Tier).
- **PDF-Parsing:** pdfplumber + Custom-Rules + LLM-Fallback für unbekannte Formate.

### 4.4 UX/UI **[NEU]**
- **Keyboard-first** (Command Palette à la Linear/Raycast).
- Dark/Light/System-Theme, hoher Kontrast für Trader-Setups (Multi-Monitor).
- Internationalisierung DE/EN von Tag 1, Währungs­formate korrekt (1.234,56 € vs. $1,234.56).
- **Accessibility:** WCAG 2.1 AA.

### 4.5 Skalierbarkeit & Wartbarkeit **[NEU]**
- Modular-Monolith mit klaren Bounded Contexts (Import, Portfolio, Tagebuch, Analytics, KI).
- Test-Pyramide: Unit ≥ 80 %, Integration für alle Importer, Golden-File-Tests für PDF-Parser, E2E für kritische Flows.
- CI mit Coverage-Gate, Pre-commit-Hooks (Format, Lint, Type-Check).

### 4.6 Rechtliches **[NEU]**
- Disclaimer „Keine Anlage­beratung" prominent.
- Datenlizenzen: Yahoo Finance ist **nicht** für kommerzielle Nutzung freigegeben – für ein veröffentlichtes Produkt zwingend Polygon/EOD/IEX Cloud-Lizenz.
- Logo-/Markenrechte Flatex prüfen (besser: „Flatex-kompatibel" statt „Flatex-Tool").

---

## 5. MVP-Definition & Roadmap **[NEU]**

### Phase 1 – MVP (8–10 Wochen)
1. Flatex-PDF-Import (Kauf/Verkauf/Dividende/Steuer).
2. SQLite-Storage, lokales Portfolio.
3. Dashboard: Positionen, TTWROR, einfacher Chart.
4. Börsentagebuch: Tags, Notizen, FIFO-Trades.
5. CSV-Export.

### Phase 2 – Pro Features (10–12 Wochen)
6. Live-Kurse, erweiterte Kennzahlen (Sharpe, Sortino, Max DD).
7. KI-News-Summary + Sentiment.
8. Watchlist + Alerts.
9. Steuer-Modul Anlage KAP.

### Phase 3 – Advanced (12+ Wochen)
10. Backtesting-Engine.
11. Multi-Broker (Trade Republic, Scalable, IBKR).
12. Cloud-Sync (E2EE).
13. Mobile-Companion-App (Read-only).

### Phase 4 – Vision
14. Krypto, Optionen, FX.
15. Community/Marktplatz für Strategien (mit Vorsicht – BaFin-Schwelle).

---

## 6. Risiken & Abhängigkeiten **[NEU]**

| Risiko | Wahrscheinlichkeit | Impact | Gegenmaßnahme |
|---|---|---|---|
| Flatex ändert PDF-Layout | hoch | mittel | Versionierte Parser, Golden-File-Tests, LLM-Fallback |
| Datenanbieter-Lizenzkosten | mittel | hoch | Provider-Abstraktion, Free-Tier-Fallback (privat) |
| BaFin-Einstufung als Finanzdienstleister | niedrig | sehr hoch | Reine Tool-Positionierung, keine Empfehlungen, Disclaimer |
| LLM-Halluzinationen in Analysen | hoch | mittel | Quellen-Zitate Pflicht, „Confidence"-Anzeige, Devil's Advocate |
| Datenlecks / Credential-Diebstahl | niedrig | sehr hoch | Local-first, E2EE, Keychain, kein Klartext, 2FA optional |

---

## 7. Offene Punkte / Entscheidungs­bedarf

- [ ] Single-User-App oder Multi-User (Familie / Gemeinschaftsdepot)?
- [ ] Zielplattformen Priorität: Desktop (Win/Mac/Linux), Web, Mobile?
- [ ] Monetarisierung später (Open-Source + Pro-Tier? Komplett privat?)?
- [ ] LLM-Provider: Cloud (Claude/OpenAI) vs. lokal (Llama/Mistral via Ollama)?
- [ ] Cutoff für historische Daten (z. B. ab 2010 oder ab Depot-Eröffnung)?
