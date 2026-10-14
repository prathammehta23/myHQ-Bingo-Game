# myHQ Bingo Bonanza — Diwali event runbook

For the hosts running the game on Zoom (~80–90 players). Read it once before the event and keep it open during the game.

## Where everything lives

| Thing | Where | Owner |
|---|---|---|
| Code | GitHub `prathammehta23/myHQ-Bingo-Game`. Event work is on branch `diwali-bingo`. | **Pratham's personal GitHub account** (Pratham has left). Samarth has push access, but not admin, so can't change settings or transfer the repo. **To do:** copy or fork it into a company-owned GitHub account or org. |
| Website (Vercel) | Old: project `bingo-final` → `bingo-final.vercel.app`. **New: _TBD_** (Samarth is setting this up). | Old: Pratham's Vercel account (access unknown). New: Samarth's Vercel account. |
| Database and sign-in (Firebase) | Old: project `bingo-2d4a0`. **Deleted or unreachable as of 2026-10-14.** **New: _TBD_** (see `FIREBASE_RULES.md`). | Old: Pratham. New: Samarth (ideally on a company Google account). |

Update this table once the new Vercel and Firebase projects exist.

## Links

- **Player link:** `https://<NEW-VERCEL-DOMAIN>/` (fill in once the new Vercel project exists). This is the link that goes in the invite email. **Don't put the room code in the email.** The rulebook says hosts reveal the room code live on the call.
- **Admin link:** *not written here on purpose.* It's the player link with `admin1.html` on the end. Keep it, and the host email and password, **only in the hosts' password manager**, in an entry named "myHQ Bingo host". Never paste it into Zoom chat, Slack or email. Even if it leaks, nobody can use it without a host sign-in.
- **Rehearsal (no live data):** add `?demo=1` to either link. The player page calls numbers automatically every 3 seconds, and nothing is saved.

## Before the event (day before)

1. Do a full run-through on the **preview URL** with 2–3 colleagues: join, mark, claim, verify, reset.
2. Both hosts sign in to `admin1.html` on their own laptops and check they see the dashboard.
3. Make sure the Firebase project is on the **Blaze** plan, or accept the 100-connection limit (see `FIREBASE_RULES.md` step 9).
4. Prepare the fallback sheet (see "If something breaks"): a Google Sheet with columns `#`, `Call`, `Time`.
5. Agree who does what: **Host A** calls numbers and reads the script line; **Host B** watches claims and verifies.

## Running the game

### 1. Start
1. Host A opens the admin link and signs in. A room code appears at the top, e.g. `MHQ-K7P`. That code is your room.
2. If a room code is already showing from an earlier test, click **+ New Room** for a fresh one.
3. Screen-share the rulebook, then **say the room code out loud and type it in Zoom chat.**
4. Players open the player link, enter their name, company email and the room code, and click **Get My Card**.
5. Watch **Players Joined** on the dashboard. Wait until it stops climbing, then do a final "anyone not in yet?" call.
6. Ask everyone to **take a screenshot of their card now**. This is the backup if anything breaks.

### 2. Call numbers
1. Click **Call Next Number 🪔**. Read out the letter, the number and the script line under it, e.g. "G 54 — Check the inventory once more!".
2. Leave about 10 seconds between calls so people can find and mark the number.
3. Keep the fallback sheet updated with each call if you have a spare person. It costs little and saves the game if the dashboard dies.

### 3. Verify a claim
1. When someone shouts BINGO, their claim appears under **Bingo Claims** with the prize name.
2. Click **Verify**. A pop-up shows their card with every called number highlighted.
   - **✅ Legitimate claim:** if there's a **"⚠ Joined after N numbers were called"** line, glance at the card. A normal card has 5 different numbers per column from that column's range. If it looks hand-picked (e.g. full of numbers that were already called), don't confirm and ask the player about it. Otherwise click **Confirm winner**. The prize locks, and every player sees "X won …".
   - **❌ Invalid claim:** say so on the call, kindly. The player keeps playing.
   - **Already won:** that prize was confirmed for someone else first.
3. Click **Dismiss** on the winner banner when you're ready to carry on.
4. Prizes, in order of difficulty: Quick Strike (1 line), Double Dhamaka (2 lines), Corner Capture (4 corners), Jackpot Jashn (full house). Each one can only be won once per room.

### 4. Reset (new round)
Click **↺ Reset Game** and confirm. This clears called numbers, claims, winners **and every card**. Players don't need to do anything: their pages deal them a fresh, clean card automatically. Same room code, new round.

## If something breaks mid-game

| Problem | What to do |
|---|---|
| **A player's page shows "Reconnecting…"** | Usually their Wi-Fi. It recovers by itself. If not, they refresh and rejoin with the **same email and room code**, and get the same card back. |
| **A player switched device** | Same email and room code on the new device gives them the same card. Marks made on the new device are only saved on that device. That's fine: the host verifies from the master list, not from their marks. |
| **A player says "Room not found"** | They mistyped the code. Repeat it in chat. |
| **The host's dashboard freezes or the laptop dies** | **Host B** opens the admin link with `?room=<ROOM CODE>` on the end and signs in. Everything (called numbers, claims, players) is stored in the database, not on the laptop, so nothing is lost. |
| **The dashboard says "Could not load the room…"** | Refresh once. If it persists, the Firebase rules or project setup is wrong. Switch to the paper fallback below and investigate after the event. |
| **Nobody can join / everything is down** | Switch to the paper fallback below. |

### Paper / spreadsheet fallback (keeps the game going without the website)

1. Announce: "The website is having trouble. Keep your card open or use your screenshot, and mark numbers yourself."
2. **Call numbers from the Google Sheet.** Use the `=RANDBETWEEN(1,75)` method, or better, pre-fill a shuffled list: in column A put 1–75, in column B put `=RAND()`, then sort by B. Read them out in order, saying the letter (1–15 B, 16–30 I, 31–45 N, 46–60 G, 61–75 O). **Mark each one in the sheet** as you call it, and paste each call into Zoom chat so people can catch up.
3. If the dashboard was working before it broke, **continue from where it stopped.** The called numbers are on the dashboard board, or in the Sheet if you were logging.
4. Claims: the player shouts BINGO and posts a **screenshot of their card** in chat (or shares their screen). The host checks every number in the winning line against the Sheet.
5. Prizes as before, one per pattern, first verified claim wins.

## Not yet tested at 80–90-player scale

These have only been checked with a handful of browsers. Plan the dry run accordingly.

- **Connection limit:** 100 simultaneous connections on Firebase's free plan. Every open tab counts. Unknown how close a real crowd gets.
- **Join surge:** 80–90 people clicking "Get My Card" within the same minute.
- **Reset with a full room:** about 90 players all getting a new card at the same instant.
- **Claim burst:** many people claiming the same prize within seconds. The rules should keep it orderly, but it hasn't been tried with more than two players.
- **Dashboard performance** with about 90 players marking squares; each mark updates the host's player list.
- **Corporate networks / VPNs** blocking `firebaseio.com` / `firebasedatabase.app` or Google sign-in endpoints.
- **Phones:** only checked at narrow screen widths in a desktop browser, not on real iOS/Android devices or Zoom's built-in browser.
- **The security rules themselves:** written carefully, but first exercised during the preview re-test.
