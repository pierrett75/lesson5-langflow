# Family Assistant

Rodinný AI asistent vytvořený v **LangFlow**. Komunikuje česky, pracuje s databází rodinných úkolů a návrhů událostí a používá nástroj pro Google Calendar.

## Co agent umí

- Přidat, vyhledat a dokončit rodinný úkol.
- Zjistit nadcházející události v kalendáři **FAMILY**.
- Zkontrolovat možnou kolizi termínu.
- Připravit návrh nové události a uložit ho do databáze.
- Vytvořit událost v Google Calendar až po výslovném potvrzení uživatele `ano`.

## Workflow

```text
Chat Input -> Agent -> Chat Output
                 |
                 +-> SQL Database: family_tasks
                 +-> SQL Database: pending_calendar_events
                 +-> Family Calendar (custom tool) -> Google Calendar FAMILY
```

## Použité komponenty

| Komponenta | Účel |
| --- | --- |
| Chat Input | Vstup uživatelského dotazu v češtině. |
| Agent + OpenAI LLM | Porozumění dotazu, výběr nástroje a formulace odpovědi. |
| SQL Database | SQLite databáze s úkoly a čekajícími návrhy událostí. |
| Family Calendar (Custom Component) | Nástroj pro čtení událostí a vytvoření potvrzené události v Google Calendar. |
| Chat Output | Zobrazení odpovědi agenta. |

## Databáze

Workflow používá SQLite databázi `family_assistant.db` se dvěma tabulkami:

- `family_tasks` — rodinné úkoly: název, přiřazená osoba, termín a stav.
- `pending_calendar_events` — návrhy událostí: název, začátek, konec, účastníci, místo a stav.

Stavy návrhu události jsou:

- `čeká_na_potvrzení`
- `vytvořeno`
- `zrušeno`

## Bezpečnostní pravidlo

Agent nesmí volat akci `create_event`, dokud uživatel v téže konverzaci výslovně nenapíše `ano`. Před vytvořením nejdřív ověří obsazenost kalendáře a uloží návrh do tabulky `pending_calendar_events`.

Google Calendar je připojen přes servisní účet, kterému je nasdílen pouze kalendář **FAMILY** s oprávněním upravovat události. Service-account JSON klíč není součástí exportu workflow ani tohoto repozitáře.

## Ukázkové dotazy

```text
Přidej rodinný úkol: koupit léky. Přiřaď ho mamince a termín dej na 15. srpna 2026.
```

```text
Co máme v kalendáři FAMILY v následujících 30 dnech?
```

```text
Přidej do kalendáře FAMILY 20. srpna 2026 od 16:00 do 17:30 fotbal pro Kubu.
```

Agent připraví návrh a vyžádá potvrzení. Událost se vytvoří až po odpovědi:

```text
ano
```

## Import a spuštění

1. V LangFlow importuj soubor `Family_Assistant.json`.
2. Nastav vlastní OpenAI API key v komponentě Agent.
3. Nahraj svůj service-account JSON klíč do komponenty **Family Calendar**.
4. Do instrukcí Agenta vlož Calendar ID kalendáře FAMILY.
5. Otevři Playground a vyzkoušej ukázkové dotazy.

> Nikdy nesdílej soubor service-account JSON ani OpenAI API key.
