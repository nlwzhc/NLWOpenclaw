# PROCESSES.md — Nova Light Works Operationelle Processer

*Load this file in ALL sessions. Last updated: 2026-04-30*

## ⚠️ LAGEROPTÆLLING — VIGTIGT PÅMINDELSES-REGEL

Inden fysisk lageroptælling startes, SKAL Lars og/eller Daniel mindes om:

**"Husk at identificere og fysisk adskille alle fakturerede men endnu ikke installerede/leverede produkter inden optælling. Disse skal IKKE tælles med i lagerbeholdningen."**

Benny skal aktivt minde om dette hver gang en lageroptælling planlægges eller annonceres.

**Efter lageroptælling er gennemført og fane 4 er opdateret:**
- Opdater kolonneoverskriften i fane 3 kolonne H fra "Optalt (gammel dato)" til "Optalt (ny dato)"
- Kør update_faktisk_beholdning.js for at genberegne faktisk beholdning med nyt nulpunkt

---

## 📦 Lagerstyring Regneark (NLW_Lagerstyring.xlsx)

### Placering
- Google Drive: `gdrive:shared/NLW_Lagerstyring.xlsx`
- Lokal arbejdskopi: `/tmp/NLW_Lagerstyring.xlsx`

### Fane-struktur (7 faner)
1. **Messing-lager** — indgående messing fra Kina (nyeste øverst)
2. **Under behandling** — hos overfladebehandler (TempoKrom)
3. **Færdigvare-lager** — alle SKU'er med beholdning, solgt, seneste afstemning
4. **Lageroptælling** — fysisk optælling med dato, svind, nulstilling
5. **Konverteringslog** — råvare→færdigvare bevægelser (nyeste øverst)
6. **Salgslog** — alle salgsfakturaer (nyeste øverst)
7. **Workflow-guide**

### ⚠️ OBLIGATORISK UPLOAD-PROCEDURE
**ALDRIG upload NLW_Lagerstyring.xlsx uden at køre fix_cols.js FØRST.**
```bash
node /tmp/bold_headers.js && node /tmp/fix_cols.js && rclone copyto "/tmp/NLW_Lagerstyring.xlsx" "gdrive:shared/NLW_Lagerstyring.xlsx"
```

### Kolonnebredder (fix_cols.js)
- Fane 1: [14,35,14,55,12,28,20]
- Fane 2: [14,20,12,45,18,18,15,25]
- Fane 3: [14,10,28,28,18,12,35,25,22,12]
- Fane 4: [14,10,28,22,14,16,40,18]
- Fane 5: [14,20,10,25,10,35,55]
- Fane 6: [14,14,10,35,18,22,12,40]
- Fane 7: [45,60,35]

---

## 🔄 Shopify Lager-opdatering

**Brug ALTID GraphQL — REST `/inventory_levels/set` giver 500-fejl.**

```graphql
mutation {
  inventoryAdjustQuantities(input: {
    reason: "correction",
    name: "available",
    changes: [{
      inventoryItemId: "<ITEM_GID>",
      locationId: "gid://shopify/Location/97597784406",
      delta: <ANTAL>
    }]
  }) {
    inventoryAdjustmentGroup { reason }
    userErrors { field message }
  }
}
```

- **Location GID:** `gid://shopify/Location/97597784406`
- Brug `inventoryAdjustQuantities` (delta) eller `inventorySetQuantities` (absolut)
- `write_inventory` scope er aktiv på token

---

## 🏭 Produktionsfaktura-flow (indgående fra producent)

Når ny produktionsfaktura modtages fra Kina:
1. Tilføj til Fane 1 (Messing-lager) eller direkte Fane 3 (Færdigvare) hvis NKL/færdig fra Kina
2. Opdater Shopify lager med GraphQL mutation
3. Log i Fane 5 (Konverteringslog) hvis råvare→færdigvare
4. Gem fakturakopi på Google Drive: `gdrive:shared/fakturaer/produktion/`

