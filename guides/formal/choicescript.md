---
title: A Guide to the ChoiceScript Language
description: A structured reference to ChoiceScript projects, syntax, expressions, choices, flow control, and the complete command set.
order: 2
updated: 2026-07-27
wip: false
---

# A Guide to the ChoiceScript Language

ChoiceScript is a line-oriented, indentation-sensitive scripting language for writing multiple-choice interactive fiction. A ChoiceScript game is made from plain-text scene files. Each non-blank line is either prose shown to the player, a command beginning with `*`, or an option beginning with `#` inside a choice block.

This guide presents the language as a practical reference. It covers project structure, syntax, variables, expressions, choices, scene navigation, saving, achievements, platform integrations, multimedia, and the complete recognized command set.

tl;dr: Write prose as normal text, write commands with `*command`, indent every block consistently, declare permanent variables in `startup.txt`, and move between scenes with `*goto_scene` or `*finish`.

:::important Closed command set
ChoiceScript recognizes a fixed set of 86 top-level commands. An unrecognized `*word` produces a `Non-existent command` error. A small number of additional modifiers are valid only inside `*choice` and `*fake_choice` blocks.
:::

## How a ChoiceScript project is organized

A game is a directory containing `.txt` scene files. The engine begins with `startup.txt`, which normally declares the game metadata, permanent variables, and ordered scene list.

| File | Purpose |
|---|---|
| `startup.txt` | First scene loaded; declares initial configuration and permanent stats |
| `choicescript_stats.txt` | Optional stats screen, commonly built with `*stat_chart` |
| `choicescript_screenshots.txt` | Optional screenshot-tooling scene with special restrictions |
| Other `.txt` files | Normal story scenes |
| `mygame.js` | Web-build configuration outside the ChoiceScript language |

### The top of `startup.txt`

The following commands are initial commands and must appear at the beginning of `startup.txt`, before ordinary prose or non-initial commands:

- `*create`
- `*create_array`
- `*scene_list`
- `*title`
- `*author`
- `*achievement`
- `*product`
- `*ifid`

A minimal startup file can look like this:

```choicescript
*title The Glass Road
*author Example Author
*create courage 50
*create player_name ""
*scene_list
  chapter_one
  chapter_two
  finale

*goto_scene chapter_one
```

### Scene navigation

Use `*goto_scene scene_name` to move permanently to another scene. Use `*finish` to advance to the next scene in `*scene_list`.

```choicescript
The chapter comes to an end.

*finish Continue
```

When `*finish` runs in the final listed scene, the engine opens the ending flow.

## Syntax basics

Every non-blank line has one of four roles:

1. **Prose:** displayed to the player.
2. **Command:** begins with `*` and a recognized keyword.
3. **Choice option:** begins with `#` inside a choice body.
4. **Blank line:** ends the current paragraph.

Command names are case-insensitive, although lowercase is conventional.

```choicescript
You enter the station.

*set courage +10

*choice
  #Board the train.
    *goto aboard
  #Walk away.
    *goto departure
```

## Indentation

Indentation is mandatory and structural. It tells the engine which lines belong to a choice option, conditional branch, subroutine, or other block.

You may use spaces or tabs, but do not mix them in the same file.

```choicescript
*if courage > 50
  You step forward without hesitation.
  *goto brave_path
*else
  You wait for someone else to move.
  *goto cautious_path
```

Common indentation failures include:

- mixing spaces and tabs;
- using a different indentation width for sibling lines;
- increasing indentation where no new block has started; and
- allowing a choice option or conditional branch to end without transferring control.

:::warning Explicit control flow
By default, every `*choice` option and conditional branch must leave its block using a command such as `*goto`, `*goto_scene`, `*finish`, or `*return`.

Setting the permanent variable `implicit_control_flow` to `true` allows execution to continue after a block without an explicit jump.
:::

## Prose, paragraphs, and markup

Consecutive prose lines are joined into one paragraph. A blank line begins a new paragraph.

```choicescript
This line and the next line
appear in the same paragraph.

This begins a new paragraph.
```

Use `*line_break` to force a single line break without beginning a new paragraph.

### Inline markup

ChoiceScript recognizes a small set of bracket-style tags:

| Markup | Effect |
|---|---|
| `[b]text[/b]` | Bold |
| `[i]text[/i]` | Italic |
| `[url=https://example.com]text[/url]` | Hyperlink |
| `[n/]` | Forced line break |
| `[c/]` | Consumed no-op marker |
| `•` at line start | List item |

## Comments

Use `*comment` for author notes that should not appear to the player.

```choicescript
*comment Revisit this branch after playtesting.
```

