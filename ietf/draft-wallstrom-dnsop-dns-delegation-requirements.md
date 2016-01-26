% Title = "DNS Delegation Requirements"
% abbrev = "DNS Delegation Requirements"
% category = "info"
% docName = "draft-wallstrom-dnsop-dns-delegation-requirements-00"
% ipr= "trust200902"
% area = "Internet"
% workgroup = "DNSOP"
%
% date = 2016-01-26T14:00:00Z
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
how to test a DNS delegation.

{mainmatter}


# Introduction

This document outlines a set of requirements on a well-behaved DNS delegation
of a domain name. Many domain name registries use a set of requirements on
what they may consider a valid delegation. Such requirements can be used to
implement tools that are used for pre- or post-delegation checks of the
delegations in that registry.

To test the quality of the delegation there has been a number of different
tools developed, each based on a different set of requirements. This
document documents a set of baseline requirements on a correct setup for
a delegated domain name. This document is based on current RFCs and
documents requirements that are protocol specific, but also administrative
policy requirements drawn from best practices and recommendations.

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

A secondary name server operator should follow the advice in the BCP document
[@!RFC2182].

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

The domain name MUST follow the rules defined in Section 2.3.1 of [@!RFC1123] in
order to be able to map the domain into a DNS packet. A domain name is normally
valid if the name has been registered with a domain name registry.

Internationalized domain names, [@RFC5891], are expected to be encoded using
Punycode [@RFC3492], thus following the rules outlined in Section 2.3.1 of
[@!RFC1035]. Any validation of the domain name in U-label form is out of scope
for this document.

## The domain MUST have a parent domain

A DNS delegation MUST have a parent domain from which it is delegated. The
concept of zone cuts was first described in [@!RFC1034] and later clarified in
Section 6 of [@RFC2182]. The only exception is the root zone, which do not have
a parent zone.
   
## The domain MUST have at least one working name server

A fully working DNS delegation have a parent zone delegating the zone to a set
of child name servers. At least one name server MUST be able to answer DNS
queries in order to be able to serve authoritative data for the child zone.


# Address requirements

A delegation in the public Internet DNS hierarchy will use the globally unique
address space.

## Name server address MUST be globally routable

In order for the domain and its resources to be accessible from the Internet,
authoritative name servers must have addresses in the routable public
addressing space.

IANA is responsible for global coordination of the IP addressing system. Aside
its address allocation activities, it maintains reserved address ranges for
special uses. There are two IANA registries for Special-Purpose Addresses, the
IANA IPv6 Special-Purpose Address Registry and the IANA IPv4 Special-Purpose
Address Registry.

[@RFC6890] instructs IANA on how to structure the IPv4 and IPv6 Special-Purpose
Address Registries. The registries [@IANA-IPv4-Special] and
[@IANA-IPv6-Special] are maintained by IANA, and is also described in Section
2.2 and 2.3 of [@RFC7249].

A name server MUST NOT be using any IP address out of any of these registries
that are marked with False in the Global column.

## The IP address of a name server MUST be delegated by IANA

IP addresses not delegated by IANA MUST NOT be used as an addressed used by a
name server. Thus, IP addresses within a prefix not delegated to a RIR by IANA
MUST be rejected.

The IANA registry [@IANA-IPv6-Unicast] SHOULD be used to determine the status of
an IPv6 prefix. Only prefixes with the status ALLOCATED are allowed.

The IANA registry [@IANA-IPv4-Registry] SHOULD be used to determine the status
of an IPv4 prefix. Only prefixes with the status ALLOCATED and LEGACY are
allowed. Note that IPv4 LEGACY is not allocated to a RIR.

Martians [@RFC1208] is a humorous term applied to packets that turn up
unexpectedly on the wrong network because of bogus routing entries. Bogons
[@RFC3871] are packets sourced from addresses that have not yet been allocated
by IANA or a RIR, or not delegated to a RIR by IANA as described above.
Martians and Bogons SHOULD NOT be used as an addressed used by a name server.


#  Connectivity requirements

The use of underlying protocols for DNS is described in Section 4.2 of
[@RFC1035].

The Internet supports name server access using TCP on server port 53
(decimal) as well as datagram access using UDP on UDP port 53
(decimal). Today DNS is used in conjunction with both IPv4 and IPv6,

