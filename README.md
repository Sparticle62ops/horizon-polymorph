# horizon-polymorph (1:1 port)

Exact 1:1 Rust translation of [Polymorph](https://github.com/IllusiveSoulworks/polymorph)
by Illusive Soulworks (LGPL-3.0-or-later), per the port policy (2026-11-12):
never clean-room, always exact code. The upstream license and copyright are
carried in the sources; ported by sparticle.

Polymorph resolves recipe conflicts: when several recipes produce the same
item from different ingredient layouts, the crafter picks between them instead
of the menu silently taking the first match.

## What the port carries

The full upstream logic tree, translated 1:1:

- `RecipePair` / `RecipeCache` / `RecipeData` / `BlockEntityRecipeData` /
  `PlayerRecipeData` - the conflict keying (per-block-entity and per-player
  selected recipe), cache invalidation, and data write-back.
- `RecipeManager` - the lookup ladder (`get_player_recipe` then
  `get_block_entity_recipe`), `PolymorphApiSeam` for the engine-facing
  recipe access.
- `wrappers.rs` - `IngredientWrapper` / `RecipeWrapper` /
  `CraftingRecipeWrapper` with the shaped/shapeless conflict comparison
  (`conflicts`) transcribed from upstream.
- `network.rs` - all seven packets (C2S: `PlayerRecipeSelection`,
  `BlockEntityListener`, `PersistentRecipeSelection`; S2C:
  `PlayerRecipeSync`, `RecipesList`, `HighlightRecipe`, `UpdatePreview`)
  with their `FriendlyByteBuf` field order, `ServerPacketEnv` /
  `ClientPacketEnv` seams, and the client handler ladder.
- `block_entity_ticker.rs` - the ticker-side listener registration.
- `client_widget.rs` - the recipes widget the crafting screens show.

## Carrier seams

Where Minecraft/Fabric APIs do not exist on Horizon, the counterpart is a
documented carrier seam with the same shape and semantics:

- `ServerPacketEnv` / `ClientPacketEnv` - Horizon's `native_send_packet`
  (API 409) and `native_client_packet` (API 408) play-packet surfaces carry
  the C2S/S2C traffic.
- `PolymorphApiSeam` - Horizon's recipe registry access
  (`native_recipes_of_type`, API 36) answers the manager's recipe listing.
- Screen widget - Horizon's screen lifecycle observations
  (`native_screen_opening`/`init_post`/`closing`, APIs 367/366/365) and GUI
  painting (API 411) carry the widget draw and click handling.
- Block entity data - Horizon's level/entity handles (`LevelRef`, API 381)
  carry the block-entity recipe data reads and writes.
