% Title = "DNS Delegation Requirements"
% abbrev = "DNS Delegation Requirements"
% category = "info"
% docName = "draft-wallstrom-dnsop-dns-delegation-requirements-00"
% ipr= "trust200902"
% area = "Internet"
% workgroup = "DNSOP"
%
% date = 2016-01-15T00:00:00Z
%
% [[author]]
% initials="P."
% surname="Wallstrom"
% fullname="Patrik Wallstrom"
% organization="IIS"
%     [author.address]
%     email="pawal@iis.se"
%     phone="+46-733-173956"
%     [author.address.postal]
%     city="Stockholm"
%     country="SE"
%
% [[author]]
% initials="J."
% surname="Schlyter"
% fullname="Jakob Schlyter"
% organization="Kirei AB"
%     [author.address]
%     email="jakob@kirei.se"

.# Abstract

This document outlines a set of requirements on a well-behaved DNS delegation
of a domain name. A large number of tools has been developed to test DNS
delegations, but all tools uses a different set of requirements for what is a
correct setup for a delegated domain name. However, there are few requirements
on how to set up DNS in order to just make the delegation work. In order to
have a well-behaved delegation that is robust to failures and also make DNS
resolvers behave consistently, there are a large number of things to consider.

Based on this document, it should be possible to set up a fully functional DNS
delegation for a domain, but also to create a set of test specifications for
how to test a DNS delegation. It is typically for testing the Basic section
(TODO: add in-document reference) is there.

{mainmatter}


# Introduction

The requirements are split into these different areas, to easier
differentiate between what they are for:

 1. Basic
 2. Address
 3. Connectivity
 4. Consistency
 5. Delegation
 6. DNSSEC
 7. Name server
 8. Syntax
 9. Zone

TODO: a secondary name server operator should follow the advice in the BCP
document [@!RFC2182].

## DNS Terminology

This document attempts to fully follow the DNS terminology as defined in
[@!RFC7719].

Many requirements in this document deals with the properties of a nameserver
that is used as part of a delegation, therefore the wording mentioning the use
of a name server as part of this is omitted.

## Reserved Words

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [@?RFC2119].


# Basic requirements

The Basic requirements are fundamental to a working DNS delegation. Without
these properties, the rest of the requirements are irrelevant.

## The domain name MUST be valid

The domain name MUST follow the rules defined in section 2.3.1 of [@RFC1123] in
order to be able to map the domain into a DNS packet. A domain name is normally
valid if the name has been registered with a domain name registry.

## The domain MUST have a parent domain

A DNS delegation MUST have a parent domain from which it is delegated. The
concept of zone cuts was first described in [@RFC1034] and later clarified in
section 6 of [@RFC2182]. The only exception is the root zone, which do not have
a parent zone.
   
## The domain MUST have at least one working name server

A fully working DNS delegation have a parent zone delegating the zone to a set
of child name servers. At least one name server MUST be able to answer DNS
queries in order to be able to serve authoritative data for the child zone.


# Address requirements

A delegation in the public internet DNS hierarchy will use the globally unique
address space.

## Name server address MUST be globally routable

In order for the domain and its resources to be accessible, authoritative name
servers must have addresses in the routable public addressing space.

IANA is responsible for global coordination of the IP addressing system. Aside
its address allocation activities, it maintains reserved address ranges for
special uses. There are two IANA registries for Special-Purpose Addresses, the
IANA IPv6 Special- Purpose Address Registry and the IANA IPv4 Special-Purpose
Address Registry.

[@RFC6890] instructs IANA on how to structure the IPv4 and IPv6 Special-Purpose
Address Registries. The registries [@IANA-IPv4-Special] and [@IANA-IPv6-Special] are
maintained by IANA. These IANA registries is also described in [@RFC7249] in
section 2.2 and 2.3.

A name server MUST NOT be using any IP address out of any of these registries
that are marked with False in the Global column.

## The IP address of a name server MUST be delegated by IANA

IP addresses not delegated by IANA MUST NOT be used as an addressed used by a
name server. Thus, any IP address prefix not delegated to a RIR by IANA MUST be
rejected.

The IANA registry [@IANA-IPv6-Unicast] SHOULD be used to determine the status of
an IPv6 prefix. Only prefixes with the status ALLOCATED is allowed.

The IANA registry [@IANA-IPv4-Registry] SHOULD be used to determine the status of
an IPv4 prefix. Only prefixes with the status ALLOCATED and LEGACY is allowed.
Note that IPv4 LEGACY is not allocated to an RIR. See TODO.

Martians [@RFC1208] is a humorous term applied to packets that turn up
unexpectedly on the wrong network because of bogus routing entries. Bogons
[@RFC3871] are packets sourced from addresses that have not yet been allocated
by IANA or the Regional Internet Registries (RIRs), or not delegated to a RIR
by IANA as described above. These kind of addresses SHOULD not be used for a
name server either.


