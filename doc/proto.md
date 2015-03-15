% Blottleships Player-Manager Protocol

Introduction
============

This document describes the structure and syntax of the protocol used
for communication between Blottleships Players and the Blottleships
Game Manager.

The protocol is intended to:

- Be straightforward to implement in a wide range of languages.
- Facilitate wire-level debugging using e.g., WireShark.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [RFC 2119].
      
The key words "MUST (BUT WE KNOW YOU WON'T)", "SHOULD CONSIDER",
"REALLY SHOULD NOT", "OUGHT TO", "WOULD PROBABLY", "MAY WISH TO",
"COULD", "POSSIBLE", and "MIGHT" in this document are to be
interpreted as described in [RFC 6919].

Example
=======

In this example, `>` represents communications from Manager to Player,
whereas `<` represents communications from Player to
Manager. Sequences of three vertical dots (`.`) indicate where a long
sequence of uninteresting communication has been elided.

    > hello
    < ready 'PLAYER-VERSION-1.0'
    > who
    < iam 'A Blottleships Player'
    > describe
    < bio 'Likes Pina Coladas and getting caught in the rain'
    > tournament begin
    < ok
    > match begin
    < ok
    > opponent 'Another Blottleships Player'
    < ok
    > game begin
    < ok
    > where destroyer
    < place destroyer 0 0 0
    > where cruiser
    < place cruiser 1 1 3
        .
        .
        .
    > fire
    < shoot 4 4
    > outcome 4 4 hit
    < ok
    > bombarded 5 5
    < ok
       .
       .
       .
    > lose
    < ok
    > game end
    < ok
    > match end
    < ok
    > tournament end
    < ok
    > goodbye


General Notes
=============

- All protocol communication is line-oriented plain text.
- There is a one-to-one correspondence between messages and lines.
- Blank lines are not permitted.
- (Almost) all communication is Manager-driven: the Manager sends
  one message, and the Player 


Structure
=========

With the sole exception of the `error` message, exchanges involve the
manager sending a single message to the player, and the player
responding with a single message.

Special Messages
================

### `hello`

**Manager:**

`hello`

**Player:**

`ready <version>`

The Manager MUST begin all interactions with the Player by sending the
message `hello`.

The Player MUST respond with the `ready` message, specifiying their
desired protocol version.

`<version>` is an arbitrary string specifying the protocol version the
client would like to use. If the server does not support this version,
an error results.

The version string corresponding to this document at its current state
is "PLAYER-VERSION-1.0".

If a future version of this protocol requires more sophisticated
version negotiation, it is suggested that the version string
"MAKE-ME-AN-OFFER" be used to specify protocols supporting version
negotiation (for the purposes of this message).

The Manager MUST NOT send the `hello` message after the first
interaction with the Player.

It is an error for the version string to be empty.

### `error`

**Manager:**

`error <message>`

OR

`no`

The Manager SHOULD send this message to the Player in the event of a
detectable error. The Manager OUGHT TO use the `error` form, to
provide useful messages to the Player, but MAY use the `no` form if
the Manager implementor is feeling lazy or if, in the considered
opinion of the Manager implementor, the Player' behaviour is so insane
as to not deserve an error message.

After this message has been sent, the session is in an error condition
and all bets are off.

The Player MUST NOT send error messages; the Manager defines correct
behaviour, and is in any case uninterested in the Player's opinion of
its mistakes.

### `goodbye`

**Manager:**

`goodbye`

The Manager OUGHT TO send this message to the Player before terminating
the connection: manners maketh man.

If the Player relies on the Manager to send this message for correct
termination, it is liable to be unpleasantly surprised one
day. Therefore, the Player SHOULD NOT require this message for correct
termination.

Command-Response Messages
=========================

Meatdata
--------

The Manager is permitted to send metadata messages in any order,
including duplications, may interleave metadata messages arbitrarily
at any point during a session, and is not required to send any of
them.

