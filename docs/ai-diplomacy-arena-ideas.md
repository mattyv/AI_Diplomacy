# AI Diplomacy Arena: ideas

## The experience

People bring their own AI leader to a shared Diplomacy game, give it a
personality, and watch alliances form and collapse. The broadcast makes the
stakes understandable: who promised what, who needs whom, and who is preparing
to betray an ally.

The drama comes from actual messages, decisions, and board outcomes. Commentary
must distinguish an attempted betrayal from a successful attack and a suspected
lie from a documented broken promise.

This is an ideas document, not an accepted implementation specification.

## What already exists

- Claude Code and Codex CLI providers, with separate models for each country.
- Custom prompt directories per country, private diaries, and relationship tracking.
- A game engine with multiplayer foundations.
- An animated replay viewer and analysis tools for interesting moments.

The current AI game runner invokes all players on one computer. A lobby where
remote users connect their own models still needs to be built.

## Make the game entertaining to watch

### Promises and betrayals

Track concrete commitments from negotiation messages: a demilitarized province,
a promised support order, a joint attack, or an agreement to leave a supply
center to an ally. Each commitment should retain the original message, who made
it, its recipient, any conditions, and the turn when it applies.

At resolution, compare commitments with submitted orders and outcomes. Show the
promise beside the action: “France promised to leave Burgundy empty. France
ordered its army into Burgundy.” If the move failed, show that explicitly.
Distinguish a player's deliberate order from another player's interference.

Conditional promises, ambiguous language, and changed agreements need an
“unclear” or “superseded” state. Avoid confidently declaring betrayal when the
evidence is incomplete.

### Intentions, suspense, and reactions

Ask each leader for a short strategic intention alongside its orders, such as
“Keep England friendly while preparing an attack.” This is a player-authored
statement for the show, not a request for hidden internal reasoning.

Reveal consequential orders together. Pause before contested attacks, then show
the board outcome, the affected promise, and a brief reaction from the victim.
Prefer generating intentions and reactions within calls the game already makes.

For competitive games, show private intentions only after orders resolve.
For casual broadcasts, an optional spoiler view could reveal preparations early,
but players watching that view would gain information their countries lack.

### Highlights and pacing

Give priority to broken promises, alliance changes, surprise supports, lost home
centers, successful deception, and narrow escapes. Compress routine holds and
uncontested expansion. Offer both full replay and a highlights reel.

Use short commentary grounded in the saved game. A relationship display and
recurring leader portraits can help viewers recognize rivals and follow grudges.
Avoid adding a narrator call for every minor event.

## Personalities

All players should pursue victory. Their personalities change how they negotiate,
handle uncertainty, and decide whether an alliance is still useful.

| Personality | Behavior | Weakness or breaking point |
| --- | --- | --- |
| Loyalist | Keeps agreements and builds lasting partnerships | May break an alliance when survival is threatened |
| Opportunist | Changes sides when a better deal appears | Burns trust and can become isolated |
| Schemer | Builds alliances while preparing future attacks | Can overcomplicate plans or strike too early |
| Paranoid | Demands reassurance and guards vulnerable borders | May turn an innocent move into a feud |
| Avenger | Remembers betrayals and pursues the offender | Can sacrifice expansion to settle a score |
| Gambler | Takes risky moves for large gains | Can lose everything to one failed prediction |
| Diplomat | Brokers agreements and organizes coalitions | May become dependent on allies keeping their promises |

Give each leader a recognizable voice, one weakness, and a breaking point.
Let game events influence its behavior; a personality should leave room for
adaptation rather than force a predetermined story.

Users could start from a preset, then add their own instructions. Example:
“Build trust, keep promises until victory is within reach, then strike.”

## Players bring their own models

### Joining a game

1. Join through an invite link.
2. Choose a country and name the leader.
3. Select an available model and connect a local client.
4. Choose a personality or write a custom prompt.
5. Review the turn deadline and ready up.

Lock personality prompts when the match starts. An experimental mode could allow
prompt changes between turns, with changes visible to the host, but it should
have a different ruleset from a match with fixed personalities.

### Local CLI connector

