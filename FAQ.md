# ❓ Better Stream Announcements — FAQ

---

## 🛠️ Setup & Configuration

**Q: Where are the config files stored?**

Everything lives in one shared file under your Streamer.bot base directory:

```
Announcements/Discord/settings.json
```

You should never need to edit it by hand — use the built-in Settings GUI instead. Every BsA action (Broadcaster Online/Offline, Streamer Friends, Discord Bot Presence, Presence Role Assign, Self-Assign, Custom Crosspost) reads from this same file, so a change made in one tab of the GUI is picked up by every relevant script.

---

**Q: The Settings GUI won't open / crashes Streamer.bot entirely. What do I do?**

- Check the Streamer.bot log for error details and please report it on [GitHub Issues](https://github.com/aaskjer/Better-Stream-Announcements/issues).
- If `settings.json` is malformed, the GUI (and every other BsA action) will fail to load it. Delete the corrupt file and reopen the GUI — it will be recreated with defaults.
- If closing and reopening the Settings window, then toggling **Show Debug / Testing Sections**, ever crashed the whole process for you: that was a real bug (stale cross-thread UI elements) and is fixed as of the latest version. Update if you're still on an older release.

---

**Q: How do I reset all settings to defaults?**

Click **Reset to Defaults** (red button, bottom-left of the Settings window — visible on every tab). This rewrites `settings.json` with factory values and reopens the GUI.

---

**Q: I saved settings but nothing changed. Why?**

Most scripts (Streamer Friends, Broadcaster) simply reload `settings.json` at the start of their next `Execute()`, so the next trigger picks up your change immediately — no restart needed. **Discord Bot Presence** and **Presence Role Assign** go a step further: they actively watch `settings.json` for file changes and apply relevant updates (presence text, bot profile, role lists) live, usually within a second, without waiting for the next trigger at all.

---

## 📢 Broadcaster Announcements

**Q: What's the difference between the Channel-ID and Webhook options?**

- **Channel-ID** (with `BOT_Token` set) — the bot posts as itself, and can edit or delete the last announcement (e.g. to post an "offline" follow-up in the same message).
- **Webhook URL** — no bot required, but Discord webhooks can't edit or delete a previous message, so "Broadcaster Offline" always sends a brand-new message instead.

You only need one of the two, not both.

**Q: Why didn't my announcement post when I went live?**

Check, in order:
1. **Filter by Category / Filter by Title** (Broadcaster → Online → Filters) — if either is set, the announcement *only* fires when the current category exactly matches one of your allow-listed entries, or the title contains your keyword. Empty = no filtering.
2. A **20-minute cooldown** after your last announcement suppresses duplicate posts from a flappy reconnect. This is intentional and only bypassable via the hidden `BA_SkipCooldown` debug flag (Broadcaster → Online → Debug & Testing, only visible once "Show Debug / Testing Sections" is enabled in About → Advanced).
3. Missing/invalid `BOT_Token` + `BA_ChannelId`, and no `BA_WebhookUrl` as fallback.

**Q: Random Embed Color is on — do the per-platform hex colors still matter?**

No. **Random Embed Color** overrides the fixed `Twitch/YouTube/Kick Embed Color` fields entirely whenever it's enabled; the hex fields are only used when it's off.

**Q: What does "Auto Crosspost" do, and is it the same as "Custom Crosspost"?**

Different features, similar name:
- **Auto Crosspost** (`BA_AutoCrosspost` / `SF_AutoCrosspost`) triggers Discord's own native **Publish** button on your announcement, so servers that *follow* your announcements channel automatically get it too. Requires your Discord channel to be an Announcement-type channel.
- **Custom Crosspost** (see its own section below) is a completely separate opt-in system where individual Discord members register their *own* webhook to receive copies of announcements in their own server — no "following" required.

---

## 👥 Streamer Friends

**Q: My friend's ISLIVE role never gets assigned — why?**

The role assignment here is driven purely by the platform API (Twitch/YouTube/Kick), not Discord presence. Check:
1. Their platform login/handle/slug is listed in **Twitch/YouTube/Kick Friends**.
2. They're mapped to a Discord account under **Assign Role To** for that platform.
3. A role is configured — either the platform-specific **ISLIVE Role ID(s)** (which fully *overrides* "All Platforms" for that platform, it does **not** stack with it) or the shared **All Platforms** role.
4. If **Skip Offline/Invisible Discord Users** is enabled, the friend's Discord status must not be Offline/Invisible — and that check only works while **Presence Role Assign** (Discord Bot → Role Assign) is enabled, since that's the only script that reports live Discord status.

**Q: A friend reconnects quickly after a stream drop — does that spam a new announcement?**

No. **Reconnect Window (Minutes)** (default 15) treats a quick reconnect as the *same* session, not a new one — no duplicate post. **Session Cooldown (Hours)** (default 1) is a separate safety net that force-closes a stuck "live" session if no proper "ended" event ever arrived (e.g. after a Streamer.bot restart), so a friend can't get stuck marked live forever.

**Q: How do I stop it from posting a new message every time?**

Enable **Update Announcements** (Streamer Friends → Online) — it edits the *existing* live announcement in place (title/game/viewer changes, and the final "ended" state) instead of leaving it untouched and posting separately.

**Q: Where do webhook URLs go for Custom Crosspost subscribers — are they stored safely?**

Yes. Every subscriber webhook URL is AES-encrypted before being written to disk and only decrypted in memory when actually sending an announcement.

---

## 🎭 ISLIVE Role Assignment — Presence vs. API

There are **two independent systems** that can grant the same "ISLIVE" role, and they intentionally don't talk to each other about ownership.

**Q: What's the difference between "Presence Role Assign" and Streamer Friends' role assignment?**

| | Presence Role Assign | Streamer Friends |
|---|---|---|
| Source of truth | Discord's own "Streaming" activity (gateway presence) | Twitch/YouTube/Kick API polling |
| Role combination | **Additive** — "All Platforms" role + the matching per-platform role | **Override** — per-platform role replaces "All Platforms" entirely, doesn't stack |
| Reacts to | An observed live→offline *transition* only | Whatever the API poll currently reports |
| Removes a role it didn't grant? | Never | N/A (it's the authority for its own roles) |

