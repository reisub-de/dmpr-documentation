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

## Binary Header
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

```python
msg = {
    # the unique router id, may not contain whitespace or any of the brackets:
    # []()<>{}
    # REQUIRED
    "id": "NODE_ID",

    # strictly monotonically increasing, REQUIRED
    "seq": 12,

    # Announced networks, OPTIONAL
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

    # all reachable routes, triggers full replacement in receiver when
    # present, OPTIONAL
    "routing-data": {
        # For every policy
        "POLICY": {
            # For every reachable node
            "TO_NODE_ID": {
                "path": "NODE_ID>[1]>NODE_ID>[2]>NODE_ID",
            }
        }
    },

    # Only partial upgrades, content not yet defined, OPTIONAL
    "partial-routing-data": {
        "tbd": "not yet defined"
    },

    # all nodes referenced in another section, REQUIRED, can be omitted if empty
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

    # all links referenced in "path" above, REQUIRED, can be omitted if empty
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

    # requesting reflection, OPTIONAL
    "reflector": {
        "asym-seq": 666  # Example of a reflection request
    },

    # all received reflection requests, REQUIRED, can be omitted if empty
    "reflections": {
        "NODE_ID": {
            "asym-seq": 1234  # Example of a reflected entry
        }
    }
}
```
