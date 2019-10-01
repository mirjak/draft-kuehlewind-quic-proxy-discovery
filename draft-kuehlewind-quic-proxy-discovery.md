---
title: Discovery Mechanism for QUIC-based Proxy Services
abbrev: QUIC Substrate
docname: draft-kuehlewind-quic-proxy-discovery-latest
date:
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: M. Kuehlewind
    name: Mirja Kuehlewind
    org: Ericsson
    email: mirja.kuehlewind@ericsson.com
  -
    ins: Z. Sarker
    name: Zaheduzzaman Sarker
    org: Ericsson
    email: zaheduzzaman.sarker@ericsson.com


normative:
    I-D.ietf-intarea-provisioning-domains:
    RFC4861:
    RFC2131:
    RFC8415:
    RFC6763:
    RFC6762:
    RFC1035:
    RFC2782:

informative:
    I-D.kuehlewind-quic-substrate:
    RFC2939:



--- abstract

Often proxy servers are used as an intermidate instance to connect to a web
server when the webserver is not directly reachable or the proxy can provide a
support service like, e.g., address anonymisation. To use the proxy a client
explicitly connects to it and requests forwarding to the final target server.
The client either knows the proxy address as preconfigured in the application or
can dynamically learn about available proxy services. This document describes
different discovery mechanisms for proxies that are either located in the local
network, e.g. home or enterprise network, in the access network, or somewhere
else on the Internet usually close to the traget server or even in the same
network as the target server.

This document assume that the proxy server is connected via QUIC and discusses
currently discovery mechanisms based on DNS and Provisioning Domains (PvD).

--- middle

# Introduction
 
QUIC is a new transport protocol that was initially developed as a way to
optimize HTTP traffic by supporting multiplexing without head-of-line-blocking
and integrating security directly into the transport. This tight integration of
security allows the transport and security handshakes to be combined into a
single round-trip exchange, after which both the transport connection and
authenticated encryption keys are ready.

Often proxy servers are used as an intermidate instance to connect to a web
server when the webserver is not directly reachable or the proxy can provide a
support service like, e.g., address anonymisation. QUIC's ability to multiplex,
encrypt data, and migrate between network paths makes it ideal for solutions
that need to tunnel or proxy traffic.

Existing proxies that are not based on QUIC are often transparent. That is, they
do not require the cooperation of the ultimate connection endpoints, and are
often not visible to one or both of the endpoints. If QUIC provides the basis
for future tunneling and proxying solutions, it is expected that this
relationship will change. At least one of the endpoints will be aware of the
proxy, explicitly connect to it, and coordinate with it. This allows client
hosts to make explicit decisions about the services they request from proxies
(for example, simple forward or more advance performance-optimizing services),
and to do so using a secure communication channel between themselves and the
proxy. {{I-D.kuehlewind-quic-substrate}} describes some of the use cases for
using QUIC for proxying and tunneling.

To use a proxy service, a client explicitly connects to it and requests forwarding to
the final target server. The client either knows the proxy address as
preconfigured in the application or can dynamically learn about available proxy
servers. This document describes different discovery mechanisms for proxies
that are either located in the local network, e.g. home or enterprise network,
in the access network, or somewhere else on the Internet usually close to the
traget server or even in the same network as the target server.

> At a note about mobile networks?


# Using DHCP for Local Discovery 

DHCP {{RFC2131}} can be used to announce the IP address of local proxy server in the IPv4
networks, as well DHCPv6 {{RFC8415}} in IPv6 networks. New options for both protocols are
specified below. The option can contain one or more IP addresses of QUIC-based proxy
servers. All of the addresses share the same Lifetime value. If it is desirable to have
different Lifetime values, multiple options can be used.

Comment: Type of proxy and what about multiple ones?

TODO: decide
whether we need a lifetime here. the current intention is to return
IP address(es), this means there might not be a need for name
resolution. In that case the lifetime can help indicating the
validity period of the information. In case the DHCP options returns
domain names then a resource record for the name reslution will
contain TTL value per record. hecne, in that case the lifetime
information will be reduncdant.]

TODO: will the DHCP option only include IP addresses?

~~~~~
                    0                             1
              0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
             +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
             |          Code         |          Len          |
             +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
             |                   Reserved                 |Q |
             +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
             |                    Lifetime                   |
             +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
             |                                               |
             :  IPv4 Addresses of QUIC-based Proxy Servers   :
             |                                               |
             +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
~~~~~
{: #fig-dhcpv4-option
   title="IPv4 Proxy Discovery DHCP option format"}
   
Code:

: Proxy Discovery option code (TDB) (8 bit)

Len:

: length of the option (without the Code and Len fields) in units of octets.  The 
minimum value is 6 if one IPv4 address is contained in the option. Every additional IPv4
address increases the length by 4. (8-bit unsigned integer)

Q:

: is set to one, proxy supports QUIC on port 443 (1 bit)

Lifetime:

: maximum time in seconds (relative to the time the packet is received) over which these
IP4 addresses can be used for proxy discovery. A value of all one bits (0xffff)
represents infinity. A value of zero means that the proxy addresses SHOULD no longer be
used. (16-bit unsigned integer)

IPv4 Addresses of QUIC-based Proxy Servers:

: one or more c of QUIC-based proxy servers.  The number of addresses
is determined by the Length field. That is, the number of addresses is equal to 
(Length - 2) / 4.

~~~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |          option-code          |          option-len           |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |         Reserved            |Q|            Lifetime           |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     :            IPv6 Addresses of QUIC-based Proxy Servers         :
     |                                                               |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~
{: #fig-dhcpv6-option
   title="IPv4 Proxy Discovery DHCP option format"}
   
option-code:

: Proxy Discovery option code (TDB) (16 bit)

option-len:

: length of the option (without the Type and Length fields) in units of 8 octets.  The 
minimum value is 20 if one IPv6 address is contained in the option. Every additional IPv6
address increases the length by 16. (8-bit unsigned integer)

Q:

: is set to one, proxy supports QUIC on port 443 (1 bit)


Lifetime:

: maximum time in seconds (relative to the time the packet is received) over which these
IPv6 addresses can be used for proxy discovery. A value of all one bits (0xffffffff)
represents infinity. A value of zero means that the proxy addresses SHOULD no longer be
used. (16-bit unsigned integer)

IPv6 Addresses of QUIC-based Proxy Servers:

: one or more 128-bit IPv6 addresses of QUIC-based proxy servers.  The number of addresses
is determined by the Length field. That is, the number of addresses is equal to 
(Length - 1) / 2.
   
   
   
# Using IPv6 Neighbor Discovery for Local Discovery 

If a proxy is located in the local network, information to discover a proxy service
can be provided in a new Router Advertisement (RA) Option {{RFC4861}}, the Proxy Discovery
option. 

~~~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |     Type      |     Length    |           Reserved          |Q|
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                           Lifetime                            |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |                                                               |
     :            IPv6 Addresses of QUIC-based Proxy Servers         :
     |                                                               |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~
{: #fig-RA-option
   title="Proxy Discovery RA option format"}
                       

Type:

: Proxy Discovery option type (TDB) (8 bit)

Length:

: length of the option (including the Type and Length fields) in units of 8 octets.  The 
minimum value is 3 if one IPv6 address is contained in the option. Every additional IPv6
address increases the length by 2. (8-bit unsigned integer)

Q:

: is set to one, proxy supports QUIC on port 443 (1 bit)


Lifetime:

: maximum time in seconds (relative to the time the packet is received) over which these
IPv6 addresses can be used for proxy discovery. A value of all one bits (0xffffffff)
represents infinity. A value of zero means that the proxy addresses SHOULD no longer be
used. (32-bit unsigned integer)

IPv6 Addresses of QUIC-based Proxy Servers:

: one or more 128-bit IPv6 addresses of QUIC-based proxy servers.  The number of addresses
is determined by the Length field. That is, the number of addresses is equal to 
(Length - 1) / 2.

## Using PVDs

If the local network provides configuration with an Explicit
Provisioning Domain (PvD) {{I-D.ietf-intarea-provisioning-domains}},
the RA defined above can be used with the PvD Option or alternatively
proxy information can be retrieved in the additional information JSON
files associated with the PvD ID.  The endhost resolves the URL
provided in the PvD ID into an IP address using the local DNS server
that is associated with the corresponding PvD (see also section
3.4.4. {{I-D.ietf-intarea-provisioning-domains}}). If a QUIC-based
proxy services is provided the additional information JSON file
contains the key “QuicProxyIP”. It can then optionally also contain
more information about the specific proxy services offered using the
"ProxyService" key. Or the client can connect directly to the proxy
over QUIC on port 443 and request information about the proxy service
directly from the proxy server.

For remote network a Web PvD might be available that contains proxy
information. If provided, the PvD JSON configuration file retrievable
at the URI with the format:

     https://<Domain>/.well-known/pvd"

# DNS-based service discovery

