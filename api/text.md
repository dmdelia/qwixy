# Qwixy – Matching & Swipe Logic

**Version:** 1.0  
**Status:** Proposed Draft  
**Copyright:** Copyright (C) 2026 Qwixy Development. All rights reserved.

---

## 1. Overview

This document defines the proposed shared swipe, matching, and discovery behavior for Qwixy.

The logic is intended to be consistent across all supported platforms, including Discord and Telegram.

All users are identified by their unique Qwixy ID.

## 2. Swipe & Match Logic

### 2.1 Skip

When a user skips another profile:

- Save the action as `SKIP`.
- Hide the skipped profile from the user's personal feed for 24 hours.
- Overwrite any previous swipe between the same directed user pair.
- Continue to the next available profile.

### 2.2 Like without Message

When a user likes another profile:

- Save the action as `LIKE`.
- Store the like persistently, subject to the retention policy.
- Hide the liked profile from the user's feed for 36 hours.
- Check whether the target user has already liked the initiating user.

### 2.3 Like with Message

- Save a `LIKE` action with an optional text message.
- Maximum message length: 200 characters.
- Cancelling message composition must not save the swipe.
- After saving, follow the regular matching procedure.

### 2.4 Mutual Like Detection

**If a reverse LIKE exists:**

1. Create one match between both users.
2. Prevent duplicate match records.
3. Notify both users.
4. Provide a contact button.
5. Continue to the next profile.

**If no reverse LIKE exists:**

1. Notify the receiving user.
2. Display the sender's profile card.
3. Provide `SKIP` and `LIKE BACK` actions.
4. Allow an optional message when liking back.
5. Continue to the next profile for the initiating user.

## 3. Discovery Algorithm

### 3.1 Base Eligibility Criteria

The following conditions apply to every discovery mode:

- Candidate is not the requesting user.
- Candidate has a profile photo or avatar.
- Candidate is not suspended or banned.
- No existing match exists between both users.
- Candidate has not been reported by the requesting user.
- Candidate has not been skipped within the last 24 hours.
- Candidate has not been liked within the last 36 hours.

### 3.2 Normal Filtered Search (NFS)

Candidates must:

- Have at least one game in common with the requesting user.
- Meet all base eligibility criteria.

Select up to 200 of the most recently active eligible candidates.

#### Scoring

| Criterion | Score |
|---|---:|
| Game language compatibility | +0 to +350 |
| Same game rank | +150 |
| Rank difference of 1 | +90 |
| Rank difference of 2 | +30 |
| Rank difference greater than 2 | −50 |
| Shared roles | +30 per applicable match |
| Never previously swiped | +200 |
| Previously liked | −50 |
| Previous expired skip | 0 |
| Random bonus | +0 to +40 |

For multiple shared games, use the highest individual game compatibility score.

The candidate with the highest score is selected.

**Scoring frequency:** Recalculate candidate scores for each new profile request.

### 3.3 All Players Search (APS)

If NFS has no more candidates, offer the user the option to switch to All Players Search.

APS rules:

- No shared game is required.
- Candidate must have at least one game profile.
- All base eligibility criteria still apply.
- No point-based scoring is used.

Candidates are ordered by:

1. Exact game-language match.
2. Most recent activity.
3. Most recent account creation.

### 3.4 End of Feed

**NFS exhausted:** Offer APS.

**APS exhausted:** Display an end-of-feed message.

## 4. Data Storage

### 4.1 Users

- Each user has one unique Qwixy ID.
- Profiles, language preferences, and contact information are associated with that ID.
- Games, ranks, and roles are stored separately in `game_profiles`.

### 4.2 Swipes

| Field | Description |
|---|---|
| `user_1` | User performing the action |
| `user_2` | Target user |
| `action` | `LIKE` or `SKIP` |
| `message` | Optional text, maximum 200 characters |
| `time` | Timestamp of the action |

**Constraint:** `UNIQUE(user_1, user_2)`

Only one current swipe may exist per directed user pair.

### 4.3 Matches

| Field | Description |
|---|---|
| `user_1` | First matched user |
| `user_2` | Second matched user |
| `time` | Match creation timestamp |

Only one match may exist per unordered user pair.

## 5. Edge Cases & Safety

### 5.1 Swipe Overwriting

A new swipe between the same directed user pair replaces the previous:

- Action
- Timestamp
- Optional message

### 5.2 Swipe Retention

The previous implementation included a cleanup job deleting swipes older than 30 days.

Existing matches remain unaffected.

**TODO:** Decide whether to retain all likes indefinitely and delete only expired skips, or retain the original 30-day cleanup behavior.

### 5.3 Test Profiles

The previous design reserves numeric IDs `0000–9999` for test profiles.

Test profiles:

- May appear in discovery feeds.
- Do not react to likes.
- Do not generate notifications or matches.
- Do not have automated interactions.

**TODO:** Replace numeric ID ranges with an explicit test-profile flag, as Qwixy IDs are intended to be UUIDs.

Test profiles should be clearly identified as non-real profiles.

### 5.4 Match Safety

The following operations must be performed atomically:

1. Save or update the LIKE.
2. Check for an existing reverse LIKE.
3. Create the match if mutual interest exists.

Additional rules:

- Reject self-likes.
- Prevent duplicate matches.
- Handle concurrent likes safely.
- Avoid duplicate notifications for unchanged likes.

### 5.5 Account Linking

Linked Discord and Telegram accounts must use the same Qwixy ID.

Account merging and conflict resolution are separate from the swipe and matching operations.

## 6. Open Decisions

- [ ] Finalize swipe retention policy.
- [ ] Confirm test-profile behavior and storage.
- [ ] Define notification rules for repeated likes.
- [ ] Define account-merging behavior for existing swipes and matches.
- [ ] Confirm scoring weights and language compatibility rules.

---

Copyright (C) 2026 Qwixy Development. All rights reserved.