Name servers configured for a zone in a delegation MUST be able to answer
queries using the DNS protocol.

## All name servers MUST have UDP connectivity over port 53

DNS queries are sent using UDP on port 53, as described in Section 4.2.1 of
[@!RFC1035]. A name server MUST respond to DNS queries over UDP.

## All name servers MUST have TCP connectivity over port 53

In addition to UDP, DNS queries can also be sent using TCP on port 53, as
described in Section 4.2.2 of [@!RFC1035]. A name server MUST respond to DNS
queries over TCP. This requirement has also been further clarified in
[@RFC5966], which makes TCP a REQUIRED part of a full DNS protocol
implementation.

It should be noted that even though [@RFC5966] requires TCP for a DNS protocol
implementation, it does not make specific recommendations to operators of DNS
servers. However, it also notes that failure to support TCP (or the blocking of
DNS over TCP at the network layer) may result in resolution failure and/or
application-level timeouts.


# Consistency requirements

For DNS resolver behaviour to be consistent for a domain, it is important that
the authoritative data for the domain to be consistent. All name servers
authoritative domain should serve the same data. An indicator of operator
failure is that the SOA record or the NS RR set differs between the
authoratitative name servers.

Section 4.6 in [@RFC4786] advices that data synchronisation in an anycast setup
should be done in a manner so that anycast nodes operate in a consistent manner.

## All name servers SHOULD respond with the same SOA serial number

An indication that not all authoritative name servers have a consistent
and updated copy of the zone is that the serial numbers differ. When quering
for the SOA RR all name servers SHOULD respond with the same serial record.

Section 4.3.5 in [@!RFC1034] explains the typical function of the serial
numbers in zone Maintenance and transfers.

One should note that even though different SOA serial numbers are a strong
indicator of an inconsistent setup, there are several scenarios where the
serial number vary between name servers. One example is a zone with frequent
updates to zone data, where propagation delay between the name servers may
result in limited inconsistency.

## All name servers SHOULD respond with the same SOA RNAME

As per Section 3.3.13 of [@!RFC1035], the RNAME field in the SOA RDATA refers
to the mailbox of the person responsible for the zone. An indication that not
all authoritative name servers have a consistent and updated copy of the zone
is that the RNAME differs. When quering for the SOA RR all name servers SHOULD
respond with the same SOA RNAME.

## All name servers SHOULD respond with the same SOA parameters

The inconsistency of the SOA parameters REFRESH, RETRY, EXPIRE and MINIMUM,
defined in Section 3.3.13 of [@!RFC1035], might lead to operational problems
for the zone. These SOA parameters SHOULD be consistent for all authoritative
name servers for the zone.

## All name servers MUST respond with the same NS RR Set

All authoritative name servers MUST serve the same NS record set in
order to ensure consistency in the zone cut as described in Section
4.2.2 of [!@RFC1034]. Any inconsistency of NS records descibed in Section
3.3.11 of RFC 1035 might result in operational failures.


# Delegation requirements

[@RFC2182] is a BCP on how to select and operate secondary name servers,
and summarize many operational issues with the delegation of a zone.
For a delegation to work continously if one component fails, there
are operational considerations to ensure this.

## The delegation SHOULD contain at least two name servers

Section 4.1 [@!RFC1034] states that by administrative fiat we require
every zone to be available on at least two name servers. Section 5
of [@RFC2182] that answers the question on how many name servers are
needed, the recommendation is that "three servers be provided for most
organisation level zones, with at least one which must be well
removed from the others."

In order to avoid any operational problems, a delegation SHOULD contain
at least two (2) name servers.

## The name servers SHOULD NOT belong to the same AS

[@RFC2182], Section 3.1 states that distinct authoritative name servers for a
child domain should be placed in different topological and geographical
locations. The objective is to minimise the likelihood of a single failure
disabling all of them. Further support for this is given in Section 5:

> It is recommended that three servers be provided for most
> organisation level zones, with at least one which must be well
> removed from the others.

To avoid any single point of failure in routing, all name servers SHOULD NOT be
placed within a single routing domain, or AS (autonomous system).

## The name servers MUST have distinct IP addresses

A common workaround to a registry policy that requires at least two
name servers is to create two (2) names with the same IP address.

