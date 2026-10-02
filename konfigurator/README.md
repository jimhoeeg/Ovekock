# Ove Kock – leadgenererende projektkonfigurator

`ove-kock-konfigurator.html` er en selvstændig side i Ove Kocks designlinje: topbjælke med telefonnumre, header med logo (`../assets/ovekock-logo.png`), hero, konfigurator og footer med de fire afdelinger. Farver (navy `#27346a`, rød `#e2001a`) og skrifttyper (Roboto og Roboto Condensed fra Google Fonts) matcher ovekock.dk.

**Header, hero og footer bruges kun på den selvstændige side.** Ved indlejring på ovekock.dk kopieres kun selve konfiguratoren, fordi sitet allerede har sin egen header og footer.

**Heroens baggrundsbillede:** Sæt `--ok-hero-img` øverst i sidens CSS til et af Ove Kocks egne fotos.

**Dybe links:** Tilføj `?produkt=renovation`, `kran`, `lift`, `sugebil` eller `hejs` til adressen for at springe direkte til den valgte produktlinje. Det er oplagt fra knapper på produktsiderne.

## Flow (8 trin)

1. **Produkt**: renovationsbil, kranbil, krog-/wirehejs, sugebil eller lift
2. **Opgave**: produktspecifikke spørgsmål, fx indsamlingstype og område eller hvad kranen skal løfte
3. **Opbygning og kapacitet**: anbefalinger ud fra trin 2
4. **Chassis og drivlinje**: hvem leverer chassis, aksler, diesel/HVO/el og mærke (springes over ved lift)
5. **Udstyr**: valgfrit og kan flervælges
6. **Projekt**: tidshorisont, baggrund (udbud), antal, service og postnummer (nærmeste afdeling)
7. **Prisindikation** *(før kontaktoplysninger)*: prisinterval, opdeling, leveringstid, leasing-overslag og faglige bemærkninger
8. **Kontaktformular**, derefter **kvittering** med reference og mulighed for at gemme som PDF

Den indbyggede viden virker på fire måder:
- "Godt at vide" i sidepanelet på hvert trin
- Markeringen "Anbefalet" på valgmuligheder
- Valg der ikke kan lade sig gøre, låses med en begrundelse (fx "Kraner over 30 tm kræver flere aksler")
- Advarsler og anbefalinger som "Tjek nyttelasten", "Kort tidshorisont" og "Kranførercertifikat"

## Værdiskabende funktioner (v3)

| # | Funktion | Hvor | Tilpasses i |
|---|---|---|---|
| 1 | **Start fra en bil, vi har bygget**: 5 cases udfylder konfigurationen. Banneret giver mulighed for at springe direkte til projekttrinnet | Første trin, under produktfliserne | `CASES` (udskift med rigtige cases og fotos) |
| 2 | **Live nyttelastmåler**: chassis, opbygning, udstyr og nyttelast vist som bjælke. Teksten oversætter til tømninger, gods eller m³. Hver akselkonfiguration viser sin nyttelast | Sidepanel. På mobil som kompakt bjælke øverst | `WEIGHTS` (kg – skal kalibreres) |
| 3 | **«Ved ikke – vælg det mest almindelige»** på alle tekniske spørgsmål. Valget markeres «afklares» over for sælger | Under hvert spørgsmål | `def: true` på valgmuligheder |
| 4 | **Lokal kontaktperson** med navn og billede ud fra postnummer, inkl. svartid («senest mandag kl. 12») | Sidepanel, CTA, kvittering | `CONTACTS` (bekræft navne, tilføj `photo`) |
| 7 | **Upload af filer**: udbudsmateriale, fotos af byttebil, skitser (træk og slip, maks. 15 MB) | Kontaktformularen | `maxUploadMB` |
| 8 | **Del med en kollega**: link, mail eller SMS. Linket åbner præcis samme konfiguration og pris | Prissiden og kvitteringen | – |
| 9 | **Bliv ringet op**: navn og telefon på ethvert trin. Sendes som `type: "callback"` med konfigurationen indtil nu | Sidepanelet | – |
| 10 | **Byttebil**: vises kun ved «Udskiftning af eksisterende bil» | Projekttrinnet | – |
| 13 | **«Ofte savnet»**: udstyr som kunder oftest får eftermonteret | Udstyrstrinnet | `miss: true` |
| 14 | **Tjek af de 5 dyreste fejl**: nyttelast, leveringstid mod opstart, produktspecifik fejl, eftermontering og service. Hver fejl får status ✓ / ? / ! | Prissiden | `mistakes()` |
| 17 | **Arbejdsmiljø**: mærke på kortene og en oversigt over valgte fordele plus ét forslag | Sidepanelet fra udstyrstrinnet | `env:` på valgmuligheder |

