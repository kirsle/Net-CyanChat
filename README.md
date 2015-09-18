# NAME

Net::CyanChat - Perl interface for connecting to Cyan Worlds' chat room.

# SYNOPSIS

    use Net::CyanChat;

    my $cyan = new Net::CyanChat (
          host    => 'cho.cyan.com', # default
          port    => 1812,           # main port--1813 is for testing
          proto   => 1,              # use protocol 1.0
          refresh => 60,             # ping rate (default)
    );

    # Set up handlers.
    $cyan->setHandler (foo => \&bar);

    # Connect
    $cyan->start();

# DESCRIPTION

Net::CyanChat is a Perl module for object-oriented connections to Cyan Worlds, Inc.'s
chat room.

# NOTE TO DEVELOPERS

CyanChat regulars aren't fond of having chat bots in their room. The following
guidelines should be followed when connecting to `cho.cyan.com` (the official
CyanChat server):

    1. Don't create a bot that sends messages publicly to the chat room.
    2. CyanChat regulars don't like logging bots either (ones that would i.e. allow
       users to read chat transcripts online without having to participate in the
       chat room themselves).
    3. Don't do auto-shorah (or, automatically greeting members as they enter the
       chat room).

`Net::CyanChat` was created to aid in a Perl CyanChat client program. This is
how it should stay. Use this module to program an interactive chat client, not
a bot.

# VOCABULARY

For the sake of this manpage, the following vocabulary will be used:

    nick (or nickname):
      This is the displayed name of the user, as would be seen in the Who List
      and in messages they send.

    username (or fullname):
      This is the name that CyanChat refers to users internally by. It's the same
      as the nickname but it has a number in front. See "CyanChat Auth Levels"
      below for the meaning of the numbers.

# METHODS

## new (ARGUMENTS)

Constructor for a new CyanChat object. Pass in any arguments you need. Some standard arguments
are:

    host:    The hostname or IP of a CyanChat server.
             Default: cho.cyan.com
    port:    The port number that a CyanChat server is listening on.
             Default: 1812
             Note:    Port 1812 on cho.cyan.com is the standard official CyanChat
                      service. This server is very strict about the protocol. Sending
                      malformed packets will get you banned from the server.
             Note:    Port 1813 on cho.cyan.com is the development server. The server
                      is less strict about poorly formatted commands and won't ban your
                      IP when such happens. There still is a profanity filter though.
    proto:   The number of the CyanChat protocol to use (between versions 0 and 1).
             Default: 1
             Note:    See the CyanChat Developers link below for specifications of
                      the protocol versions.
    debug:   Debug mode. When active, client/server packets are displayed in your
             terminal window (or whereever STDOUT directs to).
    refresh: The "ping rate". The CyanChat server sometimes disconnects clients who
             are idle for long periods of time. There is no "ping" system implemented
             in the protocol. Many CC clients "ping" by sending an empty private
             message to an empty nickname. The refresh rate here determines how many
             seconds it waits between doing this.
             Default: 60

Returns a `Net::CyanChat` object. See ["CHO"](#cho) for tips about the official
CyanChat server.

## version

Returns the version number of the module.

## debug (MESSAGE)

Called by the module itself for debug messages.

## send (DATA)

Send raw data to the CyanChat server. This method is dangerous to be used
manually, as the official server doesn't tolerate malformed packets.
See ["CHO"](#cho) for details.

## setHandler (EVENT\_CODE => CODEREF)

Set up a handler for the CyanChat connection. See below for a list of handlers.

## connect

Connect to the CyanChat server. Will return undef and call your `Error`
handler if the connection fails; otherwise returns 1.

## start

Start a loop of do\_one\_loop's. Will break and return undef when a do\_one\_loop
fails.

## do\_one\_loop

Perform a single loop on the server. Returns undef on error; 1 otherwise.

## login (NICK)

After receiving a "Connected" event from the server, it is okay to log in now. NICK
should be no more than 20 characters and cannot contain a pipe symbol "|" or
a comma.

Of interest, it seems that on ["CHO"](#cho) you can call the login method more than
once. Effectually you send another "has logged in" event under the new nick,
without having sent a "has left" event. The server wasn't intended to behave this
way, so your mileage may vary.

## logout

Log out of CyanChat if you're currently logged in. **Never** call this method
if your object is not currently logged in. ["CHO"](#cho) will consider it a bad
packet and ban your IP.

## sendMessage (MESSAGE)

Broadcast a message publicly to the chat room. You must have logged in to the
chat room before you can call this method.

## sendPrivate (USERNAME, MESSAGE)

Send a private message to recipient USERNAME. You must be logged in first.

## getBuddies

Returns a hashref containing "who" (normal users) and "special" (Cyan & Guests).
Under each key are keys containing the Nicknames and their Addresses as values.
Example:

    {
      who => {
        'Kirsle' => '11923769',
      },
      special => {},
    };

## getUsername (NICK)

Returns the full username of passed in NICK. If NICK is not in the room, returns undef.
This function was historically named `getFullName`. The old function is an alias to
the new one.

## getAddress (NICK)

Returns the address to NICK. This is not their IP address; CyanChat encrypts their IP into this
address, and it is basicly a unique identifier for a connection. Multiple users logged on from the
same IP address will have the same chat address. Ignoring users will ignore them by address.

## protocol

Returns the protocol version you are using. Will return 0 or 1.

## ignore (USER), unignore (USER)

Ignore and unignore a username. This sends the "Ignore" event to the server as
well as keeping track internally that the user is ignored. Unignoring a user
probably doesn't work, so if your client needs to support this you should handle
it "manually" (ex. not display messages from this user if they're ignored but
don't actually ignore them through the CC protocol).

## nick

Returns the currently signed in nickname of the CyanChat object, or the blank
string if not logged in.

# ADVANCED METHODS

**WARNING:** These methods are very dangerous to use if you don't know what you're doing.
Don't call authenticate() unless you know for sure what the CyanChat admin password is,
and don't call promote() or demote() unless you are already authenticated as a CyanChat
staff user.

Calling the authenticate() command with the wrong password will most likely get you
banned from CyanChat, and calling promote() or demote() without being an admin user
will probably have the same effect.

In other words, **don't use these methods unless you know what you're doing!**
See ["CHO"](#cho) for more information.

**Note that these commands aren't official.** The section of the CyanChat protocol
dealing with administrative functionality isn't public knowledge. Instead, some
functionality has been guessed upon based on the gaps in the protocol specification.

The following functionality **does** work when the chat server is running on the
Net::CyanChat::Server module, but it may not work with other implementations
of a CyanChat server, and it will most likely not work with ["CHO"](#cho).

## authenticate (PASSWORD)

Authenticate your connection as a Cyan Worlds staff member. Call this method before
entering the chat room. If approved by the server, you will log in with an
administrative (cyan-colored) nickname.

## promote (USER)

Promote USER to a Special Guest. Special guests are typically rendered in orange
text and appear in a special "Cyan & Guests" who list.

## demote (USER)

Demote USER to a normal user level.

# HANDLERS

Handlers are implemented via the `setHandler` function. Here is an example
handler:

    $cc->setHandler (Welcome => \&on_welcome);

    sub on_welcome {
      my ($cyanchat, $message) = @_;
      print "[ChatServer] $message\n";
    }

The handlers are listed here in the format of "HandlerName (Parameters)". All
handlers receive a copy of the `Net::CyanChat` object and then any additional
parameters based on the nature of the event.

## Connected (CYANCHAT)

Called when a connection has been established, and the server recognizes your client's
presence. At this point, you can call CYANCHAT->login (NICK) to log into the chat room.

## Disconnected (CYANCHAT)

Called when a disconnect has been detected.

## Welcome (CYANCHAT, $MESSAGE)

Called after the server recognizes your client (almost simultaneously to Connected).
MESSAGE are messages that the CyanChat server sends--mostly just includes a list of the
chat room's rules.

Note that CyanChat is different to most traditional chat rooms: new messages are
displayed on the top of your chat history. The Welcome handler is called for each
message, and these messages will arrive in reverse order to what you'd expect,
since the new messages should be displayed above the previous ones. When all the
welcome messages arrive, then it can be read from top to bottom normally.

## Message (CYANCHAT, \\%INFO)

Called when a user sends a message publicly in chat. INFO is a hash reference
containing the following keys:

    nick:     The user's nickname.
    username: Their full username.
    address:  Their encoded IP address.
    level:    Their auth level (see "CyanChat Auth Levels" below).
    message:  The text of their message.

Example:

    $cc->setHandler (Message => sub {
      my ($cyan,$info) = @_;
      print "[$info->{nick}] $info->{message}\n";
    });

All of the following handlers have the same structure for their "INFO" parameter.

## Private (CYANCHAT, \\%INFO)

Called when a user sends a private message to your client.

## Ignored (CYANCHAT, $USER)

Called when a username has ignored us in chat. This is used in the standard chat
client so that you can perform a mutual ignore (your client automatically ignores
the remote user when they ignore you). The idea is that if a user is being abusive
in chat and everybody in the room ignores them, it will appear to them as though
everybody has left (because their client will have ignored everyone else too).

## Chat\_Buddy\_In (CYANCHAT, \\%INFO)

Called when a buddy enters the chat room. NICK, USERNAME, LEVEL, and ADDRESS are the same as in the
Message and Private handlers. MESSAGE is their join message (i.e. "<links in from comcast.net age>")

## Chat\_Buddy\_Out (CYANCHAT, \\%INFO)

Called when a buddy exits. MESSAGE is their exit message (i.e. "<links safely back to their home Age>"
for normal log out, or "<mistakenly used an unsafe Linking Book without a maintainer's suit>" for
disconnected).

## WhoList (CYANCHAT, @USERS)

This handler is called whenever a "35" (WhoList) event is received from the server. USERS is an array
of hashes containing information about all the users in the order they were received from the server.

Each item in the array is a hash reference with the following keys:

    nick:     Their nickname (ex: Kirsle)
    username: Their username (ex: 0Kirsle)
    address:  Their chat address
    level:    Their auth level (ex: 0)

## Name\_Accepted (CYANCHAT)

The CyanChat server has accepted your name.

## Error (CYANCHAT, $CODE, $STRING)

Handles errors issued by CyanChat. CODE is the exact server code issued that caused the error.
STRING is either an English description or the exact text the server sent.

Potential errors that would come up:

    00 Connection error
    10 Your name is invalid

# CYAN CHAT RULES

The CyanChat server strictly enforces these rules:

    Be respectful and sensitive to others (please, no platform wars).
    Keep it "G" rated (family viewing), both in language and content.
    And HAVE FUN!

    Termination of use can happen without warning!

See ["CHO"](#cho) for what exactly "Termination of use can happen without warning!" means.

# CYAN CHAT AUTH LEVELS

Auth levels (received as LEVEL to most handlers, or prefixed onto a user's FullName) are as follows:

    0 is for regular chat user (should be in white)
    1 is for Cyan Worlds employee (should be in cyan)
    2 is for CyanChat Server message (should be in lime green)
    4 is for special guest (should be in gold or orange)
    Any other number is probably a client error message (should be in red)

# CHO

This section of the manpage provides some tips for interacting with the standard
official CyanChat server, `cho.cyan.com`.

**Cho is picky about the protocol.** If you send a malformed packet to Cho, it
will most likely ban your IP address. Usually the first offense results in a
24 hour ban. Repeat offenses last longer and longer (possibly indefinitely).

**Cho has a bad language filter.** Sending a severe swear word results in an
instant, permanent ban. Less severe swear words result in a 24 hour ban, or a
longer ban if it's a repeat offense.

**Cho has a development server.** `cho.cyan.com` port `1813` is the development
server. The server here is tolerant of packet mistakes and will warn you about
them instead of banning your client. However, the bad language filter still
exists here.

# CHANGE LOG

    Version 0.07 - Sep 18 2015
    - Update documentation.

    Version 0.06 - Oct 24 2008
    - Broke backwards compatibility *big time*.
    - Removed the Chat_Buddy_Here method. It was useless and difficult to work with.
    - All the Message handlers (Message, Private, Chat_Buddy_In, Chat_Buddy_Out)
      now receive a hashref containing the details of the event, instead of
      receiving them in array format.
    - The Who Lists are now separated internally into "Who" (normal users)
      and "Special" (Cyan & Guests). Cyanites so rarely enter the CyanChat room
      that conflicts in nicknames between normal users and Cyanites was rare, but
      possible, and the previous version of the module wouldn't be able to handle
      that.
    - The function getBuddies returns a higher level hash dividing the users into
      the "who" and "special" categories (i.e. $ret->{who}->{Kirsle} = 11223135).
    - All functions return undef on error now, and 1 on success (unless another
      value is expected), instead of returning 0 on error.
    - Removed some leftover prints in the code from the last version.
    - Revised the POD to include some bits of example code, particularly around
      the HANDLERS section.
    - Added a new section to the POD to list some tips for interacting with the
      official chat server, Cho.
    - Cleared up some of the vocabulary in the POD, since "nicknames" and
      "usernames" are two different beasts, and it's important to know which one
      to use for any given method.
    - Included a command-line CyanChat client as a demonstration of this module
      (and to complement the `ccserver` script). The client requires Term::ReadKey
      and, on Win32, Win32::Console::ANSI (if you want ANSI colors).

    Version 0.05 - Jun  1 2007
    - Fixed the end-of-line characters, it now sends a true CrLf.
    - Added the WhoList handler.
    - Added the authenticate(), promote(), and demote() methods.

    Version 0.04 - Oct 24 2006
    - The enter/exit chat messages now go by the tag number (like it's supposed to),
      not by the contained text.
    - Messages can contain pipes in them and be read okay through the module.
    - Added a "ping" function. Apparently Cho will disconnect clients who don't do
      anything in 5 minutes. The "ping" function also helps detect disconnects!
    - The Disconnected handler has been added to detect disconnects.

    Version 0.03 - Oct  1 2006
    - Bug fix: the $level received to most handlers used to be 1 (cyan staff) even
      though it should've been 0 (or any other number), so this has been fixed.

    Version 0.01 - May 14 2005
    - Initial release.
    - Fully supports both protocols 0 and 1 of CyanChat.

# SEE ALSO

Net::CyanChat::Server

CyanChat Protocol Documentation: http://cho.cyan.com/chat/programmers.html

# AUTHOR

Noah Petherbridge, http://www.kirsle.net/

# COPYRIGHT AND LICENSE

    Net::CyanChat - Perl interface to CyanChat.
    Copyright (C) 2007-2015  Noah Petherbridge

    This program is free software; you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation; either version 2 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program; if not, write to the Free Software
    Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