For Claude Code and Codex CLI players, run a small connector on each user's own
computer. It receives tasks for their country, invokes their signed-in CLI, and
returns messages and orders. Credentials remain local, and each user consumes
their own account allowance.

The connector should initiate the connection to the host so players do not need
to expose a port on their computers. It should accept narrowly defined game
tasks and approved model choices, rather than arbitrary shell commands.

Keep the player's custom personality prompt local if they want it private. The
host controls game instructions and required response formats. The model sees
both, with the game rules taking precedence over personality preferences.

### Shared host and fairness

The host owns the authoritative board, turn schedule, order validation, and
resolution. Each connector receives only public board information and messages
and memory its own country can legitimately see. A player's agent must not gain
access to another country's private messages or prompt.

Use country-scoped credentials for connectors, reconnect support, and submission
identifiers so retries cannot submit duplicate actions or orders for a stale
turn. Resolve simultaneously after the deadline or after everyone is ready.

Agree on disconnect behavior before the game: pause for a short grace period,
then submit legal hold or adjustment orders, or use a clearly labeled host bot.
Spectators should be able to see when a player missed a turn or used fallback
orders. Never silently present a fallback as the player's chosen strategy.

## Reduce token use and improve caching

The initial run logged 63 completed task records across setup and two movement
turns, with roughly 130,000 text tokens estimated from character counts. That
sample excluded system instructions and internal reasoning, did not measure cache
hits, and also had nine retry entries. It is not a billing estimate or a forecast
for future matches.

### Stable context first

The current game adds a new block of roughly 400 random characters before the
instructions on each call. This disrupts the otherwise stable prompt prefix.
Remove it for normal play, or put any requested variation after the stable
context. Random prompt text is not a provider's deterministic sampling seed.

Keep game rules and a country's personality stable. Follow them with compact
memory and board context, then the newest task. Measure the effect rather than
assuming a particular cache benefit.

### Lessons from Personal Assistant

Personal Assistant keeps stable instructions before changing details, summarizes
older history, and reconstructs prior conversation while replacing bulky past
tool results with small placeholders. Its CLI route leaves cache handling to
Claude Code; its API route separately uses explicit cache controls.

A reconstructed or resumed session still supplies history to the model. A
matching prefix can be cached; the session ID itself does not guarantee a cache
hit. The game already assembles history into each prompt, so copying the
undocumented Claude session format is not a prerequisite for improving caching.

### Smaller memory and fewer calls

- Preserve commitments, betrayals, relationships, and strategic objectives in
  compact country-specific memory.
- Keep a short recent history and summarize older events when needed.
- Combine orders, a short intention, and memory updates where practical.
- Generate extra commentary only for consequential events.
- Record actual CLI usage metadata: fresh input, cache reads, cache writes,
  output, model identity, and retries, when the CLI reports them.
- Surface refusals and avoid repeatedly sending an unchanged rejected request.

Reducing negotiation rounds saves calls but can also remove the diplomacy that
makes the game interesting. Start with smaller context and fewer administrative
calls before cutting negotiations.

Caching depends on matching content, provider behavior, model, and cache lifetime.
Its effect on subscription allowances should be measured separately from API
pricing. See [Claude prompt caching documentation](https://platform.claude.com/docs/en/build-with-claude/prompt-caching).

## Suggested order

1. **Make the existing game observable and efficient.** Record actual usage,
   stabilize prompt prefixes, and make failed turns and fallback orders visible.
2. **Give leaders character.** Add personality presets, custom prompts, and short
   strategic intentions to existing turn outputs.
3. **Create the drama layer.** Track promises, reveal betrayals with evidence,
   and produce a highlights replay from a saved game.
4. **Invite other players.** Build the lobby, local connector, deadlines, and
   reconnect behavior, beginning with one private seven-country match.

## Decisions to resolve before implementation

- Does a human only configure their bot, or may they steer it during a match?
- Are we optimizing for live matches, asynchronous matches, or recorded shows?
- How long is a turn, and what happens when someone disconnects?
- Which private information can spectators see, and when?
- Should model choice and personality prompts be revealed after the game?
- Can users set a usage budget and pause their player before exceeding it?
- What evidence is sufficient to label a broken promise as deliberate betrayal?
