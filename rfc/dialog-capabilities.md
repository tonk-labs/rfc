# Dialog: What Becomes Possible

A capabilities brief for product and design exploration.

---

## The Premise

Software today is organized around places. You go to Gmail for email, to Notion for notes, to Spotify for music. Your information is scattered across these destinations and each service becomes a silo that owns a slice of your digital life. You are the integration layer, copying, pasting, context-switching between places to assemble a complete picture.

Dialog inverts this entirely. Your data follows you. It doesn't live "in" any application the way your email lives "in" Gmail. It lives with you, a knowledge substrate that travels across devices, tools, and contexts.

Today you are a guest in the spaces that apps provide. You visit their house, play by their rules, leave your data behind when you go. Dialog flips this around. Your substrate is your space. Tools and agents are guests you invite in. They come, they contribute, they derive insights, and when they leave your data stays with you. The tool is transient; the substrate persists.

Dialog is not another silo. It is a protocol and embeddable substrate, infrastructure that any tool, extension, agent, or interface can tap into to read, contribute, and enrich your knowledge. A browser extension, a CLI tool, a mobile app, an LLM agent, each is a different surface connecting to the same underlying substrate. The tool is transient; the substrate persists.

This means cooperation between tools doesn't require corporate control, negotiated APIs, or manual effort. Tools contribute facts and derive insights from the same substrate, and cooperation emerges from overlapping concepts rather than coordinated integrations. Nobody goes anywhere. The data is already here.

Dialog is a working proof-of-concept that synthesizes ideas from semantic data modeling, logic programming, distributed version control, and content-addressed storage into something that behaves like git, but for structured knowledge, with a query engine, built-in replication, and no servers required.

What follows is a walkthrough of Dialog's properties and what each unlocks for product thinking.

---

## Semantic Data Model

### The Property

Dialog stores facts, not rows or documents. A fact is an atomic statement: "this entity has this attribute with this value." There are no tables, no schemas, no rigid structures to define upfront. You describe information the way you'd describe it in conversation, as relationships between things.

A recipe has ingredients. An ingredient has a name. A person has an allergy. These are all facts. They don't need to live in the same table or even be created by the same tool. They just exist in the substrate, ready to be connected.

### Schema-on-Read

Traditional databases require you to decide how data will be accessed before you store it. Dialog does the opposite. You store facts freely and impose structure at query time through concepts. A concept is a named pattern: "a recipe is an entity that has a title, ingredients, and instructions." The concept doesn't constrain what's stored, it's a lens for viewing what's there.

This means different tools can have completely different mental models of the same data. A meal planner sees recipes. A nutrition tracker sees ingredient-nutrient relationships. A shopping app sees ingredient quantities. None of these tools need to agree on a schema. They each define concepts that match the facts they care about.

### Metacircular

Here's where it gets interesting. Concepts, rules, and attributes are themselves facts in the database. They're not configuration files or external metadata, they're queryable, discoverable, first-class data. In the Lisp tradition everything is metacircular.

This means a tool can ask the substrate "what concepts exist here?" and discover structure it didn't know about. An interface can present available concepts to a user and let them explore. An LLM agent can introspect the data model itself, understand what's there, and reason about what's missing.

Every concept has a natural-language description (because that description is itself a fact). Every attribute has a name and a type. This makes the substrate self-describing, any interface that can read facts can understand the shape of what's inside.

This gets even more interesting when you think of facts as semantic memory. You can have facts about facts, creating episodic memories. An LLM agent can explore and discover what is in the substrate and capture its discoveries as facts, associating them with the entities they relate to. The substrate becomes something that learns about itself.

This metacircular property extends to agents themselves. People already define agents and their skills in markdown files. What if agent definitions were also just facts in the substrate? Their capabilities, their configurations, their access scopes, all queryable and discoverable. Agents could discover other agents, compose capabilities, evolve their own definitions, and link directly to all the other facts in the database. The boundary between "the data," "the rules," and "the agents that operate on them" dissolves into one self-describing system.

