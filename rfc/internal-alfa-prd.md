
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
  - This would make him enter the previos "Create Account" flow
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

## 4. Technical Notes - please spec this out!

*Template*

* Key architectural decisions (storage model, sync behavior, identity model)
* Interfaces/APIs that must exist?
* Reuse of existing components (Tinki)?
* Temporary hacks?

## 5. Risks & Unknowns

### On UX side

- UI needs to flag that data storage is ephemeral until account creation
- Unclear if account creation prompt should come from Tinki or Launcher, related to [RFC Tabs]().
- Unclear if there should be a special Home space, related to [RFC Spaces](https://www.notion.so/tonk/RFC-Spaces-2b719ce5e10b800783a5d9f5adb2d8ed?source=copy_link)

### On Eng side - please add!



## 8. Success Criteria

* 2 internal users can complete the core flow
* Time-to-Tonk (TTT) ≤ 2 minutes

