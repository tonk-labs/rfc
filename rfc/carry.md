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

## Seed

```
carry seed [<name>] [--site <SITE>]
```

`--site` defaults to `$PWD`. Creates a new Dialog DB repository at `$SITE/.carry/did:key:zSpace`.

**If a repository already exists at that location:**

If `<name>` is provided, asserts it as the space label and prints:

```
Seeded <name> repository in /path/to/.carry/did:key:zSpace
```

Otherwise prints:

```
Seeded repository in /path/to/.carry/did:key:zSpace
```

**If no repository exists:**

1. Generates an Ed25519 keypair
2. Creates the directory `$SITE/.carry/did:key:zSpace` where `did:key:zSpace` is derived from the public key
3. Saves the private key to `$SITE/.carry/did:key:zSpace/credentials`
4. If `<name>` is provided, asserts it as the space label:

```yaml
the: xyz.tonk.carry/label
of: hash("space")
is: <name>
```

5. Prints the same message as above

Running `carry seed` inside a directory that is already within an existing repository creates a nested repository. carry does not detect or warn about nesting.

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
3. Assert the invitation as claims:

```yaml
the: xyz.tonk.carry/invite
of: <member-did>
is: <ucan-cid>

the: xyz.tonk.carry/ucan
of: <ucan-cid>
is: <serialized-ucan-bytes>
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

The `<TARGET>` determines the kind of query:

- **Contains `.`**: domain query. Open-ended: specify any field names within the domain and carry will search for entities that have matching claims. You decide what to look for.
- **No `.`**: concept query. The concept defines the fields: you get all of them without having to list them. Only specify fields when you want to filter.

Fields without a value (e.g. `age`) are shorthand for `age=?`: include the field in output without filtering. Fields with a value (e.g. `name="Alice"`) filter results to matching entities. Output contains exactly the fields requested, nothing more.

---

### Domain

An open-ended query over a domain. Searches for any entities that have claims within the given domain matching the fields specified. Not constrained to a predefined set of fields.

```
carry query io.gozala.person name="Alice" age address
```

Translates to: find entities where `the=io.gozala.person/name is="Alice"`, returning `io.gozala.person/age` and `io.gozala.person/address`.

Output:

```yaml
did:key:zAlice:
  io.gozala.person:
    name: Alice
    age: 28
    address: San Francisco
did:key:zAli:
  io.gozala.person:
    name: Alice
    age: 42
    address: Paris
```

---

### Concept

Resolves the named concept via local bookmark and matches entities against it. All fields the concept defines are returned. Specify fields only when filtering.

```
carry query person name="Alice"
```

Resolves the bookmark `person` to a concept, matches all entities satisfying the concept's required attributes, and filters to those where `name` is `"Alice"`.

Output:

```yaml
did:key:zAlice:
  person:
    name: Alice
    age: 28
    address: San Francisco
did:key:zAli:
  person:
    name: Alice
    age: 42
    address: Paris
```

---

### Composition

`+` combines two query segments in the same command. By default both segments join on the same entity. Results are inlined under the same entity key.

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

Without `this`: a new entity is derived from the provided fields by the Dialog DB runtime.

With `this`: updates the specified entity. At least one field required.

`?this` refers to the entity being asserted in the current segment and can be used to join across `+` segments in the same command.

Examples:

```
carry assert person name=Alice age=28
carry assert person this=did:key:zAlice age=29
carry assert io.gozala.person name=Alice age=28
```

---

## Retract

```
carry retract <TARGET>|<FILE>|- [this=<ENTITY>] [FIELD=VALUE ...] [--site <SITE>]
```

Retracts claims. Same syntax as `assert`. Each listed field retracts the corresponding claim from the entity.

Example:

```
carry retract person this=did:key:zAlice age
```

Retracts only the `age` claim. Other claims on the entity are unaffected.

---

## Model

```
carry model [<scope>] [--site <SITE>]
```

Opens the default editor with existing definitions rendered in abbreviated YAML notation as per the [Dialog notation spec](https://github.com/dialog-db/dialog-db/blob/main/notes/notation.md). When the editor is closed carry computes a diff against the original and asserts any changes.

`<scope>` determines what is loaded into the editor:

- **Omitted**: all concepts defined in the space
- **No `.`**: the named concept and its constituent attributes
- **Contains `.`**: all attributes defined in that domain

**Editor resolution** follows the same order as git: `$CARRY_EDITOR`, then `$VISUAL`, then `$EDITOR`, then falls back to `vi`.

**On close:**

- Added definitions are asserted
- Removed definitions are retracted
- Changed definitions are asserted in their new form. Since attribute and concept identity is structural, a changed definition produces a new entity — the old one is not explicitly retracted but simply stops being referenced

**New definitions** added in the editor are treated the same as edits and asserted on close.

---

## Domain Modeling

`attribute` and `concept` are pre-registered concepts in `carry`. They can be asserted and queried like any other concept.

Asserting an attribute:

```
carry assert attribute the=io.gozala.person/name as=Text
carry assert attribute the=diy.cook/quantity as=UnsignedInteger description="Quantity as a whole number"
```

For non-trivial definitions pass a file or pipe via stdin. YAML is the canonical format; JSON is also accepted and detected automatically:

```
carry assert domain.yaml          # from file
carry assert -                    # from stdin
cat domain.yaml | carry assert -  # same
```

See the appendix for how attributes, concepts, and bookmarks compose at the claim level.

---

## File and Stdin

`assert` and `retract` accept a file path or `-` for stdin. YAML is the canonical format; JSON is also accepted and detected automatically.

```
carry assert data.yaml            # from file
carry assert -                    # from stdin
cat data.yaml | carry assert -    # same
carry retract data.yaml
carry retract -
```

### Disambiguating file from target

When the first argument to `assert` or `retract` could be either a `<TARGET>` or a file path, carry resolves it as follows:

- If the argument is `-` it is stdin
- If the argument contains `/` or ends in `.yaml`, `.yml`, or `.json` it is treated as a file path
- Otherwise it is treated as a `<TARGET>`

### Supported formats

Three formats are accepted, all of which expand to the same claims before asserting:

- **Abbreviated YAML** — the shorthand notation described in the [Dialog notation spec](https://github.com/dialog-db/dialog-db/blob/main/notes/notation.md). Recommended for human authoring.
- **Formal YAML** — the explicit full form from the same spec.
- **Formal JSON** — the JSON equivalent of the formal notation.

### Format detection

For files, format is inferred from the file extension: `.yaml` or `.yml` for YAML, `.json` for JSON.

For stdin (`-`), carry peeks at the first non-whitespace character: `{` indicates JSON, anything else is treated as YAML. Both abbreviated and formal YAML are accepted — carry distinguishes them by whether the content matches the formal schema.

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

Where `?a` and `?b` are distinct entities, joined on `owner=?a`. Output would show both entities inlined per result:

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

This section describes how carry represents domain modeling constructs as claims. It is not required reading for using the CLI but explains how the pieces fit together.

### Attribute identity

An attribute's entity is a content hash of its selector, cardinality, and type:

```
entity = hash(the, cardinality, as)
```

Two attributes with the same selector but different cardinality or type are distinct entities.

### Concept identity

A concept is a set of attributes. Its entity is a content hash of the sorted set of constituent attribute hashes. Field names are not part of the concept's identity: two concepts with the same attributes but different names are the same concept.

Membership is recorded as individual claims, one per attribute:

```yaml
the: claims.dialog.concept/with
of: concept_entity
is: attribute_entity
```

Optional attributes use a separate relation:

```yaml
the: claims.dialog.concept/maybe
of: concept_entity
is: attribute_entity
```

### Bookmarks

`bookmark` is carry's naming mechanism. It maps a local name to any entity via `xyz.tonk.carry/label`. Bookmarks are local metadata: they do not affect structural identity and do not travel with concepts on sync.

```
carry query bookmark              # list all named entities
carry query bookmark name=person  # resolve the name person
carry assert bookmark name=person this=<entity>
```

Field-level bookmarks name individual attribute slots within a concept. The target entity is the content hash of the concept-attribute pair:

```yaml
the: xyz.tonk.carry/label
of: hash(concept_entity, attribute_entity)
is: name
```

When no field bookmark exists the field name falls back to the name segment of the attribute selector. For example `io.gozala.person/name` defaults to `name`. Field bookmarks are only needed when the default is insufficient or when the same name segment appears under multiple domains in the same concept.

Asserting a concept and naming it in one command using `+` and `?this`:

```
carry assert concept with=io.gozala.person/name,io.gozala.person/age,io.gozala.person/address
  + bookmark name=person
  + bookmark this=?this,io.gozala.person/name name=name
  + bookmark this=?this,io.gozala.person/age name=age
  + bookmark this=?this,io.gozala.person/address name=address
```

`?this` binds to the entity being asserted in the current `+` segment. `?this,io.gozala.person/name` computes `hash(concept_entity, attribute_entity)` as the target for the field bookmark. Field-level bookmarks are optional and only needed when the default name segment is not sufficient.
