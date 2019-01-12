# cics-ipic-heartbeat-disabler

A method to disable outbound IPIC heartbeats in a CICS region.

## Background

CICS TS for z/OS V5.1 introduced a heartbeat feature to the IPIC communications mechanism to prevent firewalls from closing inactive connections, and to discover and report connectivity problems with idle connections. It makes use of a CISP task to check every 60 seconds for connections that have not been used in that time period. For each idle connection a CIS1 task is then attached to attempt to send a heartbeat message to the node at the other end of the connection. If a heartbeat response is not received within 10 seconds then this task makes a further attempt to test the connection and if this also times out then the connection is released.

There may be occasions when it is desirable to disable the heartbeat function. For instance if a remote region is resource constrained then it may not be able to respond to heartbeat messages before they time out and so connections may be released unnecessarily.

The heartbeat feature is controlled internally by CICS and so there is no external method to turn it off. This article explains a simple way to achieve this with CICS TS V5.1 and later releases.

## Solution

The CISP task is attached when a CICS region starts up, and continues for the lifetime of the region. It runs program DFHISPHP, which is normally found in the SDFHLOAD library within the DFHRPL concatenation of the CICS JCL job. A new version of DFHISPHP, which does very little, needs to be created as an assembler program. This then must be linked into a library that appears higher in the DFHRPL concatenation than SDFHLOAD. When the region starts up CISP will be attached, run the new version of DFHISPHP and then terminate. This will leave the region without the ability to send heartbeat messages for any of the IPIC connections that it has acquired.

The following restrictions apply to this solution:

* Once disabled, heartbeat processing will remain off for the lifetime of the region.
* This solution only prevents outbound heartbeat processing and must be applied to the regions at both ends of a connection if the aim is to completely turn off heartbeats in both directions.
* It applies to all the IPCONNs in a region, irrespective of whether they are installed and acquired during start-up, or once the region is running. It does not allow some of the IPCONNs to have heartbeat processing turned on, while others have it disabled.

## Supporting files

* [`src/dfhisphp.asm`](src/dfhisphp.asm) - New version of program DFHISPHP.

## License

This project is licensed under [Apache License Version 2.0](LICENSE).  

## Reference

* For details on using the IPIC heartbeat refer to [Intercommunication using IP interconnectivity](https://www.ibm.com/support/knowledgecenter/SSGMCP_5.5.0/fundamentals/connections/dfht1_isc_tcpip.html).
