# Edited Chest UI

A customized chest-style UI for **Minecraft Bedrock Edition**, built with the Minecraft Script API and `@minecraft/server-ui`.

This project provides a `ChestFormData` class for creating chest-style menus with item textures, custom names, descriptions, stack counts, durability indicators, enchantment glint, and reusable menu patterns.

> **Project type:** Modified chest UI source  
> **Author:** Mogley  
> **Language:** JavaScript (ES modules)  
> **Status:** Community project

## Features

- Multiple chest UI sizes.
- Custom item names and descriptions.
- Item textures using Minecraft item identifiers or supported custom content keys.
- Stack-size indicators.
- Durability indicators.
- Enchantment glint.
- Pattern-based menu layouts.
- Optional display of the player's inventory.
- Chainable methods for building menus.

## Requirements

- Minecraft Bedrock Edition.
- A compatible Minecraft Script API environment.
- The `@minecraft/server` module.
- The `@minecraft/server-ui` module.
- The project's supporting files, including `constants.js` and `typeIds.js`.

The exact supported Minecraft version depends on the API versions and dependencies configured by the complete project.

## Installation

1. Obtain the complete source code.
2. Copy `chest-ui.js` and its required supporting files into your behavior pack's script directory.
3. Ensure the relative import paths match the actual file structure.
4. Verify that your behavior pack manifest declares compatible Script API dependencies.
5. Import `ChestFormData` into your own script.
6. Create a menu and display it to a player.

Example import:

```js
import { ChestFormData } from './chest-ui.js';
```

**Important:** `chest-ui.js` is not a standalone file. It imports `constants.js` and `typeIds.js`, which must also be present and configured correctly.

## Basic usage

Create a menu, add buttons, and show it to a player:

```js
import { ChestFormData } from './chest-ui.js';

function openExampleMenu(player) {
    const menu = new ChestFormData('small');

    menu.title('Example Menu');

    menu.button(
        11,
        '§aExample Item',
        ['A custom chest UI button', 'Click to select'],
        'minecraft:diamond',
        1,
        99,
        false
    );

    return menu.show(player);
}
```

Call `openExampleMenu(player)` from an appropriate event or command handler.

The menu is displayed using Minecraft's `ActionFormData` API. The returned promise resolves with the form response.

## API reference

### `new ChestFormData(size = 'small')`

Creates a new chest-style menu.

```js
const menu = new ChestFormData('small');
```

The requested size is looked up in `CHEST_UI_SIZES`, imported from `constants.js`.

If the supplied size is not present in that map, the implementation falls back to a 27-slot menu.

Available size names depend on the `CHEST_UI_SIZES` map in your project. Check `constants.js` for the exact supported names and slot counts.

The instance exposes `slotCount`, which contains the number of button slots.

### `title(text)`

Sets additional title text for the menu.

```js
menu.title('§6Shop');
```

The constructor initializes the title data, and `title()` appends text or raw-text components to it. It returns the current instance, so calls can be chained.

Example:

```js
menu
    .title('§l§6MistyGrove')
    .title(' §8| §7Shop');
```

### `button(slot, itemName, itemDesc, texture, stackSize = 1, durability = 0, enchanted = false)`

Adds or replaces a button at the specified slot.

```js
menu.button(
    13,
    '§bDiamond',
    ['A valuable item', '§7Price: 100 coins'],
    'minecraft:diamond',
    16,
    99,
    true
);
```

#### Parameters

| Parameter | Type | Description |
|---|---|---|
| `slot` | `number` | Zero-based button slot index. |
| `itemName` | `string \| RawMessage` | Button name, optionally using Minecraft formatting codes. |
| `itemDesc` | `string[]` | Array of description lines. Raw-text objects are also supported. |
| `texture` | `string` | Minecraft item identifier or supported custom content key. |
| `stackSize` | `number` | Displayed stack count, clamped to 1–99. Defaults to `1`. |
| `durability` | `number` | Displayed durability value, clamped to 0–99. Defaults to `0`. |
| `enchanted` | `boolean` | Enables the enchantment glint when `true`. Defaults to `false`. |

#### Slot indexing

Slots start at `0`, not `1`.

For a standard nine-column layout:

```text
 0   1   2   3   4   5   6   7   8
 9  10  11  12  13  14  15  16  17
18  19  20  21  22  23  24  25  26
```

The method clamps out-of-range slot indices to the available range. Avoid relying on this behavior; provide valid indices for your selected menu size.

#### Item names

Names can be plain strings or raw-text objects.

```js
menu.button(
    10,
    '§aConfirm',
    ['Continue?'],
    'minecraft:emerald'
);
```

Minecraft formatting codes, such as `§a` and `§c`, can be used to style text.

#### Item descriptions

Descriptions are supplied as an array of strings or raw-text objects.

```js
menu.button(
    12,
    '§eMagic Book',
    [
        '§7A mysterious book',
        '§bRarity: Rare',
        '§6Cost: 250 coins'
    ],
    'minecraft:book'
);
```

Each string is added on a new line. Raw-text objects can be used for translated or dynamic text.

#### Stack size

```js
menu.button(
    14,
    '§fIron Ingots',
    ['A stack of iron'],
    'minecraft:iron_ingot',
    64
);
```

The displayed count is limited to 1–99. This controls the UI's displayed stack indicator; it does not give items to the player.

#### Durability

```js
menu.button(
    15,
    '§aDiamond Sword',
    ['A durable weapon'],
    'minecraft:diamond_sword',
    1,
    75
);
```

The durability value is clamped to 0–99 and encoded into the UI's item representation.

