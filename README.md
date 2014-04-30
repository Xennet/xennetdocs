Warning: This document is not final by any means, and is still subject to frequent changes.

Xennet RFC
==========

Ohad Asor

Rev.0: 28 Feb 2014

Rev.1: 30 Apr 2014

We present Xennet (pronounced “Zennet”), a Decentralized Application offering an automatic virtualization free-market. Not to be confused with XEN, a successful virtualization software.

Virtualization Network
======================

Generally speaking, Xennet is a Distributed Application (DA) letting **publishers** to run virtual machines on the **nodes’** hardware, and pay those nodes. A Publisher is any entity willing to hire computational resources. Nodes together form a giant cloud. The product will consist of a single Client software. All fully GNU GPL’d and decentralized.

The Xennet Foundation will implement and support the Xennet client, together with bringing into the market new products relying on Xennet, such as deecntralized video portal, decentralized search engine, short- and long-term supercomputing for companies and researchers, and many others. Our vision is that anyone on earth who owns a computing device (from a smartphone to a server farm) to be able to sell their unused computing resources. One cannot exaggerate regarding the impact of such a network on global energy consumption, global fortune distribution, accessibility of single thinkers and developers to rare computing powers, the amount of new possibilities of decentralized solution based on a virualization network, and much more.

The client will be splitted to xennetd and xennet-qt, just like Bitcoin. xennetd will function as a code library, allowing new applications to use Xennet's features from their code.

XenShares will be issues to the investors of the project. Fundraising over Mastercoin is considered. The Xennet Foundation will divide its revenues as dividends to the shareholders.

We would like to thank Mr. Alon Peleg for his significant contribution to this project, and the Israeli Facebook Bitcoin community for their Q&A about Xennet.

How it works
------------

One would naturally require the computing jobs to be provable, in a proof-of-work manner, but this is impossible when we deal with generic computing tasks. In Xennet, different mechanisms are used to prevent abuse, namely proof-of-identity, short payment intervals, implied&external reputation and more as described below.

We plan to create a subnet of Xennet which will allow provable computations, such as Singular Value Decomposition, NP Complete problem solving and so on.

### Main Flow 

The main flow can be summarized as:

1.  Publisher and Node both mine their identity (see Identity Mining below).

2.  Publisher broadcasts a task transaction.

3.  Node considers the offer. If accepted, connects via SSH to the IP specified within the task transaction.

4.  The SSH connection creates a tunnel which gives the publisher another pipe in which they gain control over the virtual machine.

5.  **Negotiation step** is taking place.

6.  The SSH pipe will be used from now on to the Rapidly-adjusted Micropayment Algorithm (RAMA), as described here: <https://en.bitcoin.it/wiki/Contracts#Example_7:_Rapidly-adjusted_.28micro.29payments_to_a_pre-determined_party>

7.  Now the Publisher can use the VM for their needs while continuously paying the node.

### Negotiation Step

After the tunneled SSH pipe is created, Publisher benchmarks the VM, calculates the rates (payment per 1GB RAM, 1GHz CPU, 1GB Bandwidth, 1GB Persistent/Temporary storage etc. ) it agrees to pay per hour (or a fraction of it), and sends the **proposal transaction **(signed with the node’s public keys) to the node, by broadcasting it (non-presistently using prunable dust transaction) over the Xennet network.

The node’s Xennet client will automatically (in a widely configurable and overridable way) consider the proposal, based on other proposals and its own benchmark. If the node discards the proposal, it just closes the SSH connection gracefully.

Since the performance of the VM is changing over time, payouts might be changed silently according to the publisher’s calculation. The Xennet client will monitor the system loads and verify that those changes are within some reasonable confidence interval.

### Task Details

The **task transaction** data includes:

1.  Task ID.

2.  Node’s Virtual Machine (VM) requirements (all optional):

    1.  Operating System (OS), typically thin distributions such as DSL (Damn Small linux), OpenWRT etc., to save resources. Multiple choices are possible (letting the node choose from a list).

    2.  Minimal network bandwidth.

    3.  Minimal storage (disk space) and performance.

    4.  Size of persistant storage. This will be saved locally by the node and mounted to the VM. Publisher can announce that they deactivate all task’s or specific node’s (identified by address) persistant storage, and cease from paying for it.

    5.  Number of cores with how much clock frequency for each (multiple choice). Similar to AWS’ Computing Units.

    6.  Memory per core and memory speed.

3.  Minimum number of described virtual machines needed network-wide.

4.  An IP address. Nodes will connect to this address to process the tasks.

5.  Preferred lease time (range).

