# Execution Binary Large Object Network

This document is the specification for the sub-protocol that supports on-demand availability of the binary large object (BLOB) lookup by their commitment.


## Overview

The BLOB network is a [Kademlia](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf) DHT that uses the [Portal Wire Protocol](./portal-wire-protocol.md) to establish an overlay network on top of the [Discovery v5](https://github.com/ethereum/devp2p/blob/master/discv5/discv5-wire.md) protocol.

The BLOB network consists of a mapping from a BLOB commitment to the BLOB.


### Data

#### Types

- BLOB


#### Retrieval

- BLOBs can be retrieved by BLOB commitments.


## Specification

### Distance Function

The BLOB network uses the stock XOR distance metric defined in the portal wire protocol specification.


### Content ID Derivation Function

??
The BLOB network uses the SHA256 Content ID derivation function from the portal wire protocol specification.


### Wire Protocol

The [Portal wire protocol](./portal-wire-protocol.md) is used as wire protocol for the BLOB network.


#### Protocol Identifier

As specified in the [Protocol identifiers](./portal-wire-protocol.md#protocol-identifiers) section of the Portal wire protocol, the `protocol` field in the `TALKREQ` message **MUST** contain the value of `0x500E`.


#### Supported Message Types

The BLOB network supports the following protocol messages:

- `Ping` - `Pong`
- `Find Nodes` - `Nodes`
- `Find Content` - `Found Content`
- `Offer` - `Accept`


#### `Ping.custom_data` & `Pong.custom_data`

??
In the BLOB network the `custom_payload` field of the `Ping` and `Pong` messages is the serialization of an SSZ Container specified as `custom_data`:

```
custom_data = Container(data_radius: uint256)
custom_payload = SSZ.serialize(custom_data)
```


### Routing Table 

The BLOB network uses the standard routing table structure from the Portal Wire Protocol.

### Node State

#### Data Radius

The BLOB network includes one additional piece of node state that should be tracked.  Nodes must track the `data_radius` from the Ping and Pong messages for other nodes in the network.  This value is a 256 bit integer and represents the data that a node is "interested" in.  We define the following function to determine whether node in the network should be interested in a piece of content.

```
interested(node, content) = distance(node.id, content.id) <= node.radius
```

A node is expected to maintain `radius` information for each node in its local node table. A node's `radius` value may fluctuate as the contents of its local key-value store change.

A node should track their own radius value and provide this value in all Ping or Pong messages it sends to other nodes.


### Data Types

#### Transaction Hash Mapping

```
BLOB_key     := Container(BLOB-commitment: Bytes32)
selector     := 0x00

content_key  := selector + SSZ.encode(BLOB_key)
content      := BLOB
```