The Player MUST deal correctly with any combination of metadata
messages. The Manager MAY WISH TO randomise the order of metadata
messages to enfore correct behaviour on the part of Players.

### `who`

**Manager:**

`who`

**Player:**

`iam <name>`

The Manager sends the `who` message to a Player to determine the short
name of the Player.

The Player responds with the `iam` message to inform the manager of
its name.

It is an error for the name of a Player to be the empty string.

It is an error for the name of a Player to change between `iam`
messages during the same session.

### `describe`

**Server:**

`describe`

**Player:**

`bio <description>`

The Manager sends the `describe` message to a Player to determine the
long-form description of the Player.

The Player responds with the `bio` message to inform the Manager of
its long-form description.

It is an error for the `<description>` value to be empty. However, it
may be the empty string.

Players COULD vary their description between `bio` messages, in order
to enliven the otherwise quite dull existence of the manager.

Tournament Management
---------------------

### `tournament`

**Manager:**

`tournament begin`

OR

`tournament <position>`

**Player:**

`ok`

The Manager sends the `tournament begin` message to a Player to inform
them that a new tournament is starting.

The Manager sends the `tournament <position>` message to a Player to
inform them that the current tournament has ended, and of their
position in the tournament.

`<position>` SHOULD be a numeric value drawn from ℕ - {0}.

The Player MUST respond with `ok`.

### `match`

**Manager:**

`match begin`

OR

`match win`

OR

`match lose`

**Player:**

`ok`

The Manager sends the `match begin` message to a Player to inform
them that a new match is starting.

The Manager sends the `match win|lose` messages to a Player to inform
them that the current match has ended, and whether they won or lost.

The Player MUST respond with `ok`.

### `game`

**Manager:**

`game begin`

OR

`game win`
`game lose`

**Player:**

`ok`


The Manager sends the `game begin` message to a Player to inform
them that a new game is starting.

The Manager sends the `game win|lose` messages to a Player to inform
them that the current game has ended, and whether they won or lost.

The Player MUST respond with `ok`.

Game Messages
-------------

### `where`

**Manager:**

`where <ship>`

**Player:**

`place <ship> <x> <y> <rotation>`

The Manager sends the `where` message to a Player to determine the
location of each of their ships.

The Player responds, in each case, with a `place` message determining
the location of the ship in question. The `<x>` and `<y>` coordinates
locate the top-left corner of the bounding box of the ship shape; the
`<rotation>` parameter specifies that the 'standard' shape should be
rotated clockwise by `<rotation> * 90` degrees.