#  Connectivity requirements

Some text about connectivity.

## All name servers MUST have UDP connectivity over port 53

DNS queries are sent using UDP on port 53, as described in section 4.2.1 of
[@RFC1035]. A name server MUST respond to DNS queries over UDP.

## All name servers MUST have TCP connectivity over port 53

DNS queries can also be sent using TCP on port 53, as described in section
4.2.2 of [@RFC1035]. A name server MUST respond to DNS queries over TCP. This
requirement has also been further clarified in [@RFC5966], which makes TCP a
REQUIRED part of a full DNS protocol implementation.

## The name servers SHOULD not be on the same AS

[@RFC2182], section 3.1 states that distinct authoritative name servers for a
child domain should be placed in different topological and geographical
locations. The objective is to minimise the likelihood of a single failure
disabling all of them. Further support for this is given in section 5:

> It is recommended that three servers be provided for most
> organisation level zones, with at least one which must be well
> removed from the others.

To avoid any single point of failure in routing, all name servers SHOULD not be
placed within a single routing domain, or AS numbers.


# Consistency requirements

For DNS resolver behaviour to be consistent for a domain, it is important
that the authoritative data for the domain to be consistent. All name servers
authoritative domain should serve the same data. An indicator of operator
failure is that the SOA record differs, and the NS RR set differs between
the authoratitative name servers.

Section 4.6 in [@RFC4786] advices that data synchronisation in an anycast
setup should be done in a manner so that anycast nodes operate in a
consistent manner.

## All name servers SHOULD respond with the same SOA serial number

An indication that not all authoritative name servers have a consistent
and updated copy of the zone is that the serial numbers differ. When quering
for the SOA RR all name servers SHOULD respond with the same serial record.

Section 4.3.5 in [@RFC1034] explains the typical function of the serial
numbers in zone Maintenance and transfers.

## All name servers SHOULD respond with the same SOA RNAME

As per section 3.3.13 of [@RFC1035], the RNAME field in the SOA RDATA refers
to the personal contect for the zone. An indication that not all
authoritative name servers have a consistent and updated copy of the zone
is that the RNAME differs. When quering for the SOA RR all name servers
SHOULD respond with the same serial record.

## All name servers SHOULD respond with the same SOA parameters

The inconsistency of the SOA parameters REFRESH, RETRY, EXPIRE
and MINIMUM might lead to operational problems for the zone. The fields are
defined in section 3.3.13 of [@RFC1035]. These SOA parameters SHOULD be
consistent for all authoritative name servers for the zone.

## All name servers MUST respond with the same NS RR Set

All authoritative name servers MUST serve the same NS record set
(section 4.2.2 of [!@RFC1034]) for the zone. Any inconsistency of NS
records descibed in section 3.3.11 of RFC 1035 might result in
operational failures.


# Delegation requirements

[@RFC2182] is a BCP on how to select and operate secondary name servers,
and summarize many operational issues with the delegation of a zone.
For a delegation to work continously if one component fails, there
are operational considerations to ensure this.

## The delegation SHOULD contain at least two name servers

Section 4.1 [@RFC1034] states that by administrative fiat we require
every zone to be available on at least two name servers. Section 5
of [@RFC2182] that answers the question on how many name servers are
needed, the recommendation is that "three servers be provided for most
organisation level zones, with at least one which must be well
removed from the others."

In order to avoid any operational problems, a delegation SHOULD contain
at least to name servers.

## The name servers MUST have distinct IP addresses

A common workaround to a registry policy that requires at least two
name servers is to create two names that uses the same IP address.

To avoid any operational errors and workaround such as this, all
name servers used for the zone MUST use distinct IP addresses.

## The referral SHOULD fit into an non-truncated 512 byte UPD packet

The DNS still defaults to using UDP, although efforts into requiring
or transitioning to use TCP have come a long way. The UDP packet limit
is 512 bytes, and although the EDNS0 ([@RFC6891]) extension mechanism
to overcome this limit have been in use for a very long time, many
middleboxes and proxies still interfere with DNS packets ([@RFC5652]).

To avoid any such problems with the delegation, and to avoid any
unexpected truncation of a referral response, the referral containing
the delegation from the parent SHOULD fit in 512 bytes.

## All name servers MUST be authoritative for the domain

A name server that does not answer authoritatively for the zone is a
clear sign of misconfiguration, and is a common cause for operational
problems.

Subsection 6.1 of [@RFC2181] mandates that the name servers MUST answer
authoritatively for the zone.

## The delegation name MUST exactly match the apex of the child zone

The configured zone on the child name servers MUST match the delegated
name of the zone. When querying the child name servers for the zone,
any authoritative data for another name MUST NOT be in the response.

