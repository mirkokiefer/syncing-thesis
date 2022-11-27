# Heidelberg University Institute of Computer Science Database Systems Research Group

**Bachelor Thesis**

## Histo: A Protocol for Peer-to-Peer Data Synchronization in Mobile Apps

Name: Mirko Kiefer  
Matricle: 2746040  
Supervisor: Prof. Dr. Gertz  
Date of Submission: September 27, 2013

I declare that this thesis was composed by myself and that the work contained therein is my own, except where explicitly stated otherwise in the text.

Date of Submission: September 27, 2013

# Abstract

Inspired by distributed version control for source code we develop Histo, which is a framework for peer-to-peer data synchronization in mobile applications.

Our objectives when developing Histo are driven by exemplaric workflows in a collabo- rative task manager. We see **offline support** as a key requirement of modern mobile applications. Histo is designed to keep all data locally on the client device to ensure a user is not blocked from using an application.

Every application has a different data model, we develop methods to support **flexible data models**. We show that by mapping an application’s data to a hierarchical model, we can efficiently synchronize data.

Locking mechanisms are shown to be non-feasible when working with loosely connected devices. The **synchronization protocol**, which is at the heart of Histo, is therefore designed to work without any locking logic. We instead rely on a concept of **optimistic synchronization**, which guarantees **eventual consistency** of an application’s data. Handling concurrent edits and resulting conflicts correctly is another core element of Histo. Histo merges concurrent edits and identifies conflicts using **three-way-merging**. We find that the only way to implement three-way-merging in a distributed setting is by tracking the edit history on each device.

Histo’s synchronization protocol is shown to perform well in a range of network topolo- gies. The most extreme cases being **client-server** and **peer-to-peer**. For each network topology, we show how assumptions we make on the data history allows to minimize the amount of data stored on each device.

We focus on developing a practical solution that works on a broad range of devices. Histo is therefore implemented with **open web standards**. To our knowledge there are no publicly available solutions with these objectives.

We see potential to continue the work by exploring other application scenarios and adding support for different types of data. This could include conflict handling support for specialized data structures like those of a spreadsheet or a text editor.

