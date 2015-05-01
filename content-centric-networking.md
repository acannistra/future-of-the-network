<!--Styles-->
<style>
	img {
		padding-left: 15%;
		padding-right: 15%;
		width: 70%
	}
	.caption{
		text-align: center;
		font-style: italic;
	}
</style>

[top]: #top "go to top"
<!-- Global References -->
[ccnx]: http://www.ccnx.org "CCNx Website" 
[ccnx_paper]: http://conferences.sigcomm.org/co-next/2009/papers/Jacobson.pdf "CCNx Paper"
[ccnx_wiki]: http://en.wikipedia.org/wiki/Content_centric_networking "CCNx Wikipedia Article"
[van_google]: https://www.youtube.com/watch?v=gqGEMQveoqg "Van Jacobson Google Talk on CCNx"
[CA_issues]: https://konklone.com/post/certificate-authorities-are-actually-a-tremendous-problem "Certificate Authorities are actually a Tremendous Problem"

[IDS:principles]: http://www.cs.tufts.edu/comp/150IDS/principles "COMP150-IDS Principles"

<!-- end: GLobal References-->
<!-- Contextual References-->

<!-- end: Contextual References-->
<a name="top"></a>
# The Future of The Network 

[Tony Cannistra](mailto:anthony.cannistra@tufts.edu) // May 01, 2015 // [COMP150-IDS](http://www.cs.tufts.edu/comp/150IDS)

<!--Table of Contents-->
[TOC] 



The way we interact with the world's information is changing, and our current TCP/IP-based network that supports the Web is faced with real limitations as a result. It is the goal of this paper to explore these limitations through the lens of a newly-proposed technology: content-centric networking. We first present the specifications of content-centric networking. We'll then explore the feasibility of the global adoption of content-centric networking (henceforth referred to as CCNx[^1]) into today's information-distribution platforms. 

## CCNx: A Primer

Content-centric networking was first described in [a paper][ccnx_paper] co-authored by developer Van Jacobson, together with colleagues at Xerox PARC, in response to the shortcomings they perceived in the current TCP/IP-based architecture of the Web. [Project CCNx][ccnx] was launched in  to maintain the CCNx specification and open-source implementations. The purpose of this section is to describe the CCNx protocol and its motivations. We begin with the latter. 

### Motivation
The TCP/IP-based architecture of the Internet today was built with the intent to connect *devices* to *other devices* in a way that satisfies several key invariants, especially: network redundancy, ease of connection, and global network interoperability. TCP/IP was built with a focus on *where machines are* and *how to talk to them*, rather than *what their content is*. To quote [Jacobson][ccnx_paper]: Today, "People value the Internet for *what* content it contains, but communication is still in terms of *where*." 

