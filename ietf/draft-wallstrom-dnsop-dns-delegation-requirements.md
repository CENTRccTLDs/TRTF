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
%     street=""
%     city="Stockholm"
%     code=""
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

This document outlines a set of requirements on a well-behaved
DNS delegation of a domain name. A large number of tools has
been developed to test DNS delegations, but all tools uses a
different set of requirements for what is a correct setup for
a delegated domain name. However, there are few requirements
on how to set up DNS in order to just make the delegation work.
In order to have a well-behaved delegation that is robust
to failures and also make DNS resolvers behave consistently,
there are a large number of things to consider.

Based on this document, it should be possible to set up a fully
functional DNS delegation for a domain, but also to create a
set of test specifications for how to test a DNS delegation.
It is typically for testing the Basic section (TODO: add
in-document reference) is there.

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

TODO: a secondary name server operator should follow the advice
in the BCP document [@!RFC2182].

# DNS Terminology

This document attempts to fully follow the DNS terminology as
defined in [@RFC7719].

Many requirements in this document deals with the properties
of a nameserver that is used as part of a delegation, therefore
the wording mentioning the use of a name server as part of this
is omitted.


# IANA Considerations

TODO

{backmatter}
