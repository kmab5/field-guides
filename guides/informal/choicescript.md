---
title: A Guide to the ChoiceScript Language
description: A beginner-friendly walkthrough of writing interactive fiction with ChoiceScript, from your first scene to choices, stats, and reusable story logic.
order: 2
updated: 2026-07-27
wip: false
---

# A Guide to the ChoiceScript Language

So you want to make a Choice of Games-style interactive story.

Good news: ChoiceScript is mostly plain text. You write the story normally, put commands on lines beginning with `*`, and indent the parts that belong inside choices or conditions.

Bad news: ChoiceScript takes indentation personally.

tl;dr: Start with `startup.txt`, declare your permanent stats there, list your scenes, write choices with `*choice`, and never let a choice option fall out of its indented block without a `*goto`, `*finish`, or another clear exit.

## What ChoiceScript actually is

ChoiceScript is a line-based scripting language for interactive fiction. A game is a folder of `.txt` files. The engine reads one line at a time.

A line can be:

- story text;
- a command such as `*set courage +10`;
- an option beginning with `#`; or
- blank space separating paragraphs.

That is the basic mental model.

```choicescript
The corridor is dark.

*choice
  #Keep walking.
    *goto forward
  #Turn around.
    *goto retreat
```

:::important There are only 86 top-level commands
ChoiceScript does not let you invent commands. If you write something like `*teleport` and it is not part of the language, the engine throws `Non-existent command`.
:::

## Your project folder

The engine begins in `startup.txt`.

A normal project includes:

- `startup.txt`;
- one or more story scenes;
- optionally `choicescript_stats.txt`; and
- web configuration outside the language itself.

Here is a tiny `startup.txt`:

```choicescript
*title My First ChoiceScript Game
*author Your Name
*create courage 50
*create player_name ""
*scene_list
  chapter_one
  ending

*goto_scene chapter_one
```

The order under `*scene_list` matters because `*finish` moves to the next scene in that list.

:::warning Put initial commands first
Commands such as `*create`, `*scene_list`, `*title`, `*author`, `*achievement`, `*product`, and `*ifid` belong at the beginning of `startup.txt`. Do not begin with story text and try to declare them later.
:::

## Write some prose

Plain text is printed to the player.

```choicescript
Rain runs down the window.
The train does not slow down.
```

Those two lines become one paragraph. Add a blank line to start a new paragraph.

Use `*line_break` when you want a smaller break without starting a new paragraph.

You can also use:

- `[b]bold[/b]`;
- `[i]italics[/i]`;
- `[url=https://example.com]links[/url]`; and
- `[n/]` for an inline break.

## Indentation: the thing that breaks everything

ChoiceScript uses indentation instead of braces.

```choicescript
*if courage > 50
  You raise your hand.
  *goto volunteer
*else
  You avoid eye contact.
  *goto silence
```

The two indented lines belong to the `*if`. The next two belong to the `*else`.

Choose spaces or tabs and stick to that choice for the whole file.

Do not do this:

```choicescript
*choice
  #Go left.
     *goto left
  #Go right.
    *goto right
```

The first option body and second option body do not line up. ChoiceScript notices.

PS. A code editor that visibly marks tabs and spaces will save you a lot of frustration.

## Make your first choice

A real branching choice looks like this:

```choicescript
*choice
  #Open the letter.
    *set curiosity +10
    *goto opened_letter
  #Burn it.
    *set caution +10
    *goto burned_letter
```

Every option needs somewhere to go.

You can use:

- `*goto label` to move inside the same scene;
- `*goto_scene scene_name` to move to another file;
- `*finish` to move to the next scene;
- `*return` when leaving a subroutine; or
- another command that clearly ends or redirects the flow.

If you forget, you will probably see the famous “illegal to fall out of a choice statement” error.

### When every option comes back together

Use `*fake_choice` when the options should all continue after the menu.

```choicescript
*temp breakfast ""

*fake_choice
  #Toast.
    *set breakfast "toast"
  #Fruit.
    *set breakfast "fruit"

You chose ${breakfast}.
```

