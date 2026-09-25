# Data boundaries and publication scope

This file exists because there is a very real difference between documenting a client architecture and packaging the last ten percent into a turnkey gameplay-assistance tool.

I am publishing the first one.

## Evidence boundary

Every interesting claim falls into one of four buckets.

### Confirmed

Directly reproduced with enough consistency that I am comfortable treating the behavior as established for the researched client build.

Examples in this repository include the owned Potential representation and the basic live `ClientPuppet` identity/position model.

### Observed

Directly seen, but with fewer repetitions, narrower conditions or less reason to claim long-term stability.

Many internal symbol names belong here. They describe the September 2026 client, not a public SDK.

### Inferred

The evidence supports an interpretation, but another explanation has not been eliminated strongly enough.

Inferences are useful as hypotheses. They stay labeled as hypotheses.

### Unknown

The available evidence does not justify a conclusion.

This category is intentional. I would rather leave a hole in a diagram than fill it with a convenient invention.

## Data provenance boundary

A value is only as trustworthy as its layer.

I use the following conceptual provenance stack:

```text
STATIC TEMPLATE
    ↓
STATIC SCENE / SPAWNER CONTEXT
    ↓
LIVE ENTITY
    ↓
RUNTIME / REPLICATED PROPERTY
    ↓
INDIVIDUAL RECORD
    ↓
UI PRESENTATION
```

The arrows are not “more true” in every situation. They represent different ownership/context.

For example:

- a species template is authoritative for a species default;
- a live entity is authoritative for current existence/position;
- an owned individual record can be authoritative for that Aniimo's individual Potential;
- the UI can reveal useful state but is not automatically the storage source.

The common failure mode is taking a correct value from the wrong layer and attaching the wrong meaning to it.

## Public material

I am comfortable publishing:

- object/class names needed to explain the architecture;
- static vs. live distinctions;
- AOI/entity lifecycle concepts;
- sanitized normalized schemas;
- the owned Potential field interpretation;
- personality talent-ID mapping;
- research chronology;
- failed paths;
- evidence/confidence labels;
- architectural diagrams;
- statements that an internal/dormant mechanism exists.

That is enough for the research to be independently understandable.

## Private material

I am intentionally keeping the following out of the public repository:

- working radar source;
- the private dashboard/application layer;
- compiled extractors;
- injection/hooking implementation;
- exact patch bytes;
- exact patch locations or signatures;
- offsets/addresses used for a live build;
- instructions for activating internal interfaces;
- local endpoint/port details;
- protocol/framing details needed to talk to the internal interface;
- live command/request recipes;
- protection or anti-cheat circumvention steps;
- packaged automation that creates a gameplay advantage.

These are not missing documentation tasks.

They are the publication boundary.

## Why object/field names are still here

Removing every real symbol would make the repository nearly useless as technical research.

Names such as `ClientPuppet`, `basePropertyList`, `indLv`, `onEntityJoin` and `getPosition()` establish what was actually observed and make the conclusions falsifiable.

What is omitted is the operational chain that turns those observations into a working external tool against the current client.

That is the line I chose.

## Official rules at the time of writing

This research snapshot was prepared in September 2026.

At that time:

- Aniimo's official [Fair Play Announcement](https://www.aniimo.com/newslist/detail/100067) prohibited third-party tools/client modifications that interfere with fair play;
- Pawprint Studio's [Terms of Use](https://pawprintstudio.com/terms-of-use/en) included restrictions on reverse engineering/decompilation, security probing/circumvention and data mining.

Those documents can change. Anyone working with the client should check the current versions rather than treating this repository as legal or policy advice.

## No license

No license file is included.

The documentation is public to read, not automatically granted for unrestricted redistribution, repackaging or commercial reuse.

If substantial reuse is wanted, ask first.

## Update policy

I may update this repository when there is something genuinely interesting to document, for example:

- a client update materially changes the entity model;
- an earlier interpretation proves wrong;
- a static/runtime distinction can be documented more accurately;
- a new finding improves the research story without becoming a turnkey gameplay tool.

I do not intend to chase client updates solely to keep an external radar implementation functional.

There are enough repositories on the internet devoted to turning every interesting discovery into a download button. This one can survive being a research notebook instead.