To avoid any operational errors and workaround such as this, all
name servers used for the zone MUST use distinct IP addresses.

## The referral SHOULD fit into an non-truncated 512 byte UDP packet

The DNS still defaults to using UDP, although efforts into requiring
or transitioning to use TCP have come a long way. The UDP packet limit
is 512 bytes, and although the EDNS0 [@RFC6891] extension mechanism
to overcome this limit have been in use for a very long time, many
middleboxes and proxies still interfere with DNS packets ([@RFC5625]).

To avoid any such problems with the delegation, and to avoid any
unexpected truncation of a referral response, the referral containing
the delegation from the parent SHOULD fit within 512 bytes.

## All name servers MUST be authoritative for the domain

A name server that does not answer authoritatively for the zone is a
clear sign of misconfiguration, and is a common cause for operational
problems.

Section 6.1 of [@RFC2181] mandates that the name servers MUST answer
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

## SOA MNAME MUST be authoritative for the zone

The hostname of the MNAME field may or may not be listed among the delegated
name servers, but SHOULD still be authoritative for the zone. MNAME may be used
for other services, e.g., DNS NOTIFY [RFC1996] and DNS Dynamic Updates
[RFC2136].

It should be noted that there are no formal requirement that the name server
listed in the SOA MNAME is reachable from the public Internet. Because of this,
it may be difficult to implement a reasonable test for this requirement.


# DNSSEC requirements

If DNSSEC is used for the zone, either by indicating that the zone is
signed with a DS record, or the use of a DNSKEY in the zone itself, a
number of things are required for a fully functional delegation.

The Domain Name System Security Extensions (DNSSEC) add data origin
authentication and data integrity to the Domain Name System, and was
first introduced with the RFCs [@RFC4033], [@RFC4034] and [@!RFC4035].
The are also a number of additions to DNSSEC such as NSEC3 described
in [@RFC5155], and a number of algorithms to the cryptographic functions.

## The DS Digest Type MUST be assigned by IANA

The The Digest Type Field is defined as part of the DS RDATA Wire Format
of Section 5.1.3 in [@RFC4034]. The appendix A.2 defines the defined
digest algorithm types with possible future algorithms. The IANA registry
for DS Digest Types [@IANA-DNSSEC-DS] was defined by [@RFC3658].

Any DS Digest Type used for a zone MUST be assigned by IANA.

## The DNSKEY algorithm MUST be assigned by IANA

The DNSKEY RR is defined in Section 2 of [@RFC4034] as part of the DNSKEY
RDATA Wire Format. The appendix A.1 defines the initial list of DNSKEY
Algorithm Types. The IANA Registry for DNSKEY Algorithm Types
[@IANA-DNSSEC-DNSKEY] was created with [@RFC3755].

Any DNSKEY algorithm number used for in a zone MUST be assigned by IANA.

## Chain of trust of delegation

A valid authentication chain from the parent DS, as described in Section 3.1 of
[@RFC4033], MUST exist for the SOA, DNSKEY and NS records of the child zone
if a DS record is published in the parent zone.

## One DS MUST match a least one DNSKEY in the child zone

DNS delegations from a parent to a child are secured with DNSSEC by publishing
one or several Delegation Signer (DS) resource records in the parent zone,
along with the NS records for the delegation.

As stated in Section 2.4 of [@!RFC4035], a DS RR SHOULD point to a DNSKEY RR
that is present in the child's apex DNSKEY RRset. If there is a DS RR published
at the parent, there MUST be at least one DNSKEY RR in the child zone that
matches at least one DS RR for every signature algorithm, otherwise the
authentication of the referral will fail, as described in Section 5.2 of
[@!RFC4035].

For each unique algorithm from the DS RRs present, there MUST be a
matching DNSKEY using that algorithm in use in the child.

## The number of NSEC3 iterations must not be higher than what is allowed

Section 10.3 of [RFC5155] specifies the max number of NSEC3 iterations allowed
for different key sizes. This requirement is enforced by several resolver
implementations.

The number of NSEC3 iterations MUST NOT be higher than what is allowed by
Section 10.3 of [RFC5155]. It should be noted that the values in the table MUST
be used independent of the key algorithm.

## RRSIG validity period neither SHOULD NOT be too short nor too long

