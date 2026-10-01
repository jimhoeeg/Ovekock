# Ove Kock – leadgenererende projektkonfigurator

`ove-kock-konfigurator.html` er en selvstændig HTML-fil. Den har ingen eksterne afhængigheder, så CSS og JavaScript ligger i filen. Filen kan åbnes direkte i en browser og testes.

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
| `fallbackEmail` | Modtager, hvis der ikke er sat et endpoint |
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
