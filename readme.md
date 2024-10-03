# DHCP schema

We assume a network with **two** DHCP servers (**Server1** and **Server2**). The client is initially connected to Server1, but due to server outages and other circumstances, the client will interacts with Server2.

1. **Initial connection** to Server1.
2. **Disconnection and fallback** to Server2 after Server1 becomes unavailable.
3. **IP lease renewal** and reconnection events during the session.
4. Final **disconnection** after the client completes using the IP.

## Sequence Diagram (Mermaid)


```mermaid
sequenceDiagram
    participant Client
    participant Server1
    participant Server2

    %% Phase 1: Initial connection to Server1
    Client->>Server1: DHCPDISCOVER
    Server1-->>Client: DHCPOFFER
    Client->>Server1: DHCPREQUEST
    Server1-->>Client: DHCPACK
    note right of Client: Connected for 1 hour

    %% Phase 2: Network outage (Server1 goes down)
    note over Client,Server1: 3-minute network outage
    Client->>Server1: DHCPREQUEST (attempt to renew)
    Server1--x Client: No response (Server1 down)
    
    %% Phase 3: Client switches to Server2 after Server1 is down
    Client->>Server2: DHCPDISCOVER
    Server2-->>Client: DHCPOFFER
    Client->>Server2: DHCPREQUEST
    Server2-->>Client: DHCPACK
    note right of Client: Connected for 12 hours

    %% Phase 4: IP lease renewal after 8 hours
    Client->>Server2: DHCPREQUEST (Renew lease)
    Server2-->>Client: DHCPACK
    note right of Client: Lease renewal successful

    %% Phase 5: 1-hour disconnection
    note over Client: Disconnected for 1 hour

    %% Phase 6: Reconnection to Server2
    Client->>Server2: DHCPREQUEST (Reuse IP)
    Server2-->>Client: DHCPACK
    note right of Client: Connected for 5 hours

    %% Phase 7: Permanent disconnection
    note over Client: Permanently disconnected
