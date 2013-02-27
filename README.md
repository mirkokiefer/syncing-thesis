
#Syncing Strategies for Mobile Devices
##Motivation
Building collaborative apps that sync data among a group of users is hard.  
The widespread adoption of mobile devices with limited network access requires the offline availability of data and apps.  
Some aspects of syncing are often application specific and can therefore not be solved in a generic way.  
However there are recurring patterns that can be used to build application specific solutions. The goal of this thesis is to develop a syncing framework that speeds up the development of collaborative apps.

##Problem Scenarios
###Relational Data Synchronization
The Wunderlist app serves as an example for a common data schema that requires syncing of a relational schema.

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

###Hierarchical Data Synchronization
Dropbox synchronizes a file system - it is therefore a good example for syncing of hierarchical data.

The data model is simple:

- Tree Item (name)
- Tree extends Tree Item (has children of type Tree Items)
- Data extends Tree Item (data)

The list of child Tree Items can either be ordered or unordered. While Dropbox does not sync the order of files there are scenarios where this is required.

Syncing trees can trigger conflicts if sub trees have been modified concurrently.

###Text Synchronization
Collaborative document editors like Google Docs need to synchronize text that is concurrently edited.  
Google Docs currently does not support offline editing.

Syncing text is equal to the problem of syncing an ordered list and can trigger conflicts.

##General Concepts
###Causality Preservation
###Optimistic Synchronization
###State vs. Edit-Based Syncing
###Three-Way Merging
###Vector Clocks
###Operational Transformation
###Commutative Replicated Data Types

##Requirements and Evaluation Criteria
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

##Existing Systems
###git

###CouchDB

###DynamoDB/Riak

##Syncing Framework
###Design Guidelines
- **no timestamps**: state-based 3-way merging
- **no change tracing**: change tracing is not necessary - support diff computation on the fly
- **data agnostic**: leave diff and merge of the actual data to plugins
- **distributed**: syncing does not require a central server
- **be small**: only implement the functional parts of syncing - leave everything else to the application (transport, persistence)
- **sensitive defaults**: have defaults that *just work* but still support custom logic (e.g. for conflict resolution)

###Proof-Of-Concept Implementation
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

##Evaluation
Evaluate the proof-of-concept by simulating syncing of data structures used in the problem scenarios with realistic network latency and disconnection.
