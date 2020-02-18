---
layout: default
title: dAuth - Distributed Authentication and use in Roaming contexts
nav_order: 10
date: 13-02-2020
categories: LTE, 5G, Blockchain, Networking
nav_exclude: true
summary: In many rural contexts around the world, close relationships within clusters of neighboring communities in a region are typical and culturally important, for example in Indigenous tribal settings. Network users may frequently travel between these related communities and want to use the cellular service present in each one.
---

# {{page.title}}
{% if page.categories %}
{% assign categories = page.categories | split: ',' %}
{% for category in categories %}
{{category}}
{: .label }
{% endfor %}
{% endif %}

##### By: [Sudheesh Singanamalla](https://sudheesh.info/), [Esther Jang](https://homes.cs.washington.edu/~infrared/) on {{page.date | date_to_string }}
##### Acknowledgements: [Matthew Johnson](https://matt9j.net/), [Spencer Sevilla](https://spencersevilla.com/bio/), [Kurtis Heimerl](https://kurti.sh/)

---

In our [research lab](https://ictd.cs.washington.edu/) we work on small-scale, community-managed cellular networks, which usually consist of a single base station attached to an edge network core. Our primary use case is to address the [long tail of connectivity in remote rural areas](https://www.gsma.com/mobilefordevelopment/resources/unlocking-rural-coverage-enablers-commercially-sustainable-mobile-network-expansion/). In recent years our lab developed a variety of technologies to enable the adoption of community LTE networks for rural Internet access [dLTE](https://dl.acm.org/doi/10.1145/3286062.3286064), [colte](https://github.com/uw-ictd/colte), building on top of open source stacks such as OpenAirInterface [(OAI)](https://www.openairinterface.org/) and more recently [Open5GS (formerly NextEPC)](https://open5gs.org/). 

In support of this we’ve developed “distributed authentication” making roaming between multiple independent community networks possible. But why can’t we do roaming the old-fashioned way? What even is roaming, why do we need it, and how do we change the way it works? Read on for the answers to all of these questions.

## Rural Community Cellular Use Case

In many rural contexts around the world, close relationships within clusters of neighboring communities in a region are typical and culturally important, for example in Indigenous tribal settings. Network users may frequently travel between these related communities and want to use the cellular service present in each one. At the same time, there are many reasons for these communities to not want to share the same EPC and backhaul connection, including political autonomy and reducing shared points of failure or network complexity. Therefore, we expect the capability of “friendly” networks to easily establish roaming agreements with each other to greatly improve usability and scalability of community cellular networks in these scenarios. We will talk more about this concept, which we call “cooperative cellular” networks, in another blog post!

## Current LTE Authentication - What you need to know

LTE networks use the Evolved Packet System Authentication and Key Agreement (EPS-AKA) procedure for bidirectionally authenticating the user to the network and the network to the user. To understand how authentication works in LTE, take a look at our post on LTE Authentication and the relevant cryptography that makes it possible [link]. In this post let us take a look at a critical part of the LTE Authentication i.e. the SQN numbers and understand how they are actually computed and work.

### How do SQNs really work?

The key to implementing the authentication procedure lies in the generation and the usage of SQN values. When a UE tries to attach to the network it discloses its identity. The MME uses the identity and makes an Authentication-Information-Request to the HSS. The HSS contains pre-shared key information which is also available with the UE. These include the LTE Symmetric Key (Ki), the AMF value and the OP/OPc value. The HSS uses a value of SQN that it last knew from the state and increments it by 32 and uses a randomly generated RAND value to generate the values of XRES, AUTN which are parts of the authentication vector.

SQNs are 4 octet sequences which are different for every authentication and are one time use only to avoid replay attacks. The first 27 bits of the SQN comprise the sequence number SEQ which is incremented by 32 and the remaining 5 bits correspond to the IND value. We can visualize this to be a matrix of values (for IND=0):

| 0   |  32 |  64 | 96  | 128 | ... |
|-----|:---:|----:|-----|-----|-----|
| 1   |  33 |  65 | 97  | 129 | ... |
| 2   |  34 |  66 | 98  | 130 | ... |
| ... | ... | ... | ... | ... | ... |
| 31  | 63  | 95  | 127 | ... | ... |

A UE traverses and continues to process SEQ numbers in a given row / vector i.e. [0,32, 64 …] and uses them in sequence to authenticate with the EPC. In case of a synch failure, the UE switches the vector and challenges the network. The network validates the new SQN number and increments the value in the vector, updates its state of the SQN and computes the necessary authentication vector and challenges the UE in an attempt to authenticate. If the network decides to skip sequences for e.g. the network uses SEQ=128 and computes the SQN but the UE expects SEQ=64, since the vector being used is still the expected vector, the UE authenticates itself but invalidates the usage of SEQ=64 and SEQ=96.

### When do UEs re-auth? What happens if SQNs are not in Sync anymore?

UEs trying to authenticate the network using the Authentication Requests with the Authentication Vector `(AUTN=SQN||AK||AMF||MAC)` need to perform re-authentication if:

1. MAC failure: If the network provides an AUTN value which is incorrect, the UE sends an Authentication Failure message to the network and may decide to terminate the authentication procedure for this session. The UEs reinitiate a session for authentication.
2. SQN Failure: These are the failures we are interested in, where the UE finds that the supplied SQN is outside the current vector it is traversing or older which means that the HSS has a stale state. In an attempt to inform the network about the current state, the UE as discussed before, switches the operating vector and challenges the network by sending an Authentication Failure along with the reason as a Synch Failure defined by EMM Cause #21. The UE computes a resynchronization token AUTS and begins the network re-synchronization procedure. The MME requests the HSS for updated vectors after validating the AUTS value. The HSS updates the state of the SQN and increments the value being used and re-initiates the Authentication Request with the UE. The UE now validates the authentication vectors and can attach to the network. If two consecutive synch failures happen at the UE, the network terminates the authentication procedures with an Authentication Reject message.


## Changes to the current LTE Authentication

In our trust model, the subscriber has a trusted relationship with the community cellular network provider and the LTE symmetric key Ki is stored in the network operators’ HSS. Peer cellular providers and their infrastructure do not trust the roaming user devices. Additionally the Home network operator does not trust the peer operator with billing and reports of data consumption. 

### Why is pre-computing the values the answer to this problem?

To address the problem we pre-compute the authentication vectors corresponding to a specific SQN vector which the home telecom operator decides to dedicate for roaming usage. In the effort of pre-computing the authentication values and sharing the information to the rest of the network, we aim to avoid sharing the LTE key or allowing prolonged attacks. Each authentication vector can also be attached to a predefined data allowance resulting in local breakouts from the roaming operator to provide better Internet connectivity to the subscriber instead of tunneling the roaming subscriber to their home network (which is done currently). Additionally, these vectors can be used in offline mode by community cellular networks  where the cellular providers face tremendous connectivity challenges and cannot contact the original subscriber’s network core.

## Implementation using Hyperledger Sawtooth

We implemented a distributed HSS based authentication where users could roam between the networks without prior roaming agreements in place or needing the symmetric keys to be shared. Our lab setup involves three EPC nodes running a modified version of Open5Gs which updates the database with authentication vectors for each user in the network. Each of these EPC nodes behaving as different network operators also run individual Hyperledger Sawtooth Validator nodes and a custom transaction processor (ccellular-tp) with all the nodes running the PBFT/PoET consensus protocols. The custom transaction processor listens to the changes happening in the HSSs and packages the updates to the database i.e. new vector generation / vector updates etc.., as transactions and sends the transactions using a ccellular client. These transactions received by each of the validators in the network are processed by the validators and packaged into a block in the blockchain.

## Results & Roadmap Ahead

Building dAuth has been a fun experience with a lot of learning in the process. At the end of the implementation and experiments we were able to:
1. Successfully perform an authentication where the UE attaches to a different EPC which uses previously generated vectors provided by the home EPC.
2. Successfully able to create and deploy a blockchain network running PoET consensus and corresponding transaction processor which performs reads and updates on the HSS by reading and writing authentication vectors into the database.

![Benchmarks](/img/benchmark.png)

## Future Work and Next Steps

We’ve identified several improvements to make in future iterations of the project. There are some performance optimizations which need attention. On a local network, the performance of the blockchain for small transaction sizes seems to be much poorer than expected. Part of this reason could be the time needed to sign each of the transactions and send it to the network. We currently do this serially and could potentially explore batching of database updates or exploit parallelism to speed the signature timings. The second optimization is with respect to the data actually being serialized into the transaction, for the proof of concept, we take the database cursor and serialize it into the transaction which is deserialized and directly executed on the other EPCs connected to the blockchain network. These cursors can sometimes be large for partial updates to a collection in the database. Having more efficient gRPC message definitions to package only the necessary parts of the commands and perform the read/write independently would make this process much faster.

---

