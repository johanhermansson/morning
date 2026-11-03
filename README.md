# Morgonrutin

En Claude Code-skill som gör grundjobbet med dagsplaneringen. Den går igenom:

| Källa | Ger |
|---|---|
| Jira (WEB, BW, NKW, WRMM2) | dina ärenden per status, och senaste kommentarerna, där det framgår vem bollen ligger hos |
| git-grenarna i `~/Sites/<repo>` | vad som *faktiskt* pågår, även när Jira inte är uppdaterat, och ocommittade ändringar |
| Gmail | obesvarade mejl, och trådar där du skrev sist och väntar på svar |
| Google Kalender | dagens och morgondagens möten, så att planen läggs runt dem |

Den sorterar allt i **🔥 Bränner / 🔨 Pågående / ✅ Redo / ⏳ Väntar på andra / 🧹 Städa** och
föreslår ett tidsschema med högst tre fokusuppgifter. Resultatet hamnar i `dagar/ÅÅÅÅ-MM-DD.md`.

## Kör

```sh
cd ~/Sites/morgonrutin
claude
> /morgonrutin
```

Eller skriv bara "planera min dag". Körs den sent på dagen planerar den resten av dagen kort
och lägger tyngden på morgondagen.

Den **ändrar ingenting** i Jira, Gmail eller kalendern. Flyttar och påminnelser föreslås sist i
filen, och görs bara om du säger ja. Mejlsvar blir utkast, aldrig skickade.

## Kopplingar

Jira går via Atlassian-pluginet. Gmail och Kalender går via Google-kopplingarna, antingen
claude.ai-connectorerna eller small-business-pluginet. Saknas en koppling körs resten ändå, och
överst i planen står vad som saknades. Anslut med `/mcp`.

## Konfiguration — `config.json`

- **`workday`** — arbetsdagens ramar, lunch, max antal fokusuppgifter och blocklängder.
- **`jira.status_buckets`** — statusnamn → hink. Nya statusar som saknas klassas efter
  statuskategori.
- **`jira.waiting_prefixes`** — titlar som börjar med t.ex. `Waiting for` eller `Blocked:` räknas
  alltid som väntande.
- **`jira.stale_days`** — tilldelat dig men orört längre än så hamnar i Städa.
- **`repos`** — Jira-projekt → repokatalog, för grenkollen.
- **`mail.ignore_senders`** — notiser som inte ska räknas som mejl att svara på.

Själva arbetsgången står i `.claude/skills/morgonrutin/SKILL.md`. Ändra där om du vill att den
prioriterar annorlunda.
