# IRC

In May 2021, a hostile takeover rendered `freenode` dead (I was using it before).

It is still "alive", but majority of users have moved to "Libera Chat", it is also now advertised in the pinned message on freenode discord. Arch linux channel is also available only on Libera Chat now.

There are couple cliens:
- `irssi` - console client for Linux
- `TheLounge` - web-based self-hosted IRC client
- `ZNC` - an irc **bouncer** - we need it to persist sessions between reconnects & allow connecting from multiple devices
  - It should be hosted on the persistent server: 24/7 PC or a DO server

## IRSSI Setup - raw

```
irssi
/set nick ewancoder
/network # Show predefined networks
/connect liberachat
/msg NickServ REGISTER <simple-password> my@email.com
# Then enter a command that came in the email.
/exit
```

Modify liberachat network to log in automatically:

```
/network modify -sasl_username <username> -sasl_password <password> -sasl_mechanism PLAIN liberachat
```

Open ~/.irssi/config, set autoconnect = "yes" in libera section.

## Bouncers

We can only connect one connection at a time. So if I connected on PC, and then went out, I can't check any messages until I'm back.

In order to circumvent this and make IRC behave like a modern messenger - we need a "bouncer" - an app, that is persistently connected to the server, and acting as a server itself, allowing you multiple connections. And you are connecting to this app instead.

However, hosting our own bouncer introduces security risk - we need proper SSL on it.

For now I've decided I will use TheLounge, until I can set up a proper ZNC setup.