### Facts as Lingua Franca

Schema evolution is one of the hardest problems in collaborative software. The typical approach, migrating between schema versions, suffers from a dimensionality problem. If a schema can evolve in many directions (and in a decentralized world, it will), you need translations between every pair of versions. This is the challenge approaches like Ink & Switch's Cambria face: n² translation paths that grow combinatorially.

Dialog sidesteps this entirely. Every data model maps down to atomic facts, and every data model maps back up from facts through rules. Facts are the lingua franca. You don't need n² translators between tools, each tool just needs two: one to express its model as facts, and one to reconstitute its model from facts through rules. Rules can even reinterpret the same underlying facts into entirely different conceptual models.

This is why cooperation scales. Adding a new tool to the ecosystem is a constant-cost operation, not one that grows with the number of existing tools.

### What This Unlocks

- Tools don't need to coordinate. A meal planner and a health tracker created by different people, at different times, with no knowledge of each other, can operate on the same substrate. Where their concepts overlap, cooperation happens automatically.

- The user can define bridging rules. If two tools use slightly different models (one calls it "allergen," the other calls it "dietary restriction"), a simple rule translates between them. The user writes this rule, not the tool makers. And they can share it with others.

- Natural language becomes a discovery interface. Since every concept, attribute, and rule carries a description, you can search the substrate by meaning. Type "recipe" and discover concepts, attributes, and entities that relate to cooking, even if they were created by tools you've never used.

- Structure can evolve without migration. Adding a new concept doesn't require altering existing data. Old tools keep working. New tools see new patterns. No migrations, no breaking changes, no coordination.

---

## Temporal Immutability

### The Property

Facts in Dialog are temporal. They record not just what is true but when it became true and what it supersedes. Nothing is ever mutated or deleted. Information accretes. A fact might be succeeded by a newer fact, but the original persists in history.

This is similar to how memory works. You don't erase the belief that Pluto is a planet; you acquire a new fact that it was reclassified. Both facts exist in your memory, with temporal context about when each was current.

### Causal References

Each fact carries a causal reference to the facts it builds upon or supersedes. This establishes a partial order, not wall-clock time, but logical succession. If Alice asserts "the project is on track" and later Bob asserts "the project is delayed," both facts exist, each linked to its causal context. There's no conflict to resolve because the granularity is fine enough that concurrent assertions about different things never collide, and concurrent assertions about the same thing carry enough causal information for automatic resolution, similar to how operation-based CRDTs work.

### What This Unlocks

- Complete audit trail. Every fact traces back to the moment and context of its creation. You can reconstruct the state of the substrate at any point in time. You can ask "what did we know last Tuesday?" and get an answer.

- No data loss. Mistakes are correctable without losing history. Retracting a fact doesn't destroy it, it creates a new fact that supersedes it. You can always go back.

- Time-travel queries. Query the substrate as it was at any past moment. Compare states across time. Track how understanding evolved.

- Safe experimentation. Since nothing is destroyed, every edit is reversible. This changes the psychology of interaction, users can explore freely without fear of breaking things.

---

## Version Control

### The Property

Dialog works like git, but for structured data instead of text files. Every change produces a new revision that shares structure with the previous one through content-addressed storage. You can branch, merge, fork, and rebase, all the workflows that make git powerful for code collaboration, applied to knowledge.

Unlike git, where the unit of change is a text file (leading to frequent merge conflicts), Dialog's unit of change is an atomic fact. Two people adding different facts to the same substrate almost never conflict. And when they do, the causal references provide enough information for automatic resolution.

### Branching

Create a branch to explore an idea without affecting the main line. A meal plan for an upcoming dinner party. A speculative reorganization of project tasks. An agent's analysis that you want to review before integrating. Each branch is a lightweight, independent line of development that can be merged back when ready.

### Local yet Ubiquitous

Just like git you start local. Your substrate is fully functional without any network. When you want to collaborate or just have a backup, you add a remote. Want to switch providers? Add a new remote and push. Your data is not tied to any particular service, it's content-addressed blobs that can live anywhere.

### What This Unlocks

- Exploration without commitment. Try things out in a branch. If it works, merge. If not, discard. No consequences.

- Review workflows. An agent or collaborator works in a branch. You review the changes, seeing exactly what facts were added or modified, and merge what you approve.

- Divergent perspectives. Multiple people can maintain different views of the same base data, merging selectively. Your personal annotations stay in your branch; shared facts live in the common trunk.

- Offline-first by nature. Work locally, accumulate changes, sync when connectivity returns. Exactly like git, your local replica is fully functional without any network.

---

## Reactivity

### The Property

Queries in Dialog are not just one-shot operations, they can be subscriptions. When you define a query, you can hold it open and receive updates whenever the matching facts change. This makes the substrate a reactive system: changes propagate to all interested observers automatically.

### Blackboard Architecture

The substrate behaves like a tuple space or blackboard system. Any process can write facts. Any process can subscribe to patterns. Communication happens through the shared space rather than through direct connections between processes. This means components don't need to know about each other, they coordinate through the data.

A query is effectively an inbox. Subscribe to "all tasks assigned to me that are not done" and you have a live feed. Another process writes a new task assignment, your subscription fires. This is how message queues, job schedulers, and event-driven architectures emerge naturally from the data model.

### Propagator Network

Because subscriptions can trigger rules that produce new facts, which trigger other subscriptions, the substrate becomes a propagator network. Information flows through chains of rules across the substrate, and across replicas. A change on one device propagates to another through sync, triggers a subscription there, which produces new derived facts, which sync back.

This is computation expressed declaratively. Instead of writing imperative workflows ("when X happens, do Y"), you declare constraints and relationships. The substrate figures out what follows.

### Dedalus-Inspired Behaviors

Drawing from the Dedalus model of declarative networking, Dialog supports inductive rules, rules that define behaviors over time. "When a counter is incremented, its count increases by one" is not an imperative command but a declared relationship between states. The substrate maintains it continuously.

This means interactive behaviors, things that feel like application logic, can be expressed as rules in the substrate. They're data, not code. They're queryable, shareable, composable. As opposed to telling the system how to do things, you declare constraints that need to be met for specific conclusions to be drawn.

### What This Unlocks

- Live interfaces. UI components subscribe to queries and update automatically as facts change. No manual refresh, no polling.

- Inter-tool coordination. One tool writes a fact. Another tool's subscription picks it up and reacts. No explicit integration required, the blackboard mediates.

- Declarative workflows. Express business logic as rules rather than imperative code. "When all tasks in a project are done, the project status is complete" is a rule, not a script.

- Distributed computation. Rules fire across replicas. A fact written on your phone triggers a subscription on your laptop. The propagator network spans devices.

---

## Multiplayer

### The Property

Every transaction in Dialog is signed by a cryptographic keypair. Every single fact can be attributed to the principal that created it, whether that's a person, an LLM agent, or an automated process.

This is not optional metadata. It's structural. The substrate knows who said what, when, and in what causal context.

### Attribution and Provenance

Because authorship is cryptographic, provenance is unforgeable. You can trace any fact back to its source. You can ask "show me everything contributed by this agent" or "show me only facts from humans I trust." Since this is schema-on-read, these filters are query-time decisions, you don't need to structure the data differently to support different trust models.

### What This Unlocks

- Disentangling human and machine contributions. In a world where LLMs generate content alongside humans, knowing the provenance of every fact becomes essential. Query-time filtering lets you see the substrate with or without agent contributions. It creates opportunity to separate signal from slop.

- Agent sandboxing. An agent is a guest in your space. You grant it write access to a specific branch or scope. Its contributions are clearly marked, reviewable, and mergeable (or discardable) by a human. The agent can read broadly but write narrowly, and every boundary is visible.

