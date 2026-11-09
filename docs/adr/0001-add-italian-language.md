# ADR 0001: Add Italian as a supported language

- **Status:** Accepted
- **Date:** 2026-11-08
- **Decider:** Veronica Williams

## Context

The game already supports English, Spanish and French. Every piece of player-facing text lives in `language_data.json`, grouped by language. `Messages` loads one language's section at startup and looks up text by key (for example `GetMessage("victory")`).

Each language section has the same six parts:

| Part | Purpose |
|---|---|
| `dictionary` | All 69 game messages, keyed identically across languages |
| `raceMap` / `occupationMap` | Turn what the player types (in their language) into the internal English value, e.g. `nain` → `Dwarf` |
| `displayRaceMap` / `displayOccupationMap` / `displayWeaponMap` | Turn the internal English value back into the player's language for display |

Because of this design, adding a language is a matter of adding data, plus one menu entry in `Game.cs`. No game logic needs to change.

<!-- TODO: add the reason for choosing Italian specifically, e.g. a course requirement, the intended audience, or a team member's language skills. -->

## Decision

Add **Italian** as the fourth supported language:

1. Add an `Italian` section to `language_data.json` with all 69 dictionary keys and all five maps.
2. Add `4. Italiano` to the language selection menu in `Game.cs` and map it to `"Italian"`. Any other input still defaults to English.
3. Add unit tests that cover the Italian welcome message, all three endings in Italian, Italian race and occupation input, and a check that every non-English language contains the core message keys.

Translation conventions:

- **ASCII only.** Spanish and French in this file are written without accents, so Italian follows suit. In place of accented vowels it uses the standard Italian apostrophe fallback (`e'`, `piu'`, `agilita'`), which keeps the text readable when the console doesn't render accents correctly.
- **Halfling is "Mezzuomo"**, the usual Italian fantasy term.
- **`roll` stays in English**, because `Player.PromptRoll` checks for that exact word in every language.
- **Weapon names are shown without an article.** Italian articles and possessives depend on grammatical gender, and the message templates insert the weapon name as plain text, so templates are phrased to avoid needing one (for example `impugni la tua arma ({1})`).

## Consequences

**Positive**

- Italian-speaking players can play the full game, including typing race and occupation answers in Italian.
- No changes to game logic, so there's little risk of breaking existing behavior. All existing tests still pass.
- The new key-coverage test catches a missing message in any language, which also protects future translations.

**Negative / trade-offs**

- Every new or changed message must now be added in four languages instead of three.
- Without accents, Italian text looks slightly less polished than properly accented text would.
- A few lines read stiffly because the templates can't adjust for grammatical gender (for example `attacca Gino con artigli`). Fixing this properly would need per-language templates or gendered weapon data.
- The translations were not reviewed by a native speaker.

## Alternatives considered

- **Use accented characters.** Rejected for consistency with the other languages and to avoid console encoding problems on Windows. This can be revisited if the console output encoding is set to UTF-8 and all languages move to accented text together.
- **Store each language in its own file** (e.g. `lang/it.json`). That would scale better with more languages, but it means changing `Messages.ReadDictionary` and the build settings. Out of scope for adding one language.
- **Accept the English word `halfling` as Italian input as well.** Not done for now, to match how the other languages work. It's a one-line addition to `raceMap` if wanted.