**Q: My member intentionally sets themselves Offline/Invisible on Discord — will they still get roles?**

By default, yes — Streamer Friends doesn't care about Discord status at all, and Presence Role Assign generally can't detect a "Streaming" activity for an Invisible member anyway (Discord suppresses it). If you specifically want to *skip* granting the role to Offline/Invisible members, turn on **Skip Offline/Invisible Discord Users** in the relevant tab.

---

## 🤖 Discord Bot & Commands

**Q: What are the in-chat (Discord) commands?**

| Command | Who | Feature |
|---|---|---|
| `?bsahelp` | Everyone | Bot Presence — shows the full command list |
| `?reloadbotprofile` | Admin (`BOT_AdminUserIds`) | Bot Presence — re-applies username/avatar/banner/description |
| `?refreshpresence` | Admin | Bot Presence — re-sends the presence status to Discord |
| `?botinfo` | Admin | Bot Presence — connection/uptime/latency status embed |
| `?sfadd` / `?sfremove` | Everyone (unless role-restricted) | Self-Assign wizard — link/unlink your account |
| `?sfmy <platform>` | Everyone | Self-Assign — show your linked account |
| `?sflist <platform>` | Admin (`SF_SelfAssignAdminRoleIds`) | Self-Assign — list everyone's linked accounts |
| `?sftoggle <on\|off>` | Admin | Self-Assign — open/close self-assign for everyone |
| `?sfhelp` | Everyone | Self-Assign — shows the full command list |
| `?cc` | Everyone (unless role-restricted) | Custom Crosspost — start the subscription wizard |
| `?cclist` / `?ccremove <id>` | Everyone | Custom Crosspost — manage your own subscriptions |
| `?cctoggle <on\|off>` | Admin (`CC_AdminRoleIds`) | Custom Crosspost — open/close new subscriptions |
| `?cchelp` | Everyone | Custom Crosspost — shows the full command list |
| `?cancel` | Everyone | Cancels your active wizard |