## Variables

ChoiceScript has permanent and temporary variables.

### Permanent variables

Declare permanent variables with `*create` at the top of `startup.txt`.

```choicescript
*create courage 50
*create name ""
*create knows_secret false
```

Permanent variables persist across scenes and save data.

### Temporary variables

Declare scene-local variables with `*temp`.

```choicescript
*temp selected_door
*temp roll 0
```

Temporary variables are cleared when the game moves to another scene with `*goto_scene`.

### Variable names

Variable names:

- must begin with a letter;
- may contain word characters;
- may not be `and`, `or`, `true`, `false`, `scene`, or `scenename`; and
- may not begin with the reserved prefix `choice_`.

### Arrays

ChoiceScript arrays are numbered variables with a generated count variable.

```choicescript
*create_array inventory 3 ""
```

This creates:

- `inventory_1`
- `inventory_2`
- `inventory_3`
- `inventory_count`

The expression language also supports bracket notation:

```choicescript
${inventory[2]}
```

Use `*temp_array` for a scene-local array.

## Setting and deleting values

Use `*set` to assign a value:

```choicescript
*set courage 70
*set name "Mara"
*set knows_secret true
```

When the expression begins with an operator, the current variable value is used as the left operand:

```choicescript
*set courage +10
*set courage %-20
```

Use `*setref` when the name of the target variable is computed at runtime.

Use `*delete` and `*delete_array` to remove variables or arrays from their current scope.

## Expressions

Expressions are used in `*set`, `*if`, variable substitution, command arguments, and other contexts.

### Operators

| Operator | Meaning |
|---|---|
| `+` | Numeric addition |
| `-` | Numeric subtraction |
| `*` | Numeric multiplication |
| `/` | Numeric division |
| `^` | Exponentiation |
| `&` | String concatenation |
| `#` | Character at a 1-based position |
| `%+` | Fairmath addition |
| `%-` | Fairmath subtraction |
| `=` | Equality |
| `!=` | Inequality |
| `<`, `>`, `<=`, `>=` | Numeric comparison |
| `and` | Logical AND |
| `or` | Logical OR |
| `modulo` | Modulo |

ChoiceScript requires explicit parentheses when combining multiple operators.

```choicescript
*set result (score + 2) / 4

*if (courage > 50) and (health > 0)
  You continue.
  *goto onward
```

### Fairmath

Fairmath is useful for percentage-like stats. Gains become smaller as the value approaches 100, while losses scale with the current value.

```choicescript
*set trust %+20
*set trust %-20
```

A bare `%` is not an operator.

### Built-in functions

| Function | Effect |
|---|---|
| `not(expr)` | Boolean negation |
| `round(expr)` | Rounds to the nearest integer |
| `timestamp(dateString)` | Converts a date string to a Unix timestamp |
| `log(expr)` | Base-10 logarithm |
| `length(expr)` | Length of a value treated as a string |
| `auto(percentage, id)` | Auto-balancing threshold helper |

## Variable substitution

Use `${expression}` to insert a value into prose.

```choicescript
Your name is ${name}.
```

Capitalization variants are also available:

```choicescript
$!{name}
$!!{name}
```

Use multireplace when an expression selects one of several text fragments:

```choicescript
@{rank novice|veteran|legend}
```

The expression must resolve to a 1-based whole number. Booleans are treated as `1` for true and `2` for false.

## Choices

Use `*choice` to present branching options.

```choicescript
*choice
  #Open the door.
    *set courage +10
    *goto inside
  #Leave it closed.
    *goto hallway
```

Each option body must normally end by transferring control.

### Fake choices

Use `*fake_choice` when the options change state or display different prose but all return to the same flow.

```choicescript
*fake_choice
  #Take tea.
    *set drink "tea"
  #Take coffee.
    *set drink "coffee"

You chose ${drink}.
```

### Conditional options

Hide an option with `*if`:

```choicescript
*choice
  *if has_key
    #Unlock the gate.
      *goto courtyard
  #Turn back.
    *goto road
```

Show an unavailable option in a disabled state with `*selectable_if`:

```choicescript
*choice
  *selectable_if (coins >= 10) #Pay ten coins.
    *set coins -10
    *goto ferry
  #Stay ashore.
    *goto harbor
```

### Reuse behavior

- `*hide_reuse` hides a previously selected option.
- `*disable_reuse` leaves it visible but unavailable.
- `*allow_reuse` restores normal reuse.

These may set the behavior from that point in the scene or apply to an individual option when placed directly before it.

## Conditional flow

Use `*if`, `*elseif` or `*elsif`, and `*else`.

```choicescript
*if reputation >= 80
  The guards salute.
  *goto welcomed
*elseif reputation >= 40
  The guards inspect your papers.
  *goto inspected
*else
  The guards bar the gate.
  *goto rejected
```

## Labels and jumps

Use `*label` to create a scene-local destination and `*goto` to jump to it.

```choicescript
*label crossroads

*choice
  #Take the north road.
    *goto north
  #Take the south road.
    *goto south
```

Use `*gotoref` when a runtime expression produces the label name.

`*looplimit` changes how many times a jump may revisit one line before the engine reports a likely infinite loop. The default is 1000.

## Subroutines

Use `*gosub` to call a label within the current scene.

```choicescript
*gosub update_reputation 5
The crowd settles.
*goto continue_story

*label update_reputation
*params amount
*set reputation +amount
*return
```

Use `*gosub_scene` to call a subroutine in another scene. The engine preserves the calling scene and resumes after `*return`.

Arguments are also available as `param_1`, `param_2`, and so on, with `param_count` storing the number of arguments.

## Randomness

Use `*rand` to assign a random value.

```choicescript
*temp roll
*rand roll 1 6
```

When both bounds are whole numbers, the result is a whole number in the inclusive range. Decimal bounds produce a decimal result.

Use `*goto_random_scene` with an indented list of scene names to choose a scene randomly. By default, each scene is selected at most once per playthrough unless reuse is allowed.

## Input

Use `*input_text` to collect text:

```choicescript
*input_text name
```

Optional modifiers are `long` and `allow_blank`.

Use `*input_number` to collect a number within a defined range:

```choicescript
*input_number age 18 120
```

## Output and formatting commands

| Command | Purpose |
|---|---|
| `*print` | Prints an expression |
| `*page_break` | Pauses and continues on a fresh screen |
| `*line_break` | Inserts a line break |
| `*stat_chart` | Displays text, percentage, or opposed-pair stats |
| `*script` | Executes raw JavaScript |

### Stat charts

A stats screen commonly uses `*stat_chart`.

```choicescript
*stat_chart
  percent courage Courage
  percent empathy Empathy
  text name Name
  opposed_pair order
    Order
    Chaos
```

## Game and scene structure

| Command | Purpose |
|---|---|
| `*scene_list` | Declares the ordered scene list |
| `*title` | Sets the game title |
| `*author` | Sets the author |
| `*product` | Declares a purchasable product |
| `*ifid` | Declares the work's UUID-formatted IFID |
| `*bug` | Throws a deliberate hard error |

## Ending and restarting

| Command | Purpose |
|---|---|
| `*finish` | Ends the scene and advances |
| `*abort` | Stops immediately without a continuation |
| `*ending` | Opens the end-of-game menu |
| `*end_trial` | Ends a trial or demo |
| `*restart` | Restarts from the beginning |
| `*reset` | Restores permanent stats to their initial values |
| `*redirect_scene` | Redirects the main game from a secondary screen |

## Saving and checkpoints

| Command | Purpose |
|---|---|
| `*save_game` | Opens the platform save flow |
| `*restore_game` | Opens the restore flow |
| `*show_password` | Displays an encoded state password |
| `*save_checkpoint` | Saves an autosave checkpoint |
| `*restore_checkpoint` | Restores a checkpoint |

## Achievements

Declare achievements at the top of `startup.txt` with `*achievement`, then grant them with `*achieve`.

```choicescript
*achievement first_step visible 10 First Step
  Begin the journey.
  You began the journey.
```

Use `*check_achievements` to refresh the generated `choice_achieved_<name>` variables.

Achievement declarations have strict rules for names, points, titles, descriptions, hidden achievements, duplicates, and total counts.

## Purchases and store integration

| Command | Purpose |
|---|---|
| `*check_purchase` | Checks whether products were purchased |
| `*purchase` | Presents a product purchase |
| `*purchase_discount` | Presents a time-limited discounted purchase |
| `*print_discount` | Prints discount timing text |
| `*restore_purchases` | Restores platform purchase records |

Products must first be declared using `*product` at the top of `startup.txt`.

## Platform, advertisements, and timers

The engine includes commands for platform-specific flows:

- `*advertisement`
- `*page_break_advertisement`
- `*finish_advertisement`
- `*delay_break`
- `*delay_ending`
- `*timer`
- `*login`
- `*check_registration`
- `*subscribe`
- `*share_this_game`
- `*more_games`
- `*feedback`

These commands depend on the build and platform. Several are skipped or behave differently on Steam, prerelease builds, or unsupported platforms.

## Multimedia and links