[@RFC2181] states that the SOA RR and the NS RR indicates the origin
of the zone, and both are mandatory records in a zone. Both RRs MUST
be present and match the name of the zone.

## Glue records in delegation SHOULD exactly match records in child zone

In-bailiwick glue for name servers listed at the parent SHOULD match
the in-bailiwick glue for the name servers in the child.

If the glue address mismatch between the server and the child, this
is a strong indication of configuration error.


# DNSSEC requirements

Some text about DNSSEC.

## The DS hash digest algorithm must be one of the ones assigned by IANA

## The DNSKEY algorithm must be one of the ones assigned by IANA

## DS must match a DNSKEY in the child zone

## The number of NSEC3 iterations must to be higher than what is allowed

## RRSIG lifetimes neither must not be too short nor too long

## The name server must include RRSIG in all responses to DNSSEC queries

## If child zone includes DNSKEY then parent zone should have DS

## RRSIG of DNSKEY must be valid and must be created by a valid DNSKEY

## RRSIG of SOA must be valid and must be created by a valid DNSKEY

## The name servers must include valid NSEC/NSEC3 record in NXDOMAIN responses

## Delegation from parent to child must be properly signed


# Syntax requirements

Some text about Syntax.

## No illegal characters must be in the domain name
## No hyphen must be at start or end of the domain name
## No hyphens must be in position 3 and 4 of the domain name
## The NS name must be a valid hostname
## The SOA RNAME must not containt the '@' character
## The SOA RNAME must have no illegal characters
## The SOA MNAME to have no illegal characters
## The MX record in apex to point at a valid hostname

# Zone requirements

Some text about Zone.

## SOA MNAME must be a valid hostname and point to A or AAAA
## SOA MNAME must be an NS of the zone
## SOA REFRESH should to be too low
## SOA RETRY must be lower than 'refresh'
## SOA RETRY should to be too low
## SOA EXPIRE should to be too low
## SOA 'expire' should to be lower than 'refresh'
## SOA 'minimum' should to be too high
## SOA 'minimum' should to be too low
## MX record in apex must be a valid hostname and point to A or AAAA

# Security Considerations

This document has no security considerations (yet).

# IANA Considerations

This document has no IANA actions.

# Acknowledgements

The requirements documented in this document was developed within the CENTR
Test Requirements Task Force (TRTF).


<!-- reference we need to include -->

<reference anchor="IANA-IPv4-Special" target="https://www.iana.org/assignments/iana-ipv4-special-registry">
  <front>
    <title>IANA IPv4 Special-Purpose Address Registry</title>
    <author><organization>IANA</organization></author>
    <date month="January" year="2006"/>
  </front>
</reference>

<reference anchor="IANA-IPv6-Special" target="https://www.iana.org/assignments/iana-ipv6-special-registry">
  <front>
      <title>IANA IPv6 Special-Purpose Address Registry</title>
      <author><organization>IANA</organization></author>
      <date month="January" year="2006"/>
  </front>
</reference>

<reference anchor="IANA-IPv6-Unicast" target="http://www.iana.org/assignments/ipv6-unicast-address-assignments">
  <front>
      <title>IPv6 Global Unicast Address Assignments</title>
      <author><organization>IANA</organization></author>
      <date month="October" year="2015"/>
  </front>
</reference>

<reference anchor="IANA-IPv4-Registry" target="https://www.iana.org/assignments/ipv4-address-space">
  <front>
      <title>IANA IPv4 Address Space Registry</title>
      <author><organization>IANA</organization></author>
      <date month="August" year="2015"/>
  </front>
</reference>

<reference anchor="IANA-DNSSEC-DNSKEY" target="https://www.iana.org/assignments/dns-sec-alg-numbers/dns-sec-alg-numbers.xhtml">
  <front>
      <title>Domain Name System Security (DNSSEC) Algorithm Numbers</title>
      <author><organization>IANA</organization></author>
      <date month="November" year="2003"/>
  </front>
</reference>

<reference anchor="IANA-DNSSEC-DS" target="https://www.iana.org/assignments/ds-rr-types/ds-rr-types.xhtml">
  <front>
      <title>Delegation Signer (DS) Resource Record (RR) Type Digest Algorithms</title>
      <author><organization>IANA</organization></author>
      <date month="Oktober" year="2003"/>
  </front>
</reference>

<reference anchor="IANA-DNSSEC-NSEC3" target="https://www.iana.org/assignments/dnssec-nsec3-parameters/dnssec-nsec3-parameters.xhtml">
  <front>
      <title>Domain Name System Security (DNSSEC) NextSECure3 (NSEC3) Parameters</title>
      <author><organization>IANA</organization></author>
      <date month="December" year="2007"/>
  </front>
</reference>


{backmatter}