- [Heidelberg University Institute of Computer Science Database Systems Research Group](#heidelberg-university-institute-of-computer-science-database-systems-research-group)
  - [Histo: A Protocol for Peer-to-Peer Data Synchronization in Mobile Apps](#histo-a-protocol-for-peer-to-peer-data-synchronization-in-mobile-apps)
- [Abstract](#abstract)
- [1 Introduction](./1-introduction/index.md)
  - [1.1 Motivation](./1-introduction/index.md#11-motivation)
  - [1.2 Objectives and Approach](./1-introduction/index.md#12-objectives-and-approach)
  - [1.3 Structure of the Thesis](./1-introduction/index.md#13-structure-of-the-thesis)
- [2 Background](./2-background/index.md)
  - [2.1 Defining Data Synchronization](./2-background/index.md#21-defining-data-synchronization)
  - [2.2 Transactions and Consistency](./2-background/index.md#22-transactions-and-consistency)
  - [2.3 Stream-Based Synchronization](./2-background/index.md#23-stream-based-synchronization)
  - [2.4 History-Based Synchronization](./2-background/index.md#24-history-based-synchronization)
  - [2.5 Three-Way Merging](./2-background/index.md#25-three-way-merging)
  - [2.6 Lowest Common Ancestor](./2-background/index.md#26-lowest-common-ancestor)
  - [2.7 Content Adressable Storage](./2-background/index.md#27-content-adressable-storage)
  - [2.8 HTML5 and Offline Applications](./2-background/index.md#28-html5-and-offline-applications)
    - [2.8.1 Web Storage](./2-background/index.md#281-web-storage)
    - [2.8.2 Web SQL Database](./2-background/index.md#282-web-sql-database)
    - [2.8.3 Indexed Database](./2-background/index.md#283-indexed-database)
    - [2.8.4 Cache Manifests](./2-background/index.md#284-cache-manifests)
- [3 Application Scenario - A Collaborative Task Manager](./3-application-scenario-a-collaborative-task-manager/index.md)
  - [3.1 User Story 1: Creating Projects](./3-application-scenario-a-collaborative-task-manager/index.md#31-user-story-1-creating-projects)
  - [3.2 User Story 2: Creating and Editing Tasks](./3-application-scenario-a-collaborative-task-manager/index.md#32-user-story-2-creating-and-editing-tasks)
  - [3.3 User Story 3: Commenting on Tasks](./3-application-scenario-a-collaborative-task-manager/index.md#33-user-story-3-commenting-on-tasks)
  - [3.4 User Story 4: User Workflows](./3-application-scenario-a-collaborative-task-manager/index.md#34-user-story-4-user-workflows)
  - [3.5 Data Schema](./3-application-scenario-a-collaborative-task-manager/index.md#35-data-schema)
- [4 Requirements](./4-requirements/index.md)
  - [4.1 Flexible Data Model Support](./4-requirements/index.md#41-flexible-data-model-support)
  - [4.2 Optimistic Synchronization](./4-requirements/index.md#42-optimistic-synchronization)
  - [4.3 Causality Preservation and Conflicts](./4-requirements/index.md#43-causality-preservation-and-conflicts)
  - [4.4 Flexible Network Topologies](./4-requirements/index.md#44-flexible-network-topologies)
  - [4.5 Integration with Existing Application Logic](./4-requirements/index.md#45-integration-with-existing-application-logic)
  - [4.6 Cross-Platform](./4-requirements/index.md#46-cross-platform)
  - [4.7 Other Requirements](./4-requirements/index.md#47-other-requirements)
  - [4.8 Summary](./4-requirements/index.md#48-summary)
- [5 Architecture of CouchDB](./5-architecture-of-couchdb/index.md)
  - [5.1 Overview](./5-architecture-of-couchdb/index.md#51-overview)
  - [5.2 Synchronization Protocol](./5-architecture-of-couchdb/index.md#52-synchronization-protocol)
  - [5.3 Fulfillment of Requirements](./5-architecture-of-couchdb/index.md#53-fulfillment-of-requirements)
- [6 Architecture of Histo](./6-architecture-of-histo/index.md)
  - [6.1 Hierarchical Data Model Mapping](./6-architecture-of-histo/index.md#61-hierarchical-data-model-mapping)
  - [6.2 Committing Objects](./6-architecture-of-histo/index.md#62-committing-objects)
  - [6.3 Synchronization Protocol](./6-architecture-of-histo/index.md#63-synchronization-protocol)
    - [6.3.1 Update Detection and Propagation](./6-architecture-of-histo/index.md#631-update-detection-and-propagation)
    - [Robustness](./6-architecture-of-histo/index.md#robustness)
    - [6.3.2 Merging](./6-architecture-of-histo/index.md#632-merging)
    - [Robustness](./6-architecture-of-histo/index.md#robustness-1)
  - [6.4 Update Detection Between Branches](./6-architecture-of-histo/index.md#64-update-detection-between-branches)
    - [6.4.1 Commit History Difference](./6-architecture-of-histo/index.md#641-commit-history-difference)
    - [6.4.2 Application Data Change Detection](./6-architecture-of-histo/index.md#642-application-data-change-detection)
  - [6.5 Reconciliation Through Three-Way Merging](./6-architecture-of-histo/index.md#65-reconciliation-through-three-way-merging)
    - [6.5.1 Merge Scenario](./6-architecture-of-histo/index.md#651-merge-scenario)
    - [6.5.2 Differencing](./6-architecture-of-histo/index.md#652-differencing)
    - [6.5.3 Diff Merging](./6-architecture-of-histo/index.md#653-diff-merging)
    - [6.5.4 Patching](./6-architecture-of-histo/index.md#654-patching)
    - [6.5.5 Hierarchical Merging](./6-architecture-of-histo/index.md#655-hierarchical-merging)
  - [6.6 Handling Conflicts](./6-architecture-of-histo/index.md#66-handling-conflicts)
  - [6.7 Synchronization Topologies](./6-architecture-of-histo/index.md#67-synchronization-topologies)
    - [6.7.1 Master - Client](./6-architecture-of-histo/index.md#671-master---client)
    - [6.7.2 Client - Client](./6-architecture-of-histo/index.md#672-client---client)
    - [6.7.3 Multi Master - Client](./6-architecture-of-histo/index.md#673-multi-master---client)
    - [6.7.4 Hierarchical](./6-architecture-of-histo/index.md#674-hierarchical)
- [7 Realization](./7-realization/index.md)
  - [7.1 Technology](./7-realization/index.md#71-technology)
  - [7.2 Fullfillment of Requirements](./7-realization/index.md#72-fullfillment-of-requirements)
  - [7.3 Correctness and Robustness](./7-realization/index.md#73-correctness-and-robustness)
- [8 Conclusion and Outlook](./8-conclusion-and-outlook/index.md)
- [Bibliography](./bibliography/index.md)