| Command | Purpose |
|---|---|
| `*image` | Displays an image |
| `*text_image` | Displays a light/dark-adaptive text image |
| `*kindle_image` | Displays a Kindle-only image |
| `*kindle_search` | Opens a Kindle storefront search |
| `*kindle_product` | Alias of `*kindle_search` |
| `*sound` | Plays a sound |
| `*youtube` | Embeds a YouTube video on the web |
| `*link` | Inserts an inline link |
| `*link_button` | Displays a link as a button |

## Analytics and configuration

Use `*track_event` to submit an analytics event with optional parameters.

Use `*config` like `*set` when a value may also receive a remote configuration override.

`*ai` is recognized by the engine but currently performs no action.

## Choice-only modifiers

The following are valid only within the option parser for `*choice` or `*fake_choice`:

- `*selectable_if`
- `*random_weight`
- option-scoped `*if`
- option-group `*if`, `*elseif`, `*elsif`, and `*else`
- option-scoped `*hide_reuse`, `*disable_reuse`, and `*allow_reuse`

`*random_weight` is parsed but currently has no runtime effect.

## Built-in read-only variables

The engine exposes reserved variables beginning with `choice_`. Authors may read these variables but may not create or directly set their own variables using the prefix.

Common examples include:

| Variable | Meaning |
|---|---|
| `choice_scene` | Current scene name |
| `choice_linenum` | Current 1-based line number |
| `choice_title` | Current game title |
| `choice_time_stamp` | Current Unix timestamp |
| `choice_registered` | Whether the player is signed in |
| `choice_nightmode` | Whether night mode is active |
| `choice_is_web` | Whether the game is running as a web build |
| `choice_is_steam` | Whether the game is running on Steam |
| `choice_is_trial` | Whether the game is a trial |
| `choice_randomtest` | Whether Randomtest is running |
| `choice_quicktest` | Whether Quicktest is running |
| `choice_saved_checkpoint[_slot]` | Whether a checkpoint exists |
| `choice_purchased_<product>` | Product state set by `*check_purchase` |
| `choice_achieved_<name>` | Achievement state set by `*check_achievements` |

## Common errors

| Error | Usual cause |
|---|---|
| `Non-existent command` | Unknown command or choice-only modifier used outside a choice |
| `Non-existent variable` | Variable was never declared |
| `Illegal mixing of spaces and tabs` | Both indentation styles appear in one file |
| `invalid indent` | Block indentation does not align |
| `increasing indent not allowed` | Indentation increased outside a valid block |
| `It is illegal to fall out of a *choice statement` | Option or branch lacks explicit control flow |
| `Neither true nor false` | Non-boolean value used in a boolean context |
| `Not a number` | Numeric operation received a non-numeric value |
| `visited this line too many times` | A jump exceeded the loop limit |
| `Invalid IFID` | IFID is not a valid UUID |

:::tip Test early
Indentation and control-flow mistakes are easiest to fix while a scene is still small. Run the engine's tests regularly rather than waiting until an entire chapter is complete.
:::

## Complete command index

`abort` · `achieve` · `achievement` · `advertisement` · `ai` · `allow_reuse` · `author` · `bug` · `check_achievements` · `check_purchase` · `check_registration` · `choice` · `comment` · `config` · `create` · `create_array` · `delay_break` · `delay_ending` · `delete` · `delete_array` · `disable_reuse` · `else` · `elseif` · `elsif` · `end_trial` · `ending` · `fake_choice` · `feedback` · `finish` · `finish_advertisement` · `gosub` · `gosub_scene` · `goto` · `goto_random_scene` · `goto_scene` · `gotoref` · `hide_reuse` · `if` · `ifid` · `image` · `input_number` · `input_text` · `kindle_image` · `kindle_product` · `kindle_search` · `label` · `line_break` · `link` · `link_button` · `login` · `looplimit` · `more_games` · `page_break` · `page_break_advertisement` · `params` · `print` · `print_discount` · `product` · `purchase` · `purchase_discount` · `rand` · `redirect_scene` · `reset` · `restart` · `restore_checkpoint` · `restore_game` · `restore_purchases` · `return` · `save_checkpoint` · `save_game` · `scene_list` · `script` · `set` · `setref` · `share_this_game` · `show_password` · `sound` · `stat_chart` · `subscribe` · `temp` · `temp_array` · `text_image` · `timer` · `title` · `track_event` · `youtube`

PS. This guide is based on the supplied engine-level ChoiceScript language reference. It reorganizes the material for the field-guides renderer without expanding beyond what that source supports.

:::tip
Made with care by [kmab](https://kmab5.github.io).
:::
