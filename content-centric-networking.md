<!--Styles-->
<style>
	img {
		width: 100%
	}
	.caption{
		margin-left: auto;
		margin-right: auto;
		text-align: center;
		font-style: italic;
	};
</style>

[top]: #top "go to top"
<!-- Global References -->
[ccnx]: http://www.ccnx.org "CCNx Website" 
[ccnx_paper]: http://conferences.sigcomm.org/co-next/2009/papers/Jacobson.pdf "CCNx Paper"
[ccnx_wiki]: http://en.wikipedia.org/wiki/Content_centric_networking "CCNx Wikipedia Article"
[van_google]: https://www.youtube.com/watch?v=gqGEMQveoqg "Van Jacobson Google Talk on CCNx"
[CA_issues]: https://konklone.com/post/certificate-authorities-are-actually-a-tremendous-problem "Certificate Authorities are actually a Tremendous Problem"

<!-- end: GLobal References-->
<!-- Contextual References-->

<!-- end: Contextual References-->

# The Future of The Network 
<small>By Tony Cannistra</small>

<!--Table of Contents-->
[TOC] 

<a name="top"></a>

The way we interact with the world's information is changing, and our current TCP/IP-based network that supports the Web is faced with real limitations as a result. It is the goal of this paper to explore these limitations through the lens of a newly-proposed technology: content-centric networking. We first present the specifications of content-centric networking and how they compare to the current *status-quo*. We'll then explore the feasibility of the global adoption of content-centric networking (henceforth referred to as CCNx) into today's information-distribution platforms. 

## CCNx: A Primer

Content-centric networking was first described in [a paper][ccnx_paper] co-authored by developer Van Jacobson, together with colleagues at Xerox PARC, in response to the shortcomings they perceived in the current TCP/IP-based architecture of the Web. [Project CCNx][ccnx] was launched in  to maintain the CCNx specification and open-source implementations. The purpose of this section is to describe the CCNx protocol and its motivations. We begin with the latter. 

### Motivation
The TCP/IP-based architecture of the Internet today was built with the intent to connect *devices* to *other devices* in a way that satisfies several key invariants, especially: network redundancy, ease of connection, and global network interoperability. TCP/IP was built with a focus on *where machines are* and *how to talk to them*, rather than *what their content is*. To quote [Jacobson][ccnx_paper]: Today, "People value the Internet for *what* content it contains, but communication is still in terms of *where*." 

This ideological limitation is the root of a larger set of practical concerns. Jacobson et al. maintain that the three main issues that arise from the TCP/IP architecture of the web are **availability**, **security**, and **location-dependence**. For content to be highly-available (i.e. accessible quickly and reliably for a diverse set of users across the networks' topology), current TCP/IP-based networks require content-delivery-networks (CDNs) or Peer-to-Peer networks to be overlaid on top of existing infrastructure. In addition, only those who can afford access to high-bandwidth connections or CDN services like Akamai can provide highly-available content. 

In addition, the current web architecture allows for *pipe security* rather than the more-important *data security*. Jacobson, in [a Google Tech Talk][van_google] delivered in 2007, highlights the fact that the way we've built security on top of the TCP/IP protocol encrypts entire connections, rather than just the data transferred across those connections. Therefore, services like caching and availability are affected. In addition to this, the current verification scheme relies upon the certificate-authority structure, which has been [shown][CA_issues] to have significant trust issues. 

Finally, the TCP/IP system as optimized for locating a *machine* rather than locating *content* means that those content consumers located topologically closer to the machine that contains the desired content experience a much higher Quality of Service (QoS) than those further away. Packets must be sent through a much larger number of intervening nodes to deliver content from one connected subgraph of the Internet to another subgraph with few connections to the source network. 

It was the goal of Jacobson et al., and later [Project CCNx][ccnx], to address these issues with a content-centric networking paradigm. We explore the specifications of the CCNx protocol next. 

### The CCNx Specification






	










