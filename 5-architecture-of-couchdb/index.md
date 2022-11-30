# 5 Architecture of CouchDB

CouchDB is a document-oriented database known for its data synchronization feature. It currently is a popular tool for master-less synchronization directly between devices. With multiple implementations being available for server, smartphone or in-browser deployment it seems like an excellent fit for our requirements. In this Chapter we will therefore investigate the inner workings of CouchDB’s architecture and synchronization protocol.

## 5.1 Overview

There is no official documentation of CouchDB’s internals - the sources used for our analysis is the CouchDB Guide \[14\] written by core developers, the CouchDB Wiki \[34\] and the source code \[35\] for details of the replication protocol. Note that the CouchDB authors actually refer to synchronization as ‘replication’.

The original implementation of CouchDB exposes an HTTP interface for all interactions with the database. This makes it possible to write web applications directly targeting CouchDB as the server, eleminating any middleware in between.

If an application developer is able to design his application within the constraints of CouchDB being the only backend, it is called a **CouchApp**.

CouchApps have the interesting property of being completely replicatable between CouchDB nodes. So an entire working application can be deployed to a device just by replicating it from a remote CouchDB node.

Unfortunately only few applications get by with CouchDB being the only backend required.

With **PouchDB** there has recently emerged a CouchDB implementation inside the browser in pure JavaScript. It makes use of HTML5’s IndexedDB as the storage layer and can therefore be included into a web application without requiring any plugins.

PouchDB exposes a similar interface like CouchDB and can fully synchronize with an actual CouchDB node on a server.

CouchDB’s data model is relatively simple - it mainly supports the storage of JSON- documents. Each document has an ID under which it can be efficiently retrieved and updated.

There is no query language like SQL available as the stored JSON-documents are not required to have any fixed schema.

If effecient access to documents based on some of its properties is required, CouchDB allows the definition of **views**. Views are created by providing a map and possibly a reduce function. The map function is used to define an index, while the reduce function can be used to efficiently compute aggregates.

An important aspect of CouchDB is that all its operations are lockless. It achieves this by writing all data to an append-only data structure therefore never updating any data in-place. Every update of a document creates a new version of it - similar to how some version control systems operate.

On each write of a new version, CouchDB requires that the current version ID of the document is passed. This guarantees that the client has read the current version before he is able to write any updates. If two clients concurrently update the same JSON-document, the first update that reaches the database succeeds and thereby creates a new version ID. The second concurrent update will therefore be rejected as the client did supply an outdated version ID. CouchDB treats this as an update conflict and notifies the second client. The second client can then review the changes of the first client, possibly merge it with his changes and re-send it with the correct version ID. This concept is often referred to as **Optimistic Locking**.

In the case of concurrent edits on two nodes of the same database the conflict handling is more complex. Concurrent writes can no longer be linearized through optimistic locking as the two database nodes are possibly disconnected.

CouchDB solves this by applying concepts of **Multi-Version Concurrency Control**. Both nodes can update the same documents thereby creating two conflicting versions. All versions of a document point to its ancestor resulting in a version tree. If the database nodes synchronize each other both nodes will end up with both conflicting versions of the document.

CouchDB uses a deterministic algorithm to choose one of the nodes as a winner. As this choice is random to the user of an application it is often not the desired result. It is therefore possible to either pick a different conflicting version as the winner or merge both versions to a new revision.

## 5.2 Synchronization Protocol

CouchDB does not really have a synchronization protocol at all - it is actually a fairly simple algorithm that uses only the existing HTTP interfaces. A ‘CouchDB synchronizer’ therefore does not even have to run inside CouchDB but can be an external program that only needs access to the public interfaces of two CouchDB nodes. What makes CouchDB’s synchronization work is at the core a **changes stream** that is accessible through an HTTP interface as well. Every update of a document triggers an update of the changes stream thereby adding a new entry with an update sequence ID, the new version ID of the updated document and the document ID itself. The update sequence ID grows monotonously with every update. At any point the changes stream includes all updates made to the database. If the same document has been update multiple times, only the most recent update is included in the stream.

The synchronization process from a CouchDB node A to B follows the following steps:

1. Read the **last source sequence ID** stored on B - it represents the last update it read on the previous synchronization.
2. Read a few entries from the changes stream of A starting at the last source sequence ID.
3. Send B the set of document revisions - B responds with the subset of those not stored in B.
4. Fetch the missing document revisions from A and write them to B.
5. Update the last source sequence ID on B.
6. Restart the process if there are remaining updates.

All steps only use public interfaces exposed by both nodes. Continuous synchronization can be implemented trivially by infinitely repeating the steps.

As explained before the process may result in conflicting document versions. It is the responsibility of the application developer to handle those conflicts after each synchronization.

It is important to note that the last source sequence ID is only valid for a particular CouchDB node. When synchronizing with a new node for the first time the process has to start at the first update sequence. The protocol is therefore only efficient when continuously synchronizing with the same nodes. If a node is not identified as a previously synchronized peer, the protocol causes redundant network operations growing linearly with the size of the changes stream.

## 5.3 Fulfillment of Requirements

As a schemaless database CouchDB at least supports the storage of any kind of data model. Its awareness of the type of data is at the same time very low.

When synchronizing databases CouchDB treats every JSON-document as an **atom**. There is no way to give CouchDB an increased level of awareness of an application’s data model. Application developers are forced to write a large amount of additional merging logic inside their application.

**Flexible data model** support is therefore given while it requires additional app-specific logic to cater for CouchDB’s lack of structural awareness.

The CouchDB model of multi-version concurrency control fulfills the requirement of **optimistic synchronization**. Concurrently edited data on multiple CouchDB nodes is **eventually consistent** if synchronized with each other.

The optimistic locking mechanism combined with a document’s version tree ensures **causality preservation** and exposure of **conflicts**.

CouchDB’s distributed synchronization protocol supports flexible network topologies. **Cross-platform support** is given with implementations available for servers, mobile devices and even web browsers.

To build a more suitable solution for our requirements we can build on many of CouchDB’s design decisions.

Major room for improvement lies in stronger data model awareness thereby relieving the application developer of repetitive logic and improving **unobtrusive integration** into an application.

CouchDB only remembers the version history of a document in the form of version IDs. The actual documents are not retained not even those of common ancestors in the case of conflicts. CouchDB’s version history can therefore only be used to identify conflicts but does not support **three-way merging**.

Furthermore, we will look into more efficient ways to synchronize with unknown nodes limiting the amount of redundant network requests.