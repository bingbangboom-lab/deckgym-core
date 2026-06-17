# Cards Needing Implementation

Generated from `cargo run --bin card_status`.

## Summary

- `  Total cards:      3406`
- `  Complete:         2972 (87.3%)`
- `  Incomplete:       434 (12.7%)`
- `    Attack effect not implemented: 283`
- `    Ability not implemented: 91`
- `    Trainer logic not implemented: 52`
- `    Tool not implemented: 8`

## What "not implemented" means

`card_status` does not mean the card data is missing. It means the engine has not yet wired that card's effect text to executable game logic. Card data lives in this repository's `database.json`, and the upstream English JSON files are also available in the Pokémon TCG Pocket card database at <https://github.com/bingbangboom-lab/pokemon-tcg-pocket-card-database/tree/main/cards/en>.

These are normal contribution tasks, not a hard limitation of the repository. Most entries can be implemented by mapping an existing effect text to an existing mechanic, generalizing an existing mechanic, or adding a new mechanic plus Game-level tests. A few cards may require new engine state or hooks, but the codebase already has extension points for those cases.

The status categories are produced by `src/card_validation.rs`:

- **Attack effect not implemented**: a Pokémon attack has effect text, but that text is not present in `EFFECT_MECHANIC_MAP`.
- **Ability not implemented**: a Pokémon ability has effect text, but that text is not mapped to an `AbilityMechanic`.
- **Trainer logic not implemented**: a Trainer card cannot generate legal play actions through trainer move generation.
- **Tool not implemented**: a Tool card's effect is not registered as implemented in `src/tools.rs`.

## Information needed to implement a card

For any card, start with the exact card IDs, effect text, and any full-art/reprint variants that share the same text:

```bash
cargo run --bin search "Venusaur"
cargo run --bin search "Venusaur" --attack "Giant Bloom"
cargo run --bin card_status --incomplete-only
```

The search tool reads `database.json` and prints the relevant Pokémon or Trainer JSON, including attacks, abilities, trainer type, IDs, names, and effect text. The upstream card database can be used as a cross-check when adding or updating card data, but implementation should be driven by the exact effect text in this repository because the mechanic maps key on that text.

## Implementation guide by status

### Attack effect not implemented

1. Look up the card and attack:
   ```bash
   cargo run --bin search "Card Name" --attack "Attack Name"
   ```
2. Copy the attack's exact `effect` text.
3. Search `src/actions/effect_mechanic_map.rs` for the same or similar text.
4. Reuse an existing `Mechanic` from `src/actions/attacks/mechanic.rs` when possible; otherwise add a new variant.
5. Add or uncomment the map entry in `src/actions/effect_mechanic_map.rs` for every card variant with the same effect template.
6. Implement the mechanic in `forecast_effect_attack_by_mechanic` in `src/actions/apply_attack_action.rs`, using `Outcomes` for deterministic effects and coin-flip branches.
7. Add focused Game-level tests under the matching `tests/` area before implementation.

### Ability not implemented

1. Look up the card:
   ```bash
   cargo run --bin search "Card Name"
   ```
2. Copy the ability's exact effect text and identify all variants with that ability.
3. Search `src/actions/effect_ability_mechanic_map.rs` for matching or similar text.
4. Reuse or add an `AbilityMechanic` in `src/actions/abilities/mechanic.rs`.
5. Map all matching effect text in `src/actions/effect_ability_mechanic_map.rs`.
6. Implement active ability logic in `forecast_ability_by_mechanic` in `src/actions/apply_abilities_action.rs` and move generation in `src/move_generation/move_generation_abilities.rs`.
7. For passive or hook-driven abilities, wire the mechanic through the relevant `src/hooks/` file and make direct ability use unavailable.
8. Add focused Game-level tests before implementation.

### Trainer logic not implemented

1. Look up the Trainer:
   ```bash
   cargo run --bin search "Trainer Name"
   ```
