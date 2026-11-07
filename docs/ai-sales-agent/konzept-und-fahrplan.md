# AI Sales Agent für GEO Visibility Audits: Konzept & Fahrplan

Stand: 07.11.2026 · Annahmen sind als solche markiert · Preise sind Schätzungen, vor dem Kauf auf den Herstellerseiten prüfen

---

## 0. Kurzfassung

- **Was du baust:** keinen einzelnen "AI-Mitarbeiter", sondern eine **Pipeline aus 6 spezialisierten Agenten**. Sie läuft rund um die Uhr per Zeitplan und Webhooks und übergibt an festen Freigabepunkten an dich.
- **Dein Hebel:** Der Agent liefert jedem Prospect vorab einen **personalisierten Mini-GEO-Scan** ("So beschreiben ChatGPT, Perplexity und Gemini euch und eure 3 Wettbewerber"). Du schickst also einen Befund statt eines Pitches. Den Scan kann die Maschine automatisch erzeugen, und genau das bezahlt dein Kunde später im Audit.
- **Stack (Empfehlung):** n8n (Orchestrierung) + Claude API (Denken/Schreiben) + Clay oder Apollo (Daten) + Attio (CRM, ist bei dir schon verbunden) + Instantly/Smartlead (E-Mail-Versand) + Cal.com (Buchung) + Slack/Telegram (Freigaben).
- **Laufende Kosten:** grob 300–700 €/Monat (Schätzung), plus etwa 40–60 Stunden Aufbau über 6–8 Wochen.
- **Größter blinder Fleck:** "24/7 autonom verschicken" ist in DE rechtlich und für die Zustellbarkeit der falsche Startpunkt. Rund um die Uhr laufen Recherche, Scans, Inbound und Antworten. Kalte Ansprache bleibt anfangs halbautomatisch, mit dir als Freigabe (Details in Abschnitt 1).

---

## 1. Sparring: drei Annahmen, die du prüfen solltest

| Annahme | Problem | Konsequenz fürs Design |
|---|---|---|
| "Der Agent schreibt 24/7 kalt an" | **§ 7 UWG:** E-Mail-Werbung braucht in DE auch im B2B grundsätzlich eine vorherige ausdrückliche Einwilligung. Abmahnrisiko. Dazu kommen die DSGVO-Informationspflicht (Art. 14) und die Interessenabwägung (Art. 6 Abs. 1 lit. f). *Keine Rechtsberatung, lass das von einem Anwalt für Wettbewerbsrecht prüfen.* | Kalte E-Mail in DE nur nach rechtlicher Prüfung oder gar nicht. Den Schwerpunkt auf **Inbound, LinkedIn (manuell gesendet), Warm Intros und Events** legen und E-Mail erst nach einem Opt-in einsetzen (z. B. nach dem Scan-Download). In anderen Märkten (CH/UK/US) gelten andere Regeln. |
| "Mehr Volumen = mehr Pipeline" | Bei einem beratungsintensiven Angebot mit Ticket ≥ 3–10 k€ verbrennt Volumen Domains, Ruf und deinen ICP. Gekaufte "AI SDRs" (11x, Artisan & Co.) haben genau dafür einen schlechten Ruf. | **Wenige, sehr relevante Kontakte:** 20–40 hoch passende Accounts pro Woche mit echtem Befund statt 500 generischer Mails. |
| "LinkedIn kann ich automatisieren" | Tools zur LinkedIn-Automation verstoßen gegen die LinkedIn-ToS, und dein Profil ist dein wichtigstes Vertriebs-Asset. | Der Agent recherchiert und schreibt Entwürfe, **du klickst auf Senden** (etwa 15 Minuten pro Tag). |

**Hypothese, die das ganze System trägt:**
> H1: Ein personalisierter Mini-GEO-Scan mit Wettbewerbsvergleich erzielt mindestens 3× so viele positive Antworten wie ein Standard-Pitch (Ziel: ≥ 8 % positive Reply-Rate gegenüber etwa 2–3 % als Benchmark, Schätzung).

Validieren kannst du sie in Woche 1–2 **manuell**: 20 Scans per Hand erstellen (oder mit dem Skill `aeo-brand-scan`), 20 Kontakte ansprechen, messen. Automatisiere erst, wenn H1 hält.

---

## 2. Optionen: Build vs. Buy

