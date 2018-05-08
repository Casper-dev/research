Table of content
==

[CA](#ca)<br>
[p2p CDN](#p2p-cdn)<br>
[zk-SNARK](#zk-snark)<br>
[Data storage validation](#data-storage-validation)<br>
[Chunking](chunking)<br>
[NAT bypass](nat-bypass)<br>
[VPN/Traffic encryption support](#vpntraffic-encryption-support)<br>
[Access permissions](#access-permissions)<br>
[Hosting mode (CDN)](#hosting-mode-cdn)<br>
[Provider reputation system](#provider-reputation-system)<br>
[DDoS attack protection](#ddos-attack-protection)<br>


CA
==

-  Objective

   -  HTTPS support for connections to providers

   -  EDS authentication support for providers

-  Current solution

   -  The user submits his public key, saved in the SC, during
      registration. Later, if necessary, SC can verify ownership of any
      particular signature.

-  Potential of research

   -  Storing certificates in a decentralised manner potentially solves
      the trust issues and reduces the risk of compromising private keys
      that are used to sign the certificates using consensus mechanisms.

   -  Find a way to work with both CA or CDN (which can additionally
      increase response time of select resources)

-  Open issues

   -  How to make it so nobody other than the wallet owner could
      register a public key for that wallet? Via KYC?

p2p CDN
=======

-  Objective

   -  During peak hours Internet traffic spikes six-fold compared to
      average daily figures. This owes to users consuming video content
      in the evening. A user watching a popular video stored in the
      Casper API network can cache it for half hour and seed it to other
      users, earning a reward.

-  Current solution

   -  Potentially the ability to work in CDN mode can scale to
      accommodate current traffic accordingly. To define popular content
      either a special browser or a browser extension is needed that
      would allow to determine the popularity of any particular media
      and achieve consensus on seeding it. The alternative would be to
      negotiate cooperation with large video hosting services (e.g.
      YouTube) and leave the trend calculation mechanisms to them.

-  Potential of research

   -  Large media comprises most of Internet traffic which, potentially,
      makes this market one of the most interesting ones.

   -  Centralisation makes delivering content a resource-intensive task
      (e.g. Twitch and YouTube streaming platforms servers) which can
      lead to issues during high load and make scaling the platform more
      difficult in addition to potentially degrading the quality of
      service.



zk-SNARK
========

-  Objective

   -  Transactions containing information on payments for data storage
      and traffic volume as well as other transactions that can be
      indicative of economic activities of a Casper API user can be
      encrypted with zk-SNARK.

   -  Transactions subject to encryption

      -  with information on payments for data storage

      -  with information on payments for traffic

-  Potential of research

   -  Currently the use of zk-SNARK for encryption is too time-consuming
      (average transaction encryption time is around one minute for mere
      kilobytes worth of data)

Data storage validation
=======================

-  Objective

   -  Files stored by providers can become corrupted from hardware error
      or malicious actions at the hand of a malicious provider or other
      actors.

-  Current solution

   -  Chunk storage proof algorithm. One of the network’s participants,
      called the check initiator, sends a request to the smart contract
      for proof of other participants’ storing a chunk. The smart
      contract saves and logs UUID and the address of the provider who
      has sent a request. Based on each provider’s answers, consensus
      vectors are generated. Based on consensus vectors, unscrupulous
      providers are identified.

-  Potential of research

   -  Optimize solutions by filecoin (PoR, PoSt)

   -  Decrease the load on the smart contract (it shouldn’t handle
      cryptography, only hash function)

   -  Make checks more reliable (currently we rely on the checker’s good
      will)

Chunking
========

-  Objective

   -  Splitting each file into chunks that are processed independently
      (that can be encrypted with a key and have separate sets of
      copies)

-  Potential of research

   -  A simple algorithm for updating files (e.g. when the change
      happens on the edge of chunks)

   -  Chunks consist of IPFS blocks. Can assigning UUID to low-level
      blocks be foregone?

   -  (large) Can block size be increased to 256MB (example), but never
      store it in memory in full (provided its hash is derived from the
      first few bytes)? In IPFS this issue is partially solved with
      sharding (the file is presented as a tree, hash is derived from
      the root) – in our case the UUID is stored in the root. It is
      desirable to minimize the relationship between metadata (UUID,
      metadata, encryption data) and useful data since space = money.

NAT bypass
==========

-  Objective

   -  NAT blocks the connection to external IPs for computers behind one
      NAT. These computers can be connected within the NAT directly.

   -  NAT can close connections and take up external ip:port if there is
      no data running through that connection. A Keep alive is required.

-  Current solution

   -  A one-to-one NAT bypass by setting up and maintaining a TCP
      connection with a dedicated stun/turn server.

-  Potential of research

   -  Implement ICE to the highest possible extent (including the
      required go plugins) to try bypass other types of NAT.

   -  Determine IPFS relay use cases (nodes acting as a relay can
      improve reputation and receive CST)

   -  See if smart contract helps

VPN/Traffic encryption support
==============================

-  Objective

   -  The ability for users to work with providers connected to a
      corporate or public VPN.

   -  The ability to encrypt traffic running between users and
      providers.

-  Potential of research

   -  To have the ability of setting up connections and verify identity
      solely based on the data from the smart contract.

Access permissions
==================

-  Objective

   -  Provide the option of multi-user access to files similar to Google
      Drive.

-  Current solution

   -  File access by UUID

   -  The ability to encrypt files with a password (key generation
      algorithm from OpenSSL)

-  Potential of research 

   -  Access tokens (instead of UUID)

   -  Store *client ↔ access permission* data (in the smart contract or
      with the provider)

Hosting mode (CDN)
==================

-  Objective

   -  In hosting mode, if traffic to a file exceeds the bandwidth
      offered by 4 copies, the number of copies begins increasing to
      accommodate all the traffic.

   -  CDN function of optimal traffic routing can be too demanding for
      smart contracts. In this case it is worth it to source this task
      to providers’ server software – then the resulting routes must be
      verified.

-  Potential of research

   -  Algorithms for determining increases in load (based on the number
      of requests to the smart contract, or a local signal from
      overloaded nodes).

   -  Algorithms for choosing new nodes based on their rating.

   -  The ability to set the number of copies manually.

Provider reputation system
==

   -  Objective

      -  Provider reputation metrics. Calculate service rejection risk
             for each of N network participants.

      -  Depending on rejection risk we can store files with certain
             providers more often than with others, potentially allowing
             for increased competition among providers.

      -  Besides, reputation can be used for work with DHT to receive a
             more reliable information.

   -  Current solution

      -  Quick comparison: 

         -  EigenTrust-type algorithms (EigenTrust++) require storing
                both the reputation map and all provider check-ups in
                the memory which is time-consuming and heavy on
                processing power.

         -  Bayes algorithms
                (`*http://www.cs.cornell.edu/people/egs/cs615-spring06/rep-p2pecon.pdf* <http://www.cs.cornell.edu/people/egs/cs615-spring06/rep-p2pecon.pdf>`__)
                on the other hand, they allow storing only the
                neighbour’s data, updating it in iterations as more
                information accumulates.

   -  Potential of research

      -  EigenTrust++ is adopted in the NEM blockchain in its
             Proof-Of-Importance (POI)

      -  Adopt EigenTrust++ (utilize a smart contract as a trusted
             intermediary and attempt to simplify)

      -  Adopt the Bayes method (separate situations of false
             response/response timeout, utilize a smart contract as a
             source of additional information)

DDoS attack protection
==

-  Objective

   -  Determine how to retain file accessibility if a DDoS attack
      affects all 4 nodes storing one file.

-  Current solution

   -  **In general terms it is impossible to retain file accessibility
      in this case; the problem can be solved upon narrowing down the
      criteria of real-world feasibility.**

   -  For large files it is almost impossible: the existence of 4\*N
      (with N being the number of chunks) machines (the chance of
      switching one machine on more than once is small in case of a high
      number of machines on the network) dramatically limits the attack
      vector usually planned for a moderate concentrated server cluster.

-  Potential of research

   -  Choose methods of identifying the start of an attack, protection
      against it during file copying onto other nodes

   -  Choose the algorithm for copying files onto other nodes and
      protecting them from exposure to a DDoS attack (perhaps make the
      copies private and transfer them to the owner privately)

   -  Consider integrating existing DDoS protection solutions into
      Casper API.

   -  Small files can potentially be protected by existing privileged
      providers (DC) that will always keep copies of small files.
