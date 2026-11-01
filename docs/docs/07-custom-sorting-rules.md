# Custom Sorting Rules

Custom sorting rules let you put selected items first or last, group items together, or leave matching stacks in their current slots.
They work alongside your selected [sort type](/usage-guide#available-sort-types).

Rules are player preferences and apply to the supported inventories you sort, not just one container.
No rules are configured by default.

## Adding Rules

**With the client mod**, open the [configuration menu](/usage-guide#config-menu) and find **Sort Priority Rules**.
Enter a match expression, choose a position, click **Add**, and save the configuration.
You can edit existing rules, move them up or down, or delete them.

**On a server with Inventory Sorter installed**, you can use `/invsort priority` commands in chat, including with a vanilla client.
For example:

```text
/invsort priority add first minecraft:diamond
/invsort priority add last minecraft:cobblestone
/invsort priority list
```

The next time you sort, diamonds go before normally sorted items and cobblestone goes after them.
Adding a rule does not trigger a sort by itself.

:::note
**With the client mod**, rules are saved in your client configuration and synchronized to a supporting server.
Changes made through priority commands are also saved to your client configuration.

**With a vanilla client**, rules are saved per player on that server and kept between sessions.
:::

## Positions and Rule Order

Each rule has a match expression and one of four positions:

| Position | Effect |
|----------|--------|
| `first` | Places matching stacks before the default group. |
| `default` | Places matching stacks in the default group, ahead of items without a matching rule. |
| `last` | Places matching stacks after the default group. |
| `ignore` | Leaves matching stacks in their current slots, without merging or reordering them. |

For stacks that are not ignored, the **first matching rule in the list** chooses their position.
Within each position, items assigned to an earlier rule come before items assigned to a later rule.
Items assigned to the same rule use the normal sorting behavior, including your selected sort type.

For example, if the first rule puts diamonds `last` and a later rule puts diamonds `first`, diamonds still go last.
Move the later rule up if you want it to take precedence.

:::info
Any matching `ignore` rule takes precedence over ordering rules, regardless of where it appears in the list.
The remaining items sort around the ignored slots.
:::

When [bundle sorting](/usage-guide#bundle-sorting) is enabled, ignored loose stacks are not inserted into bundles.
An ignored bundle stays in its slot but can still receive matching loose items.
Other priority positions control the order of stacks left in the inventory; they do not prevent bundle insertion.

## Match Syntax

Use one of these forms to select items:

| Form | Example | Matches |
|------|---------|---------|
| Item ID | `minecraft:diamond` | That exact item. |
| Item tag | `#minecraft:shulker_boxes` | Items belonging to that item tag. |
| Data component | `@minecraft:bundle_contents` | Stacks with that component, regardless of its value. |
| Display name | `name:"Meza's *"` | Stacks whose display name matches the quoted pattern. |

If you leave out the namespace, `minecraft:` is assumed: `diamond` means `minecraft:diamond`.
Use the full namespace for items, tags, or components from other mods, such as `examplemod:ruby`.
Use exact, lowercase IDs. Tags depend on the tags available in your world or server.

### Display Names

Name patterns must be enclosed in double quotes and match the **whole display name**, ignoring letter case.
The `*` wildcard matches any amount of text, including no text.

- `name:"Diamond"` matches the display name Diamond.
- `name:"Meza's *"` matches names beginning with Meza's followed by a space.
- `name:"*Pickaxe*"` matches names containing Pickaxe.

Use a backslash to match a literal special character inside a name pattern: `\*` for an asterisk, `\"` for a double quote, and `\\` for a backslash.
For example, `name:"Lucky \*"` matches the literal name Lucky *.

:::warning
For server-side sorting, name rules use the server's language, which may differ from your client's language.
Use item IDs or tags when you need a rule that does not depend on translated display names.
:::

### Combining Matches

Combine expressions with these operators:

| Operator | Meaning | Example |
|----------|---------|---------|
| `!` | Not | `!minecraft:cobblestone` |
| `&` | Both expressions must match | `#minecraft:shulker_boxes & @minecraft:container` |
| `\|` | Either expression may match | `minecraft:diamond \| minecraft:emerald` |
| `(...)` | Group expressions | `(minecraft:diamond \| minecraft:emerald) & !name:"Meza's *"` |

`!` is evaluated first, then `&`, then `|`. Use parentheses to change that order.
Spaces around operators are optional.

In the final example above, the rule matches diamonds or emeralds, except those with a display name beginning with Meza's followed by a space.

## Command Reference

Run these commands as a player in chat. They change your own rules.
By default, all players can use them; servers can control access with the [priority permission](/permissions#player-permissions).

In the commands below, replace `[position]` with `first`, `default`, `last`, or `ignore`, and `[match]` with a match expression.
Replace `[index]` with the rule number shown by `list`, starting at **1**. Do not type the square brackets.

| Command | Effect |
|---------|--------|
| `/invsort priority list` | Shows your rules with their current numbers, positions, and match expressions. |
| `/invsort priority add [position] [match]` | Adds a rule to the end of the list. |
| `/invsort priority set [index] [position] [match]` | Replaces the position and expression of an existing rule. |
| `/invsort priority set [index] [position]` | Changes only the position, keeping the existing expression. |
| `/invsort priority move [index] up` | Moves a rule one place up. |
| `/invsort priority move [index] down` | Moves a rule one place down. |
| `/invsort priority remove [index]` | Deletes one rule. |
| `/invsort priority clear` | Deletes all your rules, restoring normal sorting without custom rules. |

The match expression is the rest of the command. Do not wrap the whole expression in extra quotes; only name patterns need quotes.

Commands report invalid expressions, unknown rule numbers, and attempts to move past either end of the list.
An unsuccessful update leaves your rules unchanged. Use `list` again after moving or removing rules, since their numbers can change.

### Examples

Put all shulker boxes first:

```text
/invsort priority add first #minecraft:shulker_boxes
```

Keep stacks whose names begin with Meza's followed by a space in their current slots:

```text
/invsort priority add ignore name:"Meza's *"
```

Put diamonds and emeralds first, except those matching that name pattern:

```text
/invsort priority add first (minecraft:diamond | minecraft:emerald) & !name:"Meza's *"
```

Put stacks with bundle contents last. This checks for the component, so it also matches empty bundles:

```text
/invsort priority add last @minecraft:bundle_contents
```

To edit an existing rule, first run `/invsort priority list` and use its current number.
For example, these commands change rule 1 to put emeralds last, then change its position to first without changing the expression:

```text
/invsort priority set 1 last minecraft:emerald
/invsort priority set 1 first
```

## Editing Rules in JSON

Rules can also be stored in the client config file using `sortPriorityRules`.
See the [configuration reference](/configuration#sortpriorityrules) for the JSON format and how to apply file edits.

If an expression is malformed, the config menu shows a validation error and commands reject it.
Malformed expressions loaded directly from the config file are skipped during sorting; the remaining valid rules still apply.
An item or tag ID that does not exist or has no matching items will not select any stacks.
An unknown data component is a validation error.