| Option | Beschreibung | Kosten/Monat (Schätzung) | Pro | Contra |
|---|---|---|---|---|
| **A: AI SDR kaufen** | 11x, Artisan, Qualified Piper o. ä. | 1.000–5.000 €+ | schnell live | Black Box, generisch, kein GEO-Scan als Hook, Probleme mit dem DE-Recht |
| **B: No-Code-Stack** | Clay + Instantly + Attio, ohne eigene Logik | 300–500 € | schnell, bewährt | Personalisierung bleibt flach, kein eigener Moat |
| **C: Eigener Agent (Empfehlung)** | n8n + Claude API + B-Bausteine, mit eigenem Scan-Modul | 300–700 € + Aufbauzeit | Scan als USP, volle Kontrolle, später als **Produkt/Service verkaufbar** | 40–60 Stunden Aufbau, Wartung |

**Empfehlung: C, aufgebaut auf B-Bausteinen.** Begründung: Der Scan-Agent ist dein Differenzierungsmerkmal und gleichzeitig ein Prototyp für ein eigenes Produkt ("GEO Monitoring as a Service"). Das System verkauft also und validiert nebenbei ein Venture.

---

## 3. Systemarchitektur

```
                ┌──────────────────────── n8n (Orchestrator, 24/7) ────────────────────────┐
                │                                                                           │
 Trigger:       │  ① SIGNAL-AGENT ──► ② RESEARCH- & ICP-AGENT ──► ③ GEO-SCAN-AGENT          │
 Cron 06:00     │   (Leads + Trigger)    (Score 0–100, Persona)     (ChatGPT/Perplexity/    │
 Webhooks       │                                                    Gemini/Claude APIs)    │
 Inbound-Form   │                                   │                      │                │
                │                                   ▼                      ▼                │
                │                        ④ OUTREACH-AGENT (Entwürfe pro Kanal)              │
                │                                   │                                       │
                │                     ┌─────────────┴──────────────┐                        │
                │                     ▼                            ▼                        │
                │            🟡 FREIGABE (Slack/Telegram)   Auto-Versand (nur Opt-in/Inbound)│
                │                     │                            │                        │
                │                     ▼                            ▼                        │
                │            ⑤ REPLY-AGENT: klassifiziert, antwortet, bucht (Cal.com)       │
                │                                   │                                       │
                │                                   ▼                                       │
                │            ⑥ CRM- & REPORTING-AGENT: Attio-Updates, Daily Brief, KPIs     │
                └───────────────────────────────────────────────────────────────────────────┘
```

### Die 6 Agenten im Detail

| # | Agent | Input | Output | Läuft | Autonomie |
|---|---|---|---|---|---|
| ① | **Signal** | ICP-Filter, Trigger-Liste | 30–50 neue Accounts/Woche | täglich 06:00 | voll |
| ② | **Research & ICP** | Domain, LinkedIn-URL | ICP-Score, Persona, 3 Pain-Hypothesen, Ansprechpartner | pro neuem Lead | voll |
| ③ | **GEO-Scan** | Marke, Kategorie, 3 Wettbewerber | Share of Voice, Tonalität, Quellen, 1-Seiten-PDF/Link | Score ≥ 70 oder Inbound | voll |
| ④ | **Outreach** | Research + Scan | Entwürfe: LinkedIn-Notiz, LinkedIn-DM, E-Mail (nur Opt-in), Follow-ups | nach Scan | **Freigabe durch dich** |
| ⑤ | **Reply** | eingehende Antworten (Gmail/Instantly-Webhook) | Klassifizierung + Antwortentwurf + Buchungslink | Echtzeit | gestuft (siehe Abschnitt 5) |
| ⑥ | **CRM & Reporting** | alle Events | Attio-Records, Daily Brief 07:30, Wochenreport | durchgehend + Cron | voll |

### Trigger-Signale für deinen GEO-ICP (Annahme: DACH-Mittelstand/Scale-ups, B2B-SaaS, E-Commerce, Dienstleister mit hohem Recherche-Anteil im Kaufprozess)