**Note:** This method does not convert a real item's current durability into a percentage for you. The caller supplies the value to display.

#### Enchantment glint

```js
menu.button(
    16,
    '§dEnchanted Diamond',
    ['Shiny!'],
    'minecraft:diamond',
    1,
    0,
    true
);
```

Setting `enchanted` to `true` adds the enchantment-glint flag to the generated item representation.

### `pattern(pattern, key)`

Creates multiple buttons using a character-based grid.

This method is useful for shops, settings menus, admin panels, and other layouts that need several buttons arranged consistently.

```js
const menu = new ChestFormData('large');

menu.pattern(
    [
        'abcdefghi',
        'j.......k',
        'l.......m',
        'nopqrstu'
    ],
    {
        a: {
            itemName: '§aGreen Button',
            itemDesc: ['Example button'],
            texture: 'minecraft:lime_wool'
        },
        e: {
            itemName: '§bDiamond',
            itemDesc: ['A diamond'],
            texture: 'minecraft:diamond',
            stackAmount: 16,
            durability: 99,
            enchanted: true
        },
        k: {
            itemName: '§cClose',
            itemDesc: ['Close menu'],
            texture: 'minecraft:barrier'
        }
    }
);
```

#### Parameters

| Parameter | Type | Description |
|---|---|---|
| `pattern` | `string[]` | An array of strings representing menu rows. |
| `key` | `object` | An object mapping pattern characters to button definitions. |

#### Pattern characters

Each character represents a column in its row.

- `a` through `z`, or any other character, can be used as keys.
- Characters without a matching entry in `key` are skipped.
- Each row is processed from left to right.
- The first row starts at slot `0`.
- The next row starts at slot `9`.
- Each character maps to `slot = column + row * 9`.

For example:

```text
abcdefghi
j.......k
l.......m
```

The letters are potential button positions. The periods are empty positions unless you explicitly define `.` in the key object.

#### Pattern button properties

Each key entry can contain:

| Property | Description |
|---|---|
| `itemName` | Button name as a string or raw-text object. |
| `itemDesc` | Array of description lines. |
| `texture` | Item identifier or supported custom content key. |
| `stackAmount` | Displayed stack count, clamped to 1–99. |
| `durability` | Displayed durability value, clamped to 0–99. |
| `enchanted` | Enables the enchantment glint. |

Example:

```js
const menu = new ChestFormData('small');

menu.pattern(
    [
        '.........',
        '...a.....',
        '.........'
    ],
    {
        a: {
            itemName: '§aSettings',
            itemDesc: ['Open settings'],
            texture: 'minecraft:compass',
            stackAmount: 1,
            durability: 0,
            enchanted: false
        }
    }
);
```

**Important:** Pattern rows should not exceed the intended nine-column width, and the total pattern should fit within the selected menu's slot count. The current implementation clamps generated slots, so oversized patterns may overwrite the last available slot.

### `show(player)`

Displays the constructed menu to a player.

```js
const response = await menu.show(player);
```

The method returns the promise from `ActionFormData.show()`.

When `inventory_enabled` is enabled in `constants.js`, the player's inventory is also appended to the form as additional buttons. The inventory display includes item names, lore, stack counts, and calculated durability indicators where supported.

This inventory display is informational. It does not automatically implement item transfer, buying, selling, or button-click actions.

Example:

```js
menu.show(player).then(response => {
    if (response.canceled) return;

    const selectedSlot = response.selection;

    // Handle the selected button here.
});
```

If inventory buttons are enabled, account for their additional positions when interpreting the selection.

## Custom content and textures

The implementation resolves texture identifiers through `custom_content`, `custom_content_keys`, `typeIdToDataId`, and `typeIdToID`.

These values are provided by the supporting project files.

- Standard Minecraft item identifiers can be supplied as textures.
- Custom content keys are resolved through the configured custom-content mapping.
- Unsupported identifiers may not render as expected.

Verify that any custom textures or item mappings are correctly registered in your environment.

## Method chaining

`title()`, `button()`, and `pattern()` return the current `ChestFormData` instance, allowing chained menu construction.

```js
const menu = new ChestFormData('small')
    .title('§6Example')
    .button(
        13,
        '§aPlay',
        ['Start playing'],
        'minecraft:emerald'
    )
    .button(
        15,
        '§cExit',
        ['Close this menu'],
        'minecraft:barrier'
    );

menu.show(player);
```

## Limitations

- This is a chest-style UI rendered through `ActionFormData`, not a native Minecraft chest container.
- Button selections require your own response-handling code.
- The displayed stack count and durability values are UI metadata, not changes to real inventory items.
- Inventory display depends on `inventory_enabled`.
- Texture rendering depends on the supporting mappings and Minecraft version.
- Menu size names are defined by `CHEST_UI_SIZES`.
- This project requires its supporting source files to work correctly.

## Credits and license

This project is a modification of an existing chest UI implementation.

- **Modified by:** Mogley
- **Original creator:** To be credited
- **Original project:** To be linked
- **Original license:** To be confirmed

Please see `CREDITS.md` for attribution and licensing information.

Do not redistribute the original project's files or apply a new license to them unless the original license or explicit permission allows it.

## Contributing

Bug reports, suggestions, and improvements are welcome, subject to the original project's license and contribution requirements.

When reporting an issue, include your Minecraft version, Script API versions, reproduction steps, and relevant error messages.

## Disclaimer

This is an unofficial community project and is not affiliated with or endorsed by Mojang or Microsoft.

Minecraft is a trademark of Microsoft or its affiliates.

## Credits 

[Mogley-co](https://github.com/mogley-co)