This is perfect for flavor choices, appearance choices, names, or small preference decisions.

### Hide or disable options

Use `*if` to hide an option completely:

```choicescript
*choice
  *if has_key
    #Unlock the door.
      *goto inside
  #Walk away.
    *goto street
```

Use `*selectable_if` to show the option but leave it unavailable:

```choicescript
*choice
  *selectable_if (coins >= 10) #Pay ten coins.
    *set coins -10
    *goto ferry
  #Stay here.
    *goto dock
```

That distinction matters. Hidden information and visible-but-locked information feel very different to a player.

## Stats and variables

You will use variables for almost everything:

- player names;
- relationship values;
- inventory;
- flags;
- personality stats;
- chapter progress; and
- decisions from earlier scenes.

### Permanent stats

Use `*create` in `startup.txt`:

```choicescript
*create courage 50
*create empathy 50
*create knows_secret false
*create player_name ""
```

These survive scene changes.

### Temporary variables

Use `*temp` inside a scene:

```choicescript
*temp roll
*temp chosen_door
```

These disappear when you move to another scene.

### Change a value

```choicescript
*set courage 70
*set knows_secret true
*set player_name "Ari"
```

You can also write:

```choicescript
*set courage +10
```

That means “take the current value of courage and add ten.”

## Put variables into the story

Use `${}`:

```choicescript
"${player_name}," the captain says, "we need an answer."
```

Use `$!{}` to capitalize the first letter:

```choicescript
$!{player_name} steps forward.
```

Use `$!!{}` for uppercase.

ChoiceScript also has multireplace:

```choicescript
You are known as a @{rank novice|veteran|legend}.
```

Here, `rank` must resolve to `1`, `2`, or `3`.

## Conditions

A basic conditional:

```choicescript
*if courage >= 60
  You do not hesitate.
  *goto brave
*else
  Your hand shakes.
  *goto afraid
```

Add middle branches with `*elseif` or `*elsif`.

```choicescript
*if reputation >= 80
  Everyone recognizes you.
  *goto famous
*elseif reputation >= 40
  A few people nod.
  *goto familiar
*else
  No one looks up.
  *goto unknown
```

When combining tests, use parentheses:

```choicescript
*if (courage > 50) and (health > 0)
  You keep fighting.
  *goto battle
```

ChoiceScript does not like ambiguous expressions. Parentheses are your friend.

## The expression rules you actually need first

Start with these:

| Operator | Meaning |
|---|---|
| `+` | Add numbers |
| `-` | Subtract numbers |
| `*` | Multiply |
| `/` | Divide |
| `&` | Join strings |
| `=` | Equal |
| `!=` | Not equal |
| `<`, `>`, `<=`, `>=` | Compare numbers |
| `and`, `or` | Combine conditions |
| `modulo` | Remainder |

You can also use:

- `round(expr)`;
- `not(expr)`;
- `length(expr)`;
- `log(expr)`; and
- `timestamp(dateString)`.

### Fairmath

ChoiceScript has special percentage math:

```choicescript
*set trust %+20
*set trust %-20
```

Fairmath makes stats harder to push toward the extremes. It is useful for opposed personality stats and relationships.

A bare `%` does not work.

## Labels and jumping around

Labels are destinations inside one scene.

```choicescript
*label town_square

The fountain is dry.

*choice
  #Visit the market.
    *goto market
  #Visit the inn.
    *goto inn
```

Labels:

- are local to the current scene;
- cannot contain spaces; and
- must be unique in that scene.

Use `*gotoref` when the label name comes from an expression.

## Moving between scene files

Use:

```choicescript
*goto_scene chapter_two
```

You can also target a label:

```choicescript
*goto_scene chapter_two secret_entrance
```

Use `*finish` when you simply want the next scene from `*scene_list`.

```choicescript
*finish Next Chapter
```

## Random numbers

Roll a die:

```choicescript
*temp roll
*rand roll 1 6
```

Then test the result:

