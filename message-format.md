# Message Format

Currently the message MUST be json, equivalent to the python dictionary below.
Different formats may be considered in the future, but they must be translatable
to the same format below.

The network packets consist of the binary header with the message directly
appended. There MUST NOT be other data at the end of the packet.

The message MAY be compressed with LZMA, the corresponding header flag signals
this to the receiver.

If encryption is enabled, a separate encryption header is inserted directly
after the binary header before the message, a format for this header not yet
determined

## Binary Header (W.I.P.)
```text
| 0                             | 1                             |
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| MAGIC     | TYPE              | FLAGS                         |

| 2                             | 3                             |
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| RESERVED: 0x00                | RESERVED: 0x00                |

 MAGIC: 0b101
 TYPE:  0b00000: Standard Message Type
 FLAGS: 0: compression: 0: None
                        1: LZMA
        1: encryption:  0: No
                        1: Yes
```

## Definitions

* **REQUIRED**: this entry must be present in every message
* **REQUIRED, can be omitted if empty**: the sending node must send the
  information in this entry if it has any. It does not need to include the entry
  if it would be emtpy
* **OPTIONAL**: the sending node can decide if it wishes to send this
  information i.e. it wishes to activate this feature

## Format

### Full update

```python
msg = {
    # the unique router id, may not contain whitespace or any of the brackets:
    # []()<>{}
    # REQUIRED
    "id": "NODE_ID",

    # strictly monotonically increasing, REQUIRED
    "seq": 12,

    # Type of the update, REQUIRED
    "type": "full",

    # Announced networks, REQUIRED, can be omitted if empty
    # A router with only a IPv4 address MUST NOT advertise IPv6 networks and
    # vice versa
    "networks": {
        "1.2.3.4/12": None,
        # Additional network information, OPTIONAL
        "2.3.4.5/23": {
            "retracted": True
        }
    },

    # IP addresses of sending interface, at least one is REQUIRED
    "addr-v4": "123.45.67.89",
    "addr-v6": "abcd:ef12::1234",

    # all reachable routes, REQUIRED, can be omitted if empty
    "routing-data": {
        # For every policy
        "POLICY": {
            # For every reachable node
            "NODE_ID": {
                "path": "NODE_ID>[1]>NODE_ID>[2]>NODE_ID",
            }
        }
    },

    # all nodes referenced in routing-data, REQUIRED, can be omitted if empty
    "node-data": {
        "NODE_ID": {
            "networks": {
                "1.2.3.4/12": None,
                "2.3.4.5/23": {
                    "retracted": True
                }
            }
        }
    },

    # all links referenced in "path" in the routing-data, REQUIRED, can be
    # omitted if empty
    "link-attributes": {
        "1": {
            "bandwidth": 100,
            "loss": 0.1
        },
        "2": {
            "bandwidth": 10,
            "loss": 0.01
        }
    },

    # request a full update from all or the specified nodes, OPTIONAL
    "request-full": any((True, ["NODE_ID"])),

    # requesting reflection, OPTIONAL
    "reflect": {
        "seq": 666  # Example of a reflection request
    },

    # all received reflection requests, REQUIRED, can be omitted if empty
    "reflected": {
        "NODE_ID": {
            "seq": 1234  # Example of a reflected entry
        }
    }
}
```

### Partial update

```python
msg = {
    # the unique router id, may not contain whitespace or any of the brackets:
    # []()<>{}
    # REQUIRED
    "id": "NODE_ID",

    # strictly monotonically increasing, REQUIRED
    "seq": 12,

    # Type of the update, REQUIRED
    "type": "partial",

    # The base of a partial update, REQUIRED"
    "partial-base": 5,

    # Announced networks
    # A router with only a IPv4 address MUST NOT advertise IPv6 networks and
    # vice versa
    # If present it replaces the previous data
    "networks": {
        "1.2.3.4/12": None,
        # Additional network information, OPTIONAL
        "2.3.4.5/23": {
            "retracted": True
        }
    },

    # IP addresses of sending interface
    # If present it replaces the previous data
    "addr-v4": "123.45.67.89",
    "addr-v6": "abcd:ef12::1234",

    # all reachable routes
    # If present it replaces the previous data on a NODE_ID basis, purging
    # entries with the value None
    "routing-data": {
        # For every policy
        "POLICY": {
            # For every reachable node
            "NODE_ID": {
                "path": "NODE_ID>[1]>NODE_ID>[2]>NODE_ID",
            },
            "NODE_ID": None,
        }
    },

    # all nodes referenced in routing-data
    # If present it replaces the previous data on a NODE_ID basis, purging
    # entries with the value None
    "node-data": {
        "NODE_ID": {
            "networks": {
                "1.2.3.4/12": None,
                "2.3.4.5/23": {
                    "retracted": True
                }
            }
        },
        "NODE_ID": None,
    },

    # all links referenced in "path" in the routing-data
    # If present this data will only be used for the paths defined in
    # routing-data, it does not replace previous data.
    "link-attributes": {
        "1": {
            "bandwidth": 100,
            "loss": 0.1
        },
        "2": {
            "bandwidth": 10,
            "loss": 0.01
        }
    },

    # request a full update from all or the specified nodes, OPTIONAL
    "request-full": any((True, ["NODE_ID", ...])),

    # requesting reflection, OPTIONAL
    "reflect": {
        "seq": 666  # Example of a reflection request
    },

    # all received reflection requests, REQUIRED, can be omitted if empty
    # If present replaces the previous data on a per NODE_ID basis
    "reflected": {
        "NODE_ID": {
            "seq": 1234  # Example of a reflected entry
        }
    }
}
```