- Reputation and trust. Because every fact has an author, you can build reputation signals. Exclude facts from sources you don't trust. Weight contributions by reliability. This emerges naturally from the data model, reputation rules are just more facts and rules in the substrate.

- Moderation as queries. Moderation doesn't require a central authority making deletion decisions. It's a query-time filter, each user or community defines rules about which sources to include. Facts aren't destroyed; they're selectively visible.

- Collaborative intelligence. It's not you alone with your tools. It's you, your agents, your collaborators, and their agents, a manifest collective intelligence. Everyone contributing to a shared substrate with clear attribution.

---

## Layers

### The Property

Dialog supports querying across multiple databases simultaneously. A query can operate over the union of facts from several sources, different databases, different branches, or even mounted external data sources. Each source is a composable layer.

### The Visual Metaphor

Think of layers like those in a graphics editor. Each layer contains facts. You can toggle layers on or off to change what's visible. You can reorder them to change precedence. The composite view, all visible layers merged, is what you interact with.

Some layers are yours alone (private). Some are shared with collaborators (team). Some are public. Moving a fact between layers changes its visibility. Drag from private to public to share, or from public to private to retract. What's private and what's public becomes a tangible spatial arrangement.

### Agent Layers

An agent gets its own layer, spliced into the stack. It can read everything below it (your data, shared context) but writes only to its own layer. This makes the agent's contribution physically separate and visually distinct. You see exactly what the agent added. You can merge its layer down to accept the work, or remove it to discard. It becomes much more legible what access level an agent has and what changes it made or proposed.

### Mounted Data Sources

In theory any data source that can be expressed as facts can be mounted as a read-only layer. A live feed, a public dataset, a collaborator's published substrate. You query across the union of your local facts and these external layers, kind of like GraphQL federation but with Datalog's expressive power and without requiring API coordination. This is more speculative but the architecture points in this direction naturally.

### What This Unlocks

- Composable perspectives. Toggle layers to see your data from different angles. Just your facts. Your facts plus your team's. Your facts plus an agent's analysis. Each combination tells a different story.

- Privacy as spatial arrangement. Making something private or public can be as simple as dragging it from one layer to another. The layer stack makes visibility tangible.

- Reviewable AI contributions. An agent's work is never mixed into your data until you explicitly merge it. The layer boundary is a review gate, giving complete control to the people in the loop.

- Cross-database exploration. Query your personal substrate alongside a friend's published concepts. Discover overlapping facts. Find connections you didn't know existed.

---

## Emergent Cooperation

### The Property

Because Dialog uses semantic facts rather than rigid schemas, and because rules can bridge between different conceptual models, tools that were never designed to work together can cooperate through the substrate.

This is not integration in the traditional sense. Nobody needs to build an API. Nobody needs to agree on a data format. If two tools happen to describe overlapping concepts, even using different terminology, a bridging rule connects them.

### User-Authored Interop

The person who benefits from the interop writes the bridging rule. Not the tool maker, not a platform provider, the user. "When tool A says 'allergen' and tool B says 'dietary restriction,' treat them as the same concept." This is a fact in the substrate, a rule that can be shared, forked, and improved by others.

You don't need to go and convince tool creators to support the data model from the other tool. You can independently add a rule that does the translation. Share it with others and suddenly cooperation emerges naturally.

### Shareable Rules

Rules are facts. Facts sync. When you share your substrate or publish your rules, other people can adopt your bridging logic. Cooperation patterns spread socially. Someone figures out how to connect a meal planner with a grocery app, shares the rule, and suddenly everyone who uses those tools benefits.

### Accessible Authoring

Concepts and rules don't need to look like code. They can be expressed in simple YAML or even something close to natural language (in the spirit of projects like [nl-datalog](https://github.com/harc/nl-datalog). A concept definition reads almost like documentation: it names an entity, describes its attributes in plain language, and specifies what types those attributes take. A rule reads like a set of conditions and conclusions. The notation captures intention, description, and structure all at once in a low-markup format that's approachable even to non-programmers.

