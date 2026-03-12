# Carry CLI Specification

`carry` is the command-line interface for interacting with Dialog DB spaces. It provides commands for creating and joining spaces, querying and transacting data, and synchronizing with remotes.

---

## Global Options

Every command accepts `--site <SITE>` to point directly to a repository. When omitted, commands walk up the filesystem tree from `$PWD` toward `$HOME` looking for a `.carry` directory and use the first one found.

```
--site <SITE>    Path to a specific repository, skips filesystem search
--format <FMT>   Output format: yaml (default) or json
```

The preferred output format can also be persisted as a claim in the space. Command-line `--format` always takes precedence over the persisted preference.

---

## Asserted Notation

Asserted notation is the canonical YAML format carry uses for command output and file-based input. It represents both data claims and schema definitions as a hierarchy of entity → context → fields, and expands unambiguously to a set of raw claims. Because `carry query` output and `carry assert -` input share the same format, query output can be piped directly back as assert input without transformation.

### Three-level structure

Every entry in asserted notation follows a consistent three-level hierarchy:

```
<entity-identifier>:
  <context>:
    <field>: <value>
    <field>: <value>
```

#### Level 1: Entity identifier

The outermost key identifies the entity being described.

| Form | Meaning | Example |
|---|---|---|
| Contains `:` | Global identifier (a DID or URI) | `did:key:zAlice` |
| No `:` | Local bookmark name | `quantity`, `person` |

#### Level 2: Context

The second key declares how the fields beneath it should be interpreted.

| Form | Meaning | Example |
|---|---|---|
| Contains `.` | Domain context; fields expand to `domain/field` relation identifiers | `io.gozala.person` |
| No `.` | Concept context; fields are named attributes of that concept | `attribute`, `concept`, `bookmark` |

The concept names `attribute`, `concept`, `rule`, and `bookmark` are pre-registered. User-defined concept names may appear here once bookmarked.

> ⚠️ Domains starting with `dialog.` are reserved for Dialog DB internals. User-defined domains must not use this prefix.

#### Level 3: Fields

Named values within the context.

