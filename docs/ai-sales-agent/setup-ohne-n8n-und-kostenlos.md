# Setup ohne n8n und (fast) kostenlos

Stand: 07.11.2026 · Ergänzung zu `konzept-und-fahrplan.md` · Free-Tier-Limits ändern sich häufig, vor dem Start auf den Herstellerseiten prüfen

## Kurzantwort

- **Ohne n8n:** ja. n8n übernimmt nur Zeitplan, Webhooks und den Datenfluss zwischen den Agenten, das geht auch anders.
- **Kostenlos:** ja, für das MVP (Stufe L0–L1, bis ca. 30 Accounts/Woche). Echte 0 € schaffst du allerdings nur mit Abstrichen beim GEO-Scan: kostenlos ist nur Gemini, ChatGPT- und Perplexity-APIs kosten Geld. Realistisch sind **0 € im Test und 10–30 €/Monat**, sobald der Scan mehrere Engines abfragt (Schätzung).

## Drei Optionen ohne n8n

| | **A: Claude als Agent (Empfehlung für Woche 1–4)** | **B: Eigener Code + GitHub Actions** | **C: Make / Zapier / Activepieces Free** |
|---|---|---|---|
| Orchestrierung | Geplante Aufgaben in Claude (Claude Code im Web), stündlich oder täglich | Python-Skripte, Cron per GitHub Actions | visuelle Workflows |
| Intelligenz | Claude selbst + vorhandene Skills (`aeo-brand-scan`, `sales:lead-triage`, `sales:draft-outreach`, `sales:inbox-sweep`, `sales:daily-briefing`) | Gemini API Free Tier (oder Claude API, kostenpflichtig) | eingebaute KI-Schritte oder API-Calls |
| Anbindung | Connectoren, die du schon hast: Gmail, Google Sheets, Google Calendar, Attio | APIs selbst anbinden | fertige Konnektoren |
| Kosten | 0 € zusätzlich (läuft über dein bestehendes Claude-Abo) | 0 € (Free-Tier-Kontingent bei privaten Repos, Gemini Free Tier) | 0 € bis zu kleinen Limits, danach schnell kostenpflichtig |
| Aufbau | **1–2 Tage** (Prompts statt Code) | 1–2 Wochen | 3–5 Tage |
| Grenzen | Mindestintervall ca. stündlich, keine echten Echtzeit-Webhooks, verbraucht Abo-Nutzungslimits, der Scan nutzt nur Claudes Websuche (eine Engine) | GitHub-Cron ist nicht minutengenau, du wartest den Code selbst | Free-Limits reichen für 24/7-Betrieb nicht |
| Wann wechseln | wenn die Abo-Limits knapp werden oder du Antworten in unter 10 Minuten brauchst | wenn das Volumen > 100 Accounts/Woche steigt | – |

**Empfehlung:** Starte mit A und validiere damit die Hypothese H1. Hält sie, baust du die stabilen Teile (Scan, Lead-Feed) in B nach. C lohnt sich nicht: Die Free-Limits sind zu klein und der Schritt zu n8n ist dann kürzer.

## Option A: so läuft der "Mitarbeiter" konkret

| Uhrzeit / Takt | Geplante Aufgabe | Was passiert |
|---|---|---|
| täglich 06:00 | **Lead-Feed** | neue Accounts aus Apollo-Export oder Google Sheet ziehen, mit `sales:lead-triage` bewerten, Score ins Sheet bzw. nach Attio schreiben |
| täglich 06:30 | **Scan** | für Accounts mit Score ≥ 70 `aeo-brand-scan` ausführen und den Report als Google Doc speichern |
| täglich 07:00 | **Outreach-Entwürfe** | `sales:draft-outreach` legt Gmail-Entwürfe und LinkedIn-Texte in einem Sheet ab; **du sendest** |
| stündlich | **Inbox** | `sales:inbox-sweep` sortiert Antworten, erstellt Antwortentwürfe und schlägt Termine aus dem Kalender vor |
| täglich 07:30 | **Briefing** | `sales:daily-briefing`: Pipeline, heutige Calls, was du freigeben musst |

