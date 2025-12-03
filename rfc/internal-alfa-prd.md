## 1. Purpose

We want to ship a first internal alpha to get the team "working on the same thing." We intend to use it to dogfood and iteratively improve the product, so we can eventually bring it into beta. It will also allow us to test with users.

This is *not* an internal demo, nor is it a complete reflection of *all* current product decisions. 

## 2. User Stories (Happy Path Only)

* Two users, Alex and Eileen
* Alex would like to share a memo over the Tonk Launcher for Eileen to collaborate on

### 2.1 Onboarding

- Alex goes to https://alpha.tonk.xyz which shows an empty space called "Starter Space"
- Alex has the option to rename the Starter Space
- Alex drops memo.txt file that is added to a space

### 2.2 Open Memo

- Alex opens memo as a Tinki (within the browser tab)
- Alex can edit his memo in Tinki
- The changes are saved
- Alex closes Tinki and returns to Starter Space

### 2.3 Revisiting

- Alex closes browser (without clearing cache)
- Alex reopens same browser and goes to https://alpha.tonk.xyz
- Alex sees the last state of his Starter Space

### 2.4 Create Account 

- Alex clicks on account, which is yet undefined
- Alex adds his display name
- Alex has the option to add a profile picture
- Alex saves his entry, and is prompted to add a passkey ("Add passkey to create account" instead of "Save")
- Alex's display name and profile picture are immediately shown

### 2.5 Inviting Collaborator

- Alex copies an invitation link to the Starter Space
  - If Alex has not yet added an account he will get prompted ("Create an account to share this Space")
  - This would make him enter the previous "Create Account" flow
- Alex shares invite URL with Eileen in a side channel

### 2.6 Accepting Invite

- Eileen gets Space invite link from Alex and navigates to it
- Eileen sees Alex's Starter Space and can open his memo

### 2.7 Collaborate on Memo

- Upon editing, Tinki prompts Eileen to create an account
- Eileen creates an account
- Eileen can now edit the memo
- The changes are saved
- Eileen closes the memo and returns to Starter Space, which now shows her display name and profile picture
- Alex opens the memo again and sees Eileen's changes


## 3. Scope

## In-scope

* Launcher opens tonk files
* User accounts with passkeys as auth
* Share links
* Data sync

### Out-of-scope

* Synchronous edits: Users can see each other's presence and real-time edits
* Share links expiry: Links expire after a number of uses
* Account recovery: Prompt users to add an email account after three active hours of usage
* Threat modelling: Permissions for downloads (individual tonks, a whole Space)

## 4. Technical Notes

### Spaces

- Each space is a Tonk bundle that contains a manifest with `"type": "space"` and `"spaceId": "did:key:zSpace"`
- "Personal" space is a space like any other, but hidden from UI and only contains user's private information
  - For the alpha, a user cannot extend beyond one browser so the personal space will have no network configs
  - Staying completely local is our short-term solution to keeping the data in personal spaces private to the respective users
  - In the future, we'll need a better understanding of mappings between users and VFS' to solve this problem
- Service worker will have one Tonk Core instance with a root doc ID that can be hot-swapped as users toggle between different spaces
  - Maintaining a single initialized Tonk Core instance should reduce the overhead for quickly toggling between spaces
  > The ability to hot-swap root doc IDs without instantiating a new Tonk Core instance will need to be implemented

### Auth

#### User Onboarding

1. Ephemeral ed25519 keypair generated and stored non-extractable
   > Banner shown: "Your data is temporary. Create an account to keep it."
1. Ephemeral account DID `did:key:zAlex` is derived from the public key. 
1. Space keypair is generated
1. Space delegates complete access to the account (powerline)
1. Delegation is stored in the account "personal" space `did:key:zAlex`  _(hidden from the UI)_.
1. Home space is added to the "personal" space address book associating `home ➔ did:key:zHome`
1. Space keypair gets thrown away _(no owners could be added)_
1. [history.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) updates URL to `https://alpha.tonk.xyz/home` 

#### Create Account

1. User clicks "Create Account"
1. System creates WebAuthn credential with PRF
1. Derive Authority from PRF
1. Store credential ID
1. Migrate ephemeral data (if any) to Authority
1. Derive default profile
1. Update spaces to delegate to new profile
1. Clean up ephemeral data

#### Revisiting

1. Alex goes to https://alpha.tonk.xyz
1. Attempt silent auth, otherwise prompt for passkey
1. Re-derive Authority from PRF, verify DID matches
1. Derive active profile and restore last active space
1. Last active space `did:key:zHome` is selected
1. [history.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) updates URL to `https://alpha.tonk.xyz/home`
1. State restores to what it last was

#### Personalization

1. Alex activates personalization feature
1. System prompts for the name user wants to be called
1. Alex submits "Alex"
1. System creates address book entry in the `did:key:zHome` space associating `@Alex ➔ did:key:zAlex`
1. System updates URL using [history.replaceState](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState) to `https://alpha.tonk.xyz/home` reflecting name.

#### Inviting Collaborator

1. Alex activates share function in the "home" space
1. System checks if Alex has passkey, prompts for one if not
1. System prompts Alex to enter invitee's email address
   > If Alex has not personalized yet it probably activates that to know who's inviting
1. Alex submits email address for Eileen
1. System produces invite URL  https://alpha.tonk.xyz/@Alex/home?join#glhAb...DK2YJQ==
   > Hash is base64 encoded invite that we used in tonk-cli
1. Alex shares invite URL with Eileen in a side channel

#### Accepting Invite

1. Eileen gets space invite link from Alex and navigates to it
1. System performs account/home space bootstrap (if no account is found)
1. Alex's account info added to Eileen address book in "personal" `did:key:zEileen` space
1.associating `@Alex ➔ did:key:zAlex`
1. Delegation from membership _(derived from invite)_ to Eileen account is issued and stored in the Eileen's "personal" space
1. Space content is loaded using delegations stored in Eileen's "personal" space
1. Eileen sees Alex's memo.txt
1. Eileen attempts to edit memo.txt, system prompts for passkey to edit

### Passkeys

Before passkey, UI shows toast with "Your data is temporary until you create an account".

When user clicks "Create Account" or tries to share:
1. Generate passkey using WebAuthn
1. Derive encryption 

## 5. Risks & Unknowns

### On UX side

- UI needs to flag that data storage is ephemeral until account creation
- Unclear if account creation prompt should come from Tinki or Launcher, related to [RFC Tabs]().
- Unclear if there should be a special Home space, related to [RFC Spaces](https://www.notion.so/tonk/RFC-Spaces-2b719ce5e10b800783a5d9f5adb2d8ed?source=copy_link)

## 6. Success Criteria

* 2 internal users can complete the core flow
* Time-to-Tonk (TTT) ≤ 2 minutes

