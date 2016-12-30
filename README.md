# cics-ipic-heartbeat-disabler
A method to turn off this function in a CICS region
## Background
CICS TS for z/OS V5R2 introduced a heartbeat feature to the IPIC communications mechanism to prevent firewalls from closing idle connections, and to discover and report connectivity problems with idle connections. It makes use of the CISP task to check every 60 seconds for connections that have not been used in that time period. A CIS1 task is then attached to attempt to send a heartbeat message over a connection that has been idle in that interval. If a heartbeat response is not received within 10 seconds then this task makes a further attempt to test the connection and if this also times out then the conection is released.

There may be occasions when it is desirable to disable the heartbeat function. For instance if a remote region is resource constrained then it may not be able to respond to heartbeat messages before they time out and so connections may be released unnecessarily.

The heartbeat feature is controlled internally by CICS and so there is no way to turn it off. This article explains a simple way to achieve this with CICS TS V5R2 and later releases.
## Solution
The CISP task is attached when a CICS region starts up, and runs for the lifetime of the region. It runs program DFHISPHP, which is normally found in the DSFHLOAD library in the DFHRPL concatenation of the CICS JCL. By creating a new version of DFHPHP as an assembler program, which does nothing when run, then loading this from a library that is above SDFHLOAD, the CISP task will be attached during start-up and then will terminate, leaving the region without the IPIC heartbeat monitor running. 

The following restrictions apply to this solution:-
* Once diabled, heartbeat processing will remain off for the lifetime of the region.
* It only prevents outbound heartbeat processing and so must be applied to the regions at both ends of a connection if the aim is to completely turn it off.
* This method does not permit the selective controlling of heartbeat processing for some of the connections to a region.