```choicescript
*if roll >= 5
  You succeed.
  *goto success
*else
  You fail.
  *goto failure
```

ChoiceScript can also jump to a random scene with `*goto_random_scene`.

By default, selected scenes are not reused during the same playthrough unless you allow reuse.

## Ask the player for text or numbers

Text input:

```choicescript
What is your name?

*input_text player_name
```

Long text or blank text can be allowed with optional modifiers.

Number input:

```choicescript
How old are you?

*input_number age 18 120
```

The engine checks that the answer is inside the range.

## Arrays, without pretending they are normal arrays

ChoiceScript arrays are really numbered variables.

```choicescript
*create_array inventory 3 ""
```

That gives you:

- `inventory_1`;
- `inventory_2`;
- `inventory_3`; and
- `inventory_count`.

You can also use bracket notation:

```choicescript
${inventory[2]}
```

Use `*temp_array` for an array that only needs to exist in the current scene.

## Reusable logic with subroutines

Use `*gosub` when you keep repeating the same block of logic.

```choicescript
*gosub change_reputation 5
*goto continue_story

*label change_reputation
*params amount
*set reputation +amount
*return
```

Use `*gosub_scene` when the reusable logic lives in another scene file.

The arguments are available through the names in `*params`, and also through `param_1`, `param_2`, and `param_count`.

## Build a stats screen

Create `choicescript_stats.txt` and use `*stat_chart`.

```choicescript
*stat_chart
  text player_name Name
  percent courage Courage
  percent empathy Empathy
  opposed_pair order
    Order
    Chaos
```

Available chart rows are:

- `text`;
- `percent`; and
- `opposed_pair`.

## Reusing choice options

Sometimes a menu appears more than once.

- `*hide_reuse` removes a previously chosen option.
- `*disable_reuse` leaves it visible but grayed out.
- `*allow_reuse` lets the player choose it again.

You can set this behavior for later options in the scene or apply it to one option.

## Saving

ChoiceScript includes several save systems.

- `*save_game` opens a platform save flow.
- `*restore_game` restores a saved game.
- `*show_password` displays the classic encoded ChoiceScript save password.
- `*save_checkpoint` records a checkpoint.
- `*restore_checkpoint` loads it.

Named checkpoint slots use letters, numbers, and underscores.

## Achievements

Achievements are declared in `startup.txt`.

```choicescript
*achievement first_step visible 10 First Step
  Begin the journey.
  You began the journey.
```

Grant one later with:

```choicescript
*achieve first_step
```

There are strict limits on achievement names, titles, descriptions, points, duplicates, and totals. Hidden achievements have additional rules.

## Images, sound, video, and links

ChoiceScript can display or play media:

```choicescript
*image castle.jpg center A castle at dusk
*sound bell.mp3
*youtube video_slug
*link https://example.com Read more
*link_button https://example.com Open website
```

There are also:

- `*text_image` for assets that should adapt to light and dark mode;
- `*kindle_image`;
- `*kindle_search`; and
- `*kindle_product`.

Some commands only do anything on specific platforms.

## Store and platform commands

ChoiceScript includes engine commands for commercial releases:

- purchases;
- discounts;
- restoring purchases;
- advertisements;
- timed delays;
- login and registration;
- subscriptions;
- social sharing;
- feedback prompts; and
- platform checks.

You will see commands such as:

- `*check_purchase`;
- `*purchase`;
- `*purchase_discount`;
- `*advertisement`;
- `*delay_break`;
- `*login`;
- `*subscribe`; and
- `*share_this_game`.

:::note You may not need these immediately
For a first local project, focus on prose, variables, choices, conditions, labels, scenes, and testing. Store and platform commands matter later, when you are preparing an actual release.
:::

## Useful built-in variables

ChoiceScript reserves names beginning with `choice_`.

You can read built-in variables such as:

- `choice_scene`;
- `choice_linenum`;
- `choice_title`;
- `choice_nightmode`;
- `choice_is_web`;
- `choice_is_steam`;
- `choice_is_trial`;
- `choice_randomtest`; and
- `choice_quicktest`.

