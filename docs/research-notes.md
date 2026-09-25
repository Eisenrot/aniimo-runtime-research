# Research notes

This is the less polished version of the story: what I tried, what failed, and what finally changed the shape of the problem.

I am keeping the dead ends because removing them makes reverse engineering look much more mystical than it is. Most of the work was repeatedly proving that an attractive explanation was wrong.

## 1. Start with the boring files

The first idea was the obvious one: Aniimo might already expose enough state through logs or client cache files to build a useful read-only model without touching the runtime.

I started with Unity's ordinary `Player.log` and the game's client-side cache area.

`Player.log` was useful for:

- confirming paths and startup behavior;
- correlating actions with client events;
- seeing enough vocabulary to guide later searches.

It was not a complete structured feed for nearby Aniimo, Potential or personality.

## 2. `_client_info_bin.zst` looked promising for about five minutes

The client cache contained `_client_info_bin.zst`.

A deliberately simple test killed the useful-roster theory:

1. record the file size/content;
2. change the active party in-game;
3. record it again.

The file remained **3814 bytes and unchanged** in the test.

That made it extremely unlikely to be the live roster/stat source I was looking for.

The lesson was useful: a file sitting under something called `client_cache` is not obligated to contain the particular client state I care about. Shocking behavior from a filename, I know.

## 3. Catching and appraisal still did not produce the goods

I captured multiple Aniimo and compared the resulting client-side changes.

The obvious cache/log state only gained opaque record-style booleans. I did not get clean Potential, personality, rating or appraisal values out of that path.

Appraisal itself also failed to produce a useful content change in the tested client-info cache.

At that point continuing to diff the same tiny files would have been optimism wearing a lab coat.

## 4. Static script/data archaeology was still valuable

The next useful step was not “find the values,” but “learn the client's language.”

Static/decompiled client material exposed concepts around individual values and Aniimo records, including:

```text
Individual / IV-style machinery
PetInfo
propertyScoreStage
basePropertyList
nature
```

Wild runtime object names also became visible, including `ClientPuppet` and fields such as `isWild`, `templateId`, `level` and `staticId`.

This did not yet give a live radar.

It did something more useful first: it told me what structures to look for when observing the runtime.

## 5. Generic Unity inspection was not the convenient shortcut

I also tried the ordinary “surely a generic Unity inspector will make this trivial” route.

It did not become the reliable foundation I wanted against this client. The useful game state sat behind boundaries that made generic drop-in inspection a poor path compared with understanding the game's own structures.

I am intentionally not documenting attempts aimed at defeating those boundaries.

The research moved back toward static client analysis plus narrowly targeted runtime observation.

## 6. The pivot: stop searching for a secret file

The decisive change was treating Aniimo as a live entity problem instead of a hidden-save-file problem.

The client maintains an AOI/runtime entity layer. Live wild Aniimo appear as `ClientPuppet` objects. The surrounding system exposes local-range collections and entity lifecycle concepts.

Once that was established, several previously awkward questions became straightforward:

- what does the client currently know exists?;
- which objects are wild?;
- what template/level/static context belongs to each object?;
- where is each object now?;
- when does an object join or leave local relevance?

That is the actual foundation of the runtime model.

## 7. Position and identity were the easy part

Compared with individual stats, identifying and locating a live entity was pleasantly civilized.

Observed `ClientPuppet` state included:

```text
isWild
templateId
level
staticId
born/world position
getPosition()
```

The exact extraction mechanism is private, but the conceptual result is simple: the client has a live object representing the entity, and that object knows enough to identify and place it in the world.

## 8. Potential was harder because plausible wrong answers existed everywhere

Individual Potential was the dangerous part.

The client contains species data, property lists, static definitions and runtime property structures. More than one path can produce six numbers that look suspiciously like stats.

That means “I found six values” is not a result.

A useful result needs to answer:

- are these values different across individuals of the same species?;
- do they correspond to the displayed/appraised Potential?;
- are they coming from a template inherited by every instance?;
- is the source path stable enough to identify what layer owns them?

My internal probes became deliberately conservative. Candidate values kept their source paths. Species-looking defaults were not relabeled as per-spawn Potential. If a six-stat individual candidate could not be established, the correct result was effectively “no individual candidate found.”

This was slower and much less exciting than printing every plausible number.

It was also correct.

## 9. Owned Aniimo gave the clean reference model

Owned records finally provided a clean anchor.

`basePropertyList[*].indLv` matched the six individual Potential values in this order:

```text
HP
ATK
P.DEF
REGEN
M.DEF
BREAK
```

That gave a real per-individual structure to compare against other property-looking data.

It also made the distinction between baseline/species properties and individual rolls impossible to ignore.

## 10. Personality had its own false friends

Two fields looked semantically tempting:

```text
nature
characterInfo.curCharacter
```

Neither was the displayed four-letter personality.

The actual letters lined up with talent IDs:

```text
201 E
202 I
203 S
204 N
205 T
206 F
207 J
208 P
```

Once again the lesson was irritatingly simple: field names are clues, not verdicts.

## 11. AOI made lifecycle sane

Static scene/spawner data can explain where content is configured.

The AOI/entity layer explains which objects are live and relevant now.

Observed client names around that layer include:

```text
entitiesInRange*
onEntityJoin
onEntityLeave
addEntity
removeEntity
ActorManager
```

I do not treat those symbols as a supported API. I treat them as evidence for the architecture.

A normalized consumer can conceptually mirror joins, updates and leaves without periodically pretending the entire static scene database is a list of currently spawned Aniimo.

## 12. The dormant local command path

The client contains a local command/socket manager with JSON-shaped requests.

Normal startup does not expose it as a ready-to-use public interface.

During research I confirmed enough behavior to establish that it was not dead decorative code.

That is the point where the public write-up stops.

The following remain private:

- the client modification used to expose the mechanism;
- patch locations/bytes/signatures;
- endpoint details;
- framing details;
- request/command recipes;
- offsets/addresses;
- the bridge that turns runtime inspection into a live external feed.

The existence of the mechanism is architecturally interesting. The activation recipe is operationally useful for exactly the reason I do not need to publish it.

## 13. Final model

The research ended up with three distinct layers:

```text
1. static context
   species, templates, scene/spawner definitions, vocabulary

2. live runtime state
   AOI, ClientPuppet, identity, position, runtime properties

3. normalized consumer state
   stable semantic records for whatever visualization/research UI wants them
```

Owned Aniimo data provides an additional reference point for individual Potential/personality representation.

## What I would repeat on another game

The Aniimo-specific names are less important than the process:

1. test the cheap file/log hypothesis first;
2. use controlled state changes to falsify cache theories quickly;
3. use static analysis to learn vocabulary before blindly scraping values;
4. find the runtime object that actually owns the live state;
5. separate identity, template data and individual data;
6. keep provenance for every suspiciously convenient value;
7. allow “unknown” to survive all the way to the output;
8. treat UI confidence as presentation, not evidence.

That process is the part of this repository I actually wanted to preserve.