This ideological limitation is the root of a larger set of practical concerns. Jacobson et al. maintain that the three main issues that arise from the TCP/IP architecture of the web are **availability**, **security**, and **location-dependence**. For content to be highly-available (i.e. accessible quickly and reliably for a diverse set of users across the networks' topology), current TCP/IP-based networks require content-delivery-networks (CDNs) or Peer-to-Peer networks to be overlaid on top of existing infrastructure. In addition, only those who can afford access to high-bandwidth connections or CDN services like Akamai can provide highly-available content. Jacobson gives the example in his [Google talk][van_google] of the NBC broadcast infrastructure and how it handled the considerable increase in traffic after Bode Miller's [performance](http://en.wikipedia.org/wiki/Bode_Miller#2006_Olympics_controversy) at the 2006 Olympics; after Bode missed a gate skiing downhill, spurning a very high number of people requesting the video, the main NBC router was maintaining 6,000 copies of the video, one for each TCP connection. This serves as a great example of the dissonance between the TCP/IP architecture and this model of broadcast data distribution. 

In addition, the current web architecture allows for *pipe security* rather than the more-important *data security*. Jacobson, in [a Google Tech Talk][van_google] delivered in 2007, highlights the fact that the way we've built security on top of the TCP/IP protocol encrypts entire connections, rather than just the data transferred across those connections. Therefore, services like caching and availability are affected. In addition to this, the current verification scheme relies upon the certificate-authority structure, which has been [shown][CA_issues] to have significant trust issues. 

Finally, the TCP/IP system as optimized for locating a *machine* rather than locating *content* means that those content consumers located topologically closer to the machine that contains the desired content experience a much higher Quality of Service (QoS) than those further away. Packets must be sent through a much larger number of intervening nodes to deliver content from one connected subgraph of the Internet to another subgraph with few connections to the source network.


It was the goal of Jacobson et al., and later [Project CCNx][ccnx], to address these issues with a content-centric networking paradigm. We explore the specifications of the CCNx protocol next. 

### The CCNx Specification
<!--references-->
[ccnx_arch]: https://www.ccnx.org/architecture-overview/ "CCNx Architecture Overview"
[ccnx_arch:names]: https://www.ccnx.org/names/ "CCNx Naming Schemes"
[ccnx_arch:packet]: https://www.ccnx.org/packet-format/ "CCNx Packet Format" 
[ccnx_arch:packet:interest]: https://www.ccnx.org/interest/ "CCNx Interest Spec"
[ccnx_arch:packet:content]: https://www.ccnx.org/content-object/ "CCNx Content-Object Spec"
[ccnx_arch:protocol:contentstore]: https://www.ccnx.org/content-store/ "CCNx Content Store Spec"
[ccnx_arch:protocol:forwarders]: https://www.ccnx.org/forwarders/ "CCNx Forwarder Spec"
[ccnx_semantics]: http://tools.ietf.org/html/draft-mosko-icnrg-ccnxsemantics-01 "CCNx IETF Semantics, March 2015"

[rfc_uris]: https://tools.ietf.org/html/rfc3986 "URI Generic Syntax"

<!---->

The Project CCNx [architecture page][ccnx_arch] is the central location for the specification of the architecture and protocol of the proposed content-centric networking system. It is largely from that page and from the [Jacobson et al. paper][ccnx_paper] that this architecture overview is based. The purpose of this section is not to re-create the existing CCNx documentation, and so many important considerations will be glossed over. However, we will attempt to address the important features of the CCNx architecture. 

#### Network Components
A content-centric network consists of Content Producers, Content Publishers, and Content Consumers, supported by [Forwarders][ccnx_arch:protocol:forwarders] and  [Content Stores][ccnx_arch:protocol:contentstore]. Content Producers create data to be consumed by Content Consumers, and publish it to the CCNx network using CCNx protocols via the Content Publisher nodes (more on this later). Content Consumers retrieve the data produced by Content Producers using the CCNx protocol. **Content Publishers** assign globally-identifiable names to content produced by Content Producers, which are used by Content Consumers to refer to the content and by intra-network nodes to authenticate and verify the content's authenticity. 

#### Messages + Protocols: Getting Content from the CCNx
The content-centric networking protocol has two types of messages for the acquisition of content: [`Interest`][ccnx_arch:packet:interest] packets and [`Content`][ccnx_arch:packet:content] packets. Consumers send `Interest` packets containing the name of the content to be accessed (more on naming later!). The `Interest` packets are received by [Forwarder][ccnx_arch:protocol:forwarders] nodes which, based on the hierarchical nature of the name, forward the `Interest` packet to Content Stores or other Forwarder nodes until the desired [Content Object][ccnx_arch:packet:content] is reached. 

Because content is identified uniquely by it's name, Content Objects can be cached at any Forwarder or Content Store node. This means that `Interest` messages do not have to travel all the way back to the Content Publisher node to find the Content Object being searched for. In addition, as `Interest` packets move through the Forwarder nodes in the network, they leave "breadcrumbs" as they move. This means that when an `Interest` packet arrives at a node containing the desired Content Object, it back-tracks the `Content` packet through the network to arrive back at the requester. 

There are several non-cryptographic Content verification mechanisms built into the Forwarder protocols. `Interest` packets can optionally contain a `KeyId`, which identifies the type of publishing node that is permitted to provide the desired content (publishing/Forwarder nodes compute their own KeyID based on the Content being desired and one of a series of validation algorithms). In addition, `Interest` packets can contain `ContentObjectHash` fields, which are a SHA-256 checksum of the entire on-the-wire `Content` packet which the `Interest` is expecting to receive. If both the `KeyId` and `ContentObjectHash` fields in the `Interest` match their counterparts on the Forwarder or Content Store node for the Content Object with the matching name, the `Content` packet is sent. Otherwise the `Interest` is forwarded. 

It is important to note that cryptographic verification is the responsibility of end user applications, not of any CCNx node. This allows both for the ability of users to define and verify their own cryptographic assurance strategies and for the network to be agnostic to the content of data flowing within it. As long as a piece of content has a name, encrypted or not, it can be delivered.

#### Naming
Critical to any Internet-scale content delivery system are the names used to refer to the content within it. CCNx names are often written in documentation using [URI syntax][rfc_uris] but are in fact opaque, binary data, and are usually hierarchical. Here is a representative figure from the [Jacobson et al. paper][ccnx_paper].

![CCNx Naming](http://f.cl.ly/items/1p2i2P1F010a0I3L3u2n/Image%202015-04-30%20at%209.43.34%20PM.png)<div class="caption">Figure 1. CCNx Name Scheme Example</div>

As seen in the above figure, names are comprised of multiple components. The names are binary, which means that encodings of any kind of data are permitted in the name, including complex binary structures or even encrypted data. This is enabled by the Type-Length-Value [packet format][ccnx_arch:packet], as seen in the binary encoding section of Figure 1. The example given in the figure presents a suggested naming scheme for temporally-segmented data (here, a video file). This hierarchical segmentation structure is enabled by the CCNx naming format, and thus allows for the construction of systems which exploit an ability to explore the CCNx network for available data. 

#### Security and Trust
The foundational principle of security in the CCNx architecture is the **authenticated** binding of names to content. The CCNx protocol maintains the invariant that a name will always refer to the content to which the name was bound when the content entered the network at the Content Publisher. This represents a core design decision in the construction of CCNx: trust is between *publishers* and *content publishers*. The CCNx authenticated name binding facilitates the construction of systems which rely upon that signed binding to ensure validity and authenticity of content. 


## Adoption
<!--references-->
[ip_exhaustion]: http://en.wikipedia.org/wiki/IPv4_address_exhaustion "IPv4 Address Exhaustion"

The current TCP/IP-based network architecture that we use to access the Web's information stands apart from this new CCNx protocol in several important ways. The purpose of this section is to explore whether these differences will enhance or hinder the process of CCNx adoption in the future. We structure this section around a series of principles and core functionality upon which both the TCP/IP and CCNx systems are built, but potentially differ in. 

### Naming
The first point of analysis is with regard to the naming/addressing schemes used by TCP/IP and the new CCNx protocol. For some background: internet-scale TCP/IP relies upon the Domain Name System (DNS) to locate the IP addresses of the machines specified by root domain names. For example, a DNS lookup on the domain name `www.google.com` resolves to the IP address `4.53.56.118` (among many others). Requests for the search results for the string "DNS," accessible with the URI `https://www.google.com/search?q=DNS`, are acquired via a domain lookup and the following URI: `https://4.53.56.118/search?q=DNS`. This highlights a core design difference of the TCP/IP system when compared to CCNx: names refer to *content at locations* rather than just *content*. DNS is an extraneous layer placed upon TCP/IP to allow for the human cognition of *machine locations* (or addresses), which (according to Jacobson) is going out of style. 

As an internet-using society, we care more about access to content rather than access to locations. Therefore, the CCNx paradigm of binding names to individual pieces of content seems to be more in line with today's Internet. In addition, there are several significant downsides to the [URI system][rfc_uris] upon which our current web is based. Though its hierarchical nature is convenient for exploring the web (i.e. it is possible to "guess" with certain accuracy the names of content given a hierarchical content structure), it is severely limited as a text-based system. In order to name every piece of content, CCNx allows for *any binary data* to be a name. This means that a CCNx can serve as a sort of distributed key-value store around the internet, where applications can use the network for any purpose they would like (not just content hosting). This is an example of the tradeoffs seen in the COMP150-IDS [principle][IDS:principles] "Text vs. Binary Formats". Specifically, a CCNx name is more compact and expressive than its URI counterpart, but is considerably more difficult for a human to remember and use, and encoding becomes an issue.

The downside of binary, non-hierarchical naming is that CCNx applications bear the burden of creating a logical naming hierarchy for the data that they use/produce. This is in contrast to URIs, which have a built-in enforcement of a data hierarchy. Because so much data is hierarchical or has relationships to other kinds of data, it is convenient to have the protocol have direct support for this structure. However, the CCNx [specification][ccnx] strongly advocates for the construction of standards to be used in conjunction with the open binary naming system to impose intelligent structure on the data. 

Despite these differences, both the TCP/IP-DNS architecture and the CCNx architecture respect the COMP150-IDS [principle][IDS:principles] of "Name Everything." Namely, all objects in both systems receive a name. However, the CCNx architecture allows for a name which is independent of the location of the content. This provides considerable advantages when comparing the network architecture of today's system to CCNx, a topic we explore next. 


### Network Architecture
Like TCP/IP, the CCNx architecture is built upon the COMP150-IDS [principle][IDS:principles] "Keep It Simple Stupid." One could successfully argue that CCNx is even simpler than TCP/IP. Since CCNx has only 2 types of packets (no acknowledgements) and can take advantage of existing solutions (originally created for multicast/broadcast systems) which suppress repeat `Interest` or `Content` messages in the network, the individual nodes are precisely as complicated as the logic necessary to handle those two packet types. 

Frankly, the decision to use DNS to map domain names to IP addresses in our current architecture stands in violation of one of the Modularity/layering [principle][IDS:principles]: the use of DNS to map application-level requests (URIs referring to machines and the content that resides on them) to transport-level addresses (IP addresses) disrupts the abstraction between the layers. The CCNx system, as it is currently specified, clearly differentiates itself as a layer above the transport layer, which means that any underlying transport implementation may be used. 

Another advantage over the current TCP/IP structure is the scalability built into the CCNx network. Given that the network maintains the invariant that each name is bound uniquely to a given piece of content, and that the network can find any piece of content by name regardless of its location, the network can position the content at any internal node. Because of this, the network's performance can scale to a very large number of connected devices due to the network's inherent maintenance of topological closeness between requester and content. One of our [principles][IDS:principles], "Simplicity supports Scalability," is indeed applicable in this situation. The simplicity of the CCNx architecture and protocol enables this scalability. 

An issue that is frequently raised about the TCP/IP system is [IPv4 address exhaustion][ip_exhaustion]. The IPv4 specification can only expose 2^32 IP addresses, a significant portion of which are reserved for private use and IPv6 transition. Several of the regional internet registries for the world's regions have exhausted the space of IPv4 addresses assigned to them, which has created significant headaches for network managers and has led to the development of IPv6 addresses. This is an issue that stems from TCP/IP's architecture, which maintains that each connected device must have an internet protocol (IP) address assigned to it. The CCNx network does not have this limitation. Since only content is assigned a name, the space of names is considerably less limited than the IP space. However, since the CCNx naming scheme relies at some level upon SHA-256 checksums, the naming scheme is somewhat dependent on the range of the SHA algorithms to produce unique results on the content they are hashing. 

With that said, the CCNx architecture still depends upon an existing Layer 2 networking technology, like IPv6 or Ethernet, to move Interest and Content packets around. This is a similarity to the TCP-based protocol we currently use, which means to some extent we will still experience namespace issues. However, these namespace issues are entirely internal to the implementation of CCNx, rather than exposed to the application layer as they are currently. Specifically, the IP protocol (for example) would be used to identify Forwarder and Content Store nodes, rather than content. This should allow for more intelligent distribution of the available naming space for nodes and still does not relate to the content being requested whatsoever. 

### HTTP
[ccnx:apps:http_prox]: https://github.com/ProjectCCNx/ccnx/tree/master/apps/HttpProxy "Project CCNx HTTP Proxy"
[quic]: https://www.chromium.org/quic "The QUIC Protocol"

A significant downside to the adoption of the CCNx protocol is its incompatibility with the current HTTP protocol, but an easy fix lurks around the corner. CCNx intends to replace the HTTP protocol, as the HTTP protocol relies upon URIs to identify the content they request. The CCNx project hopes to replace the URIs used by HTTP with CCNx names. However, for the time being, the CCNx project has implemented a [proxy][ccnx:apps:http_prox] for HTTP requests which leverages a content-centric network to satisfy HTTP requests. It converts HTTP requests into CCNx Interest packets, and relies on a special Forwarder node to convert those HTTP interests into CCNx Content packets by making a HTTP request to the location specified in the Interest packet. If the CC network hasn't seen the specific Interest before, the Content packet published by the HTTP-request node is stored throughout the CCNx network for future retrieval with appropriate timeouts. 

However, the need for this proxy points to a significant departure from the current state of the Internet that would need to occur for the full adoption of the CCNx architecture. HTTP will have to change entirely. `GET` requests, for example, map to CCNx interests. Making content available via HTTP is equivalent to publishing the content via a Content Publisher node (binding a name to the content with SHA). This implies that all current clients of the Internet---browsers, JavaScript applications, native applications, hardware---will have to be updated (or have their HTTP requests proxied via CCNx) in order to work with the new system. This is an enormously large change to the architecture of the Internet, and would be a very difficult sell for those with a very large amount of already-implemented HTTP code. 

However, as we see with Google's [QUIC][quic] protocol, clients (Google Chrome in this case) are free to implement the protocols which they feel are important enough to their service to be worthy of implementation. This might be a potential adoption strategy for CCNx, with individual interested clients whose application could especially benefit from CCNx implementing the protocol first. Once there is a base of clients and nodes forming a large-enough network, a more global adoption might be more realistic. 

### Security
[end-to-end]: http://web.mit.edu/Saltzer/www/publications/endtoend/endtoend.pdf "End-To-End Arguments in System Design"

While we briefly touched upon this in the previous section, there is a significant advantage to CCNx with regard to the security that it provides, one that adheres very closely to the [end-to-end][] [principle][IDS:principles]. Namely, the CCNx provides only a single service: the retrieval of published content using authenticated names which are guaranteed to retrieve the content bound to that name upon its publishing. This means that the *users* of the network (the endpoints, that is), not the network itself, control data encryption and access. Since CCNx is entirely agnostic to the data being transferred, content itself can be encrypted without affecting any of the operations of the network. This stands in significant contrast to (HTTPS) security as it stands now, which requires encrypted connections to send ordinary HTTP data between machines. There is considerable overhead to negotiate and maintain SSL/TLS connections and to encrypt all traffic between them, much of which does not need to be encrypted. An advantage of the CCNx model is that it allows for the encryption of only what is necessary, and the model itself is entirely agnostic to this security. Only the endpoints are responsible for the security of their data, which makes the task of ensuring correctness much more tractable. 

## Conclusions
The purpose of this work was to consider content-centric networking as a contender for the future of our current Internet. Faced with both the technological shortcomings of the current TCP/IP model and the changing ways in which we use the Web, it appears rather clear that content-centric networking, in some form or another is a very serious contender for the future of networking. 

Given the relatively short nature of this paper, it is impossible to analyze every facet of the adoption feasibility of content-centric networking. However, CCNx stands out over the current TCP/IP structure in the domains of naming, network architecture, data security, and more. Though we did not have time to consider the other advantages that CCNx provides, much scholarship exists which highlights CCNx as a superior alternative to TCP/IP, especially with regard to the dissemination and broadcast of data to large numbers of topologically-distributed consumers. 

To wrap up this discussion, it appears that CCNx is a very viable alternative to the current TCP/IP structure. Despite the fact that there's a significant overhead to this adoption in terms of the code that must be changed to accommodate content-centric networking, I believe that we will begin to see more traction for this technology in the near future. 

## Bibliography
Berners-Lee, T., Fielding, R., and Masinter, L. ["Uniform Resource Identifier (URI): Generic Syntax"](https://tools.ietf.org/html/rfc3986). Network Working Group. January 2005. Accessed May 1, 2015.

The CCNx Project. ["CCNx Architecture Overview"](https://www.ccnx.org/architecture-overview/). Palo Alto Research Center, Inc. Accessed April 29, 2015. 

The CCNx Project. ["Naming".](https://www.ccnx.org/names/) Palo Alto Research Center, Inc. Accessed April 29, 2015.

The CCNx Project. ["Interest"](https://www.ccnx.org/interest/). Palo Alto Research Center, Inc. Accessed April 29, 2015.

The CCNx Project. ["Packet Format"](https://www.ccnx.org/packet-format/). Palo Alto Research Center, Inc. Accessed April 29, 2015.

The CCNx Project. ["Content Object"](https://www.ccnx.org/content-object/). Palo Alto Research Center, Inc. Accessed April 29, 2015.

The CCNx Project. ["HTTPProxy"](https://github.com/ProjectCCNx/ccnx/tree/master/apps/HttpProxy). Palo Alto Research Center, Inc. Accessed May 1, 2015.

The CCNx Project. ["Content-Store"]( https://www.ccnx.org/content-store/ ). Palo Alto Research Center, Inc. Accessed May 1, 2015.

The CCNx Project. ["Forwarders"](https://www.ccnx.org/forwarders/). Palo Alto Research Center, Inc. Accessed May 1, 2015.
 
The Chromium Project. ["QUIC, a multiplexed stream transport over UDP."](https://www.chromium.org/quic) Accessed April 30, 2015.

Jacobson, V., Smetters, D.K., Thornton, D.J., Plass, M.F., Briggs, N.H., and Braynard, R.L., ["Networking named content"](http://conferences.sigcomm.org/co-next/2009/papers/Jacobson.pdf). *Proceedings of the 5th international conference on Emerging networking experiments and technologies*. ACM, 2009. Accessed April 28, 2015.

Jacobson, V. ["A New Way To Look at Networking"](https://www.youtube.com/watch?v=gqGEMQveoqg). Google Tech Talks. Oct 8, 2007. Accessed April 27, 2015.

Wikipedia contributors. ["Bode Miller"](http://en.wikipedia.org/wiki/Bode_Miller#2006_Olympics_controversy). Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 7 Feb. 2015. Accessed May 1, 2015.

Wikipedia contributors. ["Content centric networking"](http://en.wikipedia.org/wiki/Content_centric_networking). Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, Apr 8, 2015. Accessed May 1, 2015.

Wikipedia contributors. ["IPv4 address exhaustion."](http://en.wikipedia.org/wiki/IPv4_address_exhaustion) Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 27 Apr. 2015. Accessed 30 Apr. 2015.

Mendelsohn, N. ["Cross-Cutting Issues: Principles, Guidelines, and Bits of Wisdom"](http://www.cs.tufts.edu/comp/150IDS/principles). Spring 2015. Accessed May 1, 2015.

Mill, E. ["Certificate Authorities are Actually a Tremendous Problem"](https://konklone.com/post/certificate-authorities-are-actually-a-tremendous-problem). June 21, 2013. Accessed April 29, 2015.

Mosko, M. and Solis, I. ["CCNx Semantics (draft-mosko-icnrg-ccnxsemantics-01)"](http://tools.ietf.org/html/draft-mosko-icnrg-ccnxsemantics-01). ICNRG. March 9, 2015. Accessed May 1, 2015.

Saltzer, J.H., Reed, D.P., and Clark, D.D. ["End-to-end Arguments in System Design."](http://web.mit.edu/Saltzer/www/publications/endtoend/) ACM TOCS, 4 Nov. 1984. Accessed April 30, 2015. 



[^1]: CCNx® is a registered trademark of Palo Alto Research Center, Inc.


	