Do not try to create your own variables beginning with `choice_`.

## The errors you will meet

### `Non-existent command`

You misspelled a command, invented one, or used a choice-only modifier in the wrong place.

### `Non-existent variable`

You used a variable before declaring it with `*create` or `*temp`.

### `Illegal mixing of spaces and tabs`

Your file contains both indentation styles.

### `invalid indent`

Sibling lines do not line up, or a block has the wrong indentation.

### `increasing indent not allowed`

You indented even though the previous line did not open a block.

### `It is illegal to fall out of a *choice statement`

An option reached the end of its block without a jump, finish, return, or other explicit exit.

### `Neither true nor false`

A condition received something that was not a valid boolean.

### `Not a number`

A numeric expression received text or another non-numeric value.

### `visited this line too many times`

A loop kept jumping to the same line beyond the current `*looplimit`.

## A sensible learning order

Do not try to memorize every command.

Learn these first:

1. prose and paragraphs;
2. indentation;
3. `*create`, `*temp`, and `*set`;
4. `${variable}`;
5. `*choice` and `*fake_choice`;
6. `*if`, `*elseif`, and `*else`;
7. `*label` and `*goto`;
8. `*goto_scene`, `*scene_list`, and `*finish`;
9. `*input_text`, `*input_number`, and `*rand`; and
10. `*stat_chart`.

Then move into subroutines, arrays, achievements, saves, platform commands, and commercial integrations.

## A tiny complete example

`startup.txt`:

```choicescript
*title The Last Train
*author Example Author
*create courage 50
*create player_name ""
*scene_list
  station
  ending

*goto_scene station
```

`station.txt`:

```choicescript
The station clock reads 11:58.

A train waits at the platform.

What is your name?

*input_text player_name

*choice
  #Board the train.
    *set courage %+20
    *goto_scene ending boarded
  #Walk away.
    *set courage %-20
    *goto_scene ending left
```

`ending.txt`:

```choicescript
*label boarded
$!{player_name} boards just before the doors close.

*ending

*label left
$!{player_name} turns away from the platform.

*ending
```

:::tip Test constantly
The language is simple, but indentation and flow errors can become difficult to trace in a large scene. Test after every meaningful branch.
:::

## Every top-level command

Here is the full command set recognized by the engine:

`abort` · `achieve` · `achievement` · `advertisement` · `ai` · `allow_reuse` · `author` · `bug` · `check_achievements` · `check_purchase` · `check_registration` · `choice` · `comment` · `config` · `create` · `create_array` · `delay_break` · `delay_ending` · `delete` · `delete_array` · `disable_reuse` · `else` · `elseif` · `elsif` · `end_trial` · `ending` · `fake_choice` · `feedback` · `finish` · `finish_advertisement` · `gosub` · `gosub_scene` · `goto` · `goto_random_scene` · `goto_scene` · `gotoref` · `hide_reuse` · `if` · `ifid` · `image` · `input_number` · `input_text` · `kindle_image` · `kindle_product` · `kindle_search` · `label` · `line_break` · `link` · `link_button` · `login` · `looplimit` · `more_games` · `page_break` · `page_break_advertisement` · `params` · `print` · `print_discount` · `product` · `purchase` · `purchase_discount` · `rand` · `redirect_scene` · `reset` · `restart` · `restore_checkpoint` · `restore_game` · `restore_purchases` · `return` · `save_checkpoint` · `save_game` · `scene_list` · `script` · `set` · `setref` · `share_this_game` · `show_password` · `sound` · `stat_chart` · `subscribe` · `temp` · `temp_array` · `text_image` · `timer` · `title` · `track_event` · `youtube`

Choice-body-only modifiers include `*selectable_if`, `*random_weight`, and the choice-scoped forms of condition and reuse commands.

PS. The source reference goes much deeper into the exact argument grammar and engine behavior of every command. This version is meant to get you writing without losing the underlying technical accuracy.

:::tip 💖
Made with love by [kmab](https://kmab5.github.io).
:::
