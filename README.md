# raft-dezyne-model

This repository contains a Dezyne model of the Raft consensus algorithm.

[Consensus](https://en.wikipedia.org/wiki/Consensus_(computer_science)) is a fundamental problem in distributed computing and multi-agent systems. 
Objective of consensus algorithms is to achieve overall system reliability in the presence of a number of faulty processes. 
Consensus algorithms allow a collection of machines to work as a coherent group that can survive the failures of some of its members.

[Raft](https://raft.github.io) is a proven and reasonably understandable consensus algorithm, used in modern cloud server clusters.

![Raft concept](images/raft-concept.png)

The [Dezyne](https://www.verum.com/dezyne/) model in this repository is based on the paper [In Search of an Understandable Consensus Algorithm (Extended Version)](https://raft.github.io/raft.pdf) by Diego Ongaro and John Ousterhout.

The Raft algorithm decomposes consensus within a cluster of servers into three sub-problems:

1. Leader Election
A leader is elected by majority vote among the servers in the cluster. 
In case the leader fails automatically a new leader is elected.
The algorithm takes care that in each term (virtual time period) there can only be one leader.
The other servers in the cluster follow the leader's orders.

2. Log replication
The leader receives commands from clients, logs them and replicates them to the followers. 
When a log entry is replicated to a majority of servers, it is supposed to be committed. 
The command will then be executed and its result is returned to the client.

3. Guarding Consistency
The rules of the algorithm take care that commands are committed and executed by all servers in the same sequence they have been requested.
If one of the servers has committed a log entry at a particular index, no other server can apply a different log entry for that index.
	
Each server can be in one of three ![states](images/raft-states): leader, follower, or candidate. 

![Raft states](images/raft-states.png)

A leader leads the pack. A follower follows the leader. A candidate wants to become leader.

A virtual time measure, called term is used. Time is divided in terms. 
The term is increased monotonically by a candidate each time it (re)starts a new leader election.
In a term there is at most one leader.

Initially all servers start as followers.
A follower that does not hear from a leader within a (randomly selected) election timeout will become candidate.
A candidate will become leader when it has collected a majority of votes from other servers in the cluster.
A candidate or leader will step down and become follower when they hear (via messages they receive) from a leader of a higher term then their own.

Clients can ask the leader to log, replicate and execute a command using a:
- ClientRequest RPC

There are only two Remote Procedure Calls (RPC's) used to reach consensus among the servers:
- RequestVote RPC: used by candidate to collect votes to become leader
- AppendEntries RPC: used by the leader to replicate log entries and as heartbeat to confirm its leadership


