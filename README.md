# palace — Nexo content pack for High Palace

Namespace **`palace`**. Everything here is generated from the art repo
[`grntsmth/minecraftassets`](https://github.com/grntsmth/minecraftassets) by `tools/convert.py`,
driven by that repo's `assets.yml` inventory: an asset ships when its `status` is `plugin`.

```
items/            generated Nexo item YAML, one file per theme (dings, furniture, blocks, items)
pack/assets/palace/{models,textures}/   generated; wiped and rebuilt on every run
overrides/<item_id>.yml                 hand adjustments, deep-merged over the generated entry
ledger.yml        NOTEBLOCK custom_variation pins — never edit or reuse a number once a block is placed
tools/convert.py  the generator
```

## Rebuild

```bash
python3 -I tools/convert.py --art /home/opc/mc-assets/minecraftassets
```

Exit code 1 means at least one chosen model was refused for unresolved texture refs; the report lists them.
Fix the art (add the texture, or re-export the model) or set that asset back to `undecided`.

## Rules the generator follows

| art inventory entry | becomes |
|---|---|
| `model-json` in dings / furniture / stations / calligraphy | furniture item: display entity, barrier hitbox sized from the model's bounding box, rotatable, 8 facings; `*_on` / `*_lit` ids get a light level 10 at `0,1,0` |
| `model-json` in blocks | NOTEBLOCK block using that model |
| `texture` in blocks | NOTEBLOCK `cube_all` block; if `<name>_top.png` is also chosen, a `cube_column` with that end texture |
| `texture` in items / armor | plain item (`item/generated`) |
| `texture` in dings / npc / mobs | copied into the pack only (model dependencies; no standalone item) |
| bbmodel, geo, animation, jem, misc | skipped and listed |

Texture refs inside models are rewritten to `palace:block/...` for repo textures and `minecraft:block/...` for the
vanilla stems Blockbench left bare. Item ids are the file stem with a leading `model_` dropped.

## Deploying to the live server (owner runs these)

Nothing here touches `/srv/minecraft`. To deploy:

```bash
sudo rsync -a --delete items/ /srv/minecraft/server/plugins/Nexo/items/palace/
```

```bash
sudo rsync -a --delete pack/assets/palace/ /srv/minecraft/server/plugins/Nexo/pack/assets/palace/
```

```bash
sudo chown -R minecraft:minecraft /srv/minecraft/server/plugins/Nexo/items/palace /srv/minecraft/server/plugins/Nexo/pack/assets/palace
```

```bash
mc rcon "nexo reload all"
```

Nexo regenerates and re-uploads the pack to Hermes on reload; players get it on next join. Test any new
batch in a `/srv/minecraft/scratch-*` sandbox first when it changes block variations.

## Known gaps

- Furniture states (on/off, empty/full dings) are shipped as separate items for now. Nexo's furniture-states
  mechanic can fold them later.
- Hitboxes are bounding boxes, so an L-shaped or hollow model gets a solid barrier block. Trim with an override.
- Items are all `material: PAPER`. Armor pieces are plain items, not wearable, until the equipment layer exists.
