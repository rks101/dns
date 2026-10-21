# dns
DNS Domain Name System 

* dns
  * [NS Lookup](#ns-lookup) 
  * [Resolve DNS](#resolve-dns) 
  * [The One with DNS root nameservers](#the-one-with-dns-root-nameservers)
  * [Top Level Domain TLD](#Top-Level-Domain-TLD)
  * [`dig` into DNS](#dig-into-dns)
  * [DNS Hierarchy](#DNS-hierarchy)
  * [DNS Cache](#dns-cache)
  * [New domain](#new-domain)
  * [Know sub-domains](#know-sub\-domains)
  * [DNS and Firewall](#dns-and-firewall) 

## NS Lookup  

```
$ nslookup
> set q=ANY
> iitjammu.ac.in 
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
iitjammu.ac.in
	origin = ns1.iitjammu.ac.in
	mail addr = cc\@iitjammu.ac.in
	serial = 2020123155
	refresh = 2800
	retry = 3600
	expire = 1604800
	minimum = 86400
iitjammu.ac.in	nameserver = ns1.iitjammu.ac.in.
iitjammu.ac.in	nameserver = ns3.iitjammu.ac.in.
iitjammu.ac.in	nameserver = ns2.iitjammu.ac.in.
Name:	iitjammu.ac.in
Address: 14.139.53.140
iitjammu.ac.in	mail exchanger = 5 ALT2.ASPMX.L.GOOGLE.COM.
iitjammu.ac.in	mail exchanger = 5 ALT1.ASPMX.L.GOOGLE.COM.
iitjammu.ac.in	mail exchanger = 3 ASPMX.L.GOOGLE.COM.
iitjammu.ac.in	text = "google-site-verification=FbfesMgJWj_x98cASxF4B3J5t9wr0ccF_LXLmKZI1d4"
iitjammu.ac.in	text = "MS=0803D61A210443353771F37FAB6297221EF56F2E" "3600"
iitjammu.ac.in	text = "v=spf1 include:_spf.google.com ~all"

Authoritative answers can be found from:                         <== sample output, not guaranteed the same 
ns1.iitjammu.ac.in	internet address = 14.139.53.132
ns2.iitjammu.ac.in	internet address = 14.139.53.133
ns3.iitjammu.ac.in	internet address = 182.76.238.118
> exit
```

Another way to nslookup: use -debug for DNS record type.   

```
$ nslookup -type=ns -debug iitjammu.ac.in 
Server:		127.0.0.53
Address:	127.0.0.53#53

------------
    QUESTIONS:
	iitjammu.ac.in, type = NS, class = IN
    ANSWERS:
    ->  iitjammu.ac.in
	nameserver = ns3.iitjammu.ac.in.
	ttl = 8600
    ->  iitjammu.ac.in
	nameserver = ns1.iitjammu.ac.in.
	ttl = 8600
    ->  iitjammu.ac.in
	nameserver = ns2.iitjammu.ac.in.
	ttl = 8600
    AUTHORITY RECORDS:
    ADDITIONAL RECORDS:
    ->  ns1.iitjammu.ac.in
	internet address = 14.139.53.132
	ttl = 8600
    ->  ns2.iitjammu.ac.in
	internet address = 14.139.53.133
	ttl = 8600
    ->  ns3.iitjammu.ac.in
	internet address = 182.76.238.118
	ttl = 8600
------------
Non-authoritative answer:
iitjammu.ac.in	nameserver = ns3.iitjammu.ac.in.
iitjammu.ac.in	nameserver = ns1.iitjammu.ac.in.
iitjammu.ac.in	nameserver = ns2.iitjammu.ac.in.

Authoritative answers can be found from:
ns1.iitjammu.ac.in	internet address = 14.139.53.132
ns2.iitjammu.ac.in	internet address = 14.139.53.133
ns3.iitjammu.ac.in	internet address = 182.76.238.118
```

----

## Resolve DNS  

You can check /etc/resolve.conf or use resolvectl Domain Names using resolvectl.  

```
$ man resolvectl 

$ resolvectl dns
Global:
Link 2 (eno2):
Link 3 (eno1): 14.139.53.132 8.8.8.8          <== list of DNS servers, note public DNS 

$ resolvectl status
Global
       Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
resolv.conf mode: stub

Link 2 (eno2)
Current Scopes: none
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 3 (eno1)
    Current Scopes: DNS
         Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 14.139.53.132
       DNS Servers: 14.139.53.132 8.8.8.8
        DNS Domain: iitjammu.ac.in
```

When you type a URL in a web browser and get a DNS_PROBE_FINISHED_NXDOMAIN error, this suggests that the domain name you are requesting is not found, not registered, or has expired.    

----

## The One with DNS root nameservers  

DNS remains one of the most interesting topics in networking.    

Do you know that each DNS resolver always knows the IP addresses of DNS root nameservers? This is not a new thing; this has always been the case. This info is actually hardwired or [hardcoded into BIND-9](https://github.com/isc-projects/bind9/blob/main/lib/dns/rootns.c) (bind9/lib/dns/rootns.c). [Old link](https://gitlab.isc.org/isc-projects/bind9/blob/4c3b063ef8bd6e47b13c1dac3087daa1301a78ac/lib/dns/rootns.c) has moved to GitHub. Look at this code [root_ns array](https://github.com/rks101/egnet/blob/main/images/root_ns.png); there are 13 DNS root nameservers, and the names are not case-sensitive. You can find A type DNS records for these 13 root name servers named from a to m (as of Jan 2022).    

[IANA](https://www.iana.org/) has listed the [root name servers](https://www.iana.org/domains/root/servers), and [DNS root hint and root zone files](https://www.iana.org/domains/root/files). [Map](https://www.google.com/maps/d/u/0/viewer?mid=1LcHEpzl-7RzziWzDa4h3BxJcbEo&hl=en&ll=24.71341537554179%2C36.13137070989961&z=2) of geographically distributed root name servers reachable using anycast - the same IP mapped to multiple DNS root servers - the one reachable first is returned in the DNS query's answer section.    

Note: BIND (Berkeley Internet Name Domain) is an implementation of a naming service or DNS used in our endpoint devices and networks to connect to the internet. [BIND source code](https://gitlab.isc.org/isc-projects/bind9) is hosted by ISC (Internet Systems Consortium). It was developed at UCB in 1984 and later maintained by ISC.     

Note: How do we/local DNS servers reach a particular root nameserver while they are all managed by different entities globally? Using IP Anycast => As long as we get a response from any of them, it is fine to proceed further. No broadcast query or multicast query is used to locate a root nameserver.    

To get a list of root nameservers and their IP addresses, type dig and see the answer section and additional sections. An A-type record shows an IPv4 address, while an AAAA-type record shows an IPv6 address.    

```
$ dig 

; <<>> DiG 9.18.12-1-Debian <<>>
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49409
;; flags: qr rd ra; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 27

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;.                              IN      NS

;; ANSWER SECTION:
.                       7173    IN      NS      c.root-servers.net.
.                       7173    IN      NS      e.root-servers.net.
.                       7173    IN      NS      i.root-servers.net.
.                       7173    IN      NS      b.root-servers.net.
.                       7173    IN      NS      j.root-servers.net.
.                       7173    IN      NS      g.root-servers.net.
.                       7173    IN      NS      a.root-servers.net.
.                       7173    IN      NS      d.root-servers.net.
.                       7173    IN      NS      l.root-servers.net.
.                       7173    IN      NS      k.root-servers.net.
.                       7173    IN      NS      f.root-servers.net.
.                       7173    IN      NS      h.root-servers.net.
.                       7173    IN      NS      m.root-servers.net.

;; ADDITIONAL SECTION:
i.root-servers.net.     7173    IN      AAAA    2001:7fe::53
d.root-servers.net.     7173    IN      A       199.7.91.13
m.root-servers.net.     7173    IN      A       202.12.27.33
b.root-servers.net.     7173    IN      A       199.9.14.201
g.root-servers.net.     7173    IN      A       192.112.36.4
m.root-servers.net.     7173    IN      AAAA    2001:dc3::35
e.root-servers.net.     7173    IN      A       192.203.230.10
l.root-servers.net.     7173    IN      AAAA    2001:500:9f::42
e.root-servers.net.     7173    IN      AAAA    2001:500:a8::e
f.root-servers.net.     7173    IN      A       192.5.5.241
h.root-servers.net.     7173    IN      A       198.97.190.53
a.root-servers.net.     7173    IN      A       198.41.0.4
h.root-servers.net.     7173    IN      AAAA    2001:500:1::53
a.root-servers.net.     7173    IN      AAAA    2001:503:ba3e::2:30
d.root-servers.net.     7173    IN      AAAA    2001:500:2d::d
k.root-servers.net.     7173    IN      AAAA    2001:7fd::1
c.root-servers.net.     7173    IN      A       192.33.4.12
j.root-servers.net.     7173    IN      AAAA    2001:503:c27::2:30
i.root-servers.net.     7173    IN      A       192.36.148.17
k.root-servers.net.     7173    IN      A       193.0.14.129
g.root-servers.net.     7173    IN      AAAA    2001:500:12::d0d
c.root-servers.net.     7173    IN      AAAA    2001:500:2::c
b.root-servers.net.     7173    IN      AAAA    2001:500:200::b
j.root-servers.net.     7173    IN      A       192.58.128.30
l.root-servers.net.     7173    IN      A       199.7.83.42
f.root-servers.net.     7173    IN      AAAA    2001:500:2f::f

;; Query time: 0 msec
;; SERVER: 10.0.2.3#53(10.0.2.3) (UDP)
;; WHEN: Tue May 09 01:47:36 EDT 2023
;; MSG SIZE  rcvd: 811
```

Check these informative pages: [b.root-servers.org home](https://b.root-servers.org/) handling 5 billion queries a day, [a.root-servers.org metrics](https://a.root-servers.org/metrics), and [root-servers.org - map of root nameservers](https://root-servers.org/).    

Q. There are countably finite 13 root nameservers (from a to m) run by 12 operator entities.   
There are 2000+ instances of them by April 2026.   
So, how do the billions of queries per day to root name servers route? This must be interesting!    
A. In the class.   

----

There are some privately hosted **Public DNS Servers** so that everyone does not need a local DNS server:     
Type in a web browser: 1.1.1.1 or 8.8.8.8   

----

On Ubuntu or similar distro: [DNS config using BIND](https://ubuntu.com/server/docs/service-domain-name-service-dns)   

----

[DNS playground](https://messwithdns.net/) by Julia Evans   

----

## Top Level Domain TLD    

A Top-level Domain or TLD is the most right-end part of a domain name. e.g., TLD for godaddy.com is .com    

There are gTLD (Generic Top Level Domain) and ccTLD (Country Code Top Level Domain).    

A gTLD is meant to describe the domain name's purpose. e.g., gTLD .com is for commercial entities, .edu is for education, and .gov is for the government.    

A ccTLD is meant to describe a country or geography. e.g., .in for sites hosted/based in India, .co.uk for sites based in the United Kingdom.     

These days, there are too many new gTLDs like .bank, .sbi, .bharti, .biz, .coffee, .dell, etc. A full list of gTLDs is available on the [IANA](https://data.iana.org/TLD/tlds-alpha-by-domain.txt) website.   

Further, second-level domains are registered by organizations. When registering a domain name, the second-level domain name is limited to 63 characters + the TLD and can only use a-z, 0-9, and in-between hyphens (cannot start or end with hyphens or have consecutive hyphens). Subdomains added by domain owners have the same limitations. The maximum length of a domain name is 253 characters, including multiple subdomain prefixes like abc.def.lmn.pqrs.example.com     

----

## dig into DNS

Dig into DNS and query A (IP Address), SOA (Start of Authority - admin record), NS (name server), MX (mail server), TXT (domain ownership, to prevent mail spam), CNAME (canonical name or alias) records. Pay attention to the QUESTION, ANSWER, AUTHORITY, and ADDITIONAL sections in the output of dig.    

Note:- Do not ignore DNS TTL values. Sys admin should set DNS TTL values appropriately. See a few [DNS TTL basics](https://www.varonis.com/blog/dns-ttl) and [SOA TTL values](https://ns1.com/resources/understanding-ttl-values-in-dns-records).    

Note:- Before making a major change, or to make a change that propagates presumably faster,   
i) Lower the TTL of the concerned record to get it to expire sooner,   
ii) make the change, and   
iii) edit TTL to a suitable value in the next 48-72 hours.    

$ dig @server hostname recort_type

```
dig iitjammu.ac.in MX

; <<>> DiG 9.16.1-Ubuntu <<>> iitjammu.ac.in MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10684
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;iitjammu.ac.in.			IN	MX

;; ANSWER SECTION:
iitjammu.ac.in.		8600	IN	MX	3 ASPMX.L.GOOGLE.COM.
iitjammu.ac.in.		8600	IN	MX	5 ALT1.ASPMX.L.GOOGLE.COM.
iitjammu.ac.in.		8600	IN	MX	5 ALT2.ASPMX.L.GOOGLE.COM.

;; Query time: 27 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Sun Feb 07 16:56:34 IST 2021
;; MSG SIZE  rcvd: 119

```
----
You can use +nocmd +noall +answer flags for a clean and simple output.    

```
$ dig +nocmd iitjammu.ac.in MX +noall +answer   

iitjammu.ac.in.		7174	IN	MX	5 ALT1.ASPMX.L.GOOGLE.COM.
iitjammu.ac.in.		7174	IN	MX	5 ALT2.ASPMX.L.GOOGLE.COM.
iitjammu.ac.in.		7174	IN	MX	3 ASPMX.L.GOOGLE.COM.
```

----

Use dig to query the hostname IP address using public DNS, a faster way.    
```
$ dig @8.8.8.8 iitjammu.ac.in

; <<>> DiG 9.16.1-Ubuntu <<>> @8.8.8.8 iitjammu.ac.in
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52538
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;iitjammu.ac.in.			IN	A

;; ANSWER SECTION:
iitjammu.ac.in.		8599	IN	A	14.139.53.140

;; Query time: 463 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Fri Feb 12 14:25:35 IST 2021
;; MSG SIZE  rcvd: 59

```
---- 

Use dig to find the DNS trace leading to a hostname (much like traceroute).    

Pay attention to root nameservers, [DNS registrar](https://www.cloudflare.com/en-gb/learning/dns/glossary/what-is-a-domain-name-registrar/), and intermediate authoritative servers.    

This information is in the public domain. DNS is a global public directory of IP addresses and hostnames.    

We will see in the later section another iterative way to reach the same answer - the IP address of a domain name, iitjammu.ac.in.   

```
$ dig +trace iitjammu.ac.in

; <<>> DiG 9.16.1-Ubuntu <<>> +trace iitjammu.ac.in
;; global options: +cmd
.			72604	IN	NS	e.root-servers.net.
.			72604	IN	NS	j.root-servers.net.
.			72604	IN	NS	c.root-servers.net.
.			72604	IN	NS	h.root-servers.net.
.			72604	IN	NS	a.root-servers.net.
.			72604	IN	NS	b.root-servers.net.
.			72604	IN	NS	m.root-servers.net.
.			72604	IN	NS	k.root-servers.net.
.			72604	IN	NS	g.root-servers.net.
.			72604	IN	NS	i.root-servers.net.
.			72604	IN	NS	f.root-servers.net.
.			72604	IN	NS	l.root-servers.net.
.			72604	IN	NS	d.root-servers.net.
;; Received 262 bytes from 127.0.0.53#53(127.0.0.53) in 7 ms

in.			172800	IN	NS	ns1.registry.in.
in.			172800	IN	NS	ns2.registry.in.
in.			172800	IN	NS	ns3.registry.in.
in.			172800	IN	NS	ns4.registry.in.
in.			172800	IN	NS	ns5.registry.in.
in.			172800	IN	NS	ns6.registry.in.
in.			86400	IN	DS	54739 8 1 2B5CA455A0E65769FF9DF9E75EC40EE1EC1CDCA9
in.			86400	IN	DS	54739 8 2 9F122CFD6604AE6DEDA0FE09F27BE340A318F06AFAC11714A73409D4 3136472C
in.			86400	IN	RRSIG	DS 8 1 86400 20210225050000 20210212040000 42351. lgk6+SUs00ldOZQLKKvskdt9680VM6ShM5aFmpp+LNsrHzMIFufwQ592 wOqMOxRcdpvjf6W3PvNNZ1SYeWj3ETBZAwRUicNbfaLAv3aVjpO/Rjke VkHt8h8b5AUrFqG3wPbmmYegESbdbg1MphFovL/LP/0b+HW1/RKcX1Wj OPHmwF9VTrbFovqxULB0M5pTnNqisLK3nYYFLLnrNVvhlyo+vFkmMY3/ ZYQCt0a+KlgS5efJEuKCAoxruICkOFh9fbCWiJtKGfYcKbNgfA4kZMe1 HzE1V9+OG/ctnwCJNdFyGP2hs4z1K8zPwJwBNFZa6d54VpCDHLbWgR2c gRQ19w==
;; Received 795 bytes from 199.7.91.13#53(d.root-servers.net) in 279 ms

iitjammu.ac.in.		3600	IN	NS	ns2.iitjammu.ac.in.
iitjammu.ac.in.		3600	IN	NS	ns1.iitjammu.ac.in.
idj3ou7anjce3n70hpktjgs1q54d9usj.ac.in.	1800 IN	NSEC3 1 1 1 F7BDE4B2 TV5S7T20F21KQV26R05RHMAECLP59H6U NS SOA RRSIG DNSKEY NSEC3PARAM TYPE65534
idj3ou7anjce3n70hpktjgs1q54d9usj.ac.in.	1800 IN	RRSIG NSEC3 8 3 1800 20210311064755 20210209054755 5223 ac.in. WYooneOeJLAXxPUx8rxpFg8Ncvmairb6Ja4k4n9X3kClIrpgCxd2Cdn0 hHCBg+7ieNAf3j7E438d/oKFlsmpdESd3/TGzzy+3GZUwPKxJWGs3/xs QSEngsiaiJh2BxbxPMKW+TkJ+NfVQuYWhGwmGxasebov5L3yBMRueGOe ailqHPZwM8j5PkJGLdKVX+YZbun9WJfzWZw8hMQ1AX4coQ==
;; Received 426 bytes from 37.209.196.12#53(ns3.registry.in) in 47 ms

iitjammu.ac.in.		8600	IN	A	14.139.53.140
iitjammu.ac.in.		8600	IN	NS	ns1.iitjammu.ac.in.
iitjammu.ac.in.		8600	IN	NS	ns2.iitjammu.ac.in.
iitjammu.ac.in.		8600	IN	NS	ns3.iitjammu.ac.in.
;; Received 189 bytes from 14.139.53.133#53(ns2.iitjammu.ac.in) in 3 ms

```

Let's tinker more!    

You may have heard of recursive and iterative DNS queries. From your system to the nearest local DNS server, or authoritative DNS nameserver, queries are recursive - you are bound to get a DNS reply/answer. In practice, a local DNS server or authoritative DNS nameserver makes iterative queries from all the way up the root server to-> top-level-domain nameserver to-> next-level DNS registrar where your domain name is registered. Try this iterative DNS query thing:    

We ask a.root-servers.net (one of the 13 root DNS servers in the entire DNS hierarchy):   
Notice the sections in the output:   

```
$ dig @a.root-servers.net iitjammu.ac.in

; <<>> DiG 9.18.12-1-Debian <<>> @a.root-servers.net iitjammu.ac.in
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51998
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 6, ADDITIONAL: 13
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;iitjammu.ac.in.                        IN      A

;; AUTHORITY SECTION:
in.                     172800  IN      NS      ns1.registry.in.
in.                     172800  IN      NS      ns4.registry.in.
in.                     172800  IN      NS      ns5.registry.in.
in.                     172800  IN      NS      ns6.registry.in.
in.                     172800  IN      NS      ns3.registry.in.
in.                     172800  IN      NS      ns2.registry.in.

;; ADDITIONAL SECTION:
ns1.registry.in.        172800  IN      A       37.209.192.12
ns1.registry.in.        172800  IN      AAAA    2001:dcd:1::12
ns4.registry.in.        172800  IN      A       37.209.198.12
ns4.registry.in.        172800  IN      AAAA    2001:dcd:4::12
ns5.registry.in.        172800  IN      A       156.154.100.20
ns5.registry.in.        172800  IN      AAAA    2001:502:2eda::20
ns6.registry.in.        172800  IN      A       156.154.101.20
ns6.registry.in.        172800  IN      AAAA    2001:502:ad09::20
ns3.registry.in.        172800  IN      A       37.209.196.12
ns3.registry.in.        172800  IN      AAAA    2001:dcd:3::12
ns2.registry.in.        172800  IN      A       37.209.194.12
ns2.registry.in.        172800  IN      AAAA    2001:dcd:2::12

;; Query time: 159 msec
;; SERVER: 198.41.0.4#53(a.root-servers.net) (UDP)
;; WHEN: Sun May 07 23:47:41 EDT 2023
;; MSG SIZE  rcvd: 424

```
Dearest root server **a** did not tell us the IP address of our domain. It also indicated that a recursive query is not entertained. 

However, the root server **a** did return the TLD registrar for **in** domain (see TLD suffix in the domain name) and IP addresses to find those TLD registrars - as an authoritative DNS server in the AUTHORITY SECTION. Notice the ADDITIONAL SECTION for IP addresses.     

Further, let us ask the TLD in registrar ns1.registry.in:     

```
$ dig @ns1.registry.in iitjammu.ac.in   

; <<>> DiG 9.18.12-1-Debian <<>> @ns1.registry.in iitjammu.ac.in
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63974
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 3, ADDITIONAL: 4
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;iitjammu.ac.in.                        IN      A

;; AUTHORITY SECTION:
iitjammu.ac.in.         3600    IN      NS      ns1.iitjammu.ac.in.
iitjammu.ac.in.         3600    IN      NS      ns3.iitjammu.ac.in.
iitjammu.ac.in.         3600    IN      NS      ns2.iitjammu.ac.in.

;; ADDITIONAL SECTION:
ns3.iitjammu.ac.in.     3600    IN      A       182.76.238.118
ns2.iitjammu.ac.in.     3600    IN      A       14.139.53.133
ns1.iitjammu.ac.in.     3600    IN      A       14.139.53.132

;; Query time: 27 msec
;; SERVER: 37.209.192.12#53(ns1.registry.in) (UDP)
;; WHEN: Sun May 07 23:58:36 EDT 2023
;; MSG SIZE  rcvd: 145

```

Wow! Note the AUTHORITY SECTION ns1.iitjammu.ac.in, it is the local DNS server - it's configured as an authoritative DNS server. Let us ask him :)   

```
$ dig @ns1.iitjammu.ac.in iitjammu.ac.in

; <<>> DiG 9.18.12-1-Debian <<>> @ns1.iitjammu.ac.in iitjammu.ac.in
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11582
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 3, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;iitjammu.ac.in.                        IN      A

;; ANSWER SECTION:
iitjammu.ac.in.         10      IN      A       14.139.53.140

;; AUTHORITY SECTION:
iitjammu.ac.in.         8600    IN      NS      ns2.iitjammu.ac.in.
iitjammu.ac.in.         8600    IN      NS      ns3.iitjammu.ac.in.
iitjammu.ac.in.         8600    IN      NS      ns1.iitjammu.ac.in.

;; ADDITIONAL SECTION:
ns1.iitjammu.ac.in.     8600    IN      A       14.139.53.132
ns2.iitjammu.ac.in.     8600    IN      A       14.139.53.133
ns3.iitjammu.ac.in.     8600    IN      A       182.76.238.118

;; Query time: 4 msec
;; SERVER: 14.139.53.132#53(ns1.iitjammu.ac.in) (UDP)
;; WHEN: Mon May 08 00:08:27 EDT 2023
;; MSG SIZE  rcvd: 161

```

Bingo! note the ANSWER SECTION - this has an IP address of iitjammu.ac.in, and A record is returned. This completes the journey of iterative DNS queries.     
Compare these steps with the output of dig +trace iitjammu.ac.in to find the IP addresses of the domain. This is what happens in practice every single day.    

An illustration on [Life of a DNS query](https://github.com/rks101/egnet/blob/main/images/Life%20of%20a%20DNS%20query.png) summarizes DNS queries.     

----

DNS Software: What software is the DNS server using?    
Dig Dyno for version.bind, chaos class, and TXT record type; see the answer section.     

```
$ dig @ns1.iitjammu.ac.in version.bind chaos txt

; <<>> DiG 9.18.12-0ubuntu0.22.04.2-Ubuntu <<>> @ns1.iitjammu.ac.in version.bind chaos txt
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45153
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;version.bind.			CH	TXT

;; ANSWER SECTION:
version.bind.		0	CH	TXT	"9.9.4-RedHat-9.9.4-73.el7_6"

;; AUTHORITY SECTION:
version.bind.		0	CH	NS	version.bind.

;; Query time: 16 msec
;; SERVER: 14.139.53.132#53(ns1.iitjammu.ac.in) (UDP)
;; WHEN: Sat Jul 08 09:40:45 IST 2023
;; MSG SIZE  rcvd: 95
```

----

## DNS Hierarchy    

Q. Who has more entries in the DNS zone file - a root server, a TLD server, or a authoritative server?    
A. A **TLD server** has the most entries, by a wide margin.   

Detailed answer:    
**Root server**: There are 13 root servers. The root zone is small. It only holds delegations for the top-level domains (.com, .net, .org, .in, .cn, .ru, and so on), which come to roughly 1,500 TLDs. Each has a few NS records, glue A/AAAA records, and DS records for DNSSEC, so the whole zone is only a few thousand records.   
**TLD server**: Top-Level Domain server holds a delegation (NS records, plus glue A/AAAA and DS records where needed) for every registered domain under it. The .com zone has well over 150 million registered domains, so its zone file is enormous, with hundreds of millions of records.    
**Authoritative server** (for a single domain): The zone for something like example.com or iitb.ac.in usually has only a handful to a few hundred records (A, AAAA, SOA, NS, MX, TXT, CNAME, and so on). Large organizations can have more, but it's nowhere near TLD scale.    

The root servers point to the TLDs, the TLDs point to every domain's authoritative servers, and the authoritative servers hold the actual answers for their own domains.    

Q. Then, what is a recursive DNS server?    
A. Refer-how does a DNS query work.    
A **recursive DNS server** (also known as a recursive resolver) is the server that does the lookup work on behalf of a client (web browser). Your device doesn't walk the DNS hierarchy itself. It asks the recursive resolver, and the resolver goes and finds the answer.    

Say you look up www.iitb.ac.in and the resolver has nothing cached (no local copy/record for iitb.ac.in):    

- It asks a root nameserver: "Who handles .in?" The root nameserver replies with the .in TLD servers.
- It asks a TLD server: "Who handles iitb.ac.in?" The TLD replies with iitb.ac.in's authoritative servers.
- It asks the authoritative server: "What's the IP for www.iitb.ac.in?" This one gives the actual answer.
- It returns the answer to your device and caches it (stores a local copy/record) for the duration of the record's TTL (seconds).

A recursive DNS server:    
- Does the recursion: Chasing referrals from root nameserver to TLD server to authoritative server so clients don't have to.
- Caching: Storing answers (of resolved domains) so repeat lookups are answered instantly without going back through the DNS hierarchy. This caching greatly reduces load on root nameservers, TLD servers, and authoritative servers and makes browsing faster.
- Can validate and filter domains, if configured: Many recursive resolvers perform DNSSEC validation, and some block malware or adult domains.

Q. Where is recursive DNS server run?    
A. Typically your ISP (the default on most home networks), or a public resolver such as Cloudflare (1.1.1.1), Google (8.8.8.8), or Quad9 (9.9.9.9) runs it. Organizations often run their own recursive DNS server (recursive resolver) internally. 

Q. How does a recursive DNS server differ from an authoritative server?    
A. An authoritative server holds the official records (ground truth or good copy) for specific zones and only answers about those. A recursive DNS server or resolver holds no zones of its own. Yes, it has no zone information of its own. It just queries others and caches what it learns.  

----

## DNS Cache   

[**DNS Cache**](https://www.keycdn.com/support/dns-cache)    

[View and flush DNS Cache on Linux](https://www.makeuseof.com/view-and-flush-dns-cache-on-linux/), [link2](https://unix.stackexchange.com/questions/28553/how-to-read-the-local-dns-cache-contents)     

----

## New Domain 

While you add a new domain name for your organization/institute, the following steps are helpful:     
- decide on a new domain name (e.g., iiitkota.ac.in or iitjammu.ac.in)    
- register the domain at the DNS registrar (ERNET in India) or get one from a private domain service provider (e.g., godaddy.com)    
- provide the new domain name and public IP address of the authoritative nameserver (to find your new domain); this authoritative nameserver can be within another campus or with another ISP    
- Domain registrar adds two resource records (RRs) into the top-level-domain nameserver (e.g. ns1.registry.in), guess the entries from the above discussion, before you proceed further.     
e.g., 1) 
  -> (iiitkota.ac.in, dns1.iiitkota.ac.in, NS, TTL1)    
  -> (dns1.iiitkota.ac.in, 210.212.97.131, A, TTL2)     
e.g., 2) 
  -> (iitjammu.ac.in, dns8.iitd.ac.in, NS, TTL1)    
  -> (dns8.iitd.ac.in, 103.27.8.1, A, TTL2)    
- DNS authoritative name server can be updated after registration.    

----

## Under the hood of DNS   

DJB on [Secure design and coding for DNS](https://cr.yp.to/talks/2009.03.04/slides.pdf)    

[DNS source tree using BIND9](https://gitlab.isc.org/isc-projects/bind9/-/tree/4c3b063ef8bd6e47b13c1dac3087daa1301a78ac/lib/dns)    

----

## Know sub-domains    

Sometimes you wish to know sub-domains and they don't tell you :) so let us knock on the domain quietly.     
[get knock](https://github.com/guelfoweb/knock) or [puredns](https://github.com/d3mondev/puredns)    

There is this another awesome tool: **dnsenum**, that can reveal much more information about a domain.     
dnsenum can show domains, nameservers, MX records, zone transfer records for sub-domains, a range of public IP addresses, old records, etc.   
``
$dnsenum -r mydomain.ac.in   
``

You can knock :)     
```
$ python3 knockpy.py iitjammu.ac.in 

  _  __                 _                
 | |/ /                | |   v6.1.0            
 | ' / _ __   ___   ___| | ___ __  _   _ 
 |  < | '_ \ / _ \ / __| |/ / '_ \| | | |
 | . \| | | | (_) | (__|   <| |_) | |_| |
 |_|\_\_| |_|\___/ \___|_|\_\ .__/ \__, |
                            | |     __/ |
                            |_|    |___/ 

local: 10757 | remote: 33 .py                                                   

Wordlist: 10790 | Target: iitjammu.ac.in | Ip: 14.139.53.140 

12:48:27

Ip address      Code Subdomain                              Server                                 Real hostname
--------------- ---- -------------------------------------- -------------------------------------- --------------------------------------
14.139.53.136   200  apply.iitjammu.ac.in                                                                                                
10.10.120.180   200  beta.iitjammu.ac.in                                                                                                 
10.10.120.101        console.iitjammu.ac.in                                                                                              
174.138.122.103 200  ces.iitjammu.ac.in                     nginx/1.18.0 (Ubuntu)                                                        
14.139.53.135        depo.iitjammu.ac.in                                                                                                 
13.126.157.211  200  egdev.iitjammu.ac.in                   Apache                                                                       
13.126.157.211  200  eg.iitjammu.ac.in                      Apache                                                                       
13.126.157.211  200  egsec.iitjammu.ac.in                   Apache                                                                       
10.10.10.42          archive.iitjammu.ac.in                                                                                              
10.10.10.170    200  apc.iitjammu.ac.in                                                                                                  
14.139.53.135        eservices.iitjammu.ac.in                                                                                            
14.139.53.135   200  idp.iitjammu.ac.in                     nginx/1.14.0 (Ubuntu)                                                        
10.10.10.44     200  intranet.iitjammu.ac.in                                                                                             
10.10.120.55         elearn.iitjammu.ac.in                                                                                               
10.10.10.43     200  list.iitjammu.ac.in                    nginx/1.14.0 (Ubuntu)                                                        
14.139.53.139   200  lists.iitjammu.ac.in                   nginx/1.14.0 (Ubuntu)                                                        
14.139.53.130   200  lms.iitjammu.ac.in                     Apache/2.4.41 (Ubuntu)                                                       
142.250.194.179 404  mail.iitjammu.ac.in                    ghs                                    ghs.googlehosted.com
10.10.10.53          ipa.iitjammu.ac.in                                                                                                  
10.10.10.45     200  libopac.iitjammu.ac.in                 Apache/2.4.29 (Ubuntu)                                                       
10.10.10.46          dspace.iitjammu.ac.in                                                                                               
10.10.96.254         eye.iitjammu.ac.in                                                                                                  
10.10.194.50         firewall.iitjammu.ac.in                                                                                             
10.10.10.76          mrtg.iitjammu.ac.in                                                                                                 
10.10.28.7           skt.iitjammu.ac.in                                                                                                  
10.10.120.113   200  ssp.iitjammu.ac.in                     Apache/2.4.29 (Ubuntu)                                                       
206.189.202.152      videoconf.iitjammu.ac.in                                                                                            
10.10.120.196   200  wifi.iitjammu.ac.in                    Apache/2.4.41 (Ubuntu)                                                       
14.139.53.140   200  www.iitjammu.ac.in                                                                                                  
14.139.53.133        ns2.iitjammu.ac.in                                                                                                  
182.76.238.118       ns3.iitjammu.ac.in                                                                                                  
14.139.53.132        ns1.iitjammu.ac.in                                                                                                  
14.139.53.129        vpn.iitjammu.ac.in                                                                                                  
                                                                                
12:48:49

Ip address: 30 | Subdomain: 33 | elapsed time: 00:00:22 

```

There is another DNS enumeration and scanning tool **dnsrecon** - a Python script to collect DNS and sub-domain-related information as passive reconansance.    

**With dig, dnsenum, and dnsrecon, you can figure out DNS entries in detail like TTL, record type, and record values for TXT.      **

dnsrecon man page tells us the type of enumerations or queries:   
```
         -t TYPE, --type TYPE  Type of enumeration to perform.
                      Possible types:
                                   std:      SOA, NS, A, AAAA, MX and SRV.
                                   rvl:      Reverse lookup of a given CIDR or IP range.
                                   brt:      Brute force domains and hosts using a given dictionary.
                                   srv:      SRV records.
                                   axfr:     Test all NS servers for a zone transfer.
                                   bing:     Perform Bing search for subdomains and hosts.
                                   yand:     Perform Yandex search for subdomains and hosts.
                                   crt:      Perform crt.sh search for subdomains and hosts.
                                   snoop:    Perform cache snooping against all NS servers for a given domain, testing
                                             all with file containing the domains, file given with -D option.
                                   tld:      Remove the TLD of given domain and test against all TLDs registered in IANA.
                                   zonewalk: Perform a DNSSEC zone walk using NSEC records.
```

Here is sample output from dnsrecon -t axfr -d domain.com 

```
$ dnsrecon -t axfr -d iitjammu.ac.in 
[*] Checking for Zone Transfer for iitjammu.ac.in name servers
[*] Resolving SOA Record
[+] 	 SOA ns1.iitjammu.ac.in 14.139.53.132
[*] Resolving NS Records
[*] NS Servers found:
[+] 	 NS ns3.iitjammu.ac.in 182.76.238.118
[+] 	 NS ns2.iitjammu.ac.in 14.139.53.133
[+] 	 NS ns1.iitjammu.ac.in 14.139.53.132
[*] Removing any duplicate NS server IP Addresses...
[*]  
[*] Trying NS server 14.139.53.132
[+] 14.139.53.132 Has port 53 TCP Open
[+] Zone Transfer was successful!!
[*] 	 NS ns1.iitjammu.ac.in 14.139.53.132
[*] 	 NS ns2.iitjammu.ac.in 14.139.53.133
[*] 	 NS ns3.iitjammu.ac.in 182.76.238.118
[*] 	 TXT google-site-verification=FbfesMgJWj_x98cASxF4B3J5t9wr0ccF_LXLmKZI1d4
[*] 	 TXT v=spf1 include:_spf.google.com ~all
[*] 	 TXT google-site-verification=_CjOVWkrqVhYi2yABaGiRoYeJwkpyMdG6Ms1FW7VJu8
[*] 	 TXT MS=0803D61A210443353771F37FAB6297221EF56F2E; 3600
[*] 	 TXT v=DMARC1; p=none; rua=mailto:dmarc@iitjammu.ac.in
[*] 	 TXT v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA3vFaf45bo5qlObTebeNjatWQ3X372gQTM26tjrRsH3lVFYvj8dKe90thOpLcFM6w/lAA2PqPO/XenfuWtNVPg; G48ZN8zRh8MjnlSL4GCqkB1Ni+aewHjKUnfIsCUMWdjBdFm/l/gNx5q9IB+/wO56EUNrK4Y9yH9OJjq+mEeQlqHs8YA2qCGeN7csdW0Y75Izf+E0pX5dyHI3YjLb; etL3INS2hbgbX6nvegap7EaMSXgEH1znoQvUqA7qhkmbLGsec1C9lpahUeuWLtwiTulWg1dQQyEOTGzI1LOuVUdLnQCQiEn4+JUdwcWd7V7vHYZ58rwivGIblehaKsqPXcr/QIDAQAB
[*] 	 TXT v=spf1 include:_spf.google.com ~all
[*] 	 TXT 5C-P8ZVTFBylBN_fwdYQ2uirwgwm1NY9prAJVROJunc
[*] 	 TXT v=spf1 a mx ip4:14.139.53.129 include:_spf.google.com ~all
[*] 	 TXT v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAny/; YMsWkcHJ4Q4+sdycU74ISvZFFfsk7OtxXqV+uMFrUH+F7UF2yEdJtKLgFXnWIyPF2li5HHU4skJ3o2P1iujF/; HXRSw4E24n+Wfc0kErpSn0vDW05MyeApxZqvuMcwF6kq8TNRrnCTalw+zK6GdtLms+TdwOeWsQSaDFR0uxj9EkiNG5aCobiPAeThye0++toMBmCjSe7SKx86VStHE2p4mO/; kRCcW88twZgJLY+Ic8D039DM/cGF/oJf8DliAf3hbcbkFAd0Udl92/wgg+RQRNzKKOR1enAOlsts/; Il3MWvwiaUMXh6fcybWJSDkuikNRjm3QRhQGGRUSbauDOwIDAQAB
[*] 	 TXT M6bwxKX9rA6bUfvapGe0mEaX6LGFMi8qB7Q5u8YxMjM
[*] 	 MX @.iitjammu.ac.in ASPMX.L.GOOGLE.COM 172.217.194.26
[*] 	 MX @.iitjammu.ac.in ASPMX.L.GOOGLE.COM 2404:6800:4003:c1c::1a
[*] 	 MX @.iitjammu.ac.in ALT1.ASPMX.L.GOOGLE.COM 173.194.202.27
[*] 	 MX @.iitjammu.ac.in ALT1.ASPMX.L.GOOGLE.COM 2607:f8b0:400e:c00::1b
[*] 	 MX @.iitjammu.ac.in ALT2.ASPMX.L.GOOGLE.COM 173.194.65.26
[*] 	 MX @.iitjammu.ac.in ALT2.ASPMX.L.GOOGLE.COM 2607:f8b0:4023:1c03::1b
[*] 	 MX alumni.iitjammu.ac.in ASPMX.L.GOOGLE.COM 172.217.194.26
[*] 	 MX alumni.iitjammu.ac.in ASPMX.L.GOOGLE.COM 2404:6800:4003:c1c::1a
[*] 	 MX alumni.iitjammu.ac.in ALT1.ASPMX.L.GOOGLE.COM 173.194.202.27
[*] 	 MX alumni.iitjammu.ac.in ALT1.ASPMX.L.GOOGLE.COM 2607:f8b0:400e:c00::1b
[*] 	 MX alumni.iitjammu.ac.in ALT2.ASPMX.L.GOOGLE.COM 173.194.65.26
[*] 	 MX alumni.iitjammu.ac.in ALT2.ASPMX.L.GOOGLE.COM 2607:f8b0:4023:1c03::1b
[*] 	 A @.iitjammu.ac.in 14.139.53.140
[*] 	 A acs1.iitjammu.ac.in 10.10.10.252
[*] 	 A acs2.iitjammu.ac.in 10.10.120.229
[*] 	 A agastya.iitjammu.ac.in 14.139.53.143
[*] 	 A aicpmu.iitjammu.ac.in 14.139.53.138
[*] 	 A aicpmudev.iitjammu.ac.in 10.10.120.154
[*] 	 A akela.iitjammu.ac.in 10.10.52.6
...........................
```

---- 

## DNS and Firewall 

Sometimes DNS and firewall/WAF can make things complicated.    

DNS Traffic Dropped as Threat Due to iCloud Private Relay Domains[https://knowledgebase.paloaltonetworks.com/KCSArticleDetail?id=kA14u000000PRR2CAO]    
- Users may experience connectivity issues when browsing or using apps that rely on Private Relay.   
- The firewall categorizes the domains as Proxy Avoidance and Anonymizers and drops the traffic.
Resolution: Add an exception for Private Relay domain names, or at the client side - disable Private Relay.   

----
