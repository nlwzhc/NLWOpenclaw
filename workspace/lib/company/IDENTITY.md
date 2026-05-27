# CONTEXT.md — Nova Light Works Company Context

Load this file in ALL sessions (DM, group, subagent). This contains non-private company facts safe for any context.

---

## Virksomhed

- **Navn:** Nova Light Works ApS
- **CVR:** 45424847, stiftet februar 2025, Danmark
- **Domæne:** nlwzhc.com
- **Model:** Zero-human, AI-native belysningsarkitektur-virksomhed
- **Shopify:** 66528a-fd.myshopify.com

## Mennesker

| Person | Rolle | Telegram ID | Telegram username |
|---|---|---|---|
| **Jens Christian Bentzon** | CTO, IT-ansvar | 8552082057 | — |
| **Lars (Buzz₿ / bustazz)** | Primær ejer | 5592720744 | bustazz |
| **Daniel** | Primær ejer | ukendt endnu | — |

- Jens håndterer tech-setup, integrationer og systemvalidering
- Lars og Daniel er primære ejere med strategisk og driftsansvar
- Gruppen "Nova Light Works Chat" (Telegram ID: -1003994940505) = Lars + Daniel + Benny — betroet ejerchat

## Produkter

- **NLW 1** — Klassisk indbygningsspot i messing, hånddrejet, blændningsfrit lys, patina
- **NLW 2** — Påbygningsspot i massiv messing, høj output, dæmpbar uden flimren
- **NLW 3** — Coming soon: indspartlet spot
- Fremtid: Messing-kontakter/tryk (input fra Lars)

## SKU-format og produktvarianter

**Format:** `NLW{1/2/3}-{VARIANT}`

| Variant | Beskrivelse | Status |
|---|---|---|
| BRS | Rå/ubehandlet messing | I produktion |
| BBR | Bruneret messing (TempoKrom) | I produktion |
| DBB | Sort oxideret (TempoKrom) | I produktion |
| NKL | Nikkel (fuldt produceret i Kina) | I produktion |
| BCH | Bleg champagne (TempoKrom) | I produktion |
| CPR | Kobber (TempoKrom) | I produktion |
| WAL | Valnød (TempoKrom) | I produktion |
| BAL | Sort aluminium (TempoKrom) | I produktion |
| BOX | Sort oxideret prøve (TempoKrom) | Prototype |

- **NLW2-BOX og NLW2-CPR:** Kun prototype — ikke til salg endnu
- **NKL:** NLW2-NKL kommer direkte fra Kina som færdigvare (bypasser TempoKrom). NLW1-NKL konverteres af TempoKrom fra NLW1-BRS.

## Lagerstatus (snapshot 2026-04-13 — kontrollér dagsnote for nyere tal)

| SKU | Beholdning |
|---|---|
| NLW1-BRS | 339 |
| NLW2-BRS | 179 |
| NLW2-NKL | 132 |
| NLW3-BRS | 300 (reserveret, endnu ikke modtaget) |
| Andre varianter | Se LEDStock `/stock/available` endpoint |

## USP'er — KERNEVÆRDIER, brug i ALT

1. In-house designede premium messing-spots skabt i Danmark (NLW 1, 2, 3) – ikke blot udvalgte produkter
2. Personlig belysningsplan baseret på kundens konkrete plantegning – ingen generiske løsninger
3. Totalinstallation inkluderet: møbelflytning, kabeltrækning, loftsspartling, app-programmering, rengøring, møbler tilbage
4. Lav adgangsbarriere via CubiCasa-integration – kunder uden plantegning kan nemt få én via CubiCasa-appen
5. Komplet pakke til 30% lavere pris end sammenlignelige konkurrenter – ingen konkurrenter tilbyder samme service-model

## Belysningsplan-Workflow

- **Nuværende:** Kunde sender grundplan → manuelt værktøj placerer farvede prikker (farve = spottype) → inden for 24 timer modtager kunden PDF-belysningsplan med placeringer, produktpræsentation og priser
- **Fremtidig AI-vision:** Instant belysningsplan — AI genererer planen automatisk få minutter efter grundplan modtages

## Nøgleregler

- **Svar ALTID til Jens efter enhver handling** — selv hvis kommandoen fejler eller returnerer exit code 1. Ingen stilhed.
- Lars identificeres som Telegram-bruger "Buzz₿" (username: bustazz, ID: 5592720744)

## Systemadgange

- **Shopify Admin API:** Token i `~/.openclaw/.env` som `SHOPIFY_ADMIN_TOKEN`. Scopes: write_products, write_orders, write_customers, write_draft_orders, write_inventory, write_price_rules, write_discounts, write_files, write_publications, write_metaobjects. Sales channels: Online Store (244946862422), Shop (244946927958), Point of Sale (244946960726). Brug curl + API v2025-01 eller GraphQL. Auth header: `X-Shopify-Access-Token`.
- **Google Drive:** rclone remote `gdrive:` — mapper: `agents/`, `shared/`, `temp/`, `projects/`. Folder ID: 1NsgBfP3XI8_rM6COMJWFapissMmizxns
- **Email:** Mailgun EU — afsendere: `ceo@`, `salg@`, `marketing@`, `hello@nlwzhc.com`

## Igangværende fremtidige opgaver (afventer opstart)

- **NovaFinans Agent** — Auto-bogføring af Shopify-ordrer i Dinero via Visma Connect OAuth 2.0
- **Marketing Intelligence Agent** — GA4, Google Ads, Meta APIs; monitorering og rapportering
- **AI Instant Belysningsplan** — Grundplan → AI → PDF → Mailgun (afventer produktbilleder fra Lars + grønt lys)
