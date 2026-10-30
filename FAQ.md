# ❓ Better Stream Announcements — FAQ

---

## 🛠️ Setup & Configuration

**Q: Where are the config files stored?**

Everything lives in one shared file under your Streamer.bot base directory:

```
Announcements/Discord/settings.json   ← every setting for every feature (Broadcaster, Streamer Friends, Discord Bot, Role Assign, Custom Crosspost)
```

You shouldn't have to edit it by hand; the Settings GUI covers all of it. Every BsA action (Broadcaster Online/Offline, Streamer Friends, Discord Bot Presence, Presence Role Assign, Self-Assign, Custom Crosspost) reads this one file, so a change made on any page reaches every script that cares about it.

---

**Q: Where do I see what BsA actually did?**

In the Settings window under **Help → Status Log**: every line a BsA action wrote to Streamer.bot's log, one day at a time, plus what Streamer.bot itself reported about a BsA action, such as an error naming it or a script that would not compile. Filter it by day, action, level and text. **Home → Today** sums up today's log: announcements posted, warnings, errors, and the last problem.

---

**Q: Do I have to copy all those Discord IDs by hand?**

No. Run `/bsa setup` in your Discord server and pick channels, roles and bot admins straight from Discord's own menus. Every pick is saved to `settings.json` the moment you make it, and the Server ID is filled in automatically if it's still empty. The Settings GUI remains the place for everything else: texts, colors, platforms and API keys.

If the Settings window happens to be open while you do this, that's fine: saving the window only writes the fields you changed there, so picks made in Discord in the meantime survive.

---

**Q: The Settings GUI won't open / crashes Streamer.bot entirely. What do I do?**

