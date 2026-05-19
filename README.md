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