**Note:** NKL-varianter håndteres forskelligt:
- **NLW2-NKL** — produceres fuldt i Kina, kommer direkte som færdigvare → Fane 3, bypasser TempoKrom
- **NLW1-NKL** — konverteres af TempoKrom fra NLW1-BRS råvare → går via Fane 2 som andre overfladebehandlede varianter

---

## 🎨 Overfladebehandling (TempoKrom ApS)

- **Adresse:** Andantevej 1, Herlev
- **Priser fra 01/03/2026:** +5% prisstigning, mindstepris nu 520 DKK
- **Proces:** Send NLW-spots til TempoKrom → modtag tilbage som BBR/DBB/BOX/CPR varianter → opdater Fane 2→3 + Shopify lager
- **Én faktura afventer afklaring med Daniel** (se seneste dagsnote for detaljer)

---

## 📸 Packshot-håndtering

- **Mappestruktur Google Drive:** `gdrive:shared/packshots/NLW_{1/2/3}_{Variant}/`
- Upload 4 billeder pr. variant
- **Status (2026-04-13):**
  - NLW1: alle 6 varianter uploadet ✅
  - NLW2: 5 varianter uploadet, Kobber (CPR) mangler
  - NLW2-Sort_oxideret: mappe oprettet, ingen billeder endnu
  - NLW3: afventer produktion

---

## 📄 Salgsfaktura-flow (udgående til kunder)

1. Opret faktura i Dinero (manuelt indtil NovaFinans-agent er bygget)
2. Log i Fane 6 (Salgslog) i NLW_Lagerstyring.xlsx
3. Opdater Shopify lager (træk solgte enheder)
4. Gem fakturakopi på Google Drive: `gdrive:shared/fakturaer/salg/`

---

## 🌐 Belysningsplan-leveringsflow

**Nuværende (manuel):**
1. Kunde sender grundplan
2. Manuelt værktøj placerer farvede prikker (farve = spottype, position = foreslået placering)
3. PDF genereres med placeringer, produktpræsentation og priser
4. Leveres til kunde inden for 24 timer som debatoplæg

**Fremtidig (AI — afventer opstart):**
Grundplan modtages → AI-analyse (<2 min) → PDF genereres → Mailgun email → kopi på Google Drive → notifikation til ejere

---

## 🔧 Gateway & System

- **Heartbeat:** 2h interval, lightContext+isolatedSession, activeHours 08-22 Europe/Copenhagen
- **boot-md hook:** aktiveret — tjekker `tasks/pending.md` ved startup
- **Restart gateway:** `systemctl --user restart openclaw-gateway.service`
- **Telegram gruppe "Nova Light Works Chat":** ID `-1003994940505` (Lars + Daniel + Benny)

---

## 📦 LEDStock — Deleger til NovaOps

**Alle lagerforespørgsler og lageroperationer delegeres til NovaOps (`nova_ops`).**

NovaOps er en dedikeret Haiku-agent der ejer det fulde LEDStock API. Brug den frem for at lave egne API-kald mod LEDStock.

### Hvornår skal du delegere til NovaOps?
- Aktuel lagerbeholdning (available, reserved, under behandling)
- Lageroptælling (opret, opdater linjer, indsend)
- Produktionsbatch-status (indgående fra Kina)
- Behandlingsordre-status (TempoKrom)
- Shopify lageropdatering via LEDStock
- Manco-tjek (negativ disponibel beholdning)
- Lagerbevægelseshistorik

### Hvornår skal du IKKE bruge NovaOps?
- Excel-regnearksoperationer (brug TOOLS.md ovenfor)
- Shopify produkter/ordrer/kunder
- Fakturering i Dinero

### Sådan delegerer du

```
Deleger til nova_ops: "Hvad er den aktuelle lagerbeholdning for NLW2-BRS?"
```

NovaOps bruger service API-nøgle (X-Api-Key) — ikke JWT. Se NovaOps TOOLS.md for fuld API-reference.

### NovaOps Red Lines (respekter dem)
- Kan ikke godkende lageroptællinger (kræver Lars eller Daniel)
- Pusher IKKE til Shopify uden eksplicit instruktion
- Retter aldrig lagerbevægelser direkte i databasen