`?` is your configured **Discord Prefix** (Bot Presence tab) — change it there if you'd rather use something else.

**Q: Why does `?reloadbotprofile` sometimes say it's rate-limited?**

Discord rate-limits profile updates. After any successful profile update, a short cooldown (~30s, longer if Discord explicitly says so via a 429 response) is enforced before the next one is allowed — this applies regardless of whether the update came from the command, the GUI Save, or the file watcher.

**Q: What Discord permissions/intents does the bot need?**

Under your bot's **Privileged Gateway Intents** in the Discord Developer Portal, enable **Presence Intent**, **Server Members Intent**, and **Message Content Intent**. Without Presence Intent specifically, Presence Role Assign has no data to work with at all.

---

## 🙋 Self-Assign

**Q: How do "Exclusive Role ID(s)" work for Self-Assign?**

They're comma-separated, not singular despite the label — a member needs to hold *any one* of the listed roles to use `?sfadd`/`?sfremove`. Leave empty to allow everyone.

**Q: I set an Admin Role but `?sftoggle` says I don't have permission.**

`?sftoggle` requires **both** a non-empty `SF_SelfAssignAdminRoleIds` list **and** the calling member holding one of those roles — an empty admin role list blocks the command for everyone, including the broadcaster, until at least one role ID is configured.

---

## 🔀 Custom Crosspost

**Q: What does a subscriber actually get?**

Whatever scope they picked during the `?cc` wizard: every Streamer Friends announcement on a chosen platform, one specific Streamer Friend, or the Broadcaster's own announcements — delivered to their own Discord webhook, in their own server.

**Q: Is there a limit on how many webhooks one person can register?**

Yes, **Max. Per User** (default 5). Keep it reasonably low — every extra registered webhook is an additional HTTP call per announcement, so a large number of heavy users can slow down or rate-limit your announcements.

**Q: Can I restrict who's allowed to set up crosspost subscriptions?**

Yes — **User Role ID(s)** restricts who can run `?cc` at all (empty = everyone), and **Admin Role ID(s)** lets specific roles manage *other* members' subscriptions, not just their own.

---

## 🌐 Multi-Platform

**Q: Which platforms are supported?**

Twitch, YouTube, and Kick, for both Broadcaster announcements and Streamer Friends. YouTube needs a Google API key and Kick needs its own Client ID/Secret in both features. Twitch is the odd one out: **Broadcaster** needs no separate credentials at all (it reads your own channel through Streamer.bot's existing Twitch connection), but **Streamer Friends** does need its own Twitch app Client ID/Secret, since it has to look up *other* people's channel data that your own login can't provide.

**Q: Does going live on multiple platforms at once cause duplicate announcements or role conflicts?**

No — each platform is tracked independently per streamer/friend, with its own session state, its own cooldown windows, and its own role list (per-platform "ISLIVE Role ID(s)" fields exist specifically so multi-platform streamers get the right role for the right platform).

---

## 🔔 Update Notifications

**Q: How do I know when a new version is available?**

Opening the Settings GUI checks GitHub for the latest release tag. If it's newer than what's installed, a popup offers to open the releases page.

**Q: Can I check for updates without opening the full Settings window?**

Not currently — the check runs as part of opening Settings.

---

# Is Better Stream Announcements an AI Slop?

Partially it is. This project has been developed with input from the Streamer.bot community and is supported by AI.
But I spend a lot of time putting heart and soul in it, and my goal was to create a robust and reliable set of Discord integrations for streamers that's still easy to set up.
I understand that people, especially IT-savvy people, will dislike the project because of the use of AI, and I absolutely understand and support their point of view.
But I had a lot of fun making it, as with all my other projects, so I used it to "learn" coding and used AI for something valuable.

AI can create bugs, and I am not a developer in classical terms. But I spend a reasonable amount of time fixing any bugs that occur while testing.
If you still find bugs or have something to say, please let me hear it :)
