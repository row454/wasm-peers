<img style="display: block; margin-left: auto; margin-right: auto" src="logo.png" alt="wasm-peers logo">

# wasm-peers

This crate provides an easy-to-use wrapper around WebRTC and DataChannels for a peer-to-peer connections.

## Overview

As creator of agar.io famously stated [WebRTC is hard](https://news.ycombinator.com/item?id=13264952).
This library aims to help, by abstracting away all the setup, and providing a simple way to send
and receive messages over the data channel.

It's as easy as providing address to a signaling server instance from
[accompanying crate](https://github.com/wasm-peers/wasm-peers/tree/main/signaling-server) and specifying two callbacks.
One that specifies what should happen when a connection is established, and one for when a message is received.
After that you can send messages back and forth without worrying about the implementation details.

Library contains three network topologies, `one-to-one`, which creates an equal connection between two peers,
`one-to-many`, which specifies a host and arbitrary number of clients
and `many-to-many` that creates connection for each pair of peers and allows sending messages to any of them.


For a "production ready" apps built with this
library check out either [Live Document](https://github.com/wasm-peers/live-document#readme) or [Footballers](https://github.com/wasm-peers/footballers#readme).

## Example

This example shows two peers sending `ping` and `pong` messages to each other.

```rust
use wasm_peers::ConnectionType;
use wasm_peers::one_to_one::NetworkManager;
use web_sys::console;

// there must be a signaling server from accompanying crate running on this port
const SIGNALING_SERVER_URL: &str = "ws://0.0.0.0:9001/one-to-one";

fn main() {
    // there must be some mechanism for exchanging session ids between peers
    let session_id = SessionId::new("some-session-id".to_string());
    let mut peer1 = NetworkManager::new(
        SIGNALING_SERVER_URL,
        session_id.clone(),
        ConnectionType::Stun,
    ).unwrap();

    let peer1_clone = peer1.clone();
    let peer1_on_open = move || peer1_clone.send_message("ping!").unwrap();
    let peer1_on_message = {
        move |message| {
            console::log_1(&format!("peer1 received message: {}", message).into());
        }
    };
    peer1.start(peer1_on_open, peer1_on_message).unwrap();

    let mut peer2 = NetworkManager::new(
        SIGNALING_SERVER_URL,
        session_id,
        ConnectionType::Stun,
    ).unwrap();
    let peer2_on_open = || { /* do nothing */ };
    let peer2_clone = peer2.clone();
    let peer2_on_message = {
        let peer2_received_message = peer2_received_message.clone();
        move |message| {
            console::log_1(&format!("peer2 received message: {}", message).into());
            peer2_clone.send_message("pong!").unwrap();
        }
    };
    peer2.start(peer2_on_open, peer2_on_message).unwrap();
}
```

For examples of other topologies check out the [docs](https://docs.rs/wasm-peers/latest/wasm_peers/).

## Authors

Tomasz Karwowski  
[LinkedIn](https://www.linkedin.com/in/tomek-karwowski/)

## Version History

* 0.3
    * Initial release to the public

## License

This project is licensed under either of

* Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
* MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as
defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.

## Acknowledgments

These projects helped me grasp WebRTC in Rust:

* [Yew WebRTC Chat](https://github.com/codec-abc/Yew-WebRTC-Chat)
* [WebRTC in Rust](https://github.com/Charles-Schleich/WebRTC-in-Rust)

Also, special thanks to the guys with whom I did my B.Eng. thesis.