---
name: morgonrutin
description: Johans morgonrutin — går igenom mejl, Jira (WEB, BW, NKW, WRMM2), git-grenar och kalendern, sorterar i redo / pågående / väntar på andra, och föreslår en konkret dagsplan. Kör vid "/morgonrutin", "planera min dag", "vad ska jag göra idag".
---

# Morgonrutin

Johan är dålig på att planera sin dag. Ditt jobb är inte att lista allt — det är att **göra
grundjobbet och fatta ett förslag**. Var konkret: klockslag, ärendenyckel, första steget.
Hellre ett bestämt förslag han kan ändra i än fem alternativ.

All konfiguration ligger i `config.json` i den här katalogen. Läs den först.

## 0. Tid och ramar

- Kör `date "+%Y-%m-%d %H:%M %A"`. Planen gäller **från nu** till `workday.end`. Är klockan efter
  15:30, planera resten av dagen kort och lägg huvudvikten på **morgondagen**.
- `{since}` = senaste arbetsdag (fredag om det är måndag), `{since_week}` = 7 dagar bakåt.
  Gmail-format `YYYY/MM/DD`.

## 1. Hämta — kör allt parallellt

Använd de verktyg som finns i sessionen. Gmail/Kalender kan heta `mcp__claude_ai_Gmail__*`,
`mcp__claude_ai_Google_Calendar__*` eller `mcp__plugin_small-business_gmail__*` /
`..._google-calendar__*` — ladda med ToolSearch. **Faller en källa** (auth-fel, ej ansluten):
fortsätt med resten, och skriv överst i planen vilken källa som saknas och att `/mcp` ansluter
den igen. Gissa aldrig innehåll från en källa du inte läst.

**Jira** (`cloud_id` från config):
1. Mina öppna: `project in (WEB, BW, NKW, WRMM2) AND statusCategory != Done AND assignee = currentUser() ORDER BY updated DESC`
2. Rörelse hos andra senaste 5 dagarna: `project in (...) AND updated >= -5d AND (assignee != currentUser() OR assignee is EMPTY) ORDER BY updated DESC`
3. Nya, otilldelade Must/buggar: `project in (...) AND assignee is EMPTY AND statusCategory = "To Do" AND (priority = Must OR issuetype = Bug) AND created >= -14d`
4. För varje ärende i hinkarna *pågående*, *redo_deploy* och *vantar*, och för ärenden uppdaterade
   senaste 3 dagarna: hämta de 3 senaste kommentarerna (`listJiraIssueComments`, `orderBy: -created`).
   Det är där det framgår vem bollen ligger hos. Kommentarer med @Johan Hermansson
   (accountId `557058:33c4f111-6695-4cd7-9d63-2bf54a23bba1`) som han inte svarat på = han är skyldig svar.

**Git** — för varje repo i `repos`:
`git -C <sites_dir>/<repo> branch --sort=-committerdate --format='%(refname:short) %(committerdate:relative)' | head -10`
och `git status --short | head`. Gren med ärendenyckel (case-insensitive, t.ex. `patch/BW-172-…`)
committad senaste veckan = **pågående i praktiken**, även om Jira säger To Do. Flagga sådana
avvikelser ("Jira säger To Do, men grenen rördes igår — flytta till In Progress?").
Ocommittade ändringar = nämn dem, de är lätta att glömma.

**Mejl**:
- `inbox_query` → läs trådar där senaste meddelandet inte är från Johan och avsändaren inte
  matchar `ignore_senders`. Hämta tråden i `PLAIN_TEXT` när snippet inte räcker för att avgöra
  om det krävs en åtgärd.
- `awaiting_reply_query` → trådar där **Johan skrev sist** och det handlar om en fråga/leverans =
  väntar på svar. Äldre än 3 arbetsdagar = föreslå påminnelse.

**Kalender**: dagens händelser från nu, plus morgondagens. Notera tid, titel, deltagare
(extern = annan domän än sumway.se), och om Johan inte svarat på inbjudan.

## 2. Sortera

Varje sak hamnar i exakt en hink:

| Hink | Vad |
|---|---|
| 🔥 **Bränner** | Kund väntar på Johan *idag*: obesvarade mejl med fråga/brådska, @-mention i Jira utan svar, `Ready for deploy`, deadline passerad eller idag |
| 🔨 **Pågående** | In Progress/Pågår hos Johan + grenar rörda senaste veckan |
| ✅ **Redo att plocka** | To Do hos Johan, plus `Awaiting implementation` och otilldelade Must/buggar inom hans projekt |
| ⏳ **Väntar på andra** | Ready for test, Awaiting approval, `waiting_prefixes`, Johan kommenterade/mejlade sist. Ange **vem** och **sedan när** |
| 🧹 **Städa** | Tilldelade Johan men orörda > `stale_days`, eller fel projekt/fel person enligt kommentar. Föreslå: stäng, avtilldela eller ping |

Parkerade (`Mini backlog` utan väntar-prefix) tas inte med, utom i Städa om de är gamla.

## 3. Föreslå dagen

Bygg ett tidsschema från nu till `workday.end`:

1. **Möten är fasta.** Lägg `prep_min_before_external_meeting` före externa möten om det finns
   ett ärende/mejl att läsa in sig på — skriv vilket.
2. **Första blocket = 🔥.** Snabba saker först (deploy, korta svar) i ett admin-block på
   `admin_block_min` — de låser upp andra.
3. **Max `max_deep_work_tasks` fokusuppgifter.** Välj i ordning: det som är närmast klart
   (pågående med gren) → det som en kund väntar på → Must → resten. Hellre avsluta än börja nytt.
   Skriv varför just den valdes, och **första konkreta steget** ("checka ut `patch/web-955-…`,
   kör testerna").
4. Fokusblock ≈ `deep_work_block_min` minuter, inte över lunch eller möten.
5. **Sista 20 min**: följ upp väntande (påminnelser), uppdatera Jira-status.
6. Allt som inte får plats → "Inte idag" med en rad varför. Det är okej.

Var ärlig om kapacitet: en dag med tre möten har plats för en fokusuppgift, inte tre.

## 4. Skriv ut

Spara till `dagar/YYYY-MM-DD.md` (skriv över om den finns) med denna struktur:

```markdown
# Tisdag 3 november

> Saknas: Gmail, Kalender — kör /mcp och anslut, sen /morgonrutin igen.   ← bara om något saknas

## Förslag
| Tid | Vad | Varför |
|---|---|---|
| 08:30–09:00 | Admin: deploya WEB-975, svara Basak | Låser upp test hos kund |
...

## 🔥 Bränner
## 🔨 Pågående
## ✅ Redo att plocka
## ⏳ Väntar på andra
## 🧹 Städa
## Inte idag
```

Varje rad: `[KEY](https://memira.atlassian.net/browse/KEY) Titel — kund — en mening om läget`.
Mejl: avsändare, ämne, vad som krävs.

I terminalen: visa bara **Förslag**-tabellen, 🔥-listan och sökvägen till filen. Resten finns i filen.

## Regler

- **Ändra ingenting** i Jira, Gmail eller kalendern utan att fråga. Föreslå flyttar
  ("BW-172 → In Progress?", "Påminn Tommy om WEB-903?") i slutet och gör dem bara om Johan säger ja.
  Vill han ha ett svarsmejl, skapa ett **utkast**, skicka aldrig.
- Kundnamn stavas som i `jira.projects` / `~/Sites/tidsunderlag/config.json`.
- Svenska, kort, inga utfyllnadsord.