{{RFC6763}} describes the use of SRV records to discover the available
instances of a type of service. To get a list of names of the
available instance for a certain service a client requests records of
type "PTR" (pointer from one name to another in the DNS namespace
{{RFC1035}} for a name containing the service and domain.

As specified in {{RFC6763}} the client can perform a PTR query for a
list of available proxy instance in following way:

    _quicproxy._udp.<domain>

here the \<domain> portion is the domain name where the service is
registered. The domain name can be obtained via DHCP options or
preconfigured.

The result of this PTR lookup is a set of zero or more PTR records
giving Service Instance names. Then to contact a particular service,
the client can query for the SRV {{RFC2782}} and TXT records of the
selected service instance name. The SRV record contains the IP address
of the proxy service instance as well as the port number. The port
number of QUIC-based proxy is usually expected to be 443 but may
differ. The TXT can contain additional information describing the kind
of proxy services that is offered.

"ToDo: format of TXT record using "key=value""
comment: {zahed} I would see the TXT record as a way to send
information about what functions the proxy can perform, hence would
rather suggest to focus on it later

## Local discovery using mDNS

{{RFC6762}} defines the use of ".local." for performing DNS like
operations on the local link. Any DNS query for a name ending "local."
will be send to predefined IPv4 or IPv6 link local multicast address.

To discovery QUIC-based proxy services locally, the client request the
PTR record for the name:

    _quicproxy._udp.local. 

The result of this PTR lookup is a set of zero or more PTR records
giving Service Instance Names of the form:

    <Instance>._quicproxy._udp.local.

Editors' Note: Or _masque._udp ? Or _proxy._quic._udp or _quicproxy._http._udp ...? 
However in the later case the proxy should also actually ofter a webpage...

## Discovery for a Remote Domains

If a client wants to discover a QUIC-based proxy server for a remote
domain, this domain has to be known by the client, e.g. being
preconfigured in the application.

# Using PCP options

Port Control Protocol (PCP), described in {{RFC6887}}, defines mechanism to do packet forwarding for different types of IPv4/Ipv6 Network Address Translators (NAT) or firewall. The usual deployment on PCP include Carrier-Grade NAT (CGN), Customer Permisis Equipment (CPE) and as well as residential NATs. Hence, the discovery of QUIC-based proxy can also be realized via PCP implementations.

PCP allows options tobe included in the PCP request and response header. The QUIC-based proxy information can be included in the response header as options. As {{RFC6887}} describes the client receiving the options that is dont understand should ignore them. 

A PCP option with QUIC-based proxy information is speficied below -

~~~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |  <Option Code>  |  Reserved     |            Length           |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     :           IP Addresses of QUIC-based Proxy Servers            :
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~
{: #fig-PCP-option
   title="Proxy Discovery PCP option format"}

The fields are described below -

Option Code:  

: 8 bits.  Its most significant bit indicates if this option is mandatory (0) or optional (1) to process.

Reserved:  

: 8 bits.  MUST be set to 0 on transmission and MUST be ignored on reception.

Option Length:  

:16 bits.  Indicates the length of the enclosed data, in octets.  Options with length of 0 are allowed.  Options that are not a multiple of 4 octets long are followed by one, two, or three 0 octets to pad their effective length in the packet to be a multiple of 4 octets.  The Option Length reflects the semantic length of the option, not including any padding octets.

IP Addresses of QUIC-based Proxy Servers:

: one or more 128-bit IPv6 addresses and/or 32-bit IPv4 addresses of QUIC-based proxy servers.  The number of addresses is determined by the Length field. That is, the number of addresses is equal to 
(Length - 1) / 2.

# Using Anycast address

Wellknown IP anycast address can be used to start communicating with
QUIC proxy or to discovery any/list of unicast address of a QUIC
proxy. When the proxy recieves the request for proxy functionalites
then it can either decide to reposond to the client from the anycast
address as source address or it can send back a list of unicast
address with a redirect command.

TODO: this needs more thinking before adding this as an
option. treat the current text as placeholder and for further
discussion on it

# IANA Considerations

IANA is requested to assign two DHCP options, one for IPv4 and one for
IPv6, in the "BOOTP Vendor Extensions and DHCP Options" registry
(http://www.iana.org/assignments/bootp-dhcp-parameters), as specified
in {{RFC2939}}, and the "Option Codes" registry under DHCPv6
parameters (http://www.iana.org/assignments/dhcpv6-parameters),
respectively, as well a new value for the Proxy Discovery Option in
the IPv6 Neighbor Discovery Option Formats registry.

This document adds a key to the “Additional Information PvD Keys”
registry, defined by {{I-D.ietf-intarea-provisioning-domains}}.

~~~~~
JSON key      | Description                        | Type             | Example
------------- | ---------------------------------- | ---------------- | ---
QuicProxyIP   | IP adress for QUIC-based proxies   | Array ot Strings | "["2001:db8:::1", "2001:db8:::2"]"
ProxyService  | IDs identifying a specific service | Array ot Strings | "["Forwarding", "DNSResolution"]"
~~~~~

Further, IANA is requested to register a new service name "quicproxy" in the "Service Name
and Transport Protocol Port Number Registry" 
(https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml).

# Security Consideration

> TBD

# Contributors



# Acknowledgments
