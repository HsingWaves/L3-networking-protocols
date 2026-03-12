| **Feature**      | **Route Distinguisher (RD)**                  | **Route Target (RT)**                       |
| ---------------- | --------------------------------------------- | ------------------------------------------- |
| **Primary Goal** | Uniqueness                                    | Connectivity/Policy                         |
| **Function**     | Makes an IPv4 address a unique VPNv4 address. | Determines which VRF receives which routes. |
| **Format**       | 64-bit value (e.g., `100:1`)                  | 64-bit BGP Extended Community               |

This describes the role of a **VRF (Virtual Routing and Forwarding)** instance. While Route Distinguishers (RDs) are configured within a VRF, it is the VRF itself that allows the coexistence of multiple routing tables.



L**abels (LDP or RSVP)** are used for label bindings to facilitate packet forwarding through the MPLS core. RDs are part of the control plane (BGP), not the data plane forwarding labels.



The primary purpose of a Route Distinguisher is to prepend a 64-bit value to a standard 32-bit IPv4 prefix. This creates a 96-bit **VPNv4 address**, ensuring that even if two different customers use the same private IP space (e.g., `10.1.1.0/24`), their routes remain unique and distinguishable as they are carried across the MPLS provider core via MP-BGP.



 This describes **Route Targets (RTs)**. RTs are BGP extended communities that determine which routes are imported into or exported from specific VRFs.