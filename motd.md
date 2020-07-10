## The Random Startup Message (motd)

A classic feature of many terminal interfaces, the "Message of the Day" (motd).

In VisiData, a short (<50 characters) message displayed in the status on startup.
The list of messages is stored on visidata.org, with a different file per version, so upgrades can be announced.
The file is cached for 24 hours, so only one request is made per day.

Otherwise, a random entry from the motd file is displayed on startup.

The `motd_url` option can set a custom url to use for these messages.
To disable them entirely, add this to your .visidatarc:

    options.motd_url = ''

### Related

- [How to Visualize VisiData Usage Data Using VisiData]()
