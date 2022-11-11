# Libp2p Tutorial

`Libp2p` is a library for creating decentralized peer-to-peer network.

</br>

## Concepts and terminologies

- How a peer-to-peer network works

    A p2p system is generally composed of three or four elements:

    - Each node on the network is identified by its public key.
    - A peer discovery system: how we join the network.
    - Optionally, a publish-subscribe system: how we broadcast events over the
      network.
    - Optionally, a distrbuted hash table (DHT): how we store data.

    </br>

- Network (peer) identities

    Every node in the P2P network always has an identifier called `PeerId`, you
    connect to another P2P node by its `PeerId`.

    `PeerId` derived from their public key (any asymmetric encryption system like):

    - RAS
    - ED25519
    - SEC256k1
    - ECDSA

    </br>

    Here is an example for the `PeerId`

    ```rust
    PeerId("12D3KooWGnRcFKtaH4RzFDn1TwFcQ82jdptRYot2FJ8NfNrDZtxe")
    ```

    </br>

- Multiaddress

    Once we have a P2P network, a `PeerID`, we need a to way find each other on
    the P2P network, it's called `addressing`. This is why `Multiaddr` comes to
    play.

    A `Multiaddr` is a self-describing network address and protocol stack that
    is used to establish connections to peers. It's just a string, easy to read
    and understand. For example:

    ```rust
    // Peer that can be reached via localhost loopback (IPv4) and UDP on port 1234
    "/ip4/127.0.0.1/udp/1234"

    // Peer that can be reached via local network (IPv4) and TCP on port 24915
    "/ip4/192.168.178.25/tcp/24915"
    ```

    You should read from left to right to understand a reachable transport address.

    For example, `"/ip4/192.168.178.25/tcp/24915"` means 2 parts:

    - `"/ip4/192.168.178.25"`
    - `"tcp/24915"`

    Left is bottom protocol `IP (Internet Protocol)`, right is top layer protocol
    `TCP (Transmission Control Protocol)`.

    </br>

    So `"/ip4/7.7.7.7/tcp/80/ws"` means use `WS (WebSocket)` on top of `TCP`.

    Also, `"/ip4/7.7.7.7/ws"` means `WebSocket` on `TCP`, as `UDP` can't be upgraded
    to `WS`.

    </br>


    For the `PeerId` in `p2p` protocol like below:

    ```rust
    "/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N"
    ```

    It does not give you enough addressing information to locate a peer on the
    network; it is not a transport address. That's why usually you should combine
    a `PeerId` with the transport address like this:

    ```rust
    // Peer that can be reached via the unique p2p peer id on the particular
    // public ip and TCP on port 4242, and combine with the peer id. You can
    // dial that peer over a TCP/IP transpor immediately.
    "/ip4/7.7.7.7/tcp/4242/p2p/QmYyQSo1c1Ym7orWxLYvCrM2EmxFTANf8wXmmE7DWjhx5N"
    ```

    So you can use `Multiaddr` to dial another peer.

    </br>

    More multiadd examples:

    ```
    "/dns4/ams-2.bootstrap.libp2p.io/tcp/443/wss/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb"
    "/ip6/2604:1380:2000:7a00::1/tcp/4001/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb"
    "/ip4/147.75.83.83/tcp/4001/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb"
    "/ip6/2604:1380:2000:7a00::1/udp/4001/quic/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb"
    "/ip4/147.75.83.83/udp/4001/quic/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb"
    "/dns6/ams-2.bootstrap.libp2p.io/tcp/443/wss/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb"
    ```

    </br>

    [Addressing in libp2p](https://github.com/libp2p/specs/blob/master/addressing/README.md)
    is the full and detailed documentation you won't want to miss.

    </br>


- Transport

    `Transport` controls **How to send bytes** or say use what protocol implementation
    to send bytes on an abstraction communication channel.

    ```rust
    pub trait Transport {
        fn listen_on() -> Result<>;
        fn remove_listener() -> bool;
        fn dial() -> Result<>>;
        fn dial_as_listener() -> Result<>;
        fn poll() -> Poll<>;
        fn address_translation() -> Option<>;

        fn boxed(self) -> Boxed<Self::Output>
        fn map<F, O>(self, f: F) -> Map<Self, F>
        fn map_err<F, E>(self, f: F) -> MapErr<Self, F>
        fn or_transport<U>(self, other: U) -> OrTransport<Self, U>
        fn and_then<C, F, O>(self, f: C) -> AndThen<Self, C>
        fn upgrade(self, version: Version) -> Builder<Self>
    }
    ```

    A `listener` is waiting for the incoming connection, a `dialer` is going to
    connect to the remote peer,  it's the same concept in the traditional roles
    of `server` and `client`.

    </br>

    Beyond the `TCP/UDP`, libp2p support other protocol implementation, for example:

    - [**QUIC**](https://docs.libp2p.io/concepts/transports/quic/)

    - [**WebTransport**](https://docs.libp2p.io/concepts/transports/webtransport/)

    </br>

- Network behaviours

    A `NetworkBehaviour` defines the behaviour of the local node on the network.

    In contrast to `Transport` which defines how to send bytes on the network,
    `NetworkBehaviour` defines what bytes to send and to whom.

    Each protocol (e.g. `libp2p-ping`, `libp2p-identify` or `libp2p-kad`)
    implements `NetworkBehaviour`. Multiple implementations of `NetworkBehaviour`
    can be composed into a hierarchy of `NetworkBehaviours` where parent
    implementations delegate to child implementations.

    Finally the root of the `NetworkBehaviour` hierarchy is passed to `Swarm`
    where it can then control the behaviour of the local node on a libp2p network.

    </br>

- Swarm


## Install dependencies

- `protoc`

    Make sure to install `protoc` binary, otherwise, you can't compile your
    program. You can find the install link from
    [here](https://github.com/hyperium/tonic#dependencies).

    </br>




