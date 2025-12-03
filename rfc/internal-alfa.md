# Internal Alpha

## User Stories

### Onboarding

- Alex goes to https://alpha.tonk.xyz (or a different URL)
- Page loads with an empty "home" space
  > 1. Account keypair is generated and stored in non-extractable form.
  > 1. Account DID `did:key:zAlex` is derived from the public key. 
  > 1. Space keypair is generated
  >    1. Space delegates complete access to the account (powerline)
  >    1. Delegation is stored in the account "personal" space `did:key:zAlex`  _(hidden from the UI)_.
  >    1. Home space is added to the "personal" space address book associating
  >    `home ➔ did:key:zHome`
  > 1. Space keypair gets thrown away _(no owners could be added)_
- [history.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) updates URL to `https://alpha.tonk.xyz/did:key:zAlex/home` 
- Alex drops memo.txt file that is added to a space


### Revisiting

- Alex goes to https://alpha.tonk.xyz
- Last active space `did:key:zHome` is selected
  - [history.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) updates URL to `https://alpha.tonk.xyz/did:key:zAlex/home`
- State restores to what it last was

### Personalization 

- Alex activates personalization feature
- System prompts for the name user wants to be called
- Alex submits "Alex"
- System creates address book entry in the `did:key:zHome` space associating `@Alex ➔ did:key:zAlex`
- System updates URL using [history.replaceState](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState) to `https://alpha.tonk.xyz/@Alex/home` reflecting name.

### Inviting Collaborator

- Alex activates share function in the "home" space
- System prompt Alex to enter invitees email address
  - If Alex has not personalized yet it probably activates that to know who's inviting
- Alex submits email address for Eileen
- System produces invite URL  https://alpha.tonk.xyz/@Alex/home?join#glhAb...DK2YJQ==
  > Hash is base64 encoded invite that we used in tonk-cli
- Alex shares invite URL with Eileen in a side channel

### Accepting Invite

- Eileen gets space invite link from Alex and navigates to it
- System performs account / home space bootstrap (if no account is found)
- Alex's account info added to Eileen address book in "personal" `did:key:zEileen` space
 associating `@Alex ➔ did:key:zAlex`
- Space's info is added to to Eileen address book in "home" space associating `@alex/home ➔ did:key:zHome`
- Delegation from membership _(derived from invite)_ to Eileen account is issued and stored in the Eileen's "personal" space
- Space content is loaded utilizing delegations stored in Eileen's "personal" space.
- Eileen sees Alex's memo.txt
- Eileen adds re-memo.txt responding to memo.txt


## Considerations

1. Currently we do not have a way to go from space DID to VFS
   > We may be better of doing something different here but we do need something to sync.
   > 
   > ℹ️ I say something else because doing all the queries to resolve names to DIDs and searching for all the UCANs could use different abstractions

1. Setting up account recovery using passkey seems like a good idea, but not sure at which point in the flow we want to do it (maybe during personalization or inviting). We could probably defer it to a next sprint or treat as nice stretch goal.

1. Address book mapping could be cut to reduce scope and be introduced in the future improvements.

