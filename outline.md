
#Syncing Strategies for Mobile Devices
##Scenarios
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

##Problems and Goals
We will evaluate syncing strategies for the listed application scenarios.

Requirements for strategies:

- Causality preservation
- Eventual consistency
- Optimistic synchronization
- Expose conflicts
- Support peer-to-peer or hybrid synchronization

Aspects to consider when evaluating strategies:

- How are updates detected?
- How are updates propagated? (Stream or Snapshot)
- How are updates merged/reconciled? (State or Edit-based)
- Level of structural awareness (Textual, Syntactic, Semantic/Structural)

##Comparing Existing Systems
###git

###Vector clocks (DynamoDB/Riak)

###CouchDB

##Approach

##Evaluation