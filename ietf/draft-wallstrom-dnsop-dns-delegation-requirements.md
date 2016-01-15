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
% initials="P.W."
% surname="Patrik Wallstrom"
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

bla bla

# IANA Considerations

bla bla

{backmatter}
