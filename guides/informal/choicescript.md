---
title: a guide to the choicescript language
description: how to actually write a choice of games style interactive story without the engine screaming at u about indentation.
order: 2
updated: 2026-07-27
wip: false
---

# a guide to the choicescript language

so u wanna make one of those choice of games style interactive stories. good choice honestly, theyre super fun to write.

good news: choicescript is mostly just plain text. u write ur story like normal, put commands on lines that start with `*`, and indent the bits that live inside choices or conditions.

bad news: choicescript takes indentation *personally*. like, one space out of place and it will refuse to speak to u. u have been warned.

tl;dr: start with `startup.txt`, declare ur permanent stats there, list ur scenes, write choices with `*choice`, and never ever let a choice option just fall off the end of its block without a `*goto`, `*finish`, or some other clear exit. thats like 80% of the errors ull ever hit.

## what choicescript actually is

its a line-based scripting language for interactive fiction. a game is just a folder of `.txt` files, and the engine reads it one line at a time, top to bottom.

a line can be:

- story text;
- a command like `*set courage +10`;
- an option that starts with `#`; or
- blank space to separate paragraphs.

thats the whole mental model. keep that in ur head and half of this makes sense already.

```choicescript
The corridor is dark.

*choice
  #Keep walking.
    *goto forward
  #Turn around.
    *goto retreat
```

:::important there are only 86 top-level commands
u dont get to invent commands. if u write something like `*teleport` and its not a real part of the language, the engine just throws `Non-existent command` at u and gives up. the full list is at the very bottom if u ever wanna check whether something exists.
:::

## ur project folder

the engine always starts in `startup.txt`. thats home base, dont rename it.

a normal project has:

- `startup.txt`;
- one or more story scenes;
- optionally `choicescript_stats.txt`; and
- some web config stuff that lives outside the language itself.

heres a tiny `startup.txt` to look at:

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

the order under `*scene_list` actually matters, cuz `*finish` sends the player to the *next* scene in that list. so the list is basically ur table of contents.

:::warning put ur setup commands first
stuff like `*create`, `*scene_list`, `*title`, `*author`, `*achievement`, `*product`, and `*ifid` all need to go at the very top of `startup.txt`. dont start with a paragraph of story and then try to sneak them in later, the engine hates that.
:::

## writing some actual prose

plain text just gets printed to the player. no ceremony.

```choicescript
Rain runs down the window.
The train does not slow down.
```

those two lines become *one* paragraph. wanna start a new paragraph? leave a blank line. thats it.

if u want a smaller gap without a full new paragraph, use `*line_break`.

u also get some inline formatting:

- `[b]bold[/b]`;
- `[i]italics[/i]`;
- `[url=https://example.com]links[/url]`; and
- `[n/]` for an inline break.

## indentation: the thing that ruins everything

choicescript uses indentation instead of curly braces. so the shape of ur file *is* the logic.

```choicescript
*if courage > 50
  You raise your hand.
  *goto volunteer
*else
  You avoid eye contact.
  *goto silence
```

the two indented lines belong to the `*if`. the next two belong to the `*else`. simple enough.

pick spaces OR tabs and then stick with that one choice for the entire file. mixing them is a fast way to make the engine cry.

do NOT do this:

```choicescript
*choice
  #Go left.
     *goto left
  #Go right.
    *goto right
```

see how the first `*goto` and the second `*goto` dont line up? choicescript notices. choicescript always notices.

PS. get a code editor that visibly shows u tabs vs spaces. seriously. it will save u hours of pain.

## making ur first real choice

a proper branching choice looks like this:

```choicescript
*choice
  #Open the letter.
    *set curiosity +10
    *goto opened_letter
  #Burn it.
    *set caution +10
    *goto burned_letter
```

every single option needs somewhere to go afterwards. no exceptions.

ur exit options are:

- `*goto label` to jump somewhere in the same scene;
- `*goto_scene scene_name` to jump to another file;
- `*finish` to move on to the next scene;
- `*return` when ur leaving a subroutine; or
- anything else that clearly ends or redirects the flow.

