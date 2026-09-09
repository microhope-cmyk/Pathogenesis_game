# Pathogenesis — Project Log

Plain-language running record of what's shipped, what's still open, and what a fresh
conversation needs to know to pick this up without re-explaining everything. Newest entry on
top.

---

## 2026-09-09 — Reward animations, card-set split, and an online-mode fix

**Shipped and now live on `main`** (deployed to the real site via Netlify's auto-deploy):

- **Reward animations.** Small, deliberately-infrequent visual rewards:
  - Player's own Health taking a hit: a screen flash plus a floating "−N" number.
  - Opponent's Health taking a hit: an impact ring plus a floating number (uses the game's
    teal/amber "good news" colors, not red, since a landed hit is a good thing for the player
    causing it).
  - A pathogen dying now animates differently depending on *how* it died: an "icon overload"
    (its own icon grows and bursts into particles) if it died while engaging; a "dissolve ring"
    if it died while blocking; a "flash & fold" if an effect card killed it directly
    (Antibiotic-family, Incineration, Cytokine Storm). Any other death (e.g. an aura wearing
    off) keeps the original plain fade.
- **Fixed: a Naive cell primed by a Dendritic Cell could end up "Specialized" without ever
  getting the Effector/Memory (or Plasma/Memory-B) choice.** Priming now always opens that
  choice immediately, the same as winning a fight does.
- **Card sets: "main deck" vs. "expansion."** Every card now belongs to one or the other; only
  main-deck cards are actually shuffled into a real game. Prion and Incineration moved to
  expansion (still fully defined, just not currently in play — bringing them back is a one-line
  change, not a rebuild). Added three new common, cheap pathogens to the main deck: **E. coli**
  (mild strain, Bacterium), **Athlete's Foot** (Fungus), **Pinworm** (Parasite) — each a simple
  1-cost card, so every pathogen category now has a cheap "starter" option the way Rhinovirus
  and Norovirus always did for Virus.
- **Fixed: 2-player online mode was fundamentally broken.** This was the big one. Three
  separate bugs, all specific to online play:
  1. Every move's game state was being sent to the shared online database wholesale — but
     pathogen cards don't have a "class" field and immune cards don't have a "category" field,
     so those came through as missing/blank, and the online database (Firestore) flatly refuses
     to save anything with a value like that anywhere in it. So the save silently failed on
     basically every move. This is exactly why a played card would flash onto the board and
     then revert — the move never actually saved, and the game re-displayed the last version
     that *did* save (i.e., the move undone).
  2. The check for "should the defender get asked how to block, or auto-resolve?" never had a
     case for online mode, so it always auto-resolved as if a computer were defending — the
     real second player never got a block prompt at all.
  3. Same missing-case bug for the Effector/Memory specialization decision — an online
     player-2 would have had that choice made for them automatically instead of being asked.

  All three are fixed. Verified with a stand-in for the online database that behaves exactly
  like the real one (including its exact rule about rejecting incomplete data): reproduced the
  original bug on the old code, confirmed it's gone on the new code, on both player seats,
  across several different cards.

**Known caveat — needs a real-world check:** I could not reach the live online-game backend
from the environment I was working in (its network is locked down), so the fix above is
verified against a faithful stand-in, not the real thing end-to-end with two actual phones. The
owner tried 2-player online once already and reported it still broken — but that test happened
*before* this fix had been merged to `main` and deployed (it was sitting on a branch). It's
live now. **This needs to actually be tried again with two real devices before calling it done.**
If it's still broken, what specifically goes wrong this time will be new information, since
we'd be looking at genuinely different code than before.

**Not done yet / open:**
- **Sound effects.** Owner asked about adding them; confirmed it's easy technically (a short
  audio file per moment, no architecture impact) but there's no built-in sound library to draw
  from. Waiting on the owner to either provide sound files (MP3 preferred for broad phone
  support, WAV also fine; keep each clip short — under ~1-2 seconds — so file size stays small,
  well under 100KB each) or ask for help sourcing free/CC0 ones. Once files exist, the natural
  next step is deciding which moments get a sound (the death-animation triggers are natural
  hook points) — not yet planned in detail.
- A minor **pre-existing bug**, unrelated to anything above: if you tap "Leave" at the exact
  moment the computer is mid-turn in Solo mode, there's a rare console error (the game briefly
  tries to read state that was just cleared). Doesn't appear to break anything visibly. Not
  fixed — just noting it exists in case it's ever worth a look.

**Branch status:** everything above is merged into `main` and pushed. The `reward-animations`
branch (used earlier per the owner's "keep it in branch for now" instruction) now points at the
exact same commit as `main` — there's nothing left sitting unmerged.

---

## Earlier history (pre-dates this log)

Not written up session-by-session, but for context, prior work already in place before the
entry above: Firebase/Firestore setup for online multiplayer, Netlify deployment, the GitHub
repo itself, a full interactive tutorial mode, and various UI/prompting polish (clearer
messaging when an action can't be taken, overflow/layout fixes, etc.).