- Stellenanzeige für "SEO", "Content Lead", "Head of Organic", "AI Search"
- neuer CMO oder Marketing-Leiter (≤ 90 Tage im Amt)
- Finanzierungsrunde oder Website-Relaunch
- Marke taucht in KI-Antworten zur eigenen Kategorie **nicht** auf, ein Wettbewerber aber schon (das liefert ③, das stärkste Signal)
- Inbound: Download des Scans, Besuch der Pricing-Seite, LinkedIn-Interaktion mit deinen Posts

---

## 4. Tool-Stack & Kosten

| Funktion | Tool (Empfehlung) | Alternative | Kosten/Monat (Schätzung) |
|---|---|---|---|
| Orchestrierung | **n8n** self-hosted (Hetzner VPS) | n8n Cloud Starter/Pro, Make | 5–10 € VPS / ca. 24–60 € Cloud |
| LLM (Reasoning, Texte) | **Claude API** (Sonnet für Masse, Opus für Research/Antworten) | OpenAI | 30–100 € (nutzungsabhängig) |
| GEO-Scan-Abfragen | OpenAI-, Perplexity-, Gemini- und Claude-APIs | Profound, Peec AI, Otterly (fertige Tools) | 20–80 € |
| Lead-Daten & Enrichment | **Apollo** (Start) → **Clay** Launch (Scale) | Cognism, Lusha (DACH-Telefonnummern) | 50–100 € / ab ca. 167–185 $ |
| CRM | **Attio** (bereits verbunden) | HubSpot Starter | 0–35 € |
| E-Mail-Versand (nur Opt-in/Ausland) | **Instantly** oder Smartlead + 2–3 Zweitdomains | lemlist | ca. 40–100 $ + Domains |
| Buchung | **Cal.com** | Calendly | 0–15 € |
| Freigaben & Alerts | **Slack** oder Telegram-Bot | – | 0 € |
| Scan-Landingpage (Inbound) | Webflow/Framer + n8n-Webhook | Tally-Formular | 0–30 € |
| **Summe** | | | **ca. 300–700 €** |

Hinweis: Unterschiedliche Quellen nennen leicht abweichende Preise, z. B. Clay Launch mit 167 $ (jährlich) bzw. 185 $ (monatlich), und n8n listet seine Preise in EUR, die Drittquellen in USD. Vor dem Kauf auf den Herstellerseiten prüfen.

---

## 5. Autonomie-Stufen (der eigentliche "24/7"-Hebel)

Volle Autonomie bekommt der Agent nicht ab Tag 1. Er **verdient sie sich** über messbare Gates.

| Stufe | Was der Agent allein darf | Gate zum Aufstieg |
|---|---|---|
| **L0: Assistent** | recherchieren, scannen, Entwürfe schreiben | Start |
| **L1: Copilot** | Inbound-Anfragen sofort mit Scan + Buchungslink beantworten | ≥ 50 Entwürfe, Freigabequote ohne Änderungen ≥ 80 % |
| **L2: Autopilot für Antworten** | Standardantworten (Interesse, Terminwunsch, "später", Abmeldung) selbst senden | ≥ 100 klassifizierte Replies, Klassifikationsfehler < 5 % |
| **L3: Autopilot für Follow-ups** | Follow-ups bei Opt-in-Kontakten selbst senden | 4 Wochen L2 ohne Beschwerde, Spam-Rate < 0,1 % |

**Immer bei dir (harte Regeln im Prompt + Code-Check):** Preisverhandlung, Rabatte, Vertragsfragen, negative/emotionale Antworten, C-Level von Accounts mit mehr als 500 MA, alles mit Unsicherheit über 0,3.

---

## 6. Fahrplan: 8 Wochen