- Look at the Streamer.bot log for the error and please report it on [GitHub Issues](https://github.com/aaskjer/Better-Stream-Announcements/issues).
- A malformed `settings.json` breaks the GUI and every other BsA action. Delete the broken file and reopen the GUI; it gets recreated with defaults.
- If closing and reopening the Settings window and then toggling **Show Debug / Testing Sections** ever took the whole process down: that was a real bug (stale cross-thread UI elements) and it's fixed in the latest version. Update if you're still on an older release.

---

**Q: How do I reset all settings to defaults?**

Open the **☰** menu at the top left of the Settings window and pick **Reset to Defaults…**. It rewrites `settings.json` with factory values and reopens the window. **Show Debug / Testing Sections** is a separate toggle under **About → Advanced**; it only reveals the debug checkboxes mentioned elsewhere in this FAQ and doesn't reset anything.

---

**Q: I saved settings but nothing changed. Why?**

Most scripts (Streamer Friends, Broadcaster) read `settings.json` again at the start of their next `Execute()`, so the next trigger already uses your change, no restart needed. **Discord Bot Presence** and **Presence Role Assign** go further: they watch the file and apply relevant changes (presence text, bot profile, role lists) live, usually within a second.

---

**Q: It won't compile in Streamer.bot — `CS0012: The type 'IDynamicMetaObjectProvider' is defined in an assembly that is not referenced`.**

A real bug in older releases. A few embed-building calls used `new JArray { item1, item2 }` (C#'s collection-initializer syntax), which makes the compiler check that `JArray` implements `IEnumerable`. That check pulls in `System.Core` for `IDynamicMetaObjectProvider`, and Streamer.bot's compiler doesn't reference it by default. The equivalent constructor form, `new JArray(item1, item2)`, avoids it. Update the affected script(s) to the latest version.

---

## 📢 Broadcaster Announcements

**Q: What's the difference between the Channel-ID and Webhook options?**

- **Channel-ID** (with `BOT_Token` set): the bot posts as itself. This is also the only way to get the **Watch Stream** button.
- **Webhook URL**: no bot needed. Editing and deleting work here too, so Broadcaster Offline can turn the live announcement into the "ended" one, just like with the bot. A webhook is tied to one channel, though, so a separate stream-ended channel is a bot-only option.

You only need one of the two. With both set, the channel and the bot win.

**Q: Can BsA announce on GameVox or Stoat instead of, or as well as, Discord?**

Yes. Broadcaster → Platforms and Streamer Friends → Platforms each take a GameVox webhook and a Stoat webhook. Discord, GameVox and Stoat are three delivery targets of their own: each one is posted when a stream starts, updated while it runs, switched to its ended form when it ends and, for a Streamer Friend, deleted once **Remove Old Message** takes it down. A target that isn't set up, or that refuses a post, never holds the other two back, so a GameVox or Stoat webhook on its own is enough to announce with.

What each one leaves out:
- **GameVox** speaks Discord's webhook API, so it gets the Discord post minus the buttons and the role mentions (those roles only exist on Discord). The time moves from the embed timestamp into the footer, and it posts under the webhook's own name and picture.
- **Stoat** has no author line, fields or footer in its embeds, so they become lines of text, and there's no preview picture: Stoat only takes pictures uploaded through a bot account. Message and embed share Stoat's 2000 characters, so a long description is shortened.

A message is only posted for a stream that is starting, so a webhook added while someone is live starts with their next stream.

**Q: Why didn't my announcement post when I went live?**

Check, in this order:
1. **Help → Status Log** in the Settings window. It usually says why in one line.
2. **The actions `Broadcaster Online` and `Broadcaster Offline` are enabled** in Streamer.bot. They come switched off in the import, so nothing goes out before you've set things up.
3. **Filter by Category / Filter by Title** (Broadcaster → Online → Filters). If either is set, nothing gets posted unless the category matches one of your entries exactly or the title contains your keyword. Empty means no filtering.
4. A **20-minute cooldown** after your last announcement keeps a shaky reconnect from posting twice. That's intentional; only the hidden `BA_SkipCooldown` debug flag bypasses it (Broadcaster → Online → Debug & Testing, visible once **Show Debug / Testing Sections** is on in About → Advanced).
5. Nowhere to post: `BOT_Token` + `BA_ChannelId` missing or wrong, no `BA_WebhookUrl` as fallback, and no GameVox or Stoat webhook either. **Home → Right Now** says so in its first line.

**Q: Random Embed Color is on — do the per-platform hex colors still matter?**

Not while it's on. **Random Embed Color** replaces the fixed per-platform colors completely; they're only used with it off. The one thing that beats it is **Use My Profile Color** / **Use Streamer's Profile Color**, whenever that finds a color.

**Q: How does "Use Streamer's Profile Color" pick a color?**

Per streamer, in this order:
1. **Twitch only:** their chat name color, read from Twitch's API. It's the one real "profile color" Twitch, YouTube and Kick expose.
2. **Everyone else:** the dominant color of their profile picture. Pixels count by how saturated they are, so a small vivid logo beats a large muted background, and a fully greyscale avatar gives a matching grey instead of a random hue.
3. If neither works: **Random Embed Color** (if it's on), then your fixed per-platform hex.

Results are cached in memory, so each friend's color is looked up once, not on every poll.

**Q: "Use Streamer's Profile Color" isn't working for my Kick friends.**

Kick serves profile pictures as **WebP**, and Windows only decodes WebP with the *WebP Image Extensions* component installed. Most Windows 11 installs have it, not all. Without it the color can't be read and the embed uses your configured Kick color; the Streamer.bot log then shows `avatar format not supported by Windows imaging`. Installing "WebP Image Extensions" from the Microsoft Store fixes it.

This only affects the avatar path. Twitch friends are fine, because their color comes from the API, not from an image.

**Q: A Twitch friend's embed isn't using their chat color.**

Twitch only returns a color for users who actually picked one. Everyone else gets a random preset from the Twitch client on page load; it isn't stored anywhere and can't be read back. Those friends fall through to their avatar color.

**Q: What does "Auto Crosspost" do, and is it the same as "Custom Crosspost"?**

Two different features with a similar name:
- **Auto Crosspost** (`BA_AutoCrosspost` / `SF_AutoCrosspost`) presses Discord's own **Publish** button on your announcement, so servers that *follow* your announcement channel get it too. Your channel has to be an Announcement channel.
- **Custom Crosspost** (its own section below) is an opt-in system where members register their *own* webhook and receive copies of the announcements in their own server. Nobody has to follow anything.

---

## 👥 Streamer Friends

**Q: Which platforms can Streamer Friends announce?**

Twitch, YouTube, Kick, Owncast, Rumble, Picarto and TikTok. Each platform is its own action with its own friends list, Discord map and ISLIVE role. TikTok is the one without a public API: its liveness comes from the public live page, which TikTok can change at any time.

**Q: My friend's ISLIVE role never gets assigned — why?**

This role comes from the platform's API, not from Discord presence. Check:
1. Their login, handle, slug or URL is in that platform's **Friends** list.
2. They're linked to a Discord account under **Assign Role To** for that platform. Owncast and Rumble pairs use `|` instead of `:` (`https://stream.example.org|123…`), because the URL already contains a colon.
3. A role is set up: either the platform's own **ISLIVE Role ID(s)**, which *replaces* "All Platforms" for that platform instead of adding to it, or the shared **All Platforms** role.
4. With **Skip Offline/Invisible Discord Users** on, the friend's Discord status mustn't be Offline or Invisible. That check needs **Presence Role Assign** (Discord Bot → Role Assign) switched on, since that's the only script that reports Discord status.

**Q: A friend reconnects quickly after a stream drop — does that spam a new announcement?**

No. **Reconnect Window (Minutes)** (default 15) treats a quick reconnect as the *same* stream, so nothing gets posted twice. **Session Cooldown (Hours)** (default 1) covers the longer gap: if a friend goes live again within that many hours after their stream ended, their "ended" announcement turns back into a live one instead of a new post appearing. After that, the next stream gets a fresh announcement; `0` means always a fresh one.

**Q: "Register Again" says it didn't work and the log shows nothing at all.**

Then Streamer.bot never found the method. `ExecuteMethod` resolves its target by the **Name** of the *Execute C# Code sub-action*, not by the action's name, and it fails silently when nothing matches. Open **[BsA] - Discord Bot Presence**, select its C# sub-action, and set its **Name** field to exactly `[BsA] - Discord Bot Presence`. The same Name is what lets Presence Role Assign reach the shared gateway, so it's worth setting even if you never press the button.

If the log *does* have a line, it says what Discord refused — usually the bot token, or a missing `applications.commands` scope on the invite.

**Q: Other bots list their commands in their Discord profile. Why doesn't BsA?**

That section is filled from an application's **global** commands, and BsA registers its commands to the server in **Server ID** instead. That's on purpose: the bot token is usually one you also use for other things, and global registration would offer `/bsa`, `/crosspost` and `/streamerfriends` in every server that bot has joined, not just yours. Server-scoped commands also appear the moment they're registered, while global ones can take Discord up to an hour.

The commands themselves work exactly the same either way — they just aren't advertised on the profile card. With **Server ID** left empty there is nothing to scope to, so BsA falls back to registering globally.

**Q: A friend's stream is still showing as live hours after they went offline, and it never gets deleted.**

Check the log for `ended-embed edit` warnings. BsA rewrites the announcement into its "ended" form before it closes the session, and the delete timer only starts afterwards, so an edit Discord keeps refusing holds everything up. Two causes are worth ruling out:

- **You changed the bot token.** A bot can't edit a message a different bot posted. Announcements still up from the old token can only be deleted — give the bot **Manage Messages** in the announcement channel and BsA will clear them once the session closes.
- **Rate limit `30046`.** Discord limits edits to messages older than an hour. With **Update Announcements** on and a long friends list, the viewer-count refreshes can eat that budget. Turning **Update Announcements** off, or shortening the friends list, takes the pressure off.

Either way BsA gives up on the edit after a while (10 minutes after an outright rejection, 45 minutes while rate limited) and closes the session anyway, so the role comes off and the announcement is deleted on schedule.

**Q: How do I stop a brand-new "ended" message from being posted every time?**

Turn on **Update Announcements** (Streamer Friends → Online). It edits the *existing* live announcement in place, both for title/game/viewer changes and for the final "ended" state, instead of posting separately.

**Q: Can members add themselves instead of me typing everyone into the lists?**

Yes, with self-linking (Discord Bot → Role Assign → **Enable Self-Assign**). A member runs `/streamerfriends link`, picks the platform and enters their account; the bot checks that the account exists and adds it to the friends list and the Discord map in one go. Details are in the Self-Assign section below.

---

## 🎭 ISLIVE Role Assignment — Presence vs. API

**Two independent systems** can grant the same "ISLIVE" role, and they deliberately don't coordinate who owns it. That trips people up, so it gets its own section.

**Q: What's the difference between "Presence Role Assign" and Streamer Friends' role assignment?**

| | Presence Role Assign | Streamer Friends |
|---|---|---|
| Source of truth | Discord's own "Streaming" activity (gateway presence) | Polling the platforms (Twitch, YouTube, Kick, Owncast, Rumble, Picarto through their APIs; TikTok off its public live page) |
| Role combination | **Additive**: "All Platforms" role + the matching per-platform role | **Override**: the per-platform role replaces "All Platforms" instead of stacking |
| Reacts to | A live→offline *transition* it saw itself | Whatever the latest poll reports |
| Removes a role it didn't grant? | Never | N/A (it's the authority for its own roles) |

**Q: Why did my role get stripped right after being granted?**

A real bug in older versions. With the same role ID set up in both systems, Presence Role Assign could see "not streaming, according to Discord" and remove a role Streamer Friends had just granted correctly via the platform API. That happened mostly when a member doesn't use Discord's stream detection at all, or when it was slow to catch up. Fixed: Presence Role Assign now reacts only to a live→offline transition it saw itself, and only removes roles it granted itself. Roles from anywhere else stay untouched, however long Discord presence says "not live".

**Q: My member intentionally sets themselves Offline/Invisible on Discord — will they still get roles?**

By default, yes. Streamer Friends ignores Discord status entirely, and Presence Role Assign usually can't see a "Streaming" activity for an Invisible member anyway, because Discord hides it. If you want Offline/Invisible members skipped, turn on **Skip Offline/Invisible Discord Users** on the matching page.

---

## 🤖 Discord Bot & Commands

**Q: What are the Discord commands?**

The bot uses slash commands. Type `/` in your server and Discord lists them with a short description; only the person who runs one sees the answer.

| Command | Who | What it does |
|---|---|---|
| `/bsa help` | Everyone | Overview of all commands |
| `/bsa setup` | Bot admins | Pick channels, roles and bot admins from menus, no IDs needed |
| `/bsa status` | Bot admins | Connection, uptime, latency and presence status |
| `/bsa refresh-presence` | Bot admins | Sends the bot's status and activity to Discord again |
| `/bsa reload-profile` | Bot admins | Pushes name, avatar, banner and About Me to Discord again |
| `/streamerfriends link` | Everyone (unless role-restricted) | Link one of your streaming accounts; leave the options empty to be guided |
| `/streamerfriends unlink` | Everyone (unless role-restricted) | Unlink one again, picked from your list |
| `/streamerfriends accounts` | Everyone | Show your linked accounts |
| `/streamerfriends list` | Self-Assign admins | Everyone linked on one platform |
| `/streamerfriends service on\|off` | Self-Assign admins | Open or close self-linking |
| `/streamerfriends help` | Everyone | How Streamer Friends works |
| `/crosspost setup` | Everyone (unless role-restricted) | Subscribe a channel on your own server to announcements |
| `/crosspost list` | Everyone | Show your subscriptions and remove one |
| `/crosspost remove` | Everyone | Remove a subscription by ID |
| `/crosspost service on\|off` | Crosspost admins | Open or close new subscriptions |
| `/crosspost help` | Everyone | How crossposting works |

**Bot admins** are the members, and everyone holding a role, listed under **Admin User/Role (IDs)** (Discord Bot → Bot Presence). User and role IDs can be mixed. While that list is empty, anyone with Discord's **Manage Server** permission counts as one. Self-Assign and Crosspost admins come from their own **Admin Role ID(s)** fields.

**Q: Where did `?bsahelp`, `?cc`, `?sfadd` and the other `?` commands go?**

They're replaced by the slash commands above. Discord increasingly expects bots to use slash commands, and the old prefix commands needed the *Message Content Intent*, which the bot no longer asks for. The **Discord Prefix** setting is gone with them. If your bot's activity text still says `?bsaHelp`, it's switched to `/bsa help` automatically, as long as it's still the old default.

**Q: The slash commands don't show up in my server.**

- The bot registers them by itself when it connects. With a **Server ID** set they appear right away; without one they're registered globally, which can take Discord up to an hour.
- The bot needs the **applications.commands** scope. If it was invited with only "bot", invite it again with both scopes (see the Bot Token tooltip).
- **Slash Commands → Register Again** (Discord Bot → Bot Presence) pushes them once more, for example after a re-invite.

**Q: Setting the bot's Avatar/Banner/Description in the GUI didn't do anything.**

Historically, two causes:
- `Profile Update On Start` only pushes changes when Streamer.bot starts. You don't need it for live edits.
- `Profile Banner Url` and `Profile Description` used to be settings that weren't connected to Discord at all. They work now: the banner goes through Discord's user-profile endpoint together with the avatar, and the description is Discord's application-level "About Me", updated separately. Edits to `settings.json`, including the GUI's Save button, apply on their own within about a second; no restart or admin command needed.

**Q: Why does `/bsa reload-profile` sometimes say it's rate-limited?**

Discord rate-limits profile updates. After a successful update, BsA waits a short cooldown (about 30 seconds, longer if Discord asks for it with a 429) before allowing the next one. That applies no matter where the update came from: the command, the GUI's Save or the file watcher.

**Q: What Discord permissions/intents does the bot need?**

Under **Privileged Gateway Intents** in the Discord Developer Portal, turn on **Presence Intent** and **Server Members Intent**. Without the Presence Intent, Presence Role Assign has nothing to work with. **Message Content Intent** isn't needed anymore: commands arrive as slash commands. When inviting the bot, tick the scopes **bot** and **applications.commands**.

**Q: My bot is stuck offline and the presence never updates, no matter what I change.**

Start with `/bsa status`. It shows the connection, whether the presence is switched on, whether the settings watcher is still alive, and any error that stopped the session. Then go down this list:

- **Privileged intents got switched off.** The usual reason a bot that worked yesterday is offline today. Discord refuses the connection outright (close code `4014`) when the bot asks for an intent the application isn't approved for, and it revokes them by itself once an unverified bot passes 100 servers. BsA then reconnects without them, so the bot still comes online and the slash commands keep working; the log and `/bsa status` say so. Presence Role Assign stays without data until you turn the intents back on in the Developer Portal and restart the action.
- **Status is set to "Offline".** On the Bot Presence page that means Discord's *invisible*, which looks exactly like a dead bot. Pick **Online** for the green dot.
- **Activity Type is "Streaming" without a usable Stream URL.** Discord quietly drops a Streaming activity unless it carries a `twitch.tv` or `youtube.com` link, leaving the bot with no activity line at all. BsA falls back to *Playing* and logs why.
- **The settings watcher died.** Live updates rely on a file watcher, and Windows silently kills it when its buffer overflows or the folder gets replaced. `/bsa status` shows whether it's still watching; it's rebuilt on every reconnect anyway, and the settings are read again on every connect.
- **Something else drives this bot's presence.** Switch **Set Bot Presence** off (next question) and let the other thing own it.

**Q: Can I stop BsA from touching the bot's presence at all?**

Yes: **Discord Bot → Bot Presence → Set Bot Presence**. With it off, BsA never sends a status or activity. The bot still connects and every command and feature keeps working; it just shows as plain *Online* without an activity line, and the rest of that section is ignored. Switching it off on a running bot clears the activity immediately instead of leaving the last one stuck. It's also the quickest way to rule the presence out while you troubleshoot something else.

---

## 🙋 Self-Assign

**Q: How does a member link an account?**

`/streamerfriends link`, then pick the platform and enter the account. Members who know the drill can pass everything at once: `/streamerfriends link platform:Twitch account:yourname`. What to enter per platform:

- **Twitch, Kick, Picarto:** the channel name (a pasted channel link works too)
- **YouTube:** the @handle or channel ID
- **Owncast:** the server address, e.g. `stream.example.org`
- **Rumble:** the livestream API URL, plus the channel link (optional) for the name shown in announcements
- **TikTok:** the user name, as in `tiktok.com/@yourname`

The bot looks the account up before saving it, so typos and wrong platforms are caught right away. For Twitch, YouTube, Kick and Picarto this needs the credentials under Streamer Friends → Platforms; TikTok is checked against the public profile page.

**Q: How do "Exclusive Role ID(s)" work for Self-Assign?**

Despite the label, it takes several roles, comma-separated. A member needs *any one* of them to link or unlink their own accounts. Leave it empty to allow everyone.

**Q: Can an admin link or unlink an account for someone else?**

Yes, with the `member` option: `/streamerfriends link member:@someone`. That's reserved for the **Admin Role ID(s)** under Self-Assign; everyone else can only manage their own accounts.

**Q: I set an Admin Role but `/streamerfriends service` says I don't have permission.**

Admin commands need **both** a non-empty `SF_SelfAssignAdminRoleIds` list **and** the member holding one of those roles. While the admin role list is empty, the commands are blocked for everyone, the broadcaster included.

---

## 🔀 Custom Crosspost

**Q: What does a subscriber actually get?**

Whatever they picked in `/crosspost setup`: every Streamer Friends announcement on one platform, one specific Streamer Friend, or the Broadcaster's own announcements. It's delivered to their own Discord webhook, in their own server. Stream-ended messages and edits aren't crossposted.

**Q: Do members need to allow DMs from the bot?**

Not for the setup. It all happens in replies only they can see, including pasting the webhook URL. The bot only sends a DM when a subscription gets paused (next question).

**Q: What happens when a subscriber's webhook stops working?**

After 5 failed deliveries in a row the subscription pauses, and the owner gets a DM. A paused subscription stays paused; the owner removes it with `/crosspost remove` and sets it up again with a working webhook.

**Q: Where do webhook URLs go for Custom Crosspost subscribers — are they stored safely?**

Yes. Every subscriber webhook URL is AES-encrypted before it's written to disk, and only decrypted in memory when an announcement is actually sent.

The key for that encryption (`BsA_Crosspost.key`) is itself protected by Windows (DPAPI) for the account Streamer.bot runs under. Someone who gets a copy of the `Announcements` folder, from a cloud backup, a support upload or a USB stick, can't decrypt anything on another PC or under another Windows account.

**Q: I'm moving to a new PC / reinstalling Windows. What happens to the crosspost subscriptions?**

Because the key only works for your current Windows account, you need the **recovery code**. Get it beforehand under **Discord Bot → Custom Crosspost → Webhook Key → Recovery Code → Show** and keep it somewhere safe, a password manager for example. On the new PC, copy over `BsA_Crosspost.json` as usual and use **Restore From Code**. Before replacing anything, BsA checks whether the code actually opens the stored subscriptions.

Treat the code like a password: together with `BsA_Crosspost.json` it reveals every member's webhook URL.

Without the code, the subscriptions stay locked. Nothing gets deleted: they're simply skipped when announcements go out, and new subscriptions are refused until the key is back, so the old entries can't end up mixed with a new key.

Older installs kept the key unprotected. It's wrapped automatically the first time the Custom Crosspost action or the Settings window reads it.

**Q: Is there a limit on how many webhooks one person can register?**

Yes, **Max. Per User** (default 5). Keep it fairly low: every subscription means another HTTP call per announcement, so lots of heavy users can slow your announcements down or run into rate limits.

**Q: Can I restrict who's allowed to set up crosspost subscriptions?**

Yes. **User Role ID(s)** limits who can run `/crosspost setup` at all (empty = everyone). **Admin Role ID(s)** lets specific roles remove *other* members' subscriptions too, and open or close new ones.

---

## 🌐 Multi-Platform

**Q: Which platforms are supported?**

- **Broadcaster:** Twitch, YouTube and Kick.
- **Streamer Friends:** Twitch, YouTube, Kick, Owncast, Rumble, Picarto and TikTok.

YouTube needs a Google API key and Kick its own Client ID/Secret, in both features. Twitch is the odd one out: **Broadcaster** needs no separate credentials (it reads your own channel through Streamer.bot's Twitch connection), while **Streamer Friends** needs its own Twitch app Client ID/Secret, because it looks up *other* people's channels, which your own login can't. Picarto needs a Client ID/Secret (preferred, the token renews itself) or a bearer token, which is only used while one of the two is empty; Owncast needs nothing; each Rumble friend brings their own livestream API URL; TikTok needs nothing either, since it is read off the public live page.

**Q: Does going live on multiple platforms at once cause duplicate announcements or role conflicts?**

No. Each platform is tracked separately per streamer or friend, with its own session state, its own cooldowns and its own role list. The per-platform "ISLIVE Role ID(s)" fields exist precisely so multi-platform streamers get the right role for the right platform.

---

## 🔔 Update Notifications

**Q: How do I know when a new version is available?**

Opening the Settings window checks GitHub for the latest release. While that isn't the version you have installed, a popup offers to open the releases page each time the window opens, and the window's title says **🔔 Update available** with the new version. Home and About show the version you run.

**Q: Can I check for updates without opening the full Settings window?**

Not yet. The check runs when Settings opens.

---

# Is Better Stream Announcements an AI Slop?

Partially, yes. The project was developed with input from the Streamer.bot community and with the help of AI.
But I've put a lot of time, heart and soul into it, and my goal was a robust, reliable set of Discord integrations for streamers that's still easy to set up.
I understand that some people, especially IT-savvy ones, will dislike the project because of the AI involved, and I fully understand and respect that point of view.
I had a lot of fun making it, as with all my other projects. It was my way to "learn" coding while using AI for something worthwhile.

AI can introduce bugs, and I'm not a developer in the classic sense. But I spend a fair amount of time fixing every bug that turns up while testing.
If you still find bugs or have something to say, I'd love to hear it :)
