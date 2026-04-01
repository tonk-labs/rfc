# Overview

Cooperating system needs to track some state that is local and does not belong in any repository or a user space.

We could leverage [memory](https://github.com/dialog-db/dialog-db/blob/embed/rust/dialog-effects/src/memory.rs) capabilities with a `did:web:tonk.localhost` subject to manage such information.


## Session

Tonk co-operating system is operated under some session that has associated `operator`. Session information is persistend in
`user` memory space under `session` cell address in a following record representing an operator acting on behalf of a user account.

```rs
struct Session {
    credentials: SessionCredentials,
    authorization: Option<SessionAuthorization>
}
```

Authorization is optional, and if not present implies a pre-authorization state where system offers limited, local only capabilities.

If record does not exists system is in uninitialized state. Any interactio with a system will need to initialize a `session` by generating `opertaor` keypair for the session and obtaining authorization from the user.

### Credentials

Operator credentials are persisted by the system and are used to acquire access to spaces shared through authorization session.

```rs
enum SessionCredentials {
  Ed25519([u8; 32])
}
```

> ℹ️ At the moment of writing only credentials supported are in form of `ed25519` private key material, but in the near future that should be extended to non-extractable private [`CryptoKey`](https://developer.mozilla.org/en-US/docs/Web/API/CryptoKey) as primary variant on the web.

### Authorization

Operator acting on behalf of user SHOULD have `SessionAuthorization` represented via [powerline](https://github.com/ucan-wg/delegation?tab=readme-ov-file#powerline)
UCAN delegation granting session `operator` access to everything within delegating authority.

> ℹ️ In majority of cases authorizing authority will be user account providing seamless access to all the spaces account has across different sessions. However it is worth calling out that it is possible and RECOMMENDED to excercise _The Principle of Least Authority (PoLA)_ and only grant access to subset of resources which can be accomplished by introducing intermidiery authority between user account and session operator and delegate only access to desired subset of resources to it.

It is RECOMMENDED to impose `expiry` on issued authorization to reduce potential damage that can be inflicted in case of compromised keys.

#### Pre Authorized State

> Requiring authorization as a first interaction puts carriage before the horse. It is a lot more natural to perform authorization during transition from local to shared as in it is natural to authorize publishing of some content so it can be attributed to the author, where's requiring authorization to to even enter the system creates disinviting feel.

When system initializes new session it is in **pre-authorized** state. In this state session operator has access to only local replicas and is unable pull parts that have not yet being replicated.

It is still possible to support new space creation and fact assertion in this state. In such case operator acts und own authority, that is to say that "authorization space" has same DID as the operator itself and it could be operated unde self signed authority.

Once authorization is established access to all new spaces can be redelegated to the "authority" by issuing powerline delegation from operator to that authority creating delegation loop. This would grand all the other operators acting under same authority access to the spaces that were created in pre-authorized state.

### API

Interface to obtain an operator can be somewhat along the following lines:

```rs
let mut env = IndexedDb::new();
let operator = Operator::open(&mut env).await?
```

> ℹ️ `env` here must be a provider of memory capabilities allowing session to be loaded or initialized.


If system has no `session` record it will initialize new session by generating a `ed25519` keypair and store `Session` record in memory so it can be loaded next time around.


## Access

Session operator when authorized acts on behalf of authorizing authority and can access all resources that were granted to that authority.

> ℹ️ It helps to think in terfs of role-based access control (RBAC)[https://en.wikipedia.org/wiki/Role-based_access_control] where authorizing authority corresponds to a specific role and operators are asigned that role through a [powerline] delegation.

Operator can discover what spaces it has access to by querying "authoziation space" for stored UCAN delegations and discover delegation chains that grant it access to `/storage/get` capabiliyt under `catalog: "index"` policy.

> ℹ️ This represents minimal capability set that would enable operator to query and replicat space. We could disambiguate between read / write access by querying for additional capabilitise like `/storage/put` for write access and `/memory/publish` for publishing capabilities.

Rough [sketch of this idea is available](https://github.com/dialog-db/dialog-db/blob/00694c3fbe34ec563740d72a854cc5f453251c7a/rust/dialog-artifacts/src/capability/session.rs) to draw inspiration from.

It is expected above design will enable [`Access`](https://github.com/dialog-db/dialog-db/blob/3076317dca1e74b4c0d1f21e1a7f6d434aba45fa/rust/dialog-capability/src/access.rs) control implemenation for the `Session`.

### Access Spaces

Outlined mechanism enables operator to discover and list spaces accessible by it under active session via convenient API like

```rs
let spaces = operator.spaces(&env).await?;
```

Each space can be mapped to a local `Repository` (a.k.a `Replica`) and provisioned with an _upstream_ by acquiring a delegation chain from space to a session operator.

### Create Space

Operator could create new spaces by generating (or deriving) unique Ed25519 keypair. This could be offered through convenient API:

```rs
let space = operator.create_space(&mut env).await?
```

Task consists of following steps:

1. Generating (or deriving) unique keypair for the space.
2. Issuing powerline delegation from space keypair to an authority operator acts in behalf of.
3. Storing that delegation in the "authorization space" propagates access to all active sessions under the same authority.
4. Opening Repository and setting up it's upstream.