6.  Payout interval. The publisher announces they will pay every X seconds. This points to a tradeoff between publisher’s trust, node’s trust and transaction fees. This value can be negative, meaning the publisher will pre-pay the nodes.

7.  Rate per part. Publisher will set payment for each MB of memory, GHz of computing and so on. The publisher can offer any price.

8.  Nonce. See Transaction Proof-of-Work below.

9.  Time for proposal to be accepted. See Negotiation step.

10. Additional information. Typically a short url where the publisher can tell more if they want to.

All data is embedded in a transaction in a way as in the Mastercoin protocol (dust amount transactions).

### Identities

Identity mining is used to prevent flooding by dummy publishers or nodes. For the network to respect a published task or an accepted task, the relevant transactions should contain an ID of the publisher or node. This cannot be any ID, but has to have a certain structure that proves that the creation of this ID consumed significant computing resources. Namely, the hash of the ID should be lower than a Target, computed from the Difficulty, exactly like in Bitcoin. The Difficulty mechanism is needed in order to take into account Moore’s law and so on.

Such identity mining is implemented on other projects like [Keyhotee](https://github.com/InvictusInnovations/keyhotee) and [Bitmessage](https://github.com/Bitmessage/PyBitmessage).

### Transaction Proof-of-Work

Another mechanism to avoid flooding is requiring every task transaction and acceptance transaction to include some POW, by targeted-hashing as described in Identity Mining subsection above. The transaction will contain a nonce (like it Bitcoin’s mining algorithm) to be modified until the transaction’s hash is low enough. In contrary to Identity Mining, both the publisher and the node can select what is their hash target, hence increase or decrease the amount of work they require in order to accept the other party’s task or acceptance, therefore another mean of controlling the trust between the parties.

As can be extracted from the above, the Xennet network has four different difficulty parameters: publisher and node identity mining difficulty, plus task and acceptance transactions difficulty.

Robustness and Extra Features
-----------------------------

1.  In case of a disconnection, it is the node’s responsibility to reconnect and rebuild the tunnel. The Xennet client will handle the connection monitoring and restoration. Conversly, the publisher can run a reconnection daemon on the virtual machine. In case such as power failure, the Xennet client will try to re-run the VMs with persistant storage at the expence of the node’s (unless the publisher ordered a persistent storage).

2.  The Xennet client will handle the VMs launching, closing and configuring. It is planned to support both QEMU (for common computers and devices) and XEN (for dedicated Xennet racks or any other reason).

3.  Xennet client will employ tools for VM resources guarantee, such as `mprotect()` to disable paging, `nice()` to adjust priority, CPU locking, disk space catching, network QOS and so on.

4.  The Xennet client will benchmark the VMs and try to wisely select the most profitable, secure and adequate tasks (all widely configurable by the user). Note that this might be a hard mathematical task, as in combinatorial auctions. Good enough approximations can be used, as well as another industry of pools calculating the best work for each worker. Such pools can also benchmark their members.

5.  The Xennet client will record usage of resources, to confirm the payouts. Sometimes a VM can be slower than when it was benchmarked, and this may cause a lower payout, so the Xennet client will (customably) verify that the payout changes are justified (up to some confidence interval).

6.  The Xennet client will tend to run several tasks from different publishers in parallel, in order to reduce variance.

7.  The Xennet client will allow to publish and cancel tasks and manually select tasks to perform.

8.  Default (thin) OS images will be supplied with the Xennet client. The Xennet client will also offer to automatically install and configure QEMU.

Technical Notes
---------------

1.  The Xennet client will follow the payouts and act as requested by the user in case of no payout (or no payout yet). The publisher’s responsibility is to implement such a system themselves, selecting which nodes to work with and for how much.

2.  For extra-safety, when using QEMU in a general-purpose machine, it can be ran on a separate user inside a `jail`.

3.  The task and acceptance transaction sizes should not exceed several dozens or a few hundreds of bytes, hence adequate to the Bitcoin’s blockchain.

4.  The main difference between QEMU and XEN is that QEMU is like an ordinary virtualization software such as VirtualBox, VMWare Workstation etc. XEN is a system that is being loaded before and behind all operating systems on the platform, so even the “regular” operating system actually runs in a virtual machine. For general purpose computers, or phones, QEMU is the coice. But if one would like to build a Xennet-dedicated machines, they might prefer XEN. XEN also has some support for GPGPU.

    Decentralized Organizations over Xennet
    =======================================

We shall build the first applications and function as publishers on the Xennet network. All of them will be open-source. Such applications include:

1.  CPU mining. Nodes will run various cryptocurrency miners and this will fund their payments.

2.  Peer-to-peer storage-only network, using Hadoop’s HDFS. Backup your data safely. Free up to some limit of size, paid afterwards. We might rent servers at our expense in order to help this storage network grow safely.

3.  Decentralized P2P World Wide Web. All nodes will be connected via OpenVPN or a modified TOR, allowing web hosting and serving websites on custom domains. May integrate or borrow ideas from I2P and Namecoin.

4.  Decentralized P2P mail servers.

5.  Decentralized P2P DNS (better avoid DNS poisoning? Maybe adopt the new DNS standard candidates?)

6.  Decentralized P2P search engine. Nodes shall have vast scraping abilities and fast queries. We can employ tools such as by Apache foundation and implement a search engine of our own. It will monetize itself via advertisements, just like Google. But it will be preferred on Google in a way that governments and politics can’t stop it (like in China), no one moderates the content, no one will track anyone’s usage, statistics will be freely available (this has a huge impact: just imagine the worth of the information Google has in hands, means - what people look for), Deep web will be indexed, maybe history of websites will be kept as well.

The first product deserves a separate section:

Xentube
-------

We shall implement Xentube, a P2P decentralized network for video storing, encoding and streaming, over Xennet.

Searching, browsing and streaming videos will be done using Kademlia algorithm. Uploading videos will use the Mitosis algorithm, in which hereby present. The Mitosis algorithm is used to spread the data files among as much clients as possible, and maintain several copies of each block. It acts as follows:

1.  Randomally exchange file parts with connected peers. Get parts from the peer, give parts to the peer, and delete the given parts. Note that the randomally selected parts can duplicate themselves.

2.  Exchanging is done continuously with a fixed bandwidth per connected peer (e.g. 1KB/sec).

3.  Each node remembers for a certain about of time (e.g. 1 day) which lets it route requests for the parts it recently held to the other node.

4.  If the local node has free space to use, it accepts more (e.g. twice) parts than it shares.

The client will support browsing and searching for videos using the Kademlia protocol. Streaming will begin with establishing a connection again using the Kademlia protocol, but the client will implement encoding rather done plane streaming.

This network will allow not only video services, but distributed storage and file sharing by all means, including private and public files. The distributed filesystem will be HDFS (Hadoop). Hadoop will also manage all VMs, their storange and processing (encoding), taking into account their availiability, capacity and so on. Hadoop is the leading product on this field (fully open source). A proof of concept of Hadoop based streaming and encoding can be found on the [HAVS](http://ijarcet.org/wp-content/uploads/IJARCET-VOL-3-ISSUE-2-408-413.pdf) article.

A dedicated Xentube client will be developed. This client will include xennetd, the Xennet's core, but this part of the application will not be visible to the average user. Of course, the user will have the option to interact with xennetd directly or using xennet-qt.

Note that Xentube is only a single task in Xennet. In order for Xentube to function even when it is not the most profitable task, Xentube users should be forced to contribute resources to Xentube's task. This can be done by introducing another token, XentubeCoin. Other solutions may be considered (TBD)

### Resource Requirements

We cannot give a good estimation of needed cloudlet size until we make experiments on the network’s implementation. But we can take several numbers as a rule of thumb. We took the following table from [here](https://support.google.com/youtube/answer/2853702?hl=en):

<span>|c|c|c|c|c|c|</span> & 240p & 360p & 480p & 720p & 1080pMaximum & 700 Kbps & 1000 Kbps & 2000 Kbps & 4000 Kbps & 6000 KbpsRecommended & 400 Kbps & 750 Kbps & 1000 Kbps & 2500 Kbps & 4500 KbpsMinimum & 300 Kbps & 400 Kbps & 500 Kbps & 1500 Kbps & 3000 Kbps

Assume we need 16GHz and 32GB RAM in order to encode video in realtime with resolution 480p, as used in the article at http://ijarcet.org/wp-content/uploads/IJARCET-VOL-3-ISSUE-2-408-413.pdf. Average user upload rate is around 100Kbps. Encoded videos will be kept for several days since they last have been watched, in order to avoid duplicated work.

The above numbers suggests that the serving nodes should be somewhere between 1-100 per video. When impossible, lower resolutions will apply. Note that the resolution will be selected according to the playing device, i.e. when mobile phone is detected, resolution is lowered. At the worst case, buffering will take place.

Encoding and streaming uses both CPU, RAM, Hard drive and Bandwidth, all four in at least moderate usage.

We recognize that this constellation of resourse consumption might not justify itself, and we account on that our peers will be part of the bigger Xennet network, hence increasing the incentive to run a node.
