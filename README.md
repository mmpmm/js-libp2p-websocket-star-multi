# libp2p-websocket-star

[![](https://img.shields.io/badge/made%20by-mkg20001-blue.svg?style=flat-square)](http://ipn.io)
[![Build Status](https://travis-ci.org/mkg20001/js-libp2p-websocket-star.svg?style=flat-square)](https://travis-ci.org/mkg20001/js-libp2p-websocket-star)

![](https://raw.githubusercontent.com/libp2p/interface-connection/master/img/badge.png)
![](https://raw.githubusercontent.com/libp2p/interface-transport/master/img/badge.png)

> Allows to listen on multiple websocket-star servers while ignoring offline ones

## Description

`libp2p-websocket-star-multi` allows to listen on multiple websocket-star servers while ignoring offline ones

**Note:** This module uses [pull-streams](https://pull-stream.github.io) for all stream based interfaces.

## Usage

### Installation

```bash
> npm install libp2p-websocket-star
```

### API

[![](https://raw.githubusercontent.com/libp2p/interface-transport/master/img/badge.png)](https://github.com/libp2p/interface-transport)

Currently websocket-star-multi uses the /libp2p-webrtc-star/ address prefix as we don't have our own just yet.

### Example

```js
const libp2p = require("libp2p")
const Id = require("peer-id")
const Info = require("peer-info")
const multiaddr = require("multiaddr")
const pull = require('pull-stream')

const WSStarMulti = require('libp2p-websocket-star-multi')

Id.create((err, id) => {
  if (err) throw err

  const peerInfo = new Info(id)
  peerInfo.multiaddrs.add(multiaddr("/libp2p-webrtc-star"))
  const ws = new WSStarMulti({
    servers: [ // servers are Multiaddr[]
      "/libp2p-webrtc-star/ip4/148.251.206.162/tcp/9090/ws/",
      "/libp2p-webrtc-star/ip4/136.243.31.32/tcp/4278/ws/",
      "/libp2p-webrtc-star/dns4/localhost/ws/"
    ],
    //ignore_no_online: true, //enable this to prevent wstar-multi from returning a listen error if no servers are online
    id // the id is required for the crypto challenge
  })
  const modules = {
    transport: [
      ws
    ],
    discovery: [
      ws.discovery
    ]
  }
  const swarm = new libp2p(modules, peerInfo)

  swarm.handle("/test/1.0.0", (protocol, conn) => {
    pull(
      pull.values(['hello']),
      conn,
      pull.map(s => s.toString()),
      pull.log()
    )
  })

  swarm.start(err => {
    if (err) throw err
    swarm.dial(peerInfo, "/test/1.0.0", (err, conn) => {
      if (err) throw err
      pull(
        pull.values(['hello from the other side']),
        conn,
        pull.map(s => s.toString()),
        pull.log()
      )
    })
  })
})
```

Outputs:
```
hello
hello from the other side
```

### This module uses `pull-streams`

We expose a streaming interface based on `pull-streams`, rather then on the Node.js core streams implementation (aka Node.js streams). `pull-streams` offers us a better mechanism for error handling and flow control guarantees. If you would like to know more about why we did this, see the discussion at this [issue](https://github.com/ipfs/js-ipfs/issues/362).

You can learn more about pull-streams at:

- [The history of Node.js streams, nodebp April 2014](https://www.youtube.com/watch?v=g5ewQEuXjsQ)
- [The history of streams, 2016](http://dominictarr.com/post/145135293917/history-of-streams)
- [pull-streams, the simple streaming primitive](http://dominictarr.com/post/149248845122/pull-streams-pull-streams-are-a-very-simple)
- [pull-streams documentation](https://pull-stream.github.io/)

#### Converting `pull-streams` to Node.js Streams

If you are a Node.js streams user, you can convert a pull-stream to a Node.js stream using the module [`pull-stream-to-stream`](https://github.com/pull-stream/pull-stream-to-stream), giving you an instance of a Node.js stream that is linked to the pull-stream. For example:

```js
const pullToStream = require('pull-stream-to-stream')

const nodeStreamInstance = pullToStream(pullStreamInstance)
// nodeStreamInstance is an instance of a Node.js Stream
```

To learn more about this utility, visit https://pull-stream.github.io/#pull-stream-to-stream.