Alles Kritische bleibt Entwurf (L0/L1). Den automatischen Versand schaltest du erst nach den Gates aus dem Hauptkonzept frei.

## 0-€-Stack (Option B)

| Funktion | Kostenloses Tool | Limit / Haken (laut Drittquellen, ungeprüft gegen Herstellerseiten) |
|---|---|---|
| Orchestrierung | GitHub Actions (Cron) | Free-Kontingent an Minuten bei privaten Repos, Cron mit Verzögerung |
| LLM | Gemini API Free Tier (Flash) | ca. 15 Anfragen/Min., ca. 1.500/Tag; Pro-Modelle teils nicht mehr frei; Free-Tier-Daten können zum Training genutzt werden (für EU-Nutzer laut einer Quelle nicht, ungeklärt) |
| Leads | Apollo Free | Credit-Angaben schwanken stark (ca. 50–100/Monat bis 900/Jahr), Exporte limitiert |
| CRM | Attio Free oder Google Sheets | Attio Free: 3 Seats, 50.000 Records, 200 E-Mails/User/Monat |
| Versand | eigenes Gmail (nur Opt-in/Inbound) | kein Warm-up, keine Mailbox-Rotation, Ruf deiner Hauptdomain steht auf dem Spiel |
| Buchung | Cal.com Free | – |
| Freigaben/Alerts | Telegram-Bot | – |
| Scan-Landingpage | Tally + Google Sheet | – |

## Risiken, die das Gratis-Setup mitbringt

1. **Datenschutz:** Wenn ein Free-Tier-LLM Prospect-Daten (Namen, E-Mails) zum Training nutzen darf, ist das DSGVO-kritisch. Schick nur Firmendaten (Domain, Branche) an Free-Tier-Modelle und keine personenbezogenen Daten. Alternativ ein bezahltes oder abonnementgebundenes Modell nutzen.
2. **Zustellbarkeit:** Ohne Versand-Tool sendest du über die Hauptdomain. Kalte Ansprache darf dort deshalb nicht laufen, nur Antworten und Opt-in-Mails.
3. **Scan-Qualität:** Ein Scan mit nur einer Engine ist als Hook schwächer als ein Vergleich über mehrere Engines. Für H1 reicht er wahrscheinlich, fürs bezahlte Audit nicht.

## Upgrade-Pfad

| Auslöser | Investition |
|---|---|
| H1 bestätigt (≥ 8 % positive Reply-Rate) | OpenAI- und Perplexity-API für den Multi-Engine-Scan, ca. 10–30 €/Monat |
| > 50 Accounts/Woche oder Abo-Limits knapp | Lead-Feed und Scan als Code (Option B) oder n8n |
| Versand an Opt-in-Kontakte skaliert | Zweitdomain + Instantly/Smartlead |

### Quellen

- Gemini Free Tier: [tinkerllm.com](https://tinkerllm.com/blog/gemini-api-free-tier-limits-rate-quotas/), [geotoolbox.ai](https://geotoolbox.ai/blog/gemini-api-pricing.md), [blog.laozhang.ai](https://blog.laozhang.ai/en/posts/gemini-api-free-tier)
- Apollo Free: [joinvalley.co](https://www.joinvalley.co/blog/apollo-pricing-and-credits-explained-2026), [costbench.com](https://www.costbench.com/software/ai-sales-tools/apollo/free-plan/), [prospeo.io](https://prospeo.io/s/apolloio-pricing)
- Attio Free: [marketbetter.ai](https://marketbetter.ai/blog/attio-crm-pricing-breakdown-2026/), [coffee.ai](https://www.coffee.ai/articles/attio-crm-free-forever-2026)
