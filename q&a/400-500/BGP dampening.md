| **Parameter**         | **Default Value** | **Description**                                              |
| --------------------- | ----------------- | ------------------------------------------------------------ |
| **Half-life**         | 15 minutes        | The time it takes for the penalty to be reduced by half.     |
| **Reuse Limit**       | 750               | The penalty level at which a suppressed route becomes usable again. |
| **Suppress Limit**    | 2000              | The penalty level at which a route is suppressed.            |
| **Max Suppress Time** | 60 minutes        | The maximum time a route can be suppressed.                  |

###  BGP Dampening Parameters

If you were to customize the dampening (as seen in the route-map options), here is what those numbers represent:

**bgp dampening:** This command enables the default route dampening mechanism. It assigns a "penalty" to a route each time it flaps. Once the penalty exceeds a certain threshold, the router stops advertising the route (dampens it) until it stabilizes.

**Scope:** Enabling it globally under the BGP process is the standard way to protect the BGP table from unstable prefixes.