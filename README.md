# DevOps Backlog Explorer

En fristaende, klientbaserad webbapp for att utforska och analysera Azure DevOps-backloggar.
Allt kor i en enda HTML-fil: `devops_backlog_gui.html`.

## Vad appen gor

- Hamtar projekt och team fran Azure DevOps-organisationen `BDO-Sweden`.
- Läser Work Items via WIQL och visar backlog med filtrering, sokning och gruppering.
- Visar flera vyer: Backlog, Estimering, Prioritering, Timeline, Roadmap, Dashboard, Kalkyl, Analys och Kostnader.
- Stödjer enklare redigering av vissa falt direkt i UI (PATCH mot Azure DevOps API).
- Har en separat "Audit users"-vy for att lista teammedlemmar per projekt.
- Sparar lokala UI-inställningar i webblasaren (localStorage), t.ex. PAT och kolumnval.

## Krav

- Tillgang till Azure DevOps-organisationen `BDO-Sweden`.
- Ett Personal Access Token (PAT).
- En modern webblasare (Chrome, Edge eller Safari).

## Snabbstart

1. Oppna `devops_backlog_gui.html` i en webblasare.
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

### Custom work item-typer som anvands

1. `Visual board` (krav for Board-vyn)
2. `Teamref` (krav for Team-lagring)
3. `Teamdef` (fallback vid inlasning i vissa processer)

Notera: `Epic`, `Feature`, `User Story`, `Sprint Goal` anvands ocksa i appen men ar inte custom-typer.

### Alla custom-falt som forekommer i koden

1. `Custom.BoardState`
2. `Custom.Confidence`
3. `Custom.EndDate`
4. `Custom.Maconomy`
5. `Custom.Milestone`
6. `Custom.PriorityLevel`
7. `Custom.Prioritylevel`
8. `Custom.Reach`
9. `Custom.RiskreductionOpportunityEnablement`
10. `Custom.Roadmap`
11. `Custom.StartDate`
12. `Custom.SuperEpic`
13. `Custom.Targetapps`
14. `Custom.Team`
15. `Custom.Teamprojekt` (lases endast som kompatibilitetsfallback)
16. `Custom.Tshirt`
17. `Custom.Typ`
18. `Custom.bc7657eb-88c5-4322-947f-73e86a65dc14` (Affarsomrade)
19. `Custom.teamProject` (lases endast som kompatibilitetsfallback)
20. `Custom.teamProjekt` (lases endast som kompatibilitetsfallback)
21. `Custom.teamprojekt` (primart falt, anvands vid skrivning)
22. `Custom.valueTimeSaveInt`
23. `Custom.valueTimeSaveText`
24. `Custom.wikipage`

### Praktisk rekommendation

1. Saknas `Custom.BoardState` fungerar Board-visning, men sparning/autospar till DevOps misslyckas.
2. Saknas `Teamref` och/eller `Custom.teamprojekt` fungerar inte Team-lagring fullt ut.
3. Ovriga custom-falt ar i huvudsak for metadata, filtrering, scoring eller redigering i modaler.

### Setup-checklista i Azure DevOps

1. Bekrafta work item-typ `Visual board` i processen/projektet.
2. Bekrafta custom-falt `Custom.BoardState`.
  - Rekommenderad typ: Plain text (multi-line).
  - Rekommenderad maxlangd: minst 32000 tecken.
3. Bekrafta work item-typ `Teamref`.
4. Bekrafta custom-falt `Custom.teamprojekt` pa `Teamref`.
  - Rekommenderad typ: Plain text (single-line eller multi-line).
  - Innehall lagras som serialiserad lista med projekt/team-nycklar.
  - Appen skriver till `Custom.teamprojekt` och laser aven varianterna `Custom.teamProjekt`, `Custom.teamProject` och `Custom.Teamprojekt` for bakatkompatibilitet.
5. (Valfritt fallback) Bekrafta att `Teamdef` finns om ni har aldre processvariant.
6. Bekrafta att `Custom.wikipage` finns om ni vill redigera Wikipage i ticket-modal.
  - Rekommenderad typ: Plain text (single-line).
7. Verifiera behorigheter med PAT.
  - Minst Work Items Read for lasning.
  - Work Items Read & Write for att skapa/uppdatera Teamref och spara board-state.
8. Funktionstest efter setup.
  - Skapa en ny `Visual board` i appen och spara en nodposition.
  - Ladda om och kontrollera att layouten aterlases.
  - Skapa ett Team och kontrollera att medlemskap sparas i `Custom.teamprojekt`.

## Data och sakerhet

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

## Step 7 smoke-tests (v2.0)

For en snabb regressionssignal finns tva inbyggda test-runners i `devops_backlog_gui_v2.0.html`:

- `runPhase6DomainTests()` for domanregler (rank/path identity).
- `runStep7UiSmokeTests()` for UI-smoke (sprint reorder/link/toggle + primar render-routing).

Kora dem fran terminalen med detta kommando:

```bash
perl -0777 -ne 'if (/<script>(.*)<\/script>/s){$s=$1; $s =~ s#// ──────────────────────────────────────────\n// Bootstrap\n// ──────────────────────────────────────────[\s\S]*$##s; print $s;}' devops_backlog_gui_v2.0.html > /tmp/devops_script_phase67.js && osascript -l JavaScript -e 'var window = { addEventListener: function(){}, requestAnimationFrame: function(cb){ cb(); } }; var console = { log: function(){}, error: function(){} }; var localStorage = { getItem: function(){return null;}, setItem: function(){} }; var script = $.NSString.stringWithContentsOfFileEncodingError("/tmp/devops_script_phase67.js", $.NSUTF8StringEncoding, null).js; eval(script); (async function(){ var p6 = runPhase6DomainTests(); var p7 = await runStep7UiSmokeTests(); var out = { phase6: p6, step7: p7 }; $.NSFileHandle.fileHandleWithStandardOutput.writeData($(JSON.stringify(out)+"\n").dataUsingEncoding($.NSUTF8StringEncoding)); })();'
```

Forvantad output ar ett JSON-objekt med resultat for bada runners, till exempel:

```json
{"phase6":{"ok":true,"passed":17,"failed":0,"failures":[]},"step7":{"ok":true,"passed":15,"failed":0,"failures":[]}}
```

## Filstruktur

- `devops_backlog_gui.html`: hela applikationen (UI, styling och JavaScript-logik).
- `devops_backlog_gui_v2.0.html`: aktiv refactor-version med senaste quality/smoke-test runner.

## Begransningar

- Organisationen ar hardkodad till `BDO-Sweden` i koden.
- Ingen test- eller byggpipeline finns; appen ar avsedd som en fristaende HTML-app.
