# Entity and Aniimo data model

This file collects the Aniimo-specific structures that were useful enough to name publicly without turning the repository into a working radar implementation.

## Confidence key

- **Confirmed** — directly reproduced with enough consistency to treat as established for the researched client.
- **Observed** — directly seen, but with a narrower test surface.
- **Inferred** — interpretation strongly supported by surrounding behavior.
- **Unknown** — insufficient evidence.

## `ClientPuppet`

**Confirmed / observed depending on field.**

Live wild Aniimo are represented through runtime `ClientPuppet` objects.

Useful identity/state names observed on or around the object include:

| Name | Meaning in the researched model | Confidence |
| --- | --- | --- |
| `isWild` | distinguishes wild-world context | Confirmed |
| `templateId` | template/species-side identity | Confirmed |
| `level` | entity level | Confirmed |
| `staticId` | static/world identity context | Observed |
| born/world coordinates | origin/spawn-side spatial context | Observed |
| `getPosition()` | current world position | Confirmed |

Additional state exists through the entity/property system, but the presence of a property on a live object does not automatically make it a unique per-spawn roll.

## Entity lifecycle / AOI

**Observed.**

The runtime contains an Area-of-Interest/entity-management layer. Names found in the researched client include:

```text
entitiesInRange*
onEntityJoin
onEntityLeave
addEntity
removeEntity
ActorManager
```

I treat those names as architectural evidence, not a stable supported API.

The important conclusion is that the client maintains a dynamic set of locally relevant entities and receives lifecycle changes as that set changes.

## Static scene data

**Observed.**

Static client material includes scene/spawner concepts such as:

```text
scene_entity_data
scene_spawner_data
```

These can help identify configured content and explain runtime references.

They should not be used as a substitute for the live entity set.

A spawner saying that an Aniimo *can* exist somewhere is not evidence that it currently exists there.

## `PetInfo` and individual properties

**Confirmed for owned Aniimo records.**

Static/decompiled material exposed several useful names around Aniimo individual state:

```text
PetInfo
propertyScoreStage
basePropertyList
nature
```

The strongest result came from owned Aniimo data, where `basePropertyList[*].indLv` corresponds to the individual Potential values.

Observed order:

| Index | Potential stat |
| ---: | --- |
| 0 | HP |
| 1 | ATK |
| 2 | P.DEF |
| 3 | REGEN |
| 4 | M.DEF |
| 5 | BREAK |

A conceptual record is therefore:

```text
basePropertyList
  [0].indLv -> HP
  [1].indLv -> ATK
  [2].indLv -> P.DEF
  [3].indLv -> REGEN
  [4].indLv -> M.DEF
  [5].indLv -> BREAK
```

This is important because it gives a concrete example of **true individual data** rather than a species template.

## Personality / MBTI-style letters

**Confirmed for the researched owned records.**

Personality letters are represented through talent IDs:

| Talent ID | Letter |
| ---: | :---: |
| 201 | E |
| 202 | I |
| 203 | S |
| 204 | N |
| 205 | T |
| 206 | F |
| 207 | J |
| 208 | P |

The meaningful lesson is not just the mapping. It is the fields that turned out **not** to be the answer.

`nature` is not the displayed four-letter personality.

`characterInfo.curCharacter` is also not the displayed four-letter personality.

Naming is not evidence. I only accepted the talent representation after it matched the observed personality data.

## Species baseline vs. individual Potential

This was the easiest mistake to make and the one most worth documenting.

The client has several places where property-looking numbers can appear:

```text
species/template defaults
static configuration
runtime replicated properties
owned individual values
UI/appraisal state
```

Those layers are related. They are not interchangeable.

If an inspector searches recursively for six stat-looking numbers, it can find something convincing while still being completely wrong about what those numbers represent.

My internal validation therefore kept source provenance and rejected candidates that behaved like species/static defaults.

The public rule is simpler:

> Do not call a value “this spawn's Potential” unless its source and behavior establish that it belongs to the individual spawn.

## Appraisal

**Observed.**

Appraisal did not turn the obvious client cache/log locations into a clean per-Aniimo data source during testing.

That suggests appraisal is better understood as revealing/presenting state already represented elsewhere, rather than writing a convenient standalone record into the files I tested.

I am leaving the stronger server/client timing interpretation as **unknown** here because the public evidence does not require a more aggressive claim.

## Lucky / Super Lucky

Static material indicated that some Lucky-related fields were marked server-side-only in the researched scripts/data.

That is useful negative evidence: not every mechanic visible in the UI has to be authoritatively derivable from local client state before the server decides it.

I did not use that observation as proof of the full Lucky/Super Lucky algorithm.

## Capture / destruction lifecycle

The live entity model includes capture/destruction lifecycle information around `ClientPuppet` state.

That is useful when reasoning about why an entity disappears from the runtime set: leaving AOI and being removed because of gameplay are not necessarily the same event semantically.

The repository does not publish the event-consumption implementation.

## A sanitized normalized model

The model I found useful conceptually looks like this:

```json
{
  "entity": {
    "runtime_id": "<runtime identity>",
    "template_id": "<template identity>",
    "static_id": "<static context if present>",
    "wild": true,
    "level": 0,
    "position": {
      "x": 0,
      "y": 0,
      "z": 0
    }
  },
  "individual": {
    "potential": "<only when provenance supports it>",
    "personality": "<only when provenance supports it>"
  }
}
```

The `individual` block is deliberately conditional.

A good runtime model can say “I know the entity exists, but I do not know its individual roll.” That is better than silently substituting a species value.