forget one and u get the legendary "illegal to fall out of a choice statement" error. u WILL meet this error. we all have.

### when all the options come back together

use `*fake_choice` when every option should just continue on after the menu instead of branching off.

```choicescript
*temp breakfast ""

*fake_choice
  #Toast.
    *set breakfast "toast"
  #Fruit.
    *set breakfast "fruit"

You chose ${breakfast}.
```

this is perfect for flavor stuff, appearance, names, little preference choices, whatever. stuff that colors the story but doesnt actually split it.

### hiding or disabling options

use `*if` to hide an option completely, like it was never there:

```choicescript
*choice
  *if has_key
    #Unlock the door.
      *goto inside
  #Walk away.
    *goto street
```

use `*selectable_if` to still SHOW the option but leave it grayed out and unpickable:

```choicescript
*choice
  *selectable_if (coins >= 10) #Pay ten coins.
    *set coins -10
    *goto ferry
  #Stay here.
    *goto dock
```

that difference actually matters a lot. "u dont see the option at all" vs "u see it but cant afford it yet" feel completely different to a player. use it on purpose.

## stats and variables

ur gonna use variables for basically everything:

- player names;
- relationship values;
- inventory;
- flags;
- personality stats;
- chapter progress; and
- decisions from way earlier in the story.

### permanent stats

use `*create` in `startup.txt`:

```choicescript
*create courage 50
*create empathy 50
*create knows_secret false
*create player_name ""
```

these stick around through scene changes. they live for the whole game.

### temporary variables

use `*temp` inside a scene:

```choicescript
*temp roll
*temp chosen_door
```

these vanish the moment u move to another scene. good for throwaway stuff.

### changing a value

```choicescript
*set courage 70
*set knows_secret true
*set player_name "Ari"
```

u can also do:

```choicescript
*set courage +10
```

which just means "take whatever courage is right now and add ten." handy.

## putting variables into the story

use `${}`:

```choicescript
"${player_name}," the captain says, "we need an answer."
```

use `$!{}` to capitalize the first letter:

```choicescript
$!{player_name} steps forward.
```

and `$!!{}` for full UPPERCASE if ur feeling dramatic.

choicescript also has this thing called multireplace:

```choicescript
You are known as a @{rank novice|veteran|legend}.
```

here `rank` has to come out to `1`, `2`, or `3`, and it picks the matching word. super useful for not writing three near-identical sentences.

## conditions

the basic conditional:

```choicescript
*if courage >= 60
  You do not hesitate.
  *goto brave
*else
  Your hand shakes.
  *goto afraid
```

want middle branches? use `*elseif` or `*elsif` (both spellings work, pick one):

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

when ur combining tests, wrap them in parentheses:

```choicescript
*if (courage > 50) and (health > 0)
  You keep fighting.
  *goto battle
```

choicescript really doesnt like ambiguous expressions. parentheses are ur friend. use way more of them than u think u need.

## the expression rules u actually need first

dont learn all of these at once. start with these:

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

theres also a few functions:

- `round(expr)`;
- `not(expr)`;
- `length(expr)`;
- `log(expr)`; and
- `timestamp(dateString)`.

### fairmath

choicescript has this special percentage math:

```choicescript
*set trust %+20
*set trust %-20
```

fairmath makes it harder to shove a stat all the way to 0 or 100, cuz each change is a percentage of the room thats left. its great for opposed personality stats and relationship meters where u dont want things maxing out instantly.

heads up: a bare `%` on its own does nothing. u need the `+` or `-` with it.

## labels and jumping around

labels are just destinations inside one scene.

```choicescript
*label town_square

The fountain is dry.

*choice
  #Visit the market.
    *goto market
  #Visit the inn.
    *goto inn
```

labels:

- are local to the current scene (they dont reach across files);
- cant have spaces in them; and
- have to be unique within that scene.

if the label name itself comes from an expression, use `*gotoref` instead.

## moving between scene files