**Leads:** Når der er vedhæftede filer, sendes leadet som `multipart/form-data` med feltet `payload` (JSON) og `files`. Uden filer sendes JSON. Payload indeholder nu også `to_clarify`, `trade_in`, `started_from_case`, `payload_kg`, `mistakes_check` og `share_link`, så sælgeren kan åbne kundens konfiguration direkte.

**Nye tracking-events:**
- `okc_case_selected`, `okc_case_jump`, `okc_unsure`
- `okc_callback_open`, `okc_callback_requested`
- `okc_share_open`, `okc_share_copy`, `okc_share_mail`, `okc_share_sms`, `okc_share_opened`
- `okc_files_added`

## Installation på ovekock.dk

**Mulighed A – direkte i siden (fx WordPress-blokken "Brugerdefineret HTML"):**
Kopiér alt mellem `<!-- OKC START -->` og `<!-- OKC SLUT -->` ind i blokken. Al CSS er afgrænset til `.okc`, og JavaScript ligger i en lukket funktion, så den ikke påvirker resten af sitet.

**Mulighed B – iframe (maksimal isolation):**
Upload filen, fx til `/konfigurator/`, og indsæt:
```html
<iframe id="okc-frame" src="/konfigurator/ove-kock-konfigurator.html" style="width:100%;border:0;min-height:900px" title="Konfigurator"></iframe>
<script>
window.addEventListener('message', function (e) {
  if (e.data && e.data.type === 'okc-height') document.getElementById('okc-frame').style.height = e.data.height + 'px';
});
</script>
```

## Opsætning (øverst i scriptet: `CONFIG`)

| Felt | Betydning |
|---|---|
| `endpoint` | URL der modtager leads som JSON via POST: HubSpot-, Pipedrive-, Make-, Zapier- eller n8n-webhook eller egen server. **Står den tom, åbnes brugerens mailprogram med en udfyldt mail til `fallbackEmail`. Sæt et endpoint før lancering.** |
| `fallbackEmail` | Modtager, hvis der ikke er sat et endpoint (standard: info@ovekock.dk) |
| `privacyUrl` | Link til privatlivspolitikken |
| `leasing` | Løbetid, rente og restværdi til leasing-overslaget |
| `assemblyWeeks` | Uger til montage, syn og klargøring |

### Priser og regler
- **Alle priser er vejledende placeholdere** og skal kalibreres af Ove Kock før lancering. De står som `p: [min, maks]` (DKK ekskl. moms) på hver valgmulighed i objektet `Q`.
- Leveringstider pr. produkt står i `PRODUCTS[...].weeks`. Chassisets leveringstid står i `estimate()` (diesel 16–30 uger, el 26–52 uger).
- Anbefalinger står i `rec:`, og låste valg står i `dis:` på de enkelte valgmuligheder. Faglige bemærkninger samles i `advice()`.
- Postnummer kobles til afdeling i `branchFor()`.

## Lead-data

Ove Kock modtager følgende (eksempel på struktur):
```json
{
  "reference": "OK-261001-AB12",
  "lead": { "name": "…", "company": "…", "email": "…", "phone": "…", "role": "owner", "contact_preference": "call", "consent": true, "marketing_consent": false },
  "qualification": { "score": 82, "grade": "A", "timeline": "0-3", "trigger": "bid", "quantity": "2-3", "nearest_branch": "Ove Kock Esbjerg" },
  "configuration": { "product": "ren", "readable": { "Opbygning": "2-kammer baglader", "…": "…" } },
  "estimate": { "per_unit_low": 4025000, "per_unit_high": 5925000, "lead_time_weeks": [29, 58], "lines": [] },
  "advice": ["warn: Tjek nyttelasten"],
  "attribution": { "utm_source": "…", "gclid": "…", "page": "…", "referrer": "…" }
}
```
**Leadscore** (A/B/C) beregnes ud fra tidshorisont, udbud, antal, rolle, ordreværdi og kontaktønske. Den sendes kun til Ove Kock og vises ikke for kunden.

## Tracking (Google Tag Manager)

Konfiguratoren sender følgende events til `dataLayer`:
- `okc_loaded`
- `okc_product_selected`
- `okc_answer`
- `okc_step_next` / `okc_step_back`
- `okc_validation_error`
- `okc_price_shown` (med prisinterval)
- `okc_lead_form_view`
- `okc_lead_submitted` (med reference, grade, score og værdi)
- `okc_submit_error`
- `okc_print_spec`
- `okc_restart`

Brug `okc_price_shown` og `okc_lead_submitted` som konverteringer i Google Ads og LinkedIn, og byg en tragt pr. trin.

## Øvrigt
- Halvt udfyldte konfigurationer gemmes lokalt i browseren (`localStorage`), så kunden kan vende tilbage.
- Formularen har et skjult honeypot-felt mod spam.
- Farver justeres via CSS-variablerne øverst i `.okc { --okc-primary … }`.
