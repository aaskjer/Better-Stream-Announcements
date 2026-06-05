# Import

<p align="center"><img width="281" height="103" alt="grafik" src="https://github.com/user-attachments/assets/2c17f5a0-15f3-4396-a277-a66638017fb3"></p>

Boot up `streamer.bot` and click on `Import`.

<p align="center"><img width="900" height="674" alt="grafik" src="https://github.com/user-attachments/assets/67b184cd-4799-4dba-9b68-6ddf84f2e5b5"></p>

Drag the `BetterStreamAnnouncements.sb` into the import window. Click on `Yes/Ok` on the following popup messages to succesfully import the project.

<p align="center"><img width="1280" height="720" alt="grafik" src="https://github.com/user-attachments/assets/17a5d427-e982-49ca-8f0e-57060a0d6ff4"></p>

Enable the timer seen in the screenshot, so the script will check for your friends from time to time.


---

# Install
<p align="center"><img width="1280" height="720" alt="grafik" src="https://github.com/user-attachments/assets/ac21effa-0e6b-4f24-b7e9-fb230e09c20d"></p>

Go to `[BsA] - ..Settings..` and trigger the `Test` trigger to open up the GUI.

## Broadcaster
### Online
<p align="center"><img width="634" height="661" alt="grafik" src="https://github.com/user-attachments/assets/7c63b7b3-d08d-4d2a-8a41-dc7a2cb7af47"></p>

Adjust as you like, this page is only for you as broadcaster sending out your stream announcement.

### Offline
<p align="center"><img width="634" height="661" alt="grafik" src="https://github.com/user-attachments/assets/cd525abf-d482-433d-8894-618eab6e0f93"></p>

Same as `Online` but you have the option to modify or send the offline announcement to another discord channel by adding a different discord channel ID in the first input field.
***Hover with the mouse over the options to see more details***

## Streamer Friends
### Promotion
<p align="center"><img width="634" height="661" alt="grafik" src="https://github.com/user-attachments/assets/24f95fcb-8694-4f65-bf54-79824184698f"></p>

In this tab, you can add your friends so BsA promotes them. You can additionally add discord role IDs so they will get assigned roles when they are live. The roles get removed
when they are detected as offline.

### Online
<p align="center"><img width="634" height="661" alt="grafik" src="https://github.com/user-attachments/assets/42570b93-bab2-4c2f-9309-076c50dd24c7"></p>

Basically the same as in the `Broadcaster` Section. Add unique custom descriptions to your friends announcements, in the action `[BsA] - Streamer Friends` add an argument to the

<p align="center"><img width="1280" height="720" alt="grafik" src="https://github.com/user-attachments/assets/0486c37c-4800-4be2-b578-31eee7376f9b"></p>

sub-action group `Custom Messages`. `Variable Name` must be: `%StreamerFriendNameMessage%` (case-sensitive)

### Offline
<p align="center"><img width="634" height="661" alt="grafik" src="https://github.com/user-attachments/assets/2b241d52-9c90-42c2-bc5e-0846af772870"></p>

Streamer Friends are restricted to a single channel in discord. If you're using your own discord bot, `BsA` will be able to update,
edit to offline and even remove stale announcements, so your promotion channel stays clean.

## Platforms
### Broadcaster
<p align="center"><img width="634" height="661" alt="grafik" src="https://github.com/user-attachments/assets/b5c2bcb4-6471-40d1-86ee-745baa4550cb"></p>

Here you can add from where `BsA` get it's information an you can decide if you use your own discord bot or a webhook.
***Hover with the mouse over the options to see more details***
Note: Using the webhook comes with limitations: updating and editing the last current announcement or removing old announcements aren't possible.

### Streamer Friends
<p align="center"><img width="634" height="661" alt="grafik" src="https://github.com/user-attachments/assets/326276b0-2405-4b4d-8bf9-3c5690382a5a"></p>

Same for `Streamer Friends` you can add the same details for your discord bot or webhook or use different ones.
***Hover with the mouse over the options to see more details***

### Discord Bot
<p align="center"><img width="634" height="661" alt="grafik" src="https://github.com/user-attachments/assets/77c95f90-9472-4338-93a7-1d342e9691f8"></p>

You're using your own discord bot exclusively for `BsA`? Then you can adjust it's discord presence here.
Also you find settings here so users from your discord cann assign themselves to `BsA` if they want to get promoted when they go live.
Add an exclusive role (e.g. `Members`) if you don't want literally `anybody` to be able to add themselves to the list.

#### Create a Discord bot
* Go to [discord.com/developers/applications](https://discord.com/developers/applications) and click `New Application`
* Under `Bot` go to `Privileged Gateway Intents` enable `Message Content Intent` and save it
* Under `Bot` click `Reset Token` and save it somewhere
* Go to `OAuth2` → `OAuth2 URL-Generator`, select scopes: `bot`
* Check `Administrator` in the now appeared `Bot-Permissions` 
* Open the generated URL, select your server and click Authorize

  
