# Firebase setup and security rules

The rules live in [`database.rules.json`](database.rules.json). This page explains what they do and how to set up a Firebase project for the game from scratch.

> **Status (2026-10-14):** the original Firebase project `bingo-2d4a0` no longer exists. Its database, hosting address and API key all return "not found" or "invalid", so the game needs a new project. The steps below cover that. **These rules have not been tested against a real database yet.** Test them during the preview re-test (see `EVENT_RUNBOOK.md`).

## Who is who

- **Players** sign in automatically and invisibly through Firebase *Anonymous* sign-in. Each browser gets a private ID (`uid`), and the rules use it to tie a player's card to the browser that created it.
- **Hosts** sign in on `admin1.html` with an email and password you create in Firebase. An account only counts as a host if its `uid` is listed under `/hosts` in the database. Only you can add it there, in the Firebase console.

## What each rule means

| Data | Who can read | Who can write | In plain English |
|---|---|---|---|
| Everything not listed below | nobody | nobody | Locked by default. Anything not explicitly opened stays closed. |
| `hosts/<uid>` | that account only | nobody (console only) | The host list. The admin page checks it after sign-in. No one can make themselves a host from a browser. |
| `rooms/<room>` (whole room) | hosts | hosts | Hosts can create rooms, call numbers, confirm winners, and reset or wipe a room. |
| `rooms/<room>/status`, `calledNumbers`, `lastCall`, `winner`, `wonPrizes` | any signed-in player | hosts only | Players can see the called numbers and the winners, but can't change them. |
| `rooms/<room>/callQueue` | hosts only | hosts only | **The upcoming numbers.** Players used to be able to read this, which meant anyone could see what was coming next. Now they can't. |
| `rooms/<room>/callTimes` | hosts only | hosts only | When each number was called. Used to flag cards created mid-game. |
| `rooms/<room>/claims` | hosts only | any signed-in player, add only | A player can send a new claim but can't edit or delete any claim, or read anyone else's. A claim is refused if: the prize is already won, the player isn't registered in the room, the prize name isn't one of the four real ones, or the timestamp isn't the server's own. |
| `rooms/<room>/players` (the whole list) | hosts only | — | Players can't list everyone in the room or harvest their emails. |
| `rooms/<room>/players/<emailKey>` (one record) | any signed-in player | see below | A player can look up a single record by its email key. This is what lets someone rejoin from another device and see their own card. |
| …`/uid` | | once, by the creator | The first browser to register an email in a room owns that record. Nobody else can take it over later. A record can only be created in a room the host has already opened. |
| …`/card` | | once, by the owner | The card can't be changed after it's created, so nobody can swap in a winning card mid-game. It must have exactly 25 numbers, each in its column's range (B 1–15, I 16–30, N 31–45, G 46–60, O 61–75), with no repeats in a column. |
| …`/email` | | once, by the owner | Must look like an email and must match the record's key, so you can't register under someone else's address. |
| …`/name` | | owner | 1–60 characters. |
| …`/joinedAt` | | once, by the owner | Must be the server's clock, not the player's. |
| …`/marks` | | owner only | The squares the player has ticked (positions 0–24). On a second device the player can still see their card and mark it there, but those marks are only saved on that device. |
| Any other field | — | nobody | Unknown fields are refused, so the data can't be filled with junk. |

### What the rules can't stop (and how the game handles it)

- **Cards are made in the player's browser.** Someone who joins mid-game could hand-pick a card made of numbers already called. The rules still force a valid-looking card. On top of that, the Verify pop-up shows **"⚠ Joined after N numbers were called"** for anyone who joined late, so the host can look twice before confirming.
- **Anyone signed in can send a claim using another player's email key.** It does no harm: Verify always checks the *card owner's* card and credits that person.
- **Free plan connection limit.** On the free "Spark" plan the database allows **100 simultaneous connections**. With 80–90 players, extra tabs and two hosts, you can hit that, and anyone over the limit can't connect. See step 9.

## Step-by-step: set up a new Firebase project (about 20–30 minutes)

Use an account the company controls (ideally a shared or team Google account, not a personal one), so this doesn't disappear if someone leaves.

1. **Create the project.** Go to <https://console.firebase.google.com> → **Add project**. Name it something like `myhq-bingo`. Google Analytics isn't needed, so turn it off. Click **Create project**.
2. **Register the web app.** On the project home page, click the **`</>`** (Web) icon. Give it a nickname such as `bingo-web` and leave "Firebase Hosting" unticked. Click **Register app**. Copy the `firebaseConfig = { … }` block that appears and send it to me. It isn't secret: it identifies the project, and the rules are what protect the data. I'll put it into `index.html` and `admin1.html`.
3. **Create the database.** Left menu → **Build → Realtime Database → Create database**.
   - Location: **Singapore (asia-southeast1)**, the closest to India.
   - Security: choose **Start in locked mode**. You'll replace the rules in step 7.
4. **Turn on sign-in.** Left menu → **Build → Authentication → Get started → Sign-in method** tab:
   - Click **Anonymous** → toggle **Enable** → **Save**.
   - Click **Email/Password** → toggle the first **Enable** (leave "Email link" off) → **Save**.
5. **Create the host account(s).** In **Authentication → Users**, click **Add user**. Enter a host email and a strong password, then click **Add user**. In the users list, copy that user's **User UID** (a long string like `Xy3k…`). Repeat for a backup co-host. **Two hosts is strongly recommended**, so the game isn't stuck if one person drops off Zoom.
6. **Mark them as hosts.** Go to **Realtime Database → Data** tab.
   - Hover over the top (root) line and click **+**.
   - Key: `hosts`. Leave the value empty and click the **+** next to it to add a child.
   - Child key: *paste the host's UID*. Value: `true`. Type it without quotes so it's saved as the true/false value, not as text.
   - Click **Add**. Repeat with the co-host's UID inside `hosts`.
   - The result should look like `hosts → Xy3k… : true`.
7. **Paste the rules.** Open the **Rules** tab. Select everything in the editor and delete it. Open `database.rules.json` from the repo (on GitHub, open the file → **Raw** → select all → copy) and paste it in. Click **Publish**. If the console shows a red error, don't publish; send me the message.
8. **Quick check.** Still on the Rules tab, open **Rules Playground**:
   - Simulate a **read** of `/rooms/TEST/callQueue` with **Authenticated** ticked. It should be **denied**.
   - Simulate a **read** of `/rooms/TEST/calledNumbers` with Authenticated ticked. It should be **allowed**.
9. **Avoid the 100-connection limit (recommended).** Left menu (gear icon) → **Usage and billing → Details & settings → Modify plan → Blaze (pay as you go)**, then set a **budget alert** of e.g. ₹500. A one-hour game for ~90 players costs a few rupees at most. Without Blaze, players beyond 100 connections can't join.
10. **Tell me when it's done** and send me the config from step 2. I'll update the code, push the branch, and run the full TST-001 test against the new project.

## Important: the old live site

`bingo-final.vercel.app` still runs the old code pointed at the deleted project, so it doesn't work today. The new code only works once steps 1–7 are done and the config from step 2 is in the code.
