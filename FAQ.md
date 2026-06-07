## Does it work for platform XYZ?
Yes, `Better Stream Announcements` supports all platforms that are currently supported by `streamer.bot` (1.0.4).
*Trovo may not be present anymore in later versions since they are closing down*

## I don't want to use a discord bot for the announcements
You always have the option to use webhooks, but consider: they work with limitations. (no updates, editing, removing possible)

## Can i promote friends from different platforms
Yes, you can promote your friends from any platform that is currently supported.

## I don't want others to be able to add themselves on the promotion list
No worries, this feature is OFF by default, so nobody can use it right away and you can restrict using the commands to a specific role and/or channels.

## Does Streamer Friends exhaust any API or tokens?
In general, this is unlikely to happen, the default settings are carefully set to respect all limitations.
Anyways, YouTube is a special case here. Since it depends on how google treats YouTube API for "external users",
it expires because of how Google classifies your account and how your OAuth consent screen is configured,
not necessarily because Streamer Friends exhaust your tokens with frequent API calls.

## Can i add custom messages to my friends announcements?
Yes, but you're limited to the description field inside the embeds. In order to use it, you need to go to the sub-action `Custom Messages` of the
`[BsA] - Streamer Friends` action and add your custom messages as `Set Argument` sub-action there. 
