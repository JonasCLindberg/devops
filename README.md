# DevOps Backlog Explorer

En fristaende, klientbaserad webbapp for att utforska och analysera Azure DevOps-backloggar.
Den aktiva appversionen i repot ar `bdo-devops-gui.html`.

## Vad appen gor

- Hamtar projekt och team fran Azure DevOps-organisationen `BDO-Sweden`.
- Läser Work Items via WIQL och visar backlog med filtrering, sokning och gruppering.
- Visar flera vyer: Backlog, Estimering, Prioritering, Timeline, Roadmap, Sprint, Board, Info, Dashboard, Kalkyl, Analys, Kostnader och Audit.
- Stödjer enklare redigering av vissa falt direkt i UI (PATCH mot Azure DevOps API).
- Har en separat "Audit users"-vy för att lista teammedlemmar per projekt (tillgänglig via Settings).
- Innehaller en "Importer"-vy för att skapa tickets i bulk från CSV/Excel-filer (tillgänglig via Settings).
- Sparar lokala UI-inställningar i webblasaren (localStorage), t.ex. PAT och kolumnval.
- Innehaller en native Board-vy med sparad layout, zoom/pan, axlar, tidsaxel och DevOps-synk for board-state.
- Innehaller en Info-vy dar Debrief-, Scope- och Plan-mallar kan laddas, redigeras och sparas per vald Epic eller Feature.

## Krav

- Tillgang till Azure DevOps-organisationen `BDO-Sweden`.
- Ett Personal Access Token (PAT).
- En modern webblasare (Chrome, Edge eller Safari).

## Snabbstart

1. Oppna `bdo-devops-gui.html` i en webblasare.
2. Fyll i ditt PAT-falt i sidopanelen.
3. Klicka pa `Hamta projekt`.
4. Valj ett eller flera projekt/team.
5. Klicka pa `Hamta data`.
6. Byt flikar for att jobba i onskad vy.

## Rekommenderade PAT-behorigheter

Minst lasbehorighet till Work Items och Projects kravs for hamtning.
Om du vill redigera falt i appen behovs aven skrivbehorighet till Work Items.

## Custom falt och custom work item-typer

Denna app anvander flera processanpassade falt och typer i Azure DevOps.
Nedan listas vad som ar krav respektive kompatibilitets/funktionsfalt.

### Harda krav for full funktion

1. Work item type `Visual board`
  - Anvands av Board-vyn for att skapa, ladda och lista board-tickets.
2. Faltet `Custom.BoardState` (Text/long text rekommenderas)
  - Anvands for att spara boardens JSON-state i DevOps.
3. Work item type `Teamref` (lagringstyp for Team-lage)
  - Appen ar last till denna som primar lagringstyp.
4. Faltet `Custom.teamprojekt`
  - Anvands pa Teamref-item for att lagra kopplade projekt/team-nycklar.
5. Work item type `Info`
  - Anvands av Info-vyn for att skapa och uppdatera sparade informationsmallar kopplade till Epic eller Feature.
6. Faltet `Custom.InfoType`
  - Anvands for att skilja mellan infotyperna `Debrief`, `Scope` och `Plan`.

### Custom work item-typer som anvands

1. `Visual board` (krav for Board-vyn)
2. `Teamref` (krav for Team-lagring)
3. `Teamdef` (fallback vid inlasning i vissa processer)
4. `Info` (krav for Info-vyn)

Notera: `Epic`, `Feature`, `User Story`, `Sprint Goal` anvands ocksa i appen men ar inte custom-typer.

### Alla custom-falt som forekommer i koden

1. `Custom.BoardState`
2. `Custom.Confidence`
3. `Custom.EndDate`
4. `Custom.InfoType`
5. `Custom.Maconomy`
6. `Custom.Milestone`
7. `Custom.PriorityLevel`
8. `Custom.Prioritylevel`
9. `Custom.Reach`
10. `Custom.RiskreductionOpportunityEnablement`
11. `Custom.Roadmap`
12. `Custom.StartDate`
13. `Custom.SuperEpic`
14. `Custom.Targetapps`
15. `Custom.Team`
16. `Custom.Tshirt`
17. `Custom.Typ`
18. `Custom.bc7657eb-88c5-4322-947f-73e86a65dc14` (Affarsomrade)
19. `Custom.teamprojekt`
20. `Custom.valueTimeSaveInt`
21. `Custom.valueTimeSaveText`
22. `Custom.wikipage`

### Praktisk rekommendation

1. Saknas `Custom.BoardState` fungerar Board-visning, men sparning/autospar till DevOps misslyckas.
2. Saknas `Teamref` och/eller `Custom.teamprojekt` fungerar inte Team-lagring fullt ut.
3. Saknas `Info` och/eller `Custom.InfoType` fungerar inte Info-vyns sparning per malltyp.
4. Ovriga custom-falt ar i huvudsak for metadata, filtrering, scoring eller redigering i modaler.

### Setup-checklista i Azure DevOps

1. Bekrafta work item-typ `Visual board` i processen/projektet.
2. Bekrafta custom-falt `Custom.BoardState`.
  - Rekommenderad typ: Plain text (multi-line).
  - Rekommenderad maxlangd: minst 32000 tecken.
3. Bekrafta work item-typ `Teamref`.
4. Bekrafta custom-falt `Custom.teamprojekt` pa `Teamref`.
  - Rekommenderad typ: Plain text (single-line eller multi-line).
  - Innehall lagras som serialiserad lista med projekt/team-nycklar.
5. Bekrafta work item-typ `Info`.
6. Bekrafta custom-falt `Custom.InfoType` pa `Info`.
  - Anvands for att markera om posten avser `Debrief`, `Scope` eller `Plan`.
7. (Valfritt fallback) Bekrafta att `Teamdef` finns om ni har aldre processvariant.
8. Bekrafta att `Custom.wikipage` finns om ni vill redigera Wikipage i ticket-modal.
  - Rekommenderad typ: Plain text (single-line).
9. Verifiera behorigheter med PAT.
  - Minst Work Items Read for lasning.
  - Work Items Read & Write for att skapa/uppdatera Teamref och spara board-state.
10. Funktionstest efter setup.
  - Skapa en ny `Visual board` i appen och spara en nodposition.
  - Ladda om och kontrollera att layouten aterlases.
  - Skapa ett Team och kontrollera att medlemskap sparas i `Custom.teamprojekt`.
  - Oppna Info-vyn, valj en Epic eller Feature via `Visa`, uppdatera mallinnehall och verifiera att en `Info`-post skapas eller uppdateras med ratt `Custom.InfoType`.

## Info-tabben

Info-vyn ar till for att hantera strukturerad dokumentation kopplad till Epic eller Feature direkt i appen.

- Val sker fran listan till vanster via knappen `Visa` pa Epic eller Feature.
- Själva mallen laddas i ett iframe-baserat arbetslage via `processmallar.html`.
- Tre malltyper hanteras: `Debrief`, `Scope` och `Plan`.
- Sparning sker till separata child work items av typen `Info`.
- Malltyp sparas i `Custom.InfoType`.
- Mallens HTML-innehall sparas i `System.Description`.
- Om en `Info`-post redan finns for vald parent + malltyp uppdateras den, annars skapas en ny.

## Importer-verktyget

Importer-vyn tillatar dig att skapa ett stort antal tickets i DevOps utgaende fran en CSV- eller Excel-fil.

### Anvandning

1. Oppna Settings (kugghjuls-ikonen).
2. Klicka pa "Importer".
3. Steg 1: Ladda upp CSV/Excel-fil (PAT hämtas automatiskt frân huvudappen).
4. Steg 2: Välj kolumner för ticket-namn (titel) och beskrivning.
5. Steg 3: Välj vilka rader som ska importeras (med filtrering och markering).
6. Steg 4: Välj projekt, Epic (ny eller befintlig) och tickettyp.
7. Steg 5: Körning - visa ticket-preview, testkör import, eller genomför import.

### Funktioner

- **Temaintegration**: Importer använder samma tema (mörkt/ljust/BDO) som huvudappen.
- **PAT-delning**: PAT från huvudappen delas automatiskt via `localStorage`.
- **Formatering**: Stödjer CSV (auto-detect separator) och Excel-filer.
- **Validering**: Hittar dubbletter och visar validerings-fel innan import.
- **Logg**: Detaljerad logg över varje ticket som skapas eller misslyckas.
- **Sparning**: Inställningar och senaste val sparas per session.



- PAT sparas lokalt i webblasaren via `localStorage` (nyckel: `devops_pat`).
- Flera andra UI-inställningar sparas ocksa lokalt.
- Ingen backend finns i detta repo; all API-kommunikation goras direkt fran klienten till Azure DevOps.

## Felsokning

- Tom vy eller fel vid hamtning:
  - Kontrollera att PAT ar giltigt och har ratt scopes.
  - Kontrollera att du valt minst ett projekt/team.
- `HTTP 401/403`:
  - Token saknar behorighet eller har gatt ut.
- Inga projekt laddas:
  - Kontrollera att du har access till organisationen `BDO-Sweden`.

## Testning

Det finns ingen separat inbyggd smoke-testfil i detta repo langre.
Testning bor utga fran den aktiva appfilen `bdo-devops-gui.html` och den funktionella setup-checklistan ovan.

## Filstruktur

- `bdo-devops-gui.html`: aktiv huvudversion av applikationen.
- `Importer.html`: verktyg för att skapa tickets i bulk från CSV/Excel-filer.
- `processmallar.html`: egen frame för Info-tabben.

## Begransningar

- Organisationen ar hardkodad till `BDO-Sweden` i koden.
- Ingen test- eller byggpipeline finns; appen ar avsedd som en fristaende HTML-app.
