---
layout: post
title: "Follow-Up: Synchronized real-time Multipeer Connectivity"
date: 2015-06-07
---

In my last post i was talking about using Multipeer Connectivity in real-time multiplayer games, where a lot of network messages need to arrive timely, otherwise the players would experience lag. One of my ideas to speed up the delivery process was to switch from individual data packages to streaming. My hope was that keeping a stream open would stop messages from clogging up, then be delivered all at once.

After implementing this solution i am still not fully satisfied, but i did make my game work pretty good over WiFi. Bluetooth has the same problems as before, but that might be a limitation of Bluetooth. Since i had to work through multiple tutorials for streams with Multipeer Connectivity and none seemed to give a complete overview, here is what i did.

First, create an output stream for each peer (you could have a client only create a stream to the server, but lets assume that everyone can communicate with each other):

    var outputStreams: [MCPeerID: NSOutputStream!]

    /* [MCSessionDelegate] Remote peer changed state */
    func session(session: MCSession!, peer peerID: MCPeerID!, didChangeState state: MCSessionState) {    
      switch state {
      case .Connecting:
        // ...
      case .Connected:
        if let outputStream = session.startStreamWithName("streamName", toPeer: peer, error: error) {
          outputStream.delegate = self
          outputStream.scheduleInRunLoop(NSRunLoop.mainRunLoop(), forMode: NSDefaultRunLoopMode)
          outputStream.open()
          outputStreams[peerID] = outputStream
        }
      case .NotConnected:
        if let outputStream = outputStreams[peerID] {
          outputStreams.removeValueForKey(peerID)
          // we will close the stream via NSStreamDelegate
        }
      }
    }

Incoming streams are handled in another `MCSessionDelegate` method:

    /* [MCSessionDelegate] Received a byte stream from remote peer */
    func session(session: MCSession!, didReceiveStream inputStream: NSInputStream!, withName streamName: String!, fromPeer peerID: MCPeerID!) {
      inputStream.delegate = self
      inputStream.scheduleInRunLoop(NSRunLoop.mainRunLoop(), forMode: NSDefaultRunLoopMode)
      inputStream.open()
    }

Note that streams are scheduled on the main run loop, as i was unable to receive stream events from background threads. That does not mean it's not possible, i just didn't try yet.

Now we can stream data to our peers:

    func streamData(data: NSData, toPeer peer: MCPeerID) {
      // retrieve the stream for this peer
      if let streamForPeer: NSOutputStream = self.session.outputStreams[peer] {
        let bytesWritten = streamForPeer.write(UnsafePointer<UInt8>(data.bytes), maxLength: data.length)
        if bytesWritten != data.length {
          println("Error streaming data to peer '\(peer.displayName)': Dropped \(data.length - bytesWritten) bytes!")
        }
      } else {
        println("Unable to find stream for peer '\(peer.displayName)'")
      }
    }

Streamed data is received via the `NSStreamDelegate` method, which gets notified for various stream events. We are interested in `NSStreamEvent.HasBytesAvailable`, which is called whenever an input stream has unread data left.

    /* [NSStreamDelegate] Handle stream events */
    func stream(aStream: NSStream, handleEvent eventCode: NSStreamEvent) {
      switch eventCode {
      case NSStreamEvent.HasBytesAvailable:
        if let inputStream = aStream as? NSInputStream {
          var buffer = [UInt8](count: 1024, repeatedValue: 0)
          let bufferLength = inputStream.read(&buffer, maxLength: buffer.count)
          if bufferLength <= 0 {
            println("buffer length: 0; nothing read")
            return
          }
          // Do stuff with data in buffer
      case NSStreamEvent.EndEncountered:
        aStream.close()
        aStream.removeFromRunLoop(NSRunLoop.mainRunLoop(), forMode: NSDefaultRunLoopMode)
      default:
        // ...
      }
    }

Note that we do NOT loop and try to read all available data. This is not necessary, the event will be called again when not everything was read. For me looping caused threading problems. I then was able to read all data once, but never got notified about stream events again.

At first i tried streaming objects, which seemed to work, until i stopped receiving any stream events, which always happened after a few seconds into the game. I never found the cause for this problem, but i settled on sending strings, which simply contain a code for a game event, the sender name (because, unlike receiving data via MPC, when receiving data via streams we don't know the sender) and a message separator. Strings somehow work fine and turned out to be a lot simpler to parse.
