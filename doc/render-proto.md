% Blottleships Manager-Renderer Protocol

Introduction
============

This document describes the structure and syntax of the protocol used
for communicating between the Blottleships Manager and Blottleships
tournament Renderers.

This protocol is one-way: Renderers should connect to the manager and
listen for the commands it sends.

Example
=======

    version 'RENDERER-VERSION-1.0'
    tournament start
    match start 'A' 'B'
    game start 'A' 'B'
    ship 'A' 1 1
    ship 'A' 1 2
        .
        .
        .
    hit 'A' 1 2
    miss 'A' 4 5
        .
        .
        .
    game end 'A' beats 'B'
        .
        .
        .
    game start 'A' 'B'
        .
        .
        .
    game end 'B' beats 'A'
    match end 'A' beats 'B
        .
        .
        .
    tournament end
    place 1 'A'
    place 2 'B'