The ship shapes and board coordinate system are described in more
detail under [Ship Shapes](#ship-shapes) and
[Board Shape](#board-shape).

It is an error for the Player to respond with a different value for
`<ship>` from the one provided by the Manager.

It is an error for the specified ship placement not to be entirely
contained in the board.

It is an error for the specified ship placement to overlap a
previously-placed ship.

It is an error for the value of `<rotation>` not to be drawn from the
set {0, 1, 2, 3}.

### `fire`

**Manager:**

`fire`

**Player:**

`shoot <x> <y>`

The Manager sends the `fire` message to a Player to tell the player to
fire one shot.

The Player should respond with the `shoot` message to inform the
Manager of their desired target. `<x>` and `<y>` are board
coordinates; the board coordinate system is described in more detail
under [Board Shape](#board-shape).

It is not an error for the Player to specify the same target
coordinates multiple times.

It is an error for the Player to specify target coordinates outside
the board.

### `outcome`

**Manager:**

`outcome <x> <y> hit`

OR

`outcome <x> <y> miss`

**Player:**

`ok`

The Manager sends the `outcome` message to a Player to inform them of
the success or failure of their most recent shot.

The Player MUST be prepared to accept `outcome` messages relating to
any coordinate within the board at any point during a game.

The Manager OUGHT TO only send `outcome` messages immediately
following a `shoot` message.

The Player MUST respond with `ok`.

It is an error for the specified coordinates to be outside the board.

### `bombarded`

**Manager:**

`bombarded <x> <y>`

**Player:**

`ok`

The Manager sends the `bombarded` message to a Player to inform them
that their opponent fired a shot at the given coordinate.

The Player MUST be prepared to accept `bombarded` messages relating to
any coordinate within the board at any point during a game.

The Player MUST respond with `ok`.

It is an error for the specified coordinates to be outside the board.

Ship Shapes
===========

The 'canonical' orientation of each ship is shown by the template
below. `X` indicates a square the ship occupies, `-` indicates a
square within the bounding box of the ship which it does not occupy.

    ship[carrier]       -X-
    ship[carrier]       -X-
    ship[carrier]       -X-
    ship[carrier]       XXX

    ship[hovercraft]    -X-
    ship[hovercraft]    XXX
    ship[hovercraft]    X-X

    ship[battleship]    X
    ship[battleship]    X
    ship[battleship]    X
    ship[battleship]    X

    ship[cruiser]       X
    ship[cruiser]       X
    ship[cruiser]       X

    ship[destroyer]     X
    ship[destroyer]     X

The slightly odd presentation is intended to make it trivial to
mechanically extract the ship shapes from this document.

Board Shape
===========

The shape of the board is shown by the template below. `X` represents
a valid square within the board; `-` represents a square within the
bounding box of the board on which it is not valid to place a ship
(and which is not a valid board coordinate). The coordinate system
goes southeast from the top-left; the coordinates of the corners are
shown as `(x, y)` pairs.

    (0, 0)                (11, 0)
       X X X X X X - - - - - -
       X X X X X X - - - - - -
       X X X X X X - - - - - -
       X X X X X X - - - - - -
       X X X X X X - - - - - -
       X X X X X X - - - - - -
       X X X X X X X X X X X X
       X X X X X X X X X X X X
       X X X X X X X X X X X X
       X X X X X X X X X X X X
       X X X X X X X X X X X X
       X X X X X X X X X X X X
    (0, 11)               (11, 11)


Message Grammar
===============

Messages from Manager to Player
-------------------------------

    message     ::= <who>
                |   <describe>
                |   <goodbye>
                |   <error>
                |   <tournament>
                |   <match>
                |   <game>
                |   <where>
                |   <fire>
                |   <outcome>
                |   <bombarded>
        
    who         ::= "who"

    describe    ::= "describe"

    goodbye     ::= "goodbye"

    error       ::= "error" STRING
                |   "no"

    tournament  ::= "tournament" ("begin" | NUMBER)

    match       ::= "match" <begin-end>

    game        ::= "game" <begin-end>

    where       ::= "where" <ship>

    fire        ::= "fire"

    bombarded   ::= "bombarded" <coordinate>

    outcome     ::= "outcome" <coordinate> ("hit" | "miss")

    begin-end   ::= "begin" | "win" | "lose"



Messages from Player to Manager
-------------------------------

    message     ::= <ok>
                |   <ready>
                |   <iam>
                |   <bio>
                |   <place>
                |   <shoot>

    ok          ::= "ok"

    ready       ::= "ready" STRING

    iam         ::= "iam" STRING

    bio         ::= "bio" STRING

    place       ::= "place" <ship> <placement>

    shoot       ::= "shoot" <coordinate>


Common Grammar
--------------

    ship        ::= "destroyer"
                |   "cruiser"
                |   "battleship"
                |   "hovercraft"
                |   "carrier"

    placement   ::= <coordinate> r=NUMBER           [ r == r % 4 ]

    coordinate  ::= x=NUMBER y=NUMBER               [ x == x % 12 &&
                                                      y == y % 12 &&
                                                      x < 6 || y >= 6 ]



[RFC 2119]: https://tools.ietf.org/html/rfc2119
[RFC 6919]: https://tools.ietf.org/html/rfc6919