use:

```choicescript
*goto_scene chapter_two
```

u can also aim straight at a label inside that scene:

```choicescript
*goto_scene chapter_two secret_entrance
```

use `*finish` when u just want the next scene from `*scene_list` without naming it:

```choicescript
*finish Next Chapter
```

## random numbers

roll a die:

```choicescript
*temp roll
*rand roll 1 6
```

then test what u got:

```choicescript
*if roll >= 5
  You succeed.
  *goto success
*else
  You fail.
  *goto failure
```

choicescript can also send the player off to a random scene with `*goto_random_scene`.

by default it wont reuse a scene it already picked during the same playthrough, unless u specifically tell it reuse is allowed.

## asking the player for text or numbers

text input:

```choicescript
What is your name?

*input_text player_name
```

u can allow longer text or blank text with some optional modifiers.

number input:

```choicescript
How old are you?

*input_number age 18 120
```

the engine makes sure the answer lands inside the range u gave it, so no one is entering "420" for their age. probably.

## arrays, without pretending theyre normal arrays

choicescript arrays are really just numbered variables wearing a trenchcoat.

```choicescript
*create_array inventory 3 ""
```

that hands u:

- `inventory_1`;
- `inventory_2`;
- `inventory_3`; and
- `inventory_count`.

u can also use bracket notation, which is nicer to read:

```choicescript
${inventory[2]}
```

use `*temp_array` when u only need the array to exist in the current scene.

## reusable logic with subroutines

when u keep writing the same block of logic over and over, use `*gosub`:

```choicescript
*gosub change_reputation 5
*goto continue_story

*label change_reputation
*params amount
*set reputation +amount
*return
```

if the reusable logic lives in a different scene file, use `*gosub_scene` instead.

the arguments come through the names u put in `*params`, and also through `param_1`, `param_2`, `param_count` if u prefer.

## building a stats screen

make a `choicescript_stats.txt` and use `*stat_chart`:

```choicescript
*stat_chart
  text player_name Name
  percent courage Courage
  percent empathy Empathy
  opposed_pair order
    Order
    Chaos
```

the chart row types u can use are:

- `text`;
- `percent`; and
- `opposed_pair`.

## reusing choice options

sometimes the same menu shows up more than once, and u want to control what happens to already-picked options.

- `*hide_reuse` removes an option the player already chose.
- `*disable_reuse` leaves it visible but grayed out.
- `*allow_reuse` lets them pick it again freely.

u can set this for all the later options in a scene, or slap it on one single option.

## saving

choicescript comes with a few save systems:

- `*save_game` opens the platform save flow.
- `*restore_game` restores a saved game.
- `*show_password` shows the classic encoded choicescript save password (very retro).
- `*save_checkpoint` drops a checkpoint.
- `*restore_checkpoint` loads it back.

named checkpoint slots can use letters, numbers, and underscores.

## achievements

u declare achievements up in `startup.txt`:

```choicescript
*achievement first_step visible 10 First Step
  Begin the journey.
  You began the journey.
```

then grant one later with:

```choicescript
*achieve first_step
```

theres strict limits on achievement names, titles, descriptions, points, duplicates, and totals. hidden achievements have extra rules on top. so dont go too wild.

## images, sound, video, and links

choicescript can show or play media:

```choicescript
*image castle.jpg center A castle at dusk
*sound bell.mp3
*youtube video_slug
*link https://example.com Read more
*link_button https://example.com Open website
```

theres also:

- `*text_image` for stuff that should adapt to light and dark mode;
- `*kindle_image`;
- `*kindle_search`; and
- `*kindle_product`.

fair warning, some of these only actually do anything on specific platforms.

## store and platform commands

choicescript has a whole pile of engine commands for commercial releases:

- purchases;
- discounts;
- restoring purchases;
- ads;
- timed delays;
- login and registration;
- subscriptions;
- social sharing;
- feedback prompts; and
- platform checks.

so ull run into commands like:

- `*check_purchase`;
- `*purchase`;
- `*purchase_discount`;
- `*advertisement`;
- `*delay_break`;
- `*login`;
- `*subscribe`; and
- `*share_this_game`.

:::note u dont need these yet
for ur first little local project, just focus on prose, variables, choices, conditions, labels, scenes, and testing. all the store and platform stuff matters later, when ur actually prepping a real release. dont let it scare u off now.
:::

## useful built-in variables

choicescript reserves any name that starts with `choice_` for itself.

u can read built-in ones like:

- `choice_scene`;
- `choice_linenum`;
- `choice_title`;
- `choice_nightmode`;
- `choice_is_web`;
- `choice_is_steam`;
- `choice_is_trial`;
- `choice_randomtest`; and
- `choice_quicktest`.

and dont try to make ur own variables starting with `choice_`. thats the engines turf.

## the errors ur gonna meet

ur gonna see these. everyone does. heres what theyre actually mad about:

### `Non-existent command`

u misspelled a command, made one up, or used a choice-only modifier somewhere it doesnt belong.

### `Non-existent variable`

u used a variable before declaring it with `*create` or `*temp`.

### `Illegal mixing of spaces and tabs`

ur file has both indentation styles in it. pick one, told u so.

### `invalid indent`

sibling lines dont line up, or a block is indented wrong.

### `increasing indent not allowed`

u indented even though the line before it didnt open a block.

### `It is illegal to fall out of a *choice statement`

an option hit the end of its block without a goto, finish, return, or any other clear exit. the classic.

### `Neither true nor false`

a condition got handed something that wasnt a valid true/false.

### `Not a number`

a numeric expression got text or some other non-number thrown at it.

### `visited this line too many times`

a loop kept jumping back to the same line past the current `*looplimit`. usually an infinite loop u didnt mean to write.

## a sensible order to actually learn this in

dont try to memorize every command. u will lose ur mind. learn these first:

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

once those feel natural, THEN wander into subroutines, arrays, achievements, saves, platform commands, and all the commercial integration stuff.

## a tiny complete example

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

:::tip test constantly
the language itself is simple, but indentation and flow errors get really annoying to trace once a scene gets big. so test after every meaningful branch instead of writing 500 lines and praying.
:::

## every top-level command

heres the full set of commands the engine actually knows, in case u ever wanna check if something's real:

`abort` · `achieve` · `achievement` · `advertisement` · `ai` · `allow_reuse` · `author` · `bug` · `check_achievements` · `check_purchase` · `check_registration` · `choice` · `comment` · `config` · `create` · `create_array` · `delay_break` · `delay_ending` · `delete` · `delete_array` · `disable_reuse` · `else` · `elseif` · `elsif` · `end_trial` · `ending` · `fake_choice` · `feedback` · `finish` · `finish_advertisement` · `gosub` · `gosub_scene` · `goto` · `goto_random_scene` · `goto_scene` · `gotoref` · `hide_reuse` · `if` · `ifid` · `image` · `input_number` · `input_text` · `kindle_image` · `kindle_product` · `kindle_search` · `label` · `line_break` · `link` · `link_button` · `login` · `looplimit` · `more_games` · `page_break` · `page_break_advertisement` · `params` · `print` · `print_discount` · `product` · `purchase` · `purchase_discount` · `rand` · `redirect_scene` · `reset` · `restart` · `restore_checkpoint` · `restore_game` · `restore_purchases` · `return` · `save_checkpoint` · `save_game` · `scene_list` · `script` · `set` · `setref` · `share_this_game` · `show_password` · `sound` · `stat_chart` · `subscribe` · `temp` · `temp_array` · `text_image` · `timer` · `title` · `track_event` · `youtube`

choice-body-only modifiers include `*selectable_if`, `*random_weight`, and the choice-scoped forms of the condition and reuse commands.

PS. the source reference goes way deeper into the exact argument grammar and engine behavior of every single command. this version is just meant to get u writing without lying to u about how any of it works.

:::tip 💖
made with love by [kmab](https://kmab5.github.io).
:::
