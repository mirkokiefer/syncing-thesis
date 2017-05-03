
# Syncing Strategies for Mobile Devices
## Motivation
Building collaborative apps that sync data among a group of users is hard.  
The widespread adoption of mobile devices with limited network access requires the offline availability of data and apps.  
Some aspects of syncing are often application specific and can therefore not be solved in a generic way.  
However there are recurring patterns that can be used to build application specific solutions. The goal of this thesis is to develop a syncing framework that speeds up the development of collaborative apps.

## Application Scenarios
### Relational Data Synchronization
The Wunderlist app serves as an example for a common kind of data model that requires syncing of data in a relational schema.

Wunderlist's schema could be defined as the following:

- User (name, email, has Todo Lists)
- Invited User (name, email)
- Invited User List (has Invited Users)
- Todo Item (title, description, due date, belongs to Todo List)
- Todo List (name, belongs to Users, has Todo Items)

The User type has a singleton instance who represents the user of the app.  
Users can be invited to Todo Lists. As their list of Todo Lists is hidden from the current user Invited User is a separate type.  
Invited User List is simply a cached list of all users that have been invited in the past.  
While Invited User List is an unordered list, Todo Lists and Todo Items are ordered.

Syncing lists of unordered object IDs never causes conflicts while syncing ordered object IDs can cause order conflicts.

### Hierarchical Data Synchronization
Dropbox synchronizes a file system - it is therefore a good example for syncing of hierarchical data.

The data model is simple:

- Tree Item (name)
- Tree extends Tree Item (has children of type Tree Items)
- Data extends Tree Item (data)

The list of child Tree Items can either be ordered or unordered. While Dropbox does not sync the order of files there are scenarios where this is required.

Syncing trees can trigger conflicts if sub trees have been modified concurrently.

### Text Synchronization
Collaborative document editors like Google Docs need to synchronize text that is concurrently edited.  
Google Docs currently does not support offline editing.

Syncing text is equal to the problem of syncing an ordered list and can trigger conflicts.

## General Concepts
### Causality Preservation
### Optimistic Synchronization
### State vs. Edit-Based Syncing
### Three-Way Merging
### Vector Clocks
### Operational Transformation
### Commutative Replicated Data Types

## Requirements and Evaluation Criteria
We will evaluate syncing strategies for the listed application scenarios.

Requirements for strategies:

- Causality preservation
- Eventual consistency
- Optimistic synchronization
- Expose conflicts
- Support peer-to-peer or hybrid synchronization

(TODO: need to explain why this set of requirements, constraints on mobile devices...)

Aspects to consider when evaluating strategies:

- How are updates detected?
- How are updates propagated? (Stream or Snapshot)
- How are updates merged/reconciled? (State or Edit-based)
- Level of structural awareness (Textual, Syntactic, Semantic/Structural)

## Existing Systems
### git

- Data structure: filesystem/tree
- Merging: tree-based, three-way merge
- Propagation: snapshot-based
- Supports peer-to-peer

### CouchDB

- Data structure: key-value
- Merging: tree-based
- Propagation: stream-based
- Supports peer-to-peer

### DynamoDB/Riak
Not made for offline editing - only serves as example for vector clocks.

- Data structure: key-value
- Merging: vector clocks
- Propagation: stream-based

## Syncing Framework
### Design Guidelines
- **no timestamps**: state-based 3-way merging
- **no change tracing**: change tracing is not necessary - support diff computation on the fly
- **data agnostic**: leave diff and merge of the actual data to plugins
- **distributed**: syncing does not require a central server
- **be small**: only implement the functional parts of syncing - leave everything else to the application (transport, persistence)
- **sensitive defaults**: have defaults that *just work* but still support custom logic (e.g. for conflict resolution)

### Proof-Of-Concept Implementation
As syncing is state based we need to track the entire history of a database.  
Every client has his own replica of the database and commits data locally.  
On every commit we create a commit object that links both to the new version of the data and the previous commit.  
If a client is connected to a server he will start the sync process on every commit. As synclib2's architecture is distributed a server could itself be a client who is connected to other servers.  
To the latest commit on a database we refer to as the 'head'.

Syncing follows the following protocol:

```
Client has committed to its local database.
Client pushs all commits since the last synced commit to Server.
Client asks Server for the common ancestor of client's head and the server's head
Client pushs all changed data since the common ancestor to Server.

if common ancestor == server head
  // there is no data to merge
  try fast-forward of server's head to client's head
  if failed (someone else updated server's head in the meantime) then start over
else
  Client asks Server for all commits + data since the common ancestor
  Client does a local merge and commits it to the local database
  start over
```

This protocol is able to minimize the amount of data sent between synced stores even in a distributed, peer-to-peer setting.

Updating the server's head uses optimistic locking. To update the head you need to include the last read head in your request.

## Evaluation
Evaluate the proof-of-concept by simulating syncing of data structures used in the problem scenarios with realistic network latency and disconnection.

## Sources

- [1] T. Lindholm, “XML-aware data synchronization for mobile devices,” 2009.
- [2] P. Padmanabhan, L. Gruenwald, A. Vallur, and M. Atiquzzaman, “A survey of data replication techniques for mobile ad hoc network databases,” The VLDB Journal, vol. 17, no. 5, pp. 1143–1164, May 2008.
- [3] N. Fraser, “Differential synchronization,” pp. 13–20, 2009.
- [4] S. Weiss, P. Urso, and P. Molli, “Logoot: a scalable optimistic replication algorithm for collaborative editing on P2P networks,” pp. 404–412, 2009.
- [5] A. Demers, K. Petersen, M. Spreitzer, D. Ferry, M. Theimer, and B. Welch, “The Bayou architecture: Support for data sharing among mobile users,” pp. 2–7, 1994.
- [6] M. Letia, N. Preguiça, and M. Shapiro, “CRDTs: Consistency without concurrency control,” arXiv.org, vol. cs.DC. 06-Jul-2009.
- [7] D. Ratner, P. Reiher, G. J. Popek, and G. H. Kuenning, “Replication requirements in mobile environments,” Mobile Networks and Applications, vol. 6, no. 6, pp. 525–533, 2001.
- [8] T. Lindholm, “A 3-way merging algorithm for synchronizing ordered trees—,” Master's thesis, Helsinki University of Technology, 2001.
- [9] G. DeCandia, D. Hastorun, M. Jampani, G. Kakulapati, A. Lakshman, A. Pilchin, S. Sivasubramanian, P. Vosshall, and W. Vogels, “Dynamo: amazon's highly available key-value store,” vol. 41, no. 6, pp. 205–220, 2007.
- [10]  T. Lindholm, “A three-way merge for XML documents,” pp. 1–10, 2004.
- [11]  K. Petersen, M. Spreitzer, D. Terry, and M. Theimer, “Bayou: replicated database services for world-wide applications,” pp. 275–280, 1996.
- [12]  N. Preguiça, J. M. Marques, M. Shapiro, and M. Letia, “A Commutative Replicated Data Type for Cooperative Editing,” presented at the 2009 29th IEEE International Conference on Distributed Computing Systems (ICDCS), pp. 395–403.
- [13]  G. Oster, P. Urso, P. Molli, and A. Imine, “Real time group editors without Operational transformation,” 2005.
- [14]  S. Agarwal, D. Starobinski, and A. Trachtenberg, “On the scalability of data synchronization protocols for PDAs and mobile devices,” Network, IEEE, vol. 16, no. 4, pp. 22–28, 2002.
- [15]  M. Satyanarayanan, J. J. Kistler, P. Kumar, M. E. Okasaki, E. H. Siegel, and D. C. Steere, “Coda: A highly available file system for a distributed workstation environment,” Computers, IEEE Transactions on, vol. 39, no. 4, pp. 447–459, 1990.
- [16]  L. Lamport, “Time, clocks, and the ordering of events in a distributed system,” Communications of the ACM, vol. 21, no. 7, pp. 558–565, 1978.
- [17]  R. Cox and W. Josephson, “File synchronization with vector time pairs,” 2005.
- [18]  J. N. Foster, M. B. Greenwald, C. Kirkegaard, B. C. Pierce, and A. Schmitt, “Exploiting schemas in data synchronization,” Journal of Computer and System Sciences, vol. 73, no. 4, pp. 669–689, Jun. 2007.
- [19]  R. Van Renesse, D. Dumitriu, V. Gough, and C. Thomas, “Efficient reconciliation and flow control for anti-entropy protocols,” p. 6, 2008.
