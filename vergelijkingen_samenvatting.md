# Samenvatting: vergelijkingen.html

Didactische single-file HTML-app voor het stap voor stap oplossen van eerstegraadsvergelijkingen in één onbekende. Bedoeld voor leerlingen op computer en tablet.

---

## Algemene structuur

Eén HTML-bestand met inline CSS en JS. Geen externe dependencies buiten Google Fonts.

Layout:
- **Header**: titel + navigatieknoppen (← Terug, Verder →, Reset)
- **Invoerzone**: tekstveld voor de vergelijking + knop "Visualiseer"
- **Split-layout** (twee kolommen, responsief naar één kolom op smal scherm):
  - **Links**: visueel paneel (weegschaal + knoppen)
  - **Rechts**: algebraïsch paneel (stappen onder elkaar)

---

## Instelling bovenaan JS

```js
const MANUEEL = true; // false = manuele invoer verbergen voor leerlingen
```

---

## Kleurconventies

| Element | Kleur | CSS-variabele |
|---|---|---|
| Positieve term | Blauw gevuld | `--pos: #2563eb` |
| Negatieve term | Oranje omrand | `--neg: #f97316` |
| Onbekende term | Blokje (vierkant) | — |
| Gekende term | Stip (cirkel) | — |
| Toegevoegde termen (algebra) | Lila | `--purple: #a855f7` |

---

## Weegschaal (SVG)

- viewBox: `0 0 500 270`
- Draaipunt van de balk: `(250, 222)` — rotatie via `rotate(hoek, 250, 222)` op `<g id="balk-g">`
- Schaalbladen links/rechts: rechthoeken in de roterende groep
- Symbolen (blokjes/stippen): in **aparte SVG-groepen** `<g id="sym-l">` en `<g id="sym-r">`, buiten de roterende groep. Positie wordt via JS berekend na rotatie (`schaalPos(kant)`).
- Verdeel-lijnen: `<line id="vd-l">` en `<line id="vd-r">` in de roterende groep, standaard `stroke="none"`
- Annotaties boven schalen: `<text id="ann-l">` en `<text id="ann-r">`, standaard `display="none"`
- Animatie: cubic ease-out, 420ms, via `requestAnimationFrame`

### Symbolen tekenen
Functie `tekenSym()` hertekent alle symbolen op basis van `S.schaal.l` en `S.schaal.r`. Positie volgt de geroteerde schaalbladen. Blokje = 19×19px, stip = 12px diameter. Breuken worden gedeeltelijk gevuld (proportionele hoogte/arc).

---

## Parser

- `parsVerg(s)` → splitst op `=`, geeft `{ll, rl, letter}`
- `parsLid(s)` → splitst termen, geeft `{coOnb: Br, coGet: Br, letter}`
- `parsB(s)` → parset getal of breuk (met `/`)
- Klasse `Br` (Breuk): rational arithmetic met automatische vereenvoudiging. Methoden: `plus`, `min`, `maal`, `deel`, `abs`, `neg`, `isNul`, `isPos`, `isGeh`, `eq`, `val`

---

## Toestand (`S`)

```js
S = {
  letter,          // 'x', 'y', ...
  opl,             // Br — de oplossing (voor massa-berekening weegschaal)
  fase,            // 1=invoer, 2=termen wegwerken, 3=verdelen, 4=klaar
  spinVal,         // geheel getal ≠ 0, voor de deelspinner
  LL, RL,          // {coOnb: Br, coGet: Br} — huidig linkerlid / rechterlid
  tLL, tRL,        // {coOnb: Br, coGet: Br} — toegevoegd in stap 2 (live, lila)
  schaal: {l, r},  // array van {type:'onb'|'get', sign:1|-1, waarde:Br, toegevoegd:bool}
  stappen,         // array van snapshots voor terug/verder
  stapIdx          // index in stappen
}
```

---

## Stappen

### Stap 1 — Invoer
- Leerling typt vergelijking, klikt "Visualiseer"
- Parser en oplossing berekend
- Algebra: eerste rij verschijnt
- Schaal geïnitialiseerd op basis van coëfficiënten

### Stap 2 — Termen wegwerken
- 4 knoppen per kant: +onbekende, −onbekende, +getal, −getal
- Optioneel manuele invoer (twee velden: onbekende term / gekende term)
  - Validatie bij Enter of blur; fout = rode rand + "Foute invoer"
  - Invoer blijft staan tot herleiden
- Live algebra-rij (`id="live"`) wordt bijgewerkt bij elke wijziging
- Knop "Herleiden": samenvoegen, nieuwe vaste algebra-rij, schaal reset
- Herleiden geblokkeerd als weegschaal niet in evenwicht
- Na herleiden check `klaarVD()`: als één lid puur onbekend en ander puur getal → naar stap 3

### Stap 3 — Verdelen
- Spinner (▼/▲) voor deler, slaat 0 over
  - Positief: blauwe volle lijn op weegschaal + `: n` annotatie
  - Negatief: oranje stippellijn + `: −n` annotatie
- Optioneel manuele invoer (links en rechts apart, vrije tekst, `*` → `·`)
  - Bij manueel: annotatie boven schalen i.p.v. lijnen
  - Verschillende waarden links/rechts = weegschaal uit evenwicht
- Live algebra-rij (`id="live-vd"`)
- Knop "Herleiden": berekent nieuwe coëfficiënten, nieuwe algebra-rij
- Na herleiden: check `isOpl()` → lof of terug naar stap 2

### Einde
- Lof-box met willekeurige felicitatie en de oplossing

---

## Algebra-weergave

- Grid: `1fr 26px 1fr` — linkerlid rechts uitgelijnd, `=` gecentreerd, rechterlid links uitgelijnd
- Breuken: gestapelde `<span>` met border-bottom als breukstreep
- Letters: `font-style: italic`
- Coëfficiënt 1 of −1 voor letter: geen getal geschreven (enkel `x` of `−x`)
- Toegevoegde termen: kleur `--purple`
- Stap-labels boven nieuwe rijen

---

## Bekende issues / nog niet geïmplementeerd

- Verdeel-lijnen volgen de schaalrotatie nog niet perfect (ze zitten in de roterende groep maar de positie klopt niet altijd visueel bij meerdere segmenten)
- Tussenstap bij delen door een breuk (omschrijven als vermenigvuldiging) is nog niet uitgewerkt
- Coëfficiënt −1 voor letter wordt in algebra soms nog als `−1x` i.p.v. `−x` weergegeven (controleren)
- Fase-dot update heeft een kleine bug (recursieve aanroep in `updateFase` bij herinitialisatie)
