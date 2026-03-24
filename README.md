# drammen_tms_sim

Scenarioanalyse for utgående distribusjon i Norge, med fokus på **fleet sizing, SLA-design, skiftstruktur, kostnadseffektivitet og operativ robusthet**.

## Hva prosjektet handler om

Dette prosjektet handler ikke først og fremst om kode. Det handler om å bruke data, simulering og ruteoptimalisering til å svare på reelle logistikkspørsmål:

- Hvor mange biler bør en distribusjonsmodell ha?
- Når bør bilene starte på dagen?
- Hvilke SLA-løfter er realistiske?
- Når er ekstra kapasitet faktisk verdt investeringen?
- Når ser en løsning bra ut på papiret, men svakere i operativ praksis?

Målet har vært å finne en driftsmodell som gir:

- høy og stabil service til **kontrakt**
- god service til **bedrift**
- fleksibel service til **privat**
- lavest mulig realistisk kost
- robust drift uten å skjule kapasitetsproblemer bak for fleksible kundeløfter

---

## Hovedkonklusjon

Analysen peker mot at den mest balanserte modellen er:

- **7 biler**
- **dagskift 09:00–17:00**
- **ingen buffer som standardpolicy**
- **SLA: kontrakt innen 16:00, bedrift innen 17:00, privat dag 2**

Det viktigste funnet er at **09–17 fungerer bedre enn 08–16** i dette caset, fordi ordreinngangen er tydelig frontlastet. Ved å vente én time får modellen mer ordregrunnlag inn før dispatch, noe som gir bedre konsolidering og bedre ruteforming.

Prosjektet viser også at **7 biler er et knekkpunkt**: færre biler kan fungere, men da må SLA-løftet mykes opp betydelig.

---

## Datasett og kontekst

Prosjektet bruker et ukesdatasett for utgående distribusjon med depot i Lier.

### Ordre- og volumprofil

- **1 257 ordre**
- **11 690 kolli**
- **9,3 kolli per ordre i snitt**
- median: 9
- 25-persentil: 6
- 75-persentil: 12
- maks: 29

Dette er ikke et lavvolum-case. Ordrene er relativt store, noe som gjør konsolidering og ruteforming viktig.

### Ordreinngang gjennom dagen

Ordreinngangen er tydelig frontlastet:

- 06:00–08:00: 240 ordre
- 08:00–10:00: 485 ordre
- 10:00–12:00: 387 ordre
- 12:00–14:00: 97 ordre
- 14:00–16:00: 48 ordre

Tidspunkter:

- tidligste ordre: ca. 07:00
- gjennomsnitt: ca. 09:46
- median: ca. 09:36
- seneste ordre: ca. 14:56

Dette forklarer hvorfor **senere oppstart kan gi bedre resultat enn tidligere oppstart**.

---

## Metode og verktøy

Prosjektet kombinerer:

- **OpenStreetMap / Overpass** for stoppgrunnlag
- **OSRM** for avstand og kjøretid
- **VROOM** for ruteoptimalisering / VRP

På toppen av dette er det bygget en egen scenario- og beslutningsmodell som vurderer:

- antall biler
- skiftstart og skiftslutt
- SLA-policy
- bufferpolicy
- dispatch-profil
- backlog-logikk
- servicekost
- operativ friksjon
- ekstra stopptid
- ekstra km per stopp
- faste bilkostnader i senere versjoner

Prosjektet er derfor ikke bare en ruteoptimalisering, men en **scenarioanalyse for distribusjonsdesign**.

---

## Viktigste funn

## 1. 09–17 er bedre enn 08–16

Dette er et av de tydeligste funnene i prosjektet.

En tidligere start kan virke intuitivt bedre, men i dette datasettet er ikke ordremassen ferdig “synlig” tidlig nok til at 08:00 gir beste beslutningsgrunnlag. Når dispatch utsettes til 09:00, får modellen:

- flere ordre inn før planlegging
- bedre konsolidering
- mer effektive ruter
- mindre behov for å reparere suboptimale tidlige ruter senere