This matters because the people who understand their domain best, the food blogger, the fitness coach, the project manager, are not necessarily programmers. If defining a concept is closer to describing it than to coding it, the barrier to participating in the substrate drops dramatically. LLMs make this even more accessible, you can describe what you want in a sentence and get a well-formed concept or rule back.

### A Database You Can Ask Questions

Here's something interesting about Datalog's declarative nature. Because rules describe constraints rather than procedures, an LLM doesn't need to be perfect to be useful. It just needs to generate something that can be verified. The substrate itself can check whether a rule or query is well-formed and whether it produces sensible results. This is similar in spirit to how Acorn uses a local LLM to aid theorem proving, the quality of the proof attempt is secondary because verification is what matters.

This suggests a compelling possibility: a database you can simply talk to. The LLM translates loose natural language into strict Datalog, and the substrate provides immediate feedback on whether it works. It doesn't need to be the best model, even a local LLM can be effective when its job is to translate something loose into something rigid and deterministic, especially with a live feedback loop. The combination of an imperfect translator with a perfect verifier is surprisingly powerful.

### Natural Language Discovery

Because every concept, attribute, and rule carries a natural-language description, the substrate supports rich discovery. Imagine typing a phrase and having the substrate surface matching concepts from tools you've installed, rules others have shared, and attributes that relate to your query. The descriptions make the data model navigable by meaning, not just by key.

This has an interesting connection to Mozilla's Ubiquity, a natural language interface that mapped phrases to actions through a vocabulary of verbs and nouns. Dialog's attributes are like nouns, things you can reference and discover. Behaviors defined through inductive rules are like verbs, actions that can be triggered. The substrate provides the structured memory that Ubiquity never had. What if you could have parse rules of sorts, allowing the system to interpret many different expressions as dates, or amounts, or locations? And verbs could be facts that trigger chains of deductive and inductive rules. Suddenly you have an almost magical way to talk to your knowledge substrate and do things with it.

### What This Unlocks

- Cooperation without coordination. Tools created by different people, at different times, start working together the moment their concepts overlap or the moment someone writes a bridging rule.

- User empowerment. You don't need to petition tool makers for integrations. You write the connection yourself, or adopt one someone else shared.

- Organic ecosystem growth. As more tools contribute facts and more users share bridging rules, the substrate becomes richer. Each new tool and rule increases the value of everything already there.

- LLM-assisted rule creation. Defining a bridging rule is a small, well-scoped task, perfect for an LLM to assist with. Describe what you want in natural language, and an agent can generate the Datalog rule.

---

## View Rules

### The Property

Dialog supports a special class of rules that conclude not facts but visual representations. A view rule associates a concept with an HTML rendering, a way to see and interact with instances of that concept.

This means data in the substrate isn't just queryable, it can have a visual manifestation. A recipe concept can have a recipe card view. A task can have a kanban tile. A contact can have a profile widget. These views are themselves facts in the substrate, shareable and composable like everything else.

### What This Unlocks

- Data with presence. Facts stop being invisible database entries and become visible, interactive objects. The substrate can render itself.

- Shareable views. Someone designs a beautiful recipe card view and shares it. Anyone with recipe facts in their substrate can now see them rendered that way.

- LLM-generated interfaces. Describe what you want to see, "show me my tasks as a timeline," and an LLM generates a view rule. The substrate already has the data; the view rule gives it shape.

- Incremental app building. Start with raw facts. Add a concept to give them structure. Add a view rule to give them a visual form. Add behavior rules to make them interactive. Each step builds on the last, and at no point do you need to write a traditional application.

---

## Privacy and Credible Exit

### The Property

Dialog syncs through commodity blob storage, S3, R2, or any object store. All data is encrypted client-side before leaving the device. The storage provider sees only opaque encrypted blobs. It cannot read your data, cannot censor specific content, and cannot build profiles from your usage.

### Pluggable Remotes