2. Record all card IDs/reprints and the exact effect text.
3. Add move-generation support in `src/move_generation/move_generation_trainer.rs` so the card appears only when legally playable.
4. Add apply logic in `forecast_trainer_action` in `src/actions/apply_trainer_action.rs`.
5. Use shared search/draw helpers when possible; add state, turn effects, or hooks only when the effect requires it.
6. Add focused Game-level tests before implementation if the behavior is non-trivial.

### Tool not implemented

1. Look up the Tool:
   ```bash
   cargo run --bin search "Tool Name"
   ```
2. Record all card IDs/reprints and exact effect text.
3. In `src/tools.rs`, add a static effect-text constant via `tool_effect_text_from_card_id(...)` and register the effect in `is_tool_effect_implemented()`.
4. Add attachment restrictions in `can_attach_tool_to()` if the Tool can attach only to certain Pokémon.
5. Implement ongoing effects in the appropriate hook, commonly under `src/hooks/`; HP modifiers usually belong in `PlayedCard::get_effective_total_hp()`.
6. Ensure Tool play is handled by trainer forecasting and add focused Game-level tests under `tests/tools/`.

## Incomplete cards

| ID | Name | Missing implementation |
| --- | --- | --- |
| A1 159 | Kabutops | Attack effect not implemented |
| A1 168 | Nidoqueen | Attack effect not implemented |
| A1 197 | Persian | Attack effect not implemented |
| A1 209 | Porygon | Ability not implemented |
| A1 226 | Lt. Surge | Trainer logic not implemented |
| A1 240 | Nidoqueen | Attack effect not implemented |
| A1 249 | Porygon | Ability not implemented |
| A1 273 | Lt. Surge | Trainer logic not implemented |
| A1 283 | Mew | Attack effect not implemented |
| A1a 014 | Volcarona | Attack effect not implemented |
| A1a 031 | Mew | Attack effect not implemented |
| A1a 038 | Florges | Attack effect not implemented |
| A1a 062 | Chatot | Attack effect not implemented |
| A1a 064 | Pokémon Flute | Trainer logic not implemented |
| A1a 066 | Budding Expeditioner | Trainer logic not implemented |
| A1a 067 | Blue | Trainer logic not implemented |
| A1a 080 | Budding Expeditioner | Trainer logic not implemented |
| A1a 081 | Blue | Trainer logic not implemented |
| A2 032 | Piloswine | Ability not implemented |
| A2 033 | Mamoswine | Ability not implemented |
| A2 034 | Regice | Ability not implemented |
| A2 044 | Snover | Attack effect not implemented |
| A2 062 | Rotom | Attack effect not implemented |
| A2 075 | Uxie | Attack effect not implemented |
| A2 076 | Mesprit | Attack effect not implemented |
| A2 082 | Rhyperior | Attack effect not implemented |
| A2 106 | Drapion | Attack effect not implemented |
| A2 107 | Croagunk | Attack effect not implemented |
| A2 108 | Toxicroak | Attack effect not implemented |
| A2 114 | Bastiodon | Ability not implemented |
| A2 115 | Wormadam | Attack effect not implemented |
| A2 125 | Lickilicky ex | Attack effect not implemented |
| A2 129 | Porygon-Z | Attack effect not implemented |
| A2 135 | Bidoof | Attack effect not implemented |
| A2 140 | Purugly | Attack effect not implemented |
| A2 142 | Fan Rotom | Attack effect not implemented |
| A2 149 | Lum Berry | Tool not implemented |
| A2 151 | Team Galactic Grunt | Trainer logic not implemented |
| A2 160 | Mamoswine | Ability not implemented |
| A2 164 | Rotom | Attack effect not implemented |
| A2 166 | Mesprit | Attack effect not implemented |
| A2 169 | Rhyperior | Attack effect not implemented |
| A2 173 | Croagunk | Attack effect not implemented |
| A2 177 | Bidoof | Attack effect not implemented |
| A2 189 | Lickilicky ex | Attack effect not implemented |
| A2 191 | Team Galactic Grunt | Trainer logic not implemented |
| A2 203 | Lickilicky ex | Attack effect not implemented |
| A2a 013 | Heatran | Ability not implemented |
| A2a 017 | Whiscash | Attack effect not implemented |
| A2a 021 | Abomasnow | Ability not implemented |
| A2a 026 | Raichu | Ability not implemented |
| A2a 034 | Unown | Ability not implemented |
| A2a 035 | Rotom | Ability not implemented |
| A2a 052 | Toxicroak | Attack effect not implemented |
| A2a 055 | Magnezone | Ability not implemented |
| A2a 056 | Mawile | Attack effect not implemented |
| A2a 060 | Origin Forme Dialga | Attack effect not implemented |
| A2a 061 | Giratina | Attack effect not implemented |
| A2a 065 | Noctowl | Attack effect not implemented |
| A2a 078 | Unown | Ability not implemented |
| A2b 004 | Pinsir | Attack effect not implemented |
| A2b 021 | Tatsugiri | Ability not implemented |
| A2b 051 | Grafaiai | Ability not implemented |
| A2b 075 | Tatsugiri | Ability not implemented |
| A2b 076 | Grafaiai | Ability not implemented |
| A3 021 | Wimpod | Ability not implemented |
| A3 027 | Alolan Marowak | Attack effect not implemented |
| A3 034 | Oricorio | Attack effect not implemented |
| A3 036 | Salazzle | Attack effect not implemented |
| A3 048 | Primarina | Ability not implemented |
| A3 050 | Wishiwashi | Attack effect not implemented |
| A3 051 | Wishiwashi ex | Attack effect not implemented |
| A3 053 | Araquanid | Attack effect not implemented |
| A3 054 | Pyukumuku | Ability not implemented |
| A3 068 | Tapu Koko | Attack effect not implemented |
| A3 083 | Mimikyu | Attack effect not implemented |
| A3 087 | Lunala ex | Ability not implemented |
| A3 096 | Conkeldurr | Ability not implemented |
| A3 104 | Passimian ex | Ability not implemented |
| A3 107 | Alolan Raticate | Attack effect not implemented |
| A3 111 | Alolan Muk ex | Attack effect not implemented |
| A3 118 | Alolan Dugtrio | Attack effect not implemented |
| A3 124 | Drampa | Attack effect not implemented |
| A3 127 | Kommo-o | Attack effect not implemented |
| A3 132 | Hawlucha | Attack effect not implemented |
| A3 140 | Oranguru | Attack effect not implemented |
| A3 143 | Fishing Net | Trainer logic not implemented |
| A3 145 | Rotom Dex | Trainer logic not implemented |
| A3 148 | Acerola | Trainer logic not implemented |
| A3 152 | Lana | Trainer logic not implemented |
| A3 153 | Sophocles | Trainer logic not implemented |
| A3 154 | Mallow | Trainer logic not implemented |
| A3 160 | Alolan Marowak | Attack effect not implemented |
| A3 163 | Pyukumuku | Ability not implemented |
| A3 166 | Tapu Koko | Attack effect not implemented |
| A3 176 | Drampa | Attack effect not implemented |
| A3 184 | Wishiwashi ex | Attack effect not implemented |
| A3 186 | Lunala ex | Ability not implemented |
| A3 187 | Passimian ex | Ability not implemented |
| A3 188 | Alolan Muk ex | Attack effect not implemented |
| A3 190 | Acerola | Trainer logic not implemented |
| A3 194 | Lana | Trainer logic not implemented |
| A3 195 | Sophocles | Trainer logic not implemented |
| A3 196 | Mallow | Trainer logic not implemented |
| A3 202 | Wishiwashi ex | Attack effect not implemented |
| A3 204 | Lunala ex | Ability not implemented |
| A3 205 | Passimian ex | Ability not implemented |
| A3 206 | Alolan Muk ex | Attack effect not implemented |
| A3 238 | Lunala ex | Ability not implemented |
| A3a 031 | Claydol | Ability not implemented |
| A3a 037 | Alolan Meowth | Attack effect not implemented |
| A3a 041 | Krookodile | Attack effect not implemented |
| A3a 058 | Bewear | Attack effect not implemented |
| A3a 063 | Beast Wall | Trainer logic not implemented |
| A3a 066 | Beastite | Tool not implemented |
| A3a 068 | Looker | Trainer logic not implemented |
| A3a 073 | Alolan Meowth | Attack effect not implemented |
| A3a 082 | Looker | Trainer logic not implemented |
| A3b 002 | Leafeon | Attack effect not implemented |
| A3b 005 | Tsareena | Attack effect not implemented |
| A3b 027 | Galvantula | Attack effect not implemented |
| A3b 035 | Mimikyu | Attack effect not implemented |
| A3b 037 | Alcremie | Attack effect not implemented |
| A3b 045 | Purrloin | Attack effect not implemented |
| A3b 054 | Drampa | Attack effect not implemented |
| A3b 059 | Ambipom | Ability not implemented |
| A3b 067 | Leftovers | Tool not implemented |
| A3b 069 | Penny | Trainer logic not implemented |
| A3b 070 | Leafeon | Attack effect not implemented |
| A3b 086 | Penny | Trainer logic not implemented |
| A3b 093 | Pinsir | Attack effect not implemented |
| A4 003 | Bellossom | Attack effect not implemented |
| A4 005 | Tangrowth | Attack effect not implemented |
| A4 010 | Meganium | Attack effect not implemented |
| A4 015 | Jumpluff | Ability not implemented |
| A4 023 | Cherubi | Ability not implemented |
| A4 029 | Typhlosion | Ability not implemented |
| A4 033 | Entei | Attack effect not implemented |
| A4 034 | Ho-Oh ex | Attack effect not implemented |
| A4 037 | Heatmor | Attack effect not implemented |
| A4 040 | Politoed | Ability not implemented |
| A4 045 | Gyarados | Attack effect not implemented |
| A4 050 | Azumarill | Ability not implemented |
| A4 056 | Octillery | Attack effect not implemented |
| A4 063 | Swanna | Attack effect not implemented |
| A4 065 | Lanturn ex | Attack effect not implemented |
| A4 082 | Xatu | Attack effect not implemented |
| A4 084 | Unown | Ability not implemented |
| A4 085 | Unown | Ability not implemented |
| A4 086 | Wobbuffet | Attack effect not implemented |
| A4 091 | Musharna | Attack effect not implemented |
| A4 118 | Houndoom | Attack effect not implemented |
| A4 119 | Tyranitar | Ability not implemented |
| A4 130 | Fearow | Attack effect not implemented |
| A4 133 | Kangaskhan | Attack effect not implemented |
| A4 136 | Porygon2 | Ability not implemented |
| A4 140 | Hoothoot | Ability not implemented |
| A4 142 | Aipom | Attack effect not implemented |
| A4 148 | Smeargle | Attack effect not implemented |
| A4 150 | Bouffalant | Attack effect not implemented |
| A4 152 | Squirt Bottle | Trainer logic not implemented |
| A4 154 | Dark Pendant | Tool not implemented |
| A4 155 | Rescue Scarf | Tool not implemented |
| A4 159 | Fisher | Trainer logic not implemented |
| A4 160 | Jasmine | Trainer logic not implemented |
| A4 161 | Hiker | Trainer logic not implemented |
| A4 163 | Bellossom | Attack effect not implemented |
| A4 169 | Octillery | Attack effect not implemented |
| A4 174 | Xatu | Attack effect not implemented |
| A4 175 | Wobbuffet | Attack effect not implemented |
| A4 179 | Tyranitar | Ability not implemented |
| A4 182 | Hoothoot | Ability not implemented |
| A4 184 | Smeargle | Attack effect not implemented |
| A4 187 | Ho-Oh ex | Attack effect not implemented |
| A4 189 | Lanturn ex | Attack effect not implemented |
| A4 199 | Fisher | Trainer logic not implemented |
| A4 200 | Jasmine | Trainer logic not implemented |
| A4 201 | Hiker | Trainer logic not implemented |
| A4 204 | Lanturn ex | Attack effect not implemented |
| A4 210 | Ho-Oh ex | Attack effect not implemented |
| A4 215 | Gyarados | Attack effect not implemented |
| A4 225 | Nidoqueen | Attack effect not implemented |
| A4 239 | Lickilicky ex | Attack effect not implemented |
| A4 240 | Ho-Oh ex | Attack effect not implemented |
| A4a 006 | Celebi | Attack effect not implemented |
| A4a 016 | Tentacruel | Attack effect not implemented |
| A4a 018 | Slowking | Attack effect not implemented |
| A4a 031 | Boltund | Attack effect not implemented |
| A4a 035 | Galarian Cursola | Ability not implemented |
| A4a 037 | Latios | Ability not implemented |
| A4a 041 | Dugtrio | Attack effect not implemented |
| A4a 042 | Poliwrath ex | Attack effect not implemented |
| A4a 048 | Seviper | Attack effect not implemented |
| A4a 062 | Miltank | Attack effect not implemented |
| A4a 063 | Azurill | Attack effect not implemented |
| A4a 068 | Memory Light | Tool not implemented |
| A4a 069 | Whitney | Trainer logic not implemented |
| A4a 070 | Traveling Merchant | Trainer logic not implemented |
| A4a 071 | Morty | Trainer logic not implemented |
| A4a 075 | Latios | Ability not implemented |
| A4a 077 | Azurill | Attack effect not implemented |
| A4a 082 | Poliwrath ex | Attack effect not implemented |
| A4a 083 | Whitney | Trainer logic not implemented |
| A4a 084 | Traveling Merchant | Trainer logic not implemented |
| A4a 085 | Morty | Trainer logic not implemented |
| A4a 089 | Poliwrath ex | Attack effect not implemented |
| A4a 097 | Pyukumuku | Ability not implemented |
| A4b 018 | Jumpluff | Ability not implemented |
| A4b 019 | Jumpluff | Ability not implemented |
| A4b 025 | Cherubi | Ability not implemented |
| A4b 026 | Cherubi | Ability not implemented |
| A4b 068 | Ho-Oh ex | Attack effect not implemented |
| A4b 076 | Heatran | Ability not implemented |
| A4b 077 | Heatran | Ability not implemented |
| A4b 122 | Wishiwashi | Attack effect not implemented |
| A4b 123 | Wishiwashi | Attack effect not implemented |
| A4b 124 | Wishiwashi ex | Attack effect not implemented |
| A4b 142 | Lanturn ex | Attack effect not implemented |
| A4b 184 | Lunala ex | Ability not implemented |
| A4b 187 | Alcremie | Attack effect not implemented |
| A4b 188 | Alcremie | Attack effect not implemented |
| A4b 223 | Passimian ex | Ability not implemented |
| A4b 235 | Alolan Muk ex | Attack effect not implemented |
| A4b 284 | Lickilicky ex | Attack effect not implemented |
| A4b 362 | Ho-Oh ex | Attack effect not implemented |
| A4b 367 | Lunala ex | Ability not implemented |
| B1 010 | Shiftry | Attack effect not implemented |
| B1 018 | Lilligant | Ability not implemented |
| B1 027 | Rillaboom | Ability not implemented |
| B1 029 | Arcanine | Attack effect not implemented |
| B1 032 | Ho-Oh | Attack effect not implemented |
| B1 055 | Ludicolo | Attack effect not implemented |
| B1 060 | Luvdisc | Attack effect not implemented |
| B1 069 | Jellicent | Ability not implemented |
| B1 080 | Eiscue | Ability not implemented |
| B1 084 | Ampharos | Attack effect not implemented |
| B1 094 | Dedenne | Attack effect not implemented |
| B1 105 | Dusknoir | Ability not implemented |
| B1 114 | Gothitelle | Attack effect not implemented |
| B1 129 | Hippowdon | Attack effect not implemented |
| B1 134 | Archeops | Attack effect not implemented |
| B1 136 | Golurk | Attack effect not implemented |
| B1 140 | Crabominable | Attack effect not implemented |
| B1 147 | Coalossal | Attack effect not implemented |
| B1 153 | Drapion | Attack effect not implemented |
| B1 164 | Morgrem | Attack effect not implemented |
| B1 176 | Druddigon | Attack effect not implemented |
| B1 182 | Pidgeot | Attack effect not implemented |
| B1 186 | Ambipom | Attack effect not implemented |
| B1 194 | Delcatty | Ability not implemented |
| B1 195 | Spinda | Attack effect not implemented |
| B1 203 | Stoutland | Attack effect not implemented |
| B1 211 | Wooloo | Attack effect not implemented |
| B1 213 | Prank Spinner | Trainer logic not implemented |
| B1 215 | Hitting Hammer | Trainer logic not implemented |
| B1 218 | Sitrus Berry | Tool not implemented |
| B1 220 | Lucky Mittens | Tool not implemented |
| B1 222 | Hala | Trainer logic not implemented |
| B1 229 | Rillaboom | Ability not implemented |
| B1 233 | Ludicolo | Attack effect not implemented |
| B1 234 | Jellicent | Ability not implemented |
| B1 236 | Eiscue | Ability not implemented |
| B1 241 | Hippowdon | Attack effect not implemented |
| B1 242 | Archeops | Attack effect not implemented |
| B1 248 | Delcatty | Ability not implemented |
| B1 249 | Stoutland | Attack effect not implemented |
| B1 267 | Hala | Trainer logic not implemented |
| B1 313 | Persian | Attack effect not implemented |
| B1 316 | Bidoof | Attack effect not implemented |
| B1 324 | Passimian ex | Ability not implemented |
| B1 329 | Lilligant | Ability not implemented |
| B1a 035 | Spritzee | Attack effect not implemented |
| B1a 048 | Liepard | Attack effect not implemented |
| B1a 099 | Lunala ex | Ability not implemented |
| B2 002 | Ledian | Attack effect not implemented |
| B2 005 | Roserade | Attack effect not implemented |
| B2 014 | Buzzwole | Attack effect not implemented |
| B2 016 | Eldegoss | Attack effect not implemented |
| B2 022 | Oricorio | Attack effect not implemented |
| B2 032 | Delibird | Attack effect not implemented |
| B2 042 | Aurorus | Attack effect not implemented |
| B2 050 | Alolan Raichu | Ability not implemented |
| B2 053 | Minun | Attack effect not implemented |
| B2 055 | Toxtricity ex | Attack effect not implemented |
| B2 075 | Polteageist | Ability not implemented |
| B2 076 | Indeedee | Attack effect not implemented |
| B2 078 | Sandslash | Attack effect not implemented |
| B2 079 | Machop | Attack effect not implemented |
| B2 084 | Medicham | Attack effect not implemented |
| B2 087 | Gigalith ex | Attack effect not implemented |
| B2 090 | Tyrantrum | Attack effect not implemented |
| B2 092 | Falinks | Ability not implemented |
| B2 097 | Alolan Muk | Ability not implemented |
| B2 103 | Spiritomb | Ability not implemented |
| B2 104 | Purrloin | Attack effect not implemented |
| B2 107 | Scrafty | Attack effect not implemented |
| B2 108 | Yveltal | Attack effect not implemented |
| B2 109 | Guzzlord | Attack effect not implemented |
| B2 111 | Galarian Perrserker | Ability not implemented |
| B2 117 | Galarian Stunfisk | Attack effect not implemented |
| B2 120 | Aegislash | Attack effect not implemented |
| B2 130 | Smeargle | Ability not implemented |
| B2 131 | Lugia | Attack effect not implemented |
| B2 133 | Swellow | Ability not implemented |
| B2 142 | Tandemaus | Attack effect not implemented |
| B2 143 | Maushold | Attack effect not implemented |
| B2 151 | Juggler | Trainer logic not implemented |
| B2 157 | Roserade | Attack effect not implemented |
| B2 159 | Buzzwole | Attack effect not implemented |
| B2 161 | Oricorio | Attack effect not implemented |
| B2 163 | Aurorus | Attack effect not implemented |
| B2 165 | Minun | Attack effect not implemented |
| B2 169 | Indeedee | Attack effect not implemented |
| B2 171 | Tyrantrum | Attack effect not implemented |
| B2 172 | Falinks | Ability not implemented |
| B2 173 | Alolan Muk | Ability not implemented |
| B2 174 | Purrloin | Attack effect not implemented |
| B2 175 | Yveltal | Attack effect not implemented |
| B2 177 | Galarian Perrserker | Ability not implemented |
| B2 184 | Toxtricity ex | Attack effect not implemented |
| B2 187 | Gigalith ex | Attack effect not implemented |
| B2 192 | Juggler | Trainer logic not implemented |
| B2 198 | Toxtricity ex | Attack effect not implemented |
| B2 200 | Gigalith ex | Attack effect not implemented |
| B2 217 | Latios | Ability not implemented |
| B2 221 | Kabutops | Attack effect not implemented |
| B2 226 | Ho-Oh ex | Attack effect not implemented |
| B2a 013 | Scovillain | Attack effect not implemented |
| B2a 026 | Wugtrio | Attack effect not implemented |
| B2a 028 | Palafin | Attack effect not implemented |
| B2a 031 | Veluza | Attack effect not implemented |
| B2a 062 | Ting-Lu | Attack effect not implemented |
| B2a 063 | Koraidon | Attack effect not implemented |
| B2a 070 | Grafaiai | Attack effect not implemented |
| B2a 074 | Tinkaton | Attack effect not implemented |
| B2a 080 | Oinkologne | Attack effect not implemented |
| B2a 082 | Maushold | Attack effect not implemented |
| B2a 084 | Cyclizar | Attack effect not implemented |
| B2a 089 | Iono | Trainer logic not implemented |
| B2a 092 | Penny | Trainer logic not implemented |
| B2a 099 | Maushold | Attack effect not implemented |
| B2a 106 | Iono | Trainer logic not implemented |
| B2a 109 | Penny | Trainer logic not implemented |
| B2b 004 | Illumise | Attack effect not implemented |
| B2b 006 | Trevenant | Attack effect not implemented |
| B2b 013 | Magmortar | Attack effect not implemented |
| B2b 017 | Lapras | Attack effect not implemented |
| B2b 030 | Mew | Attack effect not implemented |
| B2b 032 | Grumpig | Attack effect not implemented |
| B2b 035 | Groudon | Attack effect not implemented |
| B2b 046 | Forretress | Attack effect not implemented |
| B2b 070 | Lapras | Attack effect not implemented |
| B2b 072 | Groudon | Attack effect not implemented |
| B2b 085 | Mew | Attack effect not implemented |
| B2b 086 | Mew | Attack effect not implemented |
| B2b 090 | Trevenant | Attack effect not implemented |
| B2b 104 | Forretress | Attack effect not implemented |
| B3 009 | Surskit | Attack effect not implemented |
| B3 012 | Breloom | Attack effect not implemented |
| B3 013 | Budew | Attack effect not implemented |
| B3 025 | Victini | Ability not implemented |
| B3 035 | Politoed | Attack effect not implemented |
| B3 039 | Quagsire | Attack effect not implemented |
| B3 045 | Regice | Attack effect not implemented |
| B3 051 | Rapid Strike Urshifu | Attack effect not implemented |
| B3 061 | Toxtricity | Attack effect not implemented |
| B3 067 | Diancie | Attack effect not implemented |
| B3 068 | Oricorio | Attack effect not implemented |
| B3 086 | Sawk | Attack effect not implemented |
| B3 089 | Meloetta | Attack effect not implemented |
| B3 094 | Coalossal | Attack effect not implemented |
| B3 097 | Kubfu | Attack effect not implemented |
| B3 103 | Qwilfish | Attack effect not implemented |
| B3 108 | Amoonguss | Attack effect not implemented |
| B3 110 | Mandibuzz | Attack effect not implemented |
| B3 112 | Malamar | Attack effect not implemented |
| B3 113 | Single Strike Urshifu | Attack effect not implemented |
| B3 114 | Zarude | Attack effect not implemented |
| B3 118 | Bronzong | Attack effect not implemented |
| B3 130 | Dunsparce | Attack effect not implemented |
| B3 134 | Regigigas | Ability not implemented |
| B3 136 | Watchog | Attack effect not implemented |
| B3 137 | Lillipup | Attack effect not implemented |
| B3 151 | Cheren | Trainer logic not implemented |
| B3 159 | Budew | Attack effect not implemented |
| B3 162 | Quagsire | Attack effect not implemented |
| B3 166 | Oricorio | Attack effect not implemented |
| B3 170 | Meloetta | Attack effect not implemented |
| B3 172 | Kubfu | Attack effect not implemented |
| B3 175 | Malamar | Attack effect not implemented |
| B3 192 | Cheren | Trainer logic not implemented |
| B3 210 | Jellicent | Ability not implemented |
| B3a 022 | Espathra | Attack effect not implemented |
| B3a 025 | Scream Tail | Attack effect not implemented |
| B3a 033 | Garganacl | Ability not implemented |
| B3a 035 | Sandy Shocks | Attack effect not implemented |
| B3a 039 | Weavile | Attack effect not implemented |
| B3a 040 | Sableye | Attack effect not implemented |
| B3a 043 | Kingambit | Attack effect not implemented |
| B3a 045 | Glimmora | Ability not implemented |
| B3a 051 | Iron Treads | Attack effect not implemented |
| B3a 053 | Walking Wake | Attack effect not implemented |
| B3a 054 | Gouging Fire | Attack effect not implemented |
| B3a 055 | Raging Bolt | Attack effect not implemented |
| B3a 058 | Farigiraf | Attack effect not implemented |
| B3a 060 | Dudunsparce | Attack effect not implemented |
| B3a 078 | Glimmora | Ability not implemented |
| B3a 079 | Raging Bolt | Attack effect not implemented |
| P-A 003 | Hand Scope | Trainer logic not implemented |
| P-A 004 | Pokédex | Trainer logic not implemented |
| P-A 008 | Pokédex | Trainer logic not implemented |
| P-A 028 | Volcarona | Attack effect not implemented |
| P-A 044 | Raichu | Ability not implemented |
| P-A 047 | Staraptor | Ability not implemented |
| P-A 063 | Rayquaza | Attack effect not implemented |
| P-A 066 | Mimikyu | Attack effect not implemented |
| P-A 073 | Toucannon | Attack effect not implemented |
| P-A 081 | Ultra Necrozma ex | Attack effect not implemented |
| P-A 087 | Alcremie | Attack effect not implemented |
| P-A 095 | Chinchou | Attack effect not implemented |
| P-A 096 | Houndoom | Attack effect not implemented |
| P-A 097 | Kangaskhan | Attack effect not implemented |
| P-A 107 | Miltank | Attack effect not implemented |
| P-A 113 | Mimikyu | Attack effect not implemented |
| P-B 006 | Mega Pidgeot ex | Attack effect not implemented |
| P-B 013 | Arcanine | Attack effect not implemented |
| P-B 014 | Magikarp | Attack effect not implemented |
| P-B 029 | Mega Medicham ex | Attack effect not implemented |
| P-B 034 | Smoliv | Attack effect not implemented |
| P-B 038 | Tinkaton | Attack effect not implemented |
| P-B 049 | Victini | Ability not implemented |
| P-B 060 | Zarude | Attack effect not implemented |
| P-B 068 | Dudunsparce | Attack effect not implemented |
| P-B 070 | Sableye | Attack effect not implemented |