| Woche | Phase | Lieferobjekt | KPI / Exit-Kriterium |
|---|---|---|---|
| **1** | **Fundament** | ICP-Dokument (3 Segmente), Angebotsleiter (Free Scan → Audit → Retainer), Rechtscheck DE, Attio-Pipeline-Stages | ICP schriftlich, Anwalt-Feedback zu UWG/DSGVO |
| **1–2** | **Manueller Test von H1** | 20 Mini-Scans manuell (Skill `aeo-brand-scan`), 20 LinkedIn-Ansprachen | positive Reply-Rate ≥ 8 % → weiter, sonst Hook/ICP ändern |
| **2–3** | **MVP: ② + ③** | n8n-Workflow: Domain rein → ICP-Score + Scan-PDF raus, Ablage in Attio | Scan in < 5 Min., Qualität ≥ 4/5 bei Stichprobe |
| **3–4** | **Inbound-Maschine** | Landingpage "Kostenloser KI-Sichtbarkeits-Check" → Agent scannt → Mail mit Ergebnis + Buchungslink (Opt-in, DSGVO-konform) | Formular→Call ≥ 15 % |
| **4–5** | **① + ④** | täglicher Lead-Feed, Entwürfe für Outreach landen in Slack mit [Senden]/[Bearbeiten]/[Verwerfen] | 30+ qualifizierte Accounts/Woche, ≤ 15 Min./Tag Aufwand für dich |
| **5–6** | **⑤ Reply-Agent** | Gmail-/Instantly-Webhook → Klassifikation → Entwurf/Auto-Antwort (L1) | Antwortzeit < 10 Min., 24/7 |
| **6–7** | **⑥ Reporting** | Daily Brief 07:30 (Telegram), Wochenreport mit Funnel-KPIs | Report läuft 14 Tage fehlerfrei |
| **7–8** | **Optimieren & Autonomie** | A/B-Tests (Hook, Betreffzeile, Segment), Aufstieg auf L2 | siehe KPI-Baum |

---

## 7. KPI-Baum

| Stufe | KPI | Zielwert (Schätzung, nach 8 Wochen nachkalibrieren) |
|---|---|---|
| Input | qualifizierte Accounts/Woche (Score ≥ 70) | 30–50 |
| Aktivierung | Scans erzeugt/Woche | 20–40 |
| Response | positive Reply-Rate (Outbound) | ≥ 8 % |
| Inbound | Formular → gebuchter Call | ≥ 15 % |
| Meetings | Discovery-Calls/Woche | 3–5 |
| Conversion | Call → bezahltes Audit | ≥ 25 % |
| Effizienz | deine Zeit für Sales-Ops/Woche | ≤ 2 h |
| Kosten | Kosten pro gebuchtem Call | < 100 € |
| Qualität | Freigabequote der Entwürfe ohne Änderung | ≥ 80 % |
| Hygiene | Bounce < 2 %, Spam-Beschwerden < 0,1 % | – |

**North Star:** gebuchte, qualifizierte Discovery-Calls pro Woche.

---

## 8. Umsetzung: konkrete Bausteine

### 8.1 Datenmodell (Attio)

- **Company:** domain, branche, mitarbeiter, icp_score, icp_segment, trigger_signal, geo_share_of_voice, top_competitor, scan_url, status
- **Person:** name, rolle, linkedin_url, persona (CMO/Head of SEO/GF), consent_email (bool + Quelle + Datum)
- **Deal-Stages:** Identified → Scanned → Contacted → Replied → Call booked → Audit sold → Retainer

### 8.2 GEO-Scan-Agent: Logik

1. Aus Domain und Branche 8–12 **Käufer-Prompts** ableiten ("Beste [Kategorie]-Anbieter für [Use Case] in DACH", "[Marke] vs [Wettbewerber]", "Was ist [Marke]?").
2. Jeden Prompt an 3–4 Engines schicken (APIs mit Websuche, wo verfügbar), je 2 Durchläufe wegen der Varianz.
3. Auswerten: Erwähnung ja/nein, Position, Tonalität, zitierte Quellen, Wettbewerber-Erwähnungen.
4. Ergebnis: **Share of Voice** gegenüber 3 Wettbewerbern, die 3 größten Lücken und 1 Quick Win, als 1-Seiten-Report.
5. **Disclaimer im Report:** "Stichprobe zu Zeitpunkt X; KI-Antworten variieren; keine Garantie auf künftige Sichtbarkeit."

### 8.3 System-Prompt-Skelett: Research & ICP-Agent

```
Rolle: Du qualifizierst B2B-Accounts für GEO Visibility Audits.
ICP: [Segmente aus Woche 1 einfügen]
Aufgabe: Bewerte den Account 0–100 nach: (a) Recherche-Anteil im Kaufprozess der Zielkunden,
(b) Marketing-Reife/Budget, (c) Trigger-Signal-Stärke, (d) Wettbewerbsdruck in KI-Antworten.
Gib JSON zurück: {score, segment, persona, ansprechpartner[], pain_hypothesen[3], trigger, begruendung}
Regeln: Nichts erfinden. Unbekannt = null. Quelle für jede Behauptung angeben.
```

