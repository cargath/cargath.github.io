---
layout: post
title: "Synchronized Multipeer Connectivity in real-time games"
date: 2015-05-25
---

I’m currently working on a peer-to-peer multiplayer game as a side-project. I first tried [PeerKit](https://github.com/jpsim/PeerKit), but it’s too basic for my use in its current state. I need to make sure that all players are presented with the same state of the game at all times. I decided to use Multipeer Connectivity and implement my own framework, something similar to PeerKit, but something that ensures all messages arrive in the same order for all peers.

## Automatically determining a server

I ended up with a very simple client-server model, where the server is determined automatically. I do this by simply sorting all connected peers (including the local peer) by display name and picking the first peer from the sorted list (memo to self: check edge-cases where people use the same display names).

Now i’m handling all game events through the networking framework:

- If the local peer is the server, i broadcast an event to everyone else and in case of a success use the same delegate method that would handle incoming data to feed the event back to the game.
- If the local peer is a client, i attach a boolean value as metadata (so the server knows to distribute this event among the peer group) and send the event to the server, which will send it back, at which point i can assume it arrived in the correct order.

## Current problems

I was testing my solution with 3 devices (two iOS devices and a simulator) and apart from a few cases where someone dropped out of the session for some reason, everything was perfectly in sync. Unfortunately Multipeer Connectivity seems pretty slow, doubly so because a client has to go to the server and back before being able to react to an event. I’m using an NSTimer with an interval of 0.22 as a game loop, and a lot of times network messages keep clogging up somewhere, then arrive all at once. It’s not even as if i’m sending a lot of data - all i do is send an enum that tells an instance of the game it’s time to perform an update to the game logic.

## Possible solutions

- Using streams instead of sending individual NSData packets might help speeding up message delivery.
- The server could resolve the game logic and provide the clients with the complete state instead of only sending update commands. Then network messages would get larger, but the clients could use their own game loop and only react to messages from the server if they detect something’s gone out of sync, which would do away with the lag.