**Logistisk tolkning:**  
Det er ikke alltid best å starte så tidlig som mulig. Det er best å starte når ordrebildet er godt nok til å bygge gode ruter.

---

## 2. 7 biler er knekkpunktet

Prosjektet peker tydelig mot at **7 biler er knekkpunktet** mellom presset drift og robust drift.

7 biler gjør det mulig å:

- holde høy kontraktservice
- holde bedriftservice oppe
- unngå sluttbacklog
- gjøre dette uten å være avhengig av svært fleksible kundeløfter

8 biler kan i noen scenarioer gi enda sterkere resultater, men 7 biler ser ut til å være den mest balanserte løsningen mellom kost og kapasitet.

**Logistisk tolkning:**  
7 biler ser ut til å være nivået der systemet tåler variasjon uten at service og drift blir sårbare.

---

## 3. Færre biler kan fungere, men bare med mykere SLA

En viktig del av prosjektet var å teste om færre biler kunne fungere dersom kundeløftet ble mer fleksibelt.

Sluttfasen sammenligner derfor en **7-bilers referansemodell** mot et alternativ med færre biler og mer fleksibel SLA.

Det viser at:

- 7 biler gjør det mulig å tilby et strammere og enklere SLA
- færre biler krever mer fleksibilitet, særlig for mindre prioriterte kundegrupper
- færre biler er derfor ikke bare et kapasitetsvalg, men også et valg om hva slags kundeløfte man faktisk kan stå inne for

**Logistisk tolkning:**  
7 biler er ikke bare “mer kapasitet”. Det er også det som muliggjør et mer attraktivt og robust serviceløfte.

---

## 4. Sterk dagsmodell er bedre enn mer komplisert kapasitetsdeling

Prosjektet testet også forskjøvet kapasitet og kveldsløsninger.

Hovedinntrykket er at en **sterk dagsmodell** gir bedre resultat enn mer kompliserte oppsett i dette caset.

Hvorfor:

- ordreinngangen er frontlastet
- rutene i dagsmodellen er relativt kompakte
- kveldskapasitet blir ofte tilleggskapasitet, ikke reell erstatning
- enkelte kveldsscenarioer så kunstig sterke ut før modellen ble gjort mer realistisk med ekstra friksjon og kost

**Logistisk tolkning:**  
Et mer avansert skiftoppsett er ikke automatisk bedre. I dette datasettet ser en sterk dagsmodell ut til å være mest effektiv.

---

## 5. Buffer er ikke automatisk god drift

Det ble testet flere bufferpolicyer, inkludert policyer som skjermer kontrakt og hasteordre.

Funnene tyder på at buffer ikke bør være standard uten dokumentert gevinst:

- buffer kan beskytte enkelte prioriterte ordre
- men den kan samtidig svekke total utnyttelse
- og forskyve press til øvrige kundegrupper

**Logistisk tolkning:**  
Buffer kan være riktig i enkelte settinger, men fungerer ikke nødvendigvis som en god standardpolicy.

---

## 6. Bedriftservice er mer sårbar enn kontraktservice når friksjonen øker

Robusthetskjøringene viser at når den geografiske eller operative friksjonen øker:

- kostnaden stiger tydelig
- kontraktservice holder seg relativt bedre
- bedriftservice faller raskere
- backlog kan fortsatt holdes på 0 i de sterkeste scenarioene

Dette er nyttig fordi det viser hvilke deler av SLA-strukturen som først kommer under press.

**Logistisk tolkning:**  
Når kapasitet eller ruteeffektivitet svekkes, er det ofte bedriftsegmentet som først begynner å tape kvalitet, mens kontrakt holdes oppe lenger.

---

## 7. Modellen ble mer troverdig etter kalibrering

Tidligere scenarioer ga til tider for gode resultater, spesielt for løsninger med ekstra kapasitet.

For å gjøre modellen mer realistisk ble det lagt inn mer operativ friksjon, blant annet:

- ekstra stopptid
- rush-friksjon
- ekstra km per stopp
- faste bilkostnader i senere versjoner

Dette gjorde at:

- kostnadene steg
- servicegradene falt til mer troverdige nivåer
- forskjellene mellom scenarioene ble tydeligere
- “for gode” løsninger fremsto mindre magiske

**Metodisk tolkning:**  
Et viktig funn i prosjektet er ikke bare hva som vant, men hvordan mer realistisk kalibrering endret hvilke løsninger som faktisk så robuste ut.

---

## Anbefalt driftsmodell

Den anbefalte hovedmodellen er:

- **7 biler**
- **09:00–17:00**
- **ingen buffer**
- **kontrakt innen 16:00**
- **bedrift innen 17:00**
- **privat dag 2**

Dette er modellen som best balanserer:

- kost
- service
- robusthet
- enkelhet i kundeløftet
- operativ troverdighet

---

## Hvordan prosjektet er bygget opp

Prosjektet har to hovedfaser:

### 1. Utforskningsfase
Her ble et bredt sett av scenarioer testet for å forstå effekten av:

- antall biler
- skiftstart
- SLA
- bufferpolicy
- kapasitetsoppsett
- operativ friksjon

Denne fasen ligger primært i `v3_2b`.

### 2. Fokusert beslutningsfase
Her ble analysen spisset mot mer konkrete driftsvalg, særlig:

- en sterk 7-bilers referanse
- et alternativ med færre biler og mer fleksibel SLA
- mer realistisk kostmodell
- tydeligere operasjonell sammenligning

Denne fasen ligger primært i `v3_2d`.

---

## Viktige filer

### Hovedmotor og scenariofiler
- `python/run_scenarios_vroom_v3_2d.py`
- `python/build_policy_scenarios_v3_2d.py`

### Bredere analysefase
- `python/run_scenarios_vroom_v3_2b.py`
- `python/run_scenarios_vroom_v3_2b_fixed_vehicle_cost.py`
- `python/compare_profiles_v3_2b.py`

### Fokusert sluttsammenligning
- `python/compare_6v7_operational.py`

### Analyse og støttedata
- `scenario_rank_table.csv`
- `cost_vs_weekly_km_increase_points.csv`
- `service_vs_weekly_km_increase_points.csv`

### Resultatfiler
- `runs/scenario_results_policy_v3_2b.csv`
- `runs/scenario_results_policy_v3_2d.csv`

---

## Hva støttedataene brukes til

Prosjektet inneholder også egne oppsummeringsfiler som støtter funnene:

- `scenario_rank_table.csv` brukes til å rangere scenarioer på tvers av kost, service og robusthet
- `cost_vs_weekly_km_increase_points.csv` brukes til å vise hvordan økt geografisk friksjon påvirker kost
- `service_vs_weekly_km_increase_points.csv` brukes til å vise hvordan økt friksjon påvirker SLA-ytelse

Disse filene er spesielt nyttige fordi de gjør det lettere å formidle funnene uten å måtte lese all rå output direkte.

---

## Begrensninger

Dette er en scenarioanalyse, ikke en full digital tvilling av all virkelig drift.

Forenklinger inkluderer blant annet:

- forenklet trafikkmodell
- kalibrert friksjon i stedet for full dynamisk trafikk
- interne servicekostnader som analyseverktøy, ikke full bedriftsøkonomi
- modellert operativ logikk, ikke full sanntidsdrift

Likevel er modellen godt egnet til å sammenligne driftsoppsett og støtte beslutninger om kapasitet, SLA og distribusjonsdesign.

---

## Relevans

Dette prosjektet er relevant for roller innen:

- logistikk
- supply chain
- transportplanlegging
- analyse
- operations
- continuous improvement

Det viser evne til å:

- strukturere et komplekst logistikkproblem
- bygge en analysemodell
- tolke resultater faglig
- skille mellom teoretisk mulig og operasjonelt realistisk
- kommunisere funn som faktisk kan brukes i praksis

---

## Mulige neste steg

Naturlige videreføringer av prosjektet kan være:

- flere uker og flere ordremønstre
- mer eksplisitt trafikkmodell
- enda mer realistisk modellering av skiftbytter og ventetid
- utvidet kostmodell
- dashboard-/ledelsesversjon av de viktigste funnene
