# List of Requirements

## Background

The basis of this list comes from the
[Zonemaster test cases](https://github.com/dotse/zonemaster/blob/master/docs/specifications/tests/Report.md).

The identifiers from the original test cases or requirements is not the
same as the Zomaster requirements or test cases, but a new set of
requirements for the work of the TRTF.

The identifiers on any requirements or specifications should be changed
from the syntax used in Zonemaster in order to avoid confusion. This list
another syntax than the Zonemaster requirements. To find more specific
text for each requirement in the Zonemaster project, each specific test
case have a section named Objective which is a good start. Here is an
example:
https://github.com/dotse/zonemaster/blob/master/docs/specifications/tests/Address-TP/address01.md#objective

## About the requirements

Each requirement should have a clear and limited scope, and should have
relevant references to how DNS is supposed to work as described in i.e.
RFC documents from IETF.

From experience in the Zonemaster project, a set of BASIC tests must
executed before any other test can proceed - this is the reason why the
BASIC test level is listed first in the list. There should otherwise not
be a need for any interdependent tests - where one test is depending on
another test.

This list of requirements only desribes the short form to make it clear
what the requirement is about - however, there is a need to detail the exact
requirement in a separate document.

## Requirements

BASIC:

	BASIC-R01 The domain name to be valid
	BASIC-R02 The domain to have a parent domain
	BASIC-R03 The domain to have at least one working name server

ADDRESS:

	ADDRESS-R01 Name server address to be globally routable
    ADDRESS-R02 IPv6 address of name server not to be part of bogon prefix

CONNECTIVITY:

	CONNECTIVITY-R01 All name servers to have UDP connectivity over port 53
	CONNECTIVITY-R02 All name servers to have TCP connectivity over port 53
	CONNECTIVITY-R03 The name servers to not be on the same AS
	
CONSISTENCY:

	CONSISTENCY-R01 All name servers to respond with the same SOA serial number
	CONSISTENCY-R02 All name servers to respond with the same SOA RNAME
	CONSISTENCY-R03 All name servers to respond with the same SOA parameters
	CONSISTENCY-R04 All name servers to respond with the same NS RR Set
	
DELEGATION:

	DELEGATION-R01 The delegation to contain at least two name servers
	DELEGATION-R02 The name servers to have distinct IP addresses
	DELEGATION-R03 The referral to fit into an non-truncated 512 byte UPD packet
	DELEGATION-R04 All name servers to be authoritative for the domain
	DELEGATION-R05 The name server name not to point to CNAME alias 
	DELEGATION-R06 The delegation name to exactly match the apex of the child zone
	DELEGATION-R07 Glue records in delegation to exactly match records in child zone

DNSSEC:

	DNSSEC-R01 The DS hash digest algorithm to be one of the ones assigned by IANA
	DNSSEC-R02 DS to match a DNSKEY in the child zone
	DNSSEC-R03 The number of NSEC3 iterations not to be higher than what standard states
	DNSSEC-R04 RRSIG lifetimes neither to be too short nor too long
	DNSSEC-R05 The DNSKEY algorithm to be one of the ones assigned by IANA
	DNSSEC-R06 The name server to include RRSIG in all responds to DNSSEC queries
	DNSSEC-R07 If child zone includes DNSKEY then parent zone to have DS
	DNSSEC-R08 RRSIG of DNSKEY to be valid and to be created by a valid DNSKEY
	DNSSEC-R09 RRSIG of SOA to be valid and to be created by a valid DNSKEY
    DNSSEC-R10 The name servers to include valid NSEC/NSEC3 record in NXDOMAIN responses.
	DNSSEC-R11 Delegation from parent to child to be properly signed
	
NAMESERVER:

	NAMESERVER-R01 The name server not to be a recursor
	NAMESERVER-R02 The name server to correctly support EDNS0
	NAMESERVER-R03 Zone transfer (AXFR) not to be available from monitor
	NAMESERVER-R04 Source address of response to be the same as target address of query
	NAMESERVER-R05 The name server to handle query for AAAA correctly
	NAMESERVER-R06 NS name to be resolvable in public DNS tree
	NAMESERVER-R07 Reverse DNS entry to exist for name server IP address
	NAMESERVER-R08 Reverse DNS entry name server IP to match name server name

SYNTAX:

	SYNTAX-R01 No illegal characters to be in the domain name
	SYNTAX-R02 No hyphen to be at start or end of the domain name
	SYNTAX-R03 No hyphen to be in position 3 *and* 4 of the domain name
	SYNTAX-R04 The NS name to be a valid hostname
	SYNTAX-R05 The SOA RNAME to have no '@' character
	SYNTAX-R06 The SOA RNAME to have no illegal characters
	SYNTAX-R07 The SOA MNAME to have no illegal characters
	SYNTAX-R08 MX record in apex to point at a valid hostname
	
ZONE:

	ZONE-R01 SOA MNAME to be FQDN of authoritative name server
	ZONE-R21 SOA MNAME to be an NS of the zone
	ZONE-R02 SOA REFRESH not to be too low
	ZONE-R03 SOA RETRY to be lower than 'refresh'
	ZONE-R04 SOA RETRY not to be too low
	ZONE-R05 SOA EXPIRE not to be too low
	ZONE-R06 SOA 'expire' not to be lower than 'refresh'
	ZONE-R07 SOA 'minimum' not to be too high
	ZONE-R08 SOA 'minimum' not to be too low
	ZONE-R09 SOA MNAME is not to point at a CNAME or DNAME alias
	ZONE-R10 MX record not to point at a CNAME or DNAME alias
	ZONE-R11 MX record to be present in apex
	ZONE-R12 MX record in apex to point at an A or AAAA record


	
	
	
	