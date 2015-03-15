Blottleships Player-Manager Protocol
===================================


Introduction
------------

This document describes the structure and syntax of the protocol used
for communication between Blottleships Players and the Blottleships
Game Manager.

The protocol is designed to be straightforwad to implement in a
wide range of languages.

This document uses keywords as defined in RFC2119 and RFC6919.

Example
-------

In this example, `>` represents communications from Manager to Player,
whereas `<` represents communications from Player to Manager.

    < ready "1.0"
    > who
    < iam "A Blottleships Player"
    > describe
    < bio "Likes Pina Coladas and getting caught in the rain"
    > tournament begin
    < ok
    > match begin
    < ok
    > opponent "Another Blottleships Player"
    < ok
    > game begin
    < ok
    > where "destroyer"
    < place "destroyer" 0 0 0
    > where "cruiser"
    < place "cruiser" 1 1 3
        .
        .
        .
    > fire
    < shoot 4 4
    > hit
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
-------------

- All protocol communication is line-oriented plain text.
- There is a one-to-one correspondence between messages and lines.
- Blank lines are not permitted.
- (Almost) all communication is Manager-driven: the Manager sends
  one message, and the Player 


Structure
---------

With the sole execeptions of the opening `ready` message from the player,
and `error` messages from the manager, (of which more anon), all exchanges
involve the manager sending a single message to the
player, and the player responding with a single message.


Message Grammar
---------------

### Messages from Manager to Player

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

    tournament  ::= "tournament" <begin-end>

    match       ::= "match" <begin-end>

    game        ::= "game" <begin-end>

    where       ::= "where" <ship>

    fire        ::= "fire"

    bombarded   ::= "bombarded" <coordinate>

    outcome     ::= "hit" | "miss"

    begin-end   ::= "begin" | "win" | "lose""



### Messages from Player to Manager

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


### Common Grammar

    ship        ::= "destroyer"
                |   "cruiser"
                |   "battleship"
                |   "hovercraft"
                |   "carrier"

    placement   ::= <coordinate> r=NUMBER           [ r == r % 4 ]

    coordinate  ::= x=NUMBER y=NUMBER               [ x < 6 || y >= 6 ]