### 8.4 System-Prompt-Skelett: Outreach-Agent

```
Rolle: Du schreibst im Namen von Sam: direkt, sachlich, ohne Buzzwords.
Struktur LinkedIn-DM (max. 400 Zeichen): 1 konkreter Befund aus dem Scan → 1 Implikation → 1 leichte Frage.
Verboten: "Ich hoffe, es geht Ihnen gut", Superlative, Garantien zur KI-Sichtbarkeit, erfundene Zahlen.
Jede Zahl muss aus dem Scan-JSON stammen. Gib 2 Varianten + Begründung aus.
```

### 8.5 Reply-Klassen (Agent ⑤)

`interessiert` · `terminwunsch` · `frage_preis` (→ immer an dich) · `später` (→ Follow-up-Datum setzen) · `falscher_ansprechpartner` (→ Weiterleitung erfragen) · `kein_interesse` (→ schließen) · `abmeldung` (→ sofort sperren, Blacklist) · `negativ/unklar` (→ an dich)

### 8.6 Betrieb rund um die Uhr

- n8n auf einem VPS mit Docker, automatischem Neustart und täglichem Backup der Workflows und der DB
- **Error-Workflow:** Fehlschläge gehen sofort an Telegram, ein Retry erfolgt mit Backoff
- **Rate Limits und Budget-Deckel:** maximale Zahl an LLM-Calls und Scans pro Tag, Alarm bei 80 % des Monatsbudgets
- **Logging:** Jede Agent-Entscheidung kommt mit Begründung ins Attio-Activity-Log, das macht das System auditierbar und trainierbar
- **Wöchentlicher Review (30 Min.):** 10 zufällige Entwürfe und Antworten prüfen, Prompts nachschärfen

---

## 9. Risiken

| Risiko | Wahrscheinlichkeit | Impact | Mitigation |
|---|---|---|---|
| Abmahnung (UWG/DSGVO) | mittel | hoch | Rechtsprüfung, Opt-in-first, Art.-14-Info, Blacklist |
| Halluzinierte Fakten in der Ansprache | mittel | hoch | Zahlen nur aus Scan-JSON, Code-Validierung, Freigabe auf L0/L1 |
| Domain/Profil verbrannt | mittel | hoch | Zweitdomains, Warm-up, kleines Volumen, keine LinkedIn-Automation |
| Scan-Ergebnisse schwanken (LLM-Varianz) | hoch | mittel | mehrere Durchläufe, Zeitstempel, Disclaimer, Trend statt Einzelwert |
| API- und Plattformänderungen (Engines, Preise) | hoch | mittel | modulare Workflows, Engine-Adapter austauschbar |
| Zu viel Bauen, zu wenig Verkaufen | hoch | hoch | H1 erst manuell validieren, MVP in 2 Wochen, Rest iterativ |

---

## 10. Nächste 5 Schritte (diese Woche)

1. **ICP festlegen:** 3 Segmente, je 1 Persona, je 3 Trigger (2 h).
2. **Rechtscheck buchen:** UWG/DSGVO-Erstberatung zur Ansprache per E-Mail und LinkedIn (1 Termin).
3. **H1 manuell testen:** 20 Mini-Scans + 20 LinkedIn-Ansprachen, Tracking in Attio.
4. **Infrastruktur:** Hetzner-VPS + n8n (Docker), Claude-API-Key, Attio-API, Slack/Telegram-Bot.
5. **Ersten Workflow bauen:** Domain → Research-JSON → Scan → Attio-Record. Danach folgt das Inbound-Formular.

---

### Quellen (Preise, Stand Herbst 2026, teils Drittquellen)

- Clay: [joinvalley.co](https://www.joinvalley.co/blog/clay-pricing-and-credits-explained-2026), [crustdata.com](https://crustdata.com/reviews/clay), [prospeo.io](https://prospeo.io/s/clay-pricing)
- Instantly / Smartlead: [emelia.io](https://emelia.io/hub/smartlead-vs-instantly), [prospeo.io](https://prospeo.io/s/instantly-vs-smartlead), [bouncezero.io](https://bouncezero.io/smartlead-vs-instantly-2026)
- n8n: [sliplane.io](https://sliplane.io/blog/n8n-pricing), [toolradar.com](https://toolradar.com/blog/n8n-pricing-2026)