- **Scalar value**: a direct association: `name: Alice`, `age: 28`, `as: Text`
- **Non-scalar value** (a nested YAML map): implies a nested entity; see [Nested Entities](#nested-entities) below

### Domain context: data assertions

Under a domain context each field at level 3 expands to a claim `(the: domain/field, of: entity, is: value)`.

```yaml
did:key:zAlice:
  io.gozala.person:
    name: Alice
    age: 28
```

Expands to:

```yaml
- the: io.gozala.person/name
  of:  did:key:zAlice
  is:  Alice

- the: io.gozala.person/age
  of:  did:key:zAlice
  is:  28
```

Multiple top-level entries in a single document are expanded independently and submitted together as one transaction.

### Anonymous entities

`_` as the level-1 key means "a fresh entity; identity irrelevant to this document." Use when you need to assert claims without caring what the entity's identifier is:

```yaml
_:
  diy.cook:
    quantity:    2
    ingredient:  carrot
```

For cases where the same anonymous entity must be referenced in multiple places within one document, use a named variable `?foo`. All occurrences of `?foo` in the same document bind to the same generated entity. `_` is always a distinct fresh entity. `?foo` unifies.

### Concept context: schema definitions

Under a concept context the pre-registered concept schema determines how fields are interpreted and how the entity identity is computed.

#### `attribute`

Two attributes with the same relation identifier but different type or cardinality are distinct entities.

```yaml
quantity:
  attribute:
    description: Amount needed
    the:         diy.cook/quantity
    as:          UnsignedInteger
    cardinality: one
```

The local name `quantity` at level 1 creates a `dialog.meta/name` claim on the attribute entity.

#### `concept`

A concept's identity is derived from its `dialog.concept.with/*` claims: the set of `(field_name, attribute_entity)` pairs. Field names participate in identity: a concept with a field named `name` pointing at attribute `A` is distinct from one with a field named `fullname` pointing at the same `A`.

Concept definitions reference named attribute entities under `with`. Attributes can be defined separately and referenced by bookmark, or defined inline:

**Separate attribute definitions:**

```yaml
person-name:
  attribute:
    description: The person's name
    the:         io.gozala.person/name
    as:          Text
    cardinality: one

person-age:
  attribute:
    description: The person's age
    the:         io.gozala.person/age
    as:          UnsignedInteger
    cardinality: one

person:
  concept:
    description: A person
    with:
      name: person-name
      age:  person-age
```

**Inline attribute definitions:**

```yaml
person:
  concept:
    description: A person
    with:
      name:
        description: The person's name
        the:         io.gozala.person/name
        as:          Text
        cardinality: one
      age:
        description: The person's age
        the:         io.gozala.person/age
        as:          UnsignedInteger
        cardinality: one
```

Both produce the same claims:

```yaml
- the: dialog.concept.with/name
  of:  <person>
  is:  <person-name>

- the: dialog.concept.with/age
  of:  <person>
  is:  <person-age>

- the: dialog.meta/description
  of:  <person>
  is:  A person

- the: dialog.meta/name
  of:  <person>
  is:  person
```

Optional fields declared under `maybe` produce `dialog.concept.maybe/{name}` claims. An entity still matches the concept if all `with` fields are satisfied regardless of which `maybe` fields are present.

#### `bookmark`

A bookmark maps a local name to any entity. The `name` field is stored; `this` is provided at the command level to identify which entity receives the name.

```
carry assert bookmark this=did:key:zSomeEntity name=color
```

Produces:

```yaml
- the: dialog.meta/name
  of:  did:key:zSomeEntity
  is:  color
```

Names are shared across all members of a space and travel with synced data.

### Nested entities

A non-scalar value at level 3 under a **domain context** implies a nested entity. The nested entity's identity is derived deterministically from its parent entity and the field's relation identifier. The nested entity's domain is the parent domain with the field name appended as a segment, e.g. `address` under `io.gozala.person` produces `io.gozala.address`.

```yaml
did:key:zAlice:
  io.gozala.person:
    name: Alice
    address:
      city: San Francisco
      zip:  94107
```

Expands to:

```yaml
- the: io.gozala.person/name
  of:  did:key:zAlice
  is:  Alice

- the: io.gozala.person/address
  of:  did:key:zAlice
  is:  <address-entity>

- the: io.gozala.address/city
  of:  <address-entity>
  is:  San Francisco

- the: io.gozala.address/zip
  of:  <address-entity>
  is:  94107
```

Nesting can be recursive; each level applies the same rule.

### Entity identity for new data assertions

When a command creates a new data entity without an explicit `this`, the runtime generates a fresh entity DID. This DID is included in command output so it can be referenced in subsequent assertions.

### Round-trip property

Because `carry query` output is asserted notation and `carry assert -` accepts asserted notation, the following is always valid:

```
carry query person name="Alice" | carry assert -
```

Output for definitions queried via `carry query attribute` or `carry query concept` uses the same three-level structure and can be piped back unchanged or edited in between.

---

## Init

```
carry init [<n>] [--site <SITE>]
```

`--site` defaults to `$PWD`. Creates a new Dialog DB repository at `$SITE/.carry/did:key:zSpace`.

**If a repository already exists at that location:**

If `<n>` is provided, asserts it as the space label and prints:

```
Initialized <n> repository in /path/to/.carry/did:key:zSpace
```

Otherwise prints:

```
Initialized repository in /path/to/.carry/did:key:zSpace
```

**If no repository exists:**

1. Generates an Ed25519 keypair
2. Creates the directory `$SITE/.carry/did:key:zSpace` where `did:key:zSpace` is derived from the public key
3. Saves the private key to `$SITE/.carry/did:key:zSpace/credentials`
4. If `<n>` is provided, asserts it as the space label:

```yaml
did:key:zSpace:
  xyz.tonk.carry:
    label: <n>
```

5. Prints the same message as above

Running `carry init` inside a directory that is already within an existing repository creates a nested repository. carry does not detect or warn about nesting.

---

## Invite

```
carry invite [<member>] [--site <SITE>]
```

Generates an invite URL granting a member access to the space. Prints the URL to stdout.

`<member>` is a `did:key` identifier for the invitee. If omitted, carry generates a fresh Ed25519 keypair and uses its public key as the member DID.

**Steps:**

1. If `<member>` is not provided, generate a fresh Ed25519 keypair and use its public key as the member DID
2. Create a UCAN delegation from the repository keypair to the member DID
3. Assert the invitation:

```yaml
<member-did>:
  xyz.tonk.carry:
    invite: <ucan-cid>

<ucan-cid>:
  xyz.tonk.carry:
    ucan: <serialized-ucan-bytes>
```

4. Output the invite URL:

```
https://tonk.xyz/join?access=<ucan>#<private-key>
```

The `access` query parameter contains the UCAN delegation. The `#` fragment contains the generated private key and is never sent to the server. If `<member>` was provided by the caller the URL has no fragment, since the private key is already known to the recipient.

---

## Join

```
carry join [<invite-url>] [--site <SITE>]
```

Configures an upstream remote for the space. Errors if an upstream is already configured.

**If `<invite-url>` is provided:**

1. Extract the UCAN from the `access` query parameter
2. If the URL has a `#` fragment, extract the member private key from it. Redelegate the UCAN from the member keypair to the local repository credential and use the redelegated UCAN as the upstream credential
3. If the URL has no fragment, verify that the UCAN delegation audience (`aud`) matches the local repository DID. If it does not match, print an error and exit:

```
Cannot join: this invite was issued to <aud> but this repository is <local-did>
```

Otherwise use the local repository credential directly
4. Configure the remote using the resolved credential

**If `<invite-url>` is omitted:**

Self-provisions an upstream for the space.

---

## Pull

```
carry pull [--site <SITE>]
```

Pulls changes from the configured upstream into the local space. If no upstream is configured prints:

```
No upstream configured. Run `carry join` first.
```

---

## Push

```
carry push [--site <SITE>]
```

Pushes local changes to the configured upstream. If no upstream is configured prints:

```
No upstream configured. Run `carry join` first.
```

---

## Query

```
carry query <TARGET> [FIELD[=VALUE] ...] [--site <SITE>] [--format <FMT>]
```

Output is in asserted notation.

The `<TARGET>` determines the kind of query:

- **Contains `.`**: domain query. Searches for entities that have claims within the given domain matching any specified fields.
- **No `.`**: concept query. Resolves the named concept via local bookmark; returns all fields the concept defines.

Fields without a value (e.g., `age`) are output fields: included in results without filtering. Fields with a value (e.g., `name="Alice"`) are filter fields: only entities matching that value are returned. Output contains exactly the fields requested, nothing more.

### Domain query

Searches for entities with claims within the specified domain. You choose which fields to request.

```
carry query io.gozala.person name="Alice" age address
```

`name="Alice"` filters results. `age` and `address` are included in output.

Output:

```yaml
did:key:zAlice:
  io.gozala.person:
    name:    Alice
    age:     28
    address: San Francisco

did:key:zAli:
  io.gozala.person:
    name:    Alice
    age:     42
    address: Paris
```

### Concept query

Resolves the named concept via local bookmark, matches entities against it, and returns all of the concept's fields. Specify fields only when filtering.

```
carry query person name="Alice"
```

Output:

```yaml
did:key:zAlice:
  person:
    name:    Alice
    age:     28
    address: San Francisco

did:key:zAli:
  person:
    name:    Alice
    age:     42
    address: Paris
```

The level-2 key is the concept's local bookmark name. This makes the output readable and round-trippable: piping it back into `carry assert -` reasserts the data via the same concept context.

### Composition

`+` combines two query segments. By default both segments join on the same entity; results are merged under the same level-1 key.

```
carry query io.gozala.person name="Alice" + io.gozala.user email
```

Output:

```yaml
did:key:zAlice:
  io.gozala.person:
    name: Alice
  io.gozala.user:
    email: alice@example.com
```

---

## Assert

```
carry assert <TARGET>|<FILE>|- [this=<ENTITY>] [FIELD=VALUE ...] [--site <SITE>]
```

Asserts claims. `<TARGET>` follows the same domain vs. concept resolution as `query`.

Without `this`: a new entity DID is generated by the runtime. The generated DID is printed to stdout so it can be referenced in subsequent commands.

With `this`: targets the specified entity. At least one field is required.

`?this` refers to the entity being asserted in the current `+` segment and can be used to join across segments in the same command.

Examples:

```
carry assert person name=Alice age=28
carry assert person this=did:key:zAlice age=29
carry assert io.gozala.person name=Alice age=28
```

---

## Retract

```
carry retract <TARGET>|<FILE>|- [this=<ENTITY>] [FIELD[=VALUE] ...] [--site <SITE>]
```

Retracts claims. Same syntax as `assert`.

When a field is specified without a value, the current claim for that attribute is retracted regardless of value. When a field is specified with a value (e.g., `tag=urgent`), only the claim matching that exact value is retracted; useful for `cardinality: many` attributes.

Example:

```
carry retract person this=did:key:zAlice age
```

Retracts only the `age` claim. Other claims on the entity are unaffected.

---

## Domain Modeling

`attribute` and `concept` are pre-registered concepts in `carry`. They can be asserted and queried like any other concept.

Asserting an attribute via command line:

```
carry assert attribute the=io.gozala.person/name as=Text cardinality=one description="Name of the person"
carry assert attribute the=diy.cook/quantity as=UnsignedInteger cardinality=one description="Quantity as a whole number"
```

For non-trivial definitions pass a file or pipe via stdin:

```
carry assert domain.yaml          # from file
carry assert -                    # from stdin
cat domain.yaml | carry assert -  # same
```

See the appendix for how attributes, concepts, and bookmarks compose at the claim level.

---

## File and Stdin

`assert` and `retract` accept a file path or `-` for stdin.

> ℹ️ Using `-` to mean stdin is a Unix convention followed by tools like `cat`, `curl`, `jq`, and `diff`. It signals "read from standard input rather than a file." See [POSIX utility conventions](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html) and the [GNU Coding Standards](https://www.gnu.org/prep/standards/html_node/Command_002dLine-Interfaces.html) for background.

```
carry assert data.yaml
carry assert -
cat data.yaml | carry assert -
carry retract data.yaml
carry retract -
```

### Disambiguating file from target

- If the argument is `-` it is stdin
- If the argument contains `/` or ends in `.yaml`, `.yml`, or `.json` it is treated as a file path
- Otherwise it is treated as a `<TARGET>`

### Supported formats

Asserted YAML is the canonical format. JSON is also supported as an exact structural equivalent — the same three-level entity-first hierarchy, expressed in JSON syntax. The two representations are interchangeable.

The person example in JSON:

```json
{
  "did:key:zAlice": {
    "io.gozala.person": {
      "name": "Alice",
      "age": 28
    }
  }
}
```

### Format detection

For files, format is inferred from the extension: `.yaml` or `.yml` for YAML, `.json` for JSON.

For stdin (`-`), carry peeks at the first non-whitespace character: `{` or `[` indicates JSON, anything else is treated as YAML.

---

## Settings

Settings are persisted as claims in the space under the `xyz.tonk.carry` domain and can be asserted like any other data.

### Output format

Default output is YAML. Pass `--format json` for JSON output per-invocation.

To persist the preferred format:

```
carry assert xyz.tonk.carry output-format=json
```

---

## Future Extensions

Non-entity joins, where `this` on one side is a field value on the other, are not currently supported. For now, cross-entity queries require separate commands.

When supported, they might look like:

```
carry query io.gozala.person this=?a name="Alice" + io.gozala.org this=?b owner=?a
```

Where `?a` and `?b` are distinct entity variables, joined on `owner=?a`. Output would show both entities inlined per result:

```yaml
did:key:zAlice:
  io.gozala.person:
    name: Alice
did:key:zAcmeCorp:
  io.gozala.org:
    owner: did:key:zAlice
```

---

## Appendix: Concepts, Attributes, and Bookmarks at the Claim Level

This section documents how carry represents schema constructs as raw claims. Understanding it is not required for using the CLI, but is necessary for implementing the assert/retract/query pipeline.

### Primitive domains

The following domains are primitive; their semantics are defined by the runtime, not by concept definitions stored as claims. All user-defined concepts and attributes build on top of these.

> ⚠️ All domains starting with `dialog.` are reserved. The runtime may reject assertions that use reserved domains outside of the contexts defined here.

| Domain | Purpose |
|---|---|
| `dialog.attribute` | Stores attribute identity fields |
| `dialog.concept.with` | Stores required concept membership by field name |
| `dialog.concept.maybe` | Stores optional concept membership by field name |
| `dialog.meta` | Universal metadata: names and descriptions for any entity |

### Attribute claims

Two attributes with the same relation identifier but different type or cardinality are distinct entities. The `description` field is required in the notation but does not affect identity.

Claims for the `quantity` attribute:

```yaml
- the: dialog.attribute/id
  of:  <quantity>
  is:  diy.cook/quantity

- the: dialog.attribute/type
  of:  <quantity>
  is:  UnsignedInteger

- the: dialog.attribute/cardinality
  of:  <quantity>
  is:  one

- the: dialog.meta/description
  of:  <quantity>
  is:  Amount needed

- the: dialog.meta/name
  of:  <quantity>
  is:  quantity
```

### Concept claims

A concept's identity is derived from its complete set of `dialog.concept.with/{name}` claims. Both the field names and the attribute entities they point at participate. Two concepts with identical constituent attributes but different field names are distinct concepts.

Claims for the `person` concept with fields `name` and `age`:

```yaml
- the: dialog.concept.with/name
  of:  <person>
  is:  <person-name>

- the: dialog.concept.with/age
  of:  <person>
  is:  <person-age>

- the: dialog.meta/description
  of:  <person>
  is:  A person

- the: dialog.meta/name
  of:  <person>
  is:  person
```

Optional fields produce `dialog.concept.maybe/{name}` claims. These do not participate in concept identity. An entity satisfies the concept if all `with` fields are present, regardless of which `maybe` fields are present.

### Pre-registered concept schemas

The builtin concepts are self-describing, expressible in the same notation they are used to define. They are hardcoded in the runtime.

#### `attribute`

```yaml
attribute:
  concept:
    description: Built-in concept for modeling attributes
    with:
      description:
        description: Human-readable description, required to aid comprehension
        the:         dialog.meta/description
        as:          Text
        cardinality: one
      the:
        description: Nominal identifier capturing semantic intent of the relation
        the:         dialog.attribute/id
        as:          Symbol
        cardinality: one
      as:
        description: Value type of the attribute
        the:         dialog.attribute/type
        as:          [Text, Boolean, SignedInteger, UnsignedInteger, Float, Symbol, Bytes, Entity]
        cardinality: one
      cardinality:
        description: Cardinality of this relation
        the:         dialog.attribute/cardinality
        as:          [one, many]
        cardinality: one
```

The `as` and `cardinality` fields use enumerated symbol types; the value must be one of the listed symbols.

#### `concept`

```yaml
concept:
  concept:
    description: Built-in concept for composing attributes into a shape
    with:
      description:
        description: Human-readable description of what this concept models
        the:         dialog.meta/description
        as:          Text
        cardinality: one
      with:
        ?name:
          description: A required attribute, keyed by its field name in this concept
          the:         dialog.concept.with/?name
          as:          attribute
          cardinality: one
    maybe:
      maybe:
        ?name:
          description: An optional attribute, keyed by its field name in this concept
          the:         dialog.concept.maybe/?name
          as:          attribute
          cardinality: one
```

`?name` as a key signals a variable name segment; the key used in the document becomes the name segment in the relation identifier. A `?name` key may not appear alongside literal keys in the same block.

A `with` dictionary is non-empty by definition: if no entries are present the entity does not match the concept. A `maybe` dictionary may be absent entirely, which is why `maybe` itself lives under `maybe`.

#### `bookmark`

```yaml
bookmark:
  concept:
    description: Naming mechanism mapping a local name to any entity
    with:
      name:
        description: The name assigned to the target entity in this space
        the:         dialog.meta/name
        as:          Text
        cardinality: one
```

`this` is provided at the command level and determines which entity receives the name. It is not a stored field of the `bookmark` concept. Names are shared across all members of a space and travel with synced data.