[@RFC6781] describes operational considerations on the choice of validity
periods for RRSIGs. Having too short validity periods might cause
operational failure in case of unexpected events, but is good for
protecting against replay attacks. Having too long validity periods may
be good for operational security, but opens up for replay attacks.

The RRSIG validity periods in the zone SHOULD NOT be too short nor too long.

## The name server must include RRSIG in all responses to DNSSEC queries

If the zone is signed, the name servers MUST be able to include RRSIG RRs
as additional data in any response when the query has the DO bit set, as
described in Section 3.1.1 of [@!RFC4035].

## The name servers must include valid NSEC/NSEC3 record in NXDOMAIN responses

If the zone is signed, the name servers MUST be able to include NSEC/NSEC3 RRs
as additional data in any response when the query has the DO bit set, as
described in Section 3.1.1 of [@!RFC4035].


# Syntax requirements

All domain- and host names in DNS MUST follow the rules outlined in
Section 2.3.1 of [@!RFC1035]. The Name Syntax and LDH Label has been
further clarified in Section 11 in [@RFC2181] and Section 2.3.1 in
[@RFC5890]. From this follows the requirements below.

## Illegal characters MUST NOT be in the domain name

There MUST NOT be any illegal characters used in the domain name. The
domain name must follow the rules defined in Section 2.3.1 of
[@!RFC1035], Section 2.1 of [@!RFC1123], Section 11 of [@!RFC2182]
and Section 2 of [@RFC3696].

## Hyphens SHOULD NOT be in position 3 and 4 of the domain name

The effort of internationaliztion of domain names and the development
of IDNA brought us the extension mechanism of using the string 'xn--'
to have a special meaning. To allow future extensions to DNS there
SHOULD be no instances of labels in the DNS that start with two
characters, followed by two hyphens, where the two characters are not
"xn". This has been described in Section 5 of [@RFC3696].

## The NS names MUST be valid hostnames

The Name Server name MUST be a valid hostname according to the rules
defined in Section 2.3.1 of [@!RFC1035], in Section 2.1 in [RFC1123],
Section 11 in [@!RFC2181] and Section 2 and 5 in [@RFC3696].

## The SOA RNAME MUST not contain the '@' character

The SOA RNAME field is a mailbox address defined in Section 3.3 of [@!RFC1034]
and Section 2.2 of [@RFC1912]. The RNAME field MUST follow the rules of an
e-mail address defined in Section 3.4.1 of [@!RFC2822], and the '@' character
MUST be changed so that the whole e-mail address is converted into a single
domain name as described in Section 3.3 of [@!RFC1034] and Section 2.1 of
[@!RFC1123].

## The SOA RNAME MUST be a legal hostname

The SOA RNAME field is a mailbox address. The SOA RNAME field is defined in
Section 3.3 of [@!RFC1034] and Section 2.2 of [@RFC1912]. As a field containing
a domain name, the content of the RNAME field MUST follow the rules outlined in
Section 2.3.1 of [@!RFC1035] and Section 2.1 of [@!RFC1123].

## The SOA MNAME MUST be a legal hostname

The SOA MNAME field is a hostname. The SOA MNAME field is defined in Section
3.3.13 of [@!RFC1035]. As a field containing a domain name, the content of the
RNAME field MUST follow the rules outlined in Section 2.3.1 of [@!RFC1035].

Furthermore, Section 7.3 in [@RFC2181] makes it clear that the SOA MNAME field
SHOULD NOT be the name of the zone itself.

## The MX record in apex MUST point to a valid hostname

The requirement on the existence of an MX RR in the apex of the child zone may
vary by policy from different parent zones. However, it is strongly recommended
in Section 7 of [@RFC2142] that all domains should have a mailbox named
hostmaster@domain. SMTP can make a delivery without the MX, using the A or AAAA
record as specified in Section 5.1 of [@RFC5321].

If an MX RR exists in the apex of the child zone, the hostname that
the MX RR points to MUST follow the rules outlined in Section 2.3.1
of [@!RFC1035] and Section 2.1 of [@!RFC1123].


# Security Considerations

This document has no security considerations (yet).


# IANA Considerations

This document has no IANA actions.


# Acknowledgements

The requirements documented in this document was developed within the CENTR
Test Requirements Task Force (TRTF). Most of the original requirements and
text come from the Zonemaster project.


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
