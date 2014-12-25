# Zennet RFC

Ohad Asor

Rev.0: 28 Feb 2014

Rev.1: 30 Apr 2014

Rev.2: 9 May 2014

1 Jul 2014: Note, SDD document available here https://docs.google.com/document/d/1Y-JKJtJrnGpEISgpEpGKqwmW-atg0_Uk2rJAH5YpaLc/edit?usp=sharing

**Please refer to http://zennet.sc for more updated information.**

We present Zennet, a Decentralized Application offering an automatic virtualization free-market. Not to be confused with XEN, a successful virtualization software. We also present ZenFS, a distributed high-performance filesystem implemented over Zennet, and ZenTube, a video streaming and distribution system built over ZenFS.


**Table of Contents**

- [Zennet RFC](#user-content-Zennet-rfc)
	- [Preface](#user-content-preface)
	- [Architecture](#user-content-architecture)
		- [No POW](#user-content-no-pow)
		- [Main Flow](#user-content-main-flow)
		- [Zennet Blockchain](#user-content-Zennet-blockchain)
		- [Task Details](#user-content-task-details)
		- [Identities](#user-content-identities)
		- [Transaction Proof-of-Work](#user-content-transaction-proof-of-work)
	- [ZenFS](#user-content-ZenFS)
		- [Requirements](#user-content-requirements)
		- [Architecture](#user-content-architecture-1)
			- [Hashed Elements](#user-content-hashed-elements)
			- [Circuits](#user-content-circuits)
			- [Discovery](#user-content-discovery)
		- [Uploading a File](#user-content-uploading-a-file)
			- [Proof of Storage](#user-content-proof-of-storage)
		- [Additional Features](#user-content-additional-features)
	- [ZenTube](#user-content-ZenTube)
		- [Resource Requirements](#user-content-resource-requirements)
	- [FAQ](#user-content-faq)
	- [Appendix](#user-content-appendix)
		- [Zennet Robustness and Extra Features](#user-content-Zennet-robustness-and-extra-features)
		- [Zennet Technical Notes](#user-content-Zennet-technical-notes)
		- [Why QEMU/XEN?](#user-content-why-qemuxen)
			- [TBD](#user-content-tbd)
		- [Decentralized Organizations over Zennet](#user-content-decentralized-organizations-over-Zennet)

## Preface

Generally speaking, Zennet is a Distributed Application (DA) letting **publishers** to run virtual machines on the **nodes'** hardware, and pay those nodes. A Publisher is any entity willing to hire computational resources. Nodes together form a giant cloud. The product will consist of a single Client software. All open source, trust-free and decentralized.

The Zennet Foundation will implement and support the Zennet client, together with bringing into the market new products relying on Zennet, such as deecntralized video portal, decentralized search engine, short- and long-term supercomputing for companies and researchers, and many others. Our vision is that anyone on earth who owns a computing device (from a smartphone to a server farm) to be able to sell their unused computing resources. One cannot exaggerate regarding the impact of such a network on global energy consumption, global fortune distribution, accessibility of single thinkers and developers to rare computing powers, the amount of new possibilities of decentralized solution based on a virualization network, and much more.

The client will be splitted to Zennetd and Zennet-qt, like in Bitcoin. The design consists of a blockchain, again similarly to Bitcoin.

The underlying cryptocurrency of the Zennet and its applications ecosystem is called Zencoin. It will be very much Bitcoin like, except for a few additional features. The Zennet foundation is intending to fund the development via a premine of Zencoins.

We would like to thank Mr. Alon Peleg for his significant contribution to this project, and the Israeli Facebook Bitcoin community for their Q&A about Zennet.

## Architecture

### No POW

One would naturally require computing jobs to be provable, in a proof-of-work manner, but this is impossible when we deal with generic computing tasks. In Zennet, different mechanisms are used to prevent abuse, namely proof-of-identity, short payment intervals, performance measurments and more as described below.

In future, we plan to create a subnet of Zennet which will allow provable computations, such as Singular Value Decomposition, NP Complete problem solving and so on.

### Main Flow 

The main flow can be summarized as:

1.  Publisher and Node both mine their identity (see Identity Mining below).

2.  Publisher broadcasts a task announcement.

3.  Node considers the offer. If accepted, connects via SSH to the IP specified within the task announcement.

4.  The SSH connection creates a tunnel which gives the publisher another pipe in which they gain control over the virtual machine.

5.  The SSH pipe will be used from now on to the Rapidly-adjusted Micropayment Algorithm, as described here: <https://en.bitcoin.it/wiki/Contracts#Example_7:_Rapidly-adjusted_.28micro.29payments_to_a_pre-determined_party>

6.  Now the Publisher can use the VM for their needs while continuously paying the node.

### Zennet Blockchain

On Zennet implementation, unlike Bitcoin's one, we allow transactions without any output, and we call them *Announcements*. Announcements will have a pruning timeout field, allows them to be pruned from the blockchain after a given number of blocks.

The Zennet blockchain will be used both to Zencoin ledger, just like Bitcoin, and for broadcasting announcements which triggers the operation of the Zennet network.

We consider using a Myriadcoin-like mining algorithm, which allows mining with various hashing functions, each with a separate difficulty parameter, making it profitable for all kinds of mining hardware such as CPU, GPU or ASIC. To avoid misunderstanding, mining is for the sake of approving transactions and maintaining the blockchain. It has nothing to do with the computing resources Zennet nodes offering to publishers.

### Task Details

The **task announcement** data includes:

1.  Task ID.

2.  Node's Virtual Machine (VM) requirements (all optional):

    1.  Operating System (OS), typically thin distributions such as DSL (Damn Small linux), OpenWRT etc., to save resources. Multiple choices are possible (letting the node choose from a list).

    2.  Minimal network bandwidth.

    3.  Minimal storage (disk space) and performance.

    4.  Size of persistant storage. This will be saved locally by the node and mounted to the VM. Publisher can announce that they deactivate all task's or specific node's (identified by address) persistant storage, and cease from paying for it.

    5.  Number of cores with how much clock frequency for each (multiple choice). Similar to AWS' Computing Units.

    6.  Memory per core and memory speed.

3.  Minimum number of described virtual machines needed network-wide.

4.  An IP address. Nodes will connect to this address to process the tasks.

5.  Preferred lease time (range).

6.  Payout interval. The publisher announces they will pay every X seconds. This points to a tradeoff between publisher's trust, node's trust and transaction fees. This value can be negative, meaning the publisher will pre-pay the nodes.

7.  Rate per part. Publisher will set payment for each MB of memory, GHz of computing and so on. The publisher can offer any price.

8.  Nonce. See Transaction Proof-of-Work below.

9.  Time for proposal to be accepted.

10. Additional information. Typically a short url where the publisher can tell more if they want to.

### Identities

Identity mining is used to prevent flooding by dummy publishers or nodes. For the network to respect a published task or an accepted task, the relevant transactions should contain an ID of the publisher or node. This cannot be any ID, but has to have a certain structure that proves that the creation of this ID consumed significant computing resources. Namely, the hash of the ID should be lower than a Target, computed from the Difficulty, exactly like in Bitcoin. The Difficulty mechanism is needed in order to take into account Moore's law and so on.

Such identity mining is implemented on other projects like [Keyhotee](https://github.com/InvictusInnovations/keyhotee) and [Bitmessage](https://github.com/Bitmessage/PyBitmessage).

### Transaction Proof-of-Work

Another mechanism to avoid flooding is requiring every task transaction and acceptance transaction to include some POW, by targeted-hashing as described in Identity Mining subsection above. The transaction will contain a nonce (like it Bitcoin's mining algorithm) to be modified until the transaction's hash is low enough. In contrary to Identity Mining, both the publisher and the node can select what is their hash target, hence increase or decrease the amount of work they require in order to accept the other party's task or acceptance, therefore another mean of controlling the trust between the parties.

As can be extracted from the above, the Zennet network has four different difficulty parameters: publisher and node identity mining difficulty, plus task and acceptance transactions difficulty.

## ZenFS

ZenFS is a distributed filesystem to be used across developments of Zennet foundation. Specifically, it will serve ZenTube. The system will be compatible with torrent downloaders, and support uploading and spreading files between nodes.

Nodes storing the data are getting paid only for uploading, downloading and proving storage. Uploading will charge only the cost of bandwidth. On every download, the node will charge a payment encapsulating both the download bandwidth and the cost of persistant storage. Hence, nodes are motivated to keep the most popular files. It is possible to pay Zencoins and keep arbitrary data persistant even without downloading, as we shall see below regarding periodic proof of storage.

### Requirements

1. **Distributed**: All data should be distributed over the network and maintained by a single client.

2. **Trustless**: The network should be protected against cheating.

3. **Fair**: Each participant will pay for the services they consume, or get paid by giving services, and will have an incentive to participate.

4. **Storage**: Allow storing files over the network.

5. **Persistency**: Disappearance or corruption of files should have a very low probability.

6. **Fast**: File retrieval should have low latency.

7. **Streaming**: The network should be strong enough to support HD video streaming.

8. **Standardized**: The network should support common file retrieval protocols, such as BitTorrent, and mounting folders locally.

### Architecture

The ZenFSd executable will be deployed with Zennetd and it will be its only window to the outside world.

ZenFS will work with nodes running slim POSIX VMs, typically FreeBSD. For the simplicity of implementation, debugging, monitoring and future development, we chose to implement ZenFS with plain bash commands.

#### Hashed Elements

Elements are either parts or metainfo files. They are stored on nodes with their filenames being their SHA1 hash, which is Bitorrent compatible.

**Parts** Each node will store parts of files. Each part may be of different size, typically 256KB, and will be stored as a single file. The filename will be the SHA1 hash of the part's content, as in the bittorrent standard.

**Metainfo** The parts with their hashes are meaningless unless one has the information as in the torrent file, namely, the original filenames, attributes and directory structure of the files encapsulated on the torrent file, the number of parts and their hashes. Hence each part should be mapped to a torrent file. At the scope of ZenFS, we call this file a *metainfo* file, which is just like a torrent file, but without a tracker URL. The metainfo files will be again saved with their hash as their filenames.

#### Circuits

When Zennetd is launched, it discovers peers for the sake of downloading the blockchain and broadcasting transactions, just like Bitcoin's bootstrap. We call those peers the *Blockchain Peers*. When ZenFSd is launched, it should initiate a Zennet connection with other nodes participating in the ZenFS network, since it is planning to hire services from them (storage, download etc.). Such peers will be called the *Working Peers*. The set of all working peers of a specific node form the node's *circuit*.


To bootstrap ZenFS with working peers, the client broadcasts a standard Zennet task announcement, with default ZenFS or user-custom properties. The Zennet nodes accepting this task are the initial working peers of ZenFS. 


When a large file is downloaded, or with applications such as video streaming, the ZenFS client will initiate a connection with working peers holding the desired file parts. Until this finishes, the existing working peers transfer the parts by recursively requesting them from their own working peers.

#### Discovery

Discovery of nodes holding the desired parts is done simply by notifiying all connected peers that such part is inquired. The request contains the hash of the desired part, and the IP address of the requiring node (one may consider a flavour of ZenFS over OpenVPN). Once the peers received the request, they recursively pass it to their own peers. The stopping condition is when the request is arrived twice within a reasonable amount of time (say 1min). The part will be downloaded from the nodes that responded first, since it makes sense to assume they are the most available ones, in terms of load and latency.

Searching by certain metainfo fields will be possible by the again recursive broadcasting. Each node will search its metafiles for the desired information, and if found, it may send the whole metainfo file to the requester, or just the hash of it (to the choice of the requester).

### Uploading a File

The ZenFS client will allow users to upload files or directories. Uploading will require creating a metainfo file, in a same way of creating a torrent file, and it may contain user-defined fields and tags, to be later searchable. 

Uploading initiates a job off-block (in front of the existing circuit) for uploading them the parts of the file. It is the user's responsibility, and will be implemented on the client, to spread the parts between various nodes with several copies. The uploader will of course have to pay the working peers for this work, so more copies will cost more.

Working peers charge also for downloading. Assume they are all selfish. Then nodes will tend to store only the most downloadable parts, hence more profitable. 

If one has a file to store which is not popular (or even private), and still wants it to be stored, theoretically they can download it from time to time, so the hosts will not mark the part as non-profitable. We call this process *Periodic Proof of Storage* (PPS).

#### Proof of Storage

Assume host A holds a copy of 1MB of information which B uploaded and wants to be kept stored for long. So every time interval B needs a proof that the A actually holds the file, and pay the A with every successful check. We have to come up with a random test such that A will have to hold at least 1MB in order to pass the test with high probability, and B will have to store significantly less than 1MB in order to perform the test.

The proposed solution is as follows. Before uploading, B hashes the file with a random salt which are two bytes appended to the file, keeping its original size plus two but its content is garbled. It selects 32 random bits from the hashed file, and stores them. B repeats this process for 32 different random salts. This will result with 1024 bits and 32 salts, the bit location consists of 20 bits (1M=2^20), totalling in (1024+20)/8B+2x64+20/8=259B. We call this information the *private checksum*.

On every interval of time, B sends A a random pair of bit number and salt. We call this a *checkpoint*. A does not know the private checksum of B, therefore has to keep the whole file in order to give a correct answer. It can guess the answer and be right with probability 1/2. Assume we would like a probability of 1 over 4096 that A will not cheat. So at the 1024-log_{2}4096=1024-12=1012 B will download the whole file, and create a new private checksum.

On every checkpoint, B will have to give A an updated transaction (like in the micropayment protocol) with the storage fee up until now.

The initiation of the PPS process is as follows. Timeout-prunable task announcement is initiated by B announcing the hash of the metainfo file to be verified. Nodes storing such parts will connect to B and begin the PPS process.

### Additional Features

The ZenFS client will allow storing and retrieving files or folders encrypted with the user's private key. This allows pivate data to be stored securely.

Bitorrent interfacce implementation is straight-forward. The metainfo file is appended with a tracker field, which points to localhost to a port ZenFSd listens on, seemlessly serves any standard Bitorrent client.

## ZenTube

ZenTube is a video portal implemented over ZenFS and Zennet, with parameterization, configuration and tools allowing a distributed video streaming and encoding service, without the average user having to deal with Zencoins or even know about them.

A dedicated ZenTube client will be developed. This client will include Zennetd and ZenFSd, but this part of the application will not be visible to the average user. Of course, the user will have the option to interact with Zennetd directly or using Zennet-qt, as well as for ZenFSd.

The video uploading task is the same as in ZenFS, with an extra operation. Each video will be encoded into various common resolutions, to allow low-bandwidth streaming or lowering the resolution for i.e. handheld devides. The ZenTube client will publish such a task of uploading and encoding with appropriate parameters, to be discussed later.

The actual player to be used for watching videos can be any standard Bitorrent player such as Popcorn TV or XMBC.

The ZenTube client will be accessible via a web browser pointed to localhost. ZenTube will implement a web application allowing users to browse, search, tag and upload content, as well as downloading or streaming with torrent magnet links.

### Resource Requirements

We cannot give a good estimation of needed circuit size until we make experiments on the network's implementation. But we can take several numbers as a rule of thumb as from the table [here](https://support.google.com/youtube/answer/2853702?hl=en).

Assume we need 16GHz and 32GB RAM in order to encode video in realtime with resolution 480p, as used in the article at http://ijarcet.org/wp-content/uploads/IJARCET-VOL-3-ISSUE-2-408-413.pdf. Average user upload rate is around 100Kbps. Encoded videos will be kept for several days since they last have been watched, in order to avoid duplicated work.

The above numbers suggests that the serving nodes should be somewhere between 1-100 per video. When impossible, lower resolutions will apply. Note that the resolution will be selected according to the playing device, i.e. when mobile phone is detected, resolution is lowered. At the worst case, buffering will take place.

Encoding and streaming uses both CPU, RAM, Hard drive and Bandwidth, all four in at least moderate usage.

We recognize that this constellation of resourse consumption might not justify itself, and we account on that our peers will be part of the bigger Zennet and ZenFS network, hence increasing the incentive to run a node.

As we described, ZenTube is mainly a fine-tuned ZenFS with additional features. This fine-tuning of the parameters is yet to be well considered.

## FAQ

**Q**: How can I make sure that the VM I hired is always online, like AWS assure to me?

**A**: Zennet is not intended for such cases. Zennet is intended for distributed computing, requiring mass of machines, each performing smaller tasks. Of course, it is the node's incentive to be online, so it will get paid more, but the publishers should not assume that this is the case. Examples of distributed computing that does not require always-online nodes is Bitcoin itself, matrix inversion, NP complete problem solving, Singular Value Decomposition, Video encoding, Web crawling and so on. Those are with huge interest in the world of Big Data. See Apache Mahout for example.

**Q**: Isn't it like MaidSafe?

**A**: No. Maidsafe does not allow distributed computing. And even if so, it will be via a JVM, which is inherently dozens or hundreds inefficient than native VM.

## Appendix

### Zennet Robustness and Extra Features

1.  In case of a disconnection, it is the node's responsibility to reconnect and rebuild the tunnel. The Zennet client will handle the connection monitoring and restoration. Conversly, the publisher can run a reconnection daemon on the virtual machine. In case such as power failure, the Zennet client will try to re-run the VMs with persistant storage at the expence of the node's (unless the publisher ordered a persistent storage).

2.  The Zennet client will handle the VMs launching, closing and configuring. It is planned to support both QEMU (for common computers and devices) and XEN (for dedicated Zennet racks or any other reason).

3.  Zennet client will employ tools for VM resources guarantee, such as `mprotect()` to disable paging, `nice()` to adjust priority, CPU locking, disk space catching, network QOS and so on.

4.  The Zennet client will benchmark the VMs and try to wisely select the most profitable, secure and adequate tasks (all widely configurable by the user). Note that this might be a hard mathematical task, as in combinatorial auctions. Good enough approximations can be used, as well as another industry of pools calculating the best work for each worker. Such pools can also benchmark their members.

5.  The Zennet client will record usage of resources, to confirm the payouts. Sometimes a VM can be slower than when it was benchmarked, and this may cause a lower payout, so the Zennet client will (customably) verify that the payout changes are justified (up to some confidence interval).

6.  The Zennet client will tend to run several tasks from different publishers in parallel, in order to reduce variance.

7.  The Zennet client will allow to publish and cancel tasks and manually select tasks to perform.

8.  Default (thin) OS images will be supplied with the Zennet client. The Zennet client will also offer to automatically install and configure QEMU.

### Zennet Technical Notes

1.  The Zennet client will follow the payouts and act as requested by the user in case of no payout (or no payout yet). The publisher's responsibility is to implement such a system themselves, selecting which nodes to work with and for how much.

2.  For extra-safety, when using QEMU in a general-purpose machine, it can be ran on a separate user inside a `jail`.

3.  The task and acceptance announcements sizes should not exceed several dozens or a few hundreds of bytes, hence adequate to the Bitcoin's blockchain.

### Why QEMU/XEN?

QEMU and XEN are both very mature virtualization open source projects. They are packaged on most Linux distributions. QEMU has the advantage that it can be ran on almost any platform, since it supports plain user-space emulation. One can even run Windows over Android using QEMU.

The main difference between QEMU and XEN is that QEMU is like an ordinary virtualization software such as VirtualBox, VMWare Workstation etc. XEN is a system that is being loaded before and behind all operating systems on the platform, so even the “regular” operating system actually runs in a virtual machine. For general purpose computers, or phones, QEMU is the coice. But if one would like to build a Zennet-dedicated machines, they might prefer XEN. XEN also has some support for GPGPU.

#### TBD

* Maybe POS to avoid instant redeeming? Should converge to the risk. Can it be controlled the network?

### Decentralized Organizations over Zennet

We shall build the first applications and function as publishers on the Zennet network. All of them will be open-source. Such applications may include:

1.  CPU mining. Nodes will run various cryptocurrency miners and this will fund their payments.

2.  Decentralized P2P World Wide Web. All nodes will be connected via OpenVPN or a modified TOR, allowing web hosting and serving websites on custom domains. May integrate or borrow ideas from I2P and Namecoin.

3.  Decentralized P2P mail servers.

4.  Decentralized P2P DNS (better avoid DNS poisoning? Maybe adopt the new DNS standard candidates?)

5.  Decentralized P2P search engine. Nodes shall have vast scraping abilities and fast queries. We can employ tools such as by Apache foundation and implement a search engine of our own. It will monetize itself via advertisements, just like Google. But it will be preferred on Google in a way that governments and politics can't stop it (like in China), no one moderates the content, no one will track anyone's usage, statistics will be freely available (this has a huge impact: just imagine the worth of the information Google has in hands, means - what people look for), Deep web will be indexed, maybe history of websites will be kept as well.

6. ZenFS, ZenTube.

7. e-books repository.

Over the next sections we present two applications of Zennet built one of top another: ZenFS and ZenTube. ZenFS is a distributed file system which is torrent compatible. ZenTube relies on ZenFS and allows media encoding and streaming, playable with any torrent video player. Such an open source player might be packaged with ZenTube.