Remotes in Dialog work like git remotes. Start local. Add a remote to sync. Switch providers at will. Your data is not locked to any service, it's content-addressed blobs that can live anywhere.

This extends naturally to peer-to-peer transports. An Iroh network node, a Radicle-style remote helper, or any content-addressed transport can serve as a remote. The architecture doesn't privilege any particular transport, S3 and P2P are equally valid remotes. In theory (and this needs to be proven) the whole system could work in fully peer-to-peer mode, exchanging and replicating without any centralized infrastructure at all.

### Credible Exit

Because all data lives locally and remotes are interchangeable encrypted blob stores, there is no lock-in. You can switch storage providers, go fully local, go peer-to-peer, or run your own infrastructure. Your data is always yours, always portable, always under your control. This is a credible exit, not a theoretical one.

### What This Unlocks

- Privacy by default. Any tool that connects to the substrate inherits its privacy properties automatically. The protocol handles encryption and sync, individual tools don't need to implement their own.

- No vendor dependency. Switch from S3 to R2 to IPFS to a peer-to-peer mesh. The substrate doesn't care. Your data moves with you.

- Peer-to-peer as a natural extension. The same sync protocol that works over S3 can work over Iroh or similar P2P networks. Going fully decentralized is a configuration change, not an architectural redesign.

- Infrastructure simplicity. No sync servers. No custom protocols. No DevOps. Just encrypted blobs on commodity storage. This is particularly relevant for the new generation of creators who can vibe-code a tool in an afternoon but don't want to operate infrastructure. Their tools are guests in the user's space, they tap into the substrate and don't need to build or host anything of their own.

---

## Sync and Partial Replication

### The Property

Dialog uses probabilistic search trees, a content-addressed index structure where the same data always produces the same tree, regardless of insertion order. This means two replicas with the same facts will have identical trees, and differences between replicas can be identified by comparing tree hashes from the root down.

### Query-Driven Replication

You don't need the entire database to run a query. Dialog fetches only the tree nodes your query traverses. These nodes are cached locally, forming a partial replica that grows organically with your access patterns. You start with nothing and accumulate exactly the data you use.

### Eventual Consistency

Replicas converge deterministically. Given the same facts and the same merge strategy, any two replicas will arrive at the same state. This is eventual consistency with mathematical guarantees, not "it'll probably work out" but provably convergent.

### What This Unlocks

- No upfront replication decisions. You don't need to decide what to sync ahead of time. The substrate replicates what you access. New queries expand your local replica automatically.

- Large datasets without memory constraints. Unlike CRDTs that require full dataset in memory, Dialog can work with datasets far larger than available RAM. The index structure means you only load what's relevant.

- Bandwidth efficiency. Sync exchanges only the differences between replicas. The tree structure makes diff computation logarithmic, proportional to the number of changes, not the size of the dataset.

- Works anywhere. From a phone with limited storage to a server with terabytes, the same protocol adapts. Partial replication means every device participates at its capacity.

---

## The Synthesis

These properties don't exist in isolation. They compose. And the compositions produce emergent capabilities that no single property provides alone.

A semantic model with temporal immutability means you have a self-describing knowledge substrate with complete history. Add version control and you get exploratory branching with safe rollback. Add reactivity and the substrate becomes a live, self-updating system. Add multiplayer with signed facts and you get collaborative intelligence with provenance. Add layers and you get composable perspectives with natural privacy boundaries. Add emergent cooperation and tools that never heard of each other start working together. Add encrypted sync over commodity storage and the whole thing works without servers, without lock-in, without compromising privacy.

The result is not an application but a protocol. Infrastructure where information accretes around you, tools connect and contribute without becoming silos, agents work under your supervision, and the entire system works locally, syncs globally, and belongs to you. Your data doesn't live in Dialog any more than your code lives in git. It lives with you. Dialog is how everything else connects to it.

What gets built on this is up to the people who build it. This document describes the terrain. The paths through it are yours to find.
