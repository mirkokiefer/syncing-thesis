# 1 Introduction

## 1.1 Motivation

Developing mobile applications that work offline and can synchronize peer-to-peer is currently an extremely ambitious task. The problem has been solved for distributed version control of source code with software like git \[1\]. We aim to bring similar concepts to mobile app development.

Applications that allow users to collaborate on data on a central server are in widespread use. Popular examples are document authoring tools like Google Docs \[2\], project col- laboration apps like Basecamp \[3\] or Trello \[4\] or even large scale collaboration projects like Wikipedia \[5\].

The traditional architecture of collaborative applications follows a client-server model where the server hosts the entire application logic and persistence. Users access the application through a thin client, most commonly a web browser. The browser only has to display user interfaces that are pre-rendered by the server.This model works well when using desktop computers with a realiable, high-speed connection to the server.

Rising expectations on the user experience drove developers to increasingly move ap- plication logic to the client. Initially this has only been the logic required to render user interfaces. The server still hosted most of the application logic to pre-compute all relevant data for the client. Moving the interface rendering to the client reduces the amount of data that has to be transferred and makes the application behave more responsive.

The widespread adoption of mobile devices forces developers to re-think their architec- ture again. Users can now carry their devices with them and expect their applications to work outside their home or office network. Applications therefore have to work with limited mobile Internet access or often no access at all.

The only way to support this is by moving more of the application logic to the client and by replicating data for offline use. The clients are now not only responsible for rendering interfaces but also implement most of the application logic themselves.The new architecture comes at a high price - the additional client logic and persistence adds a lot of complexity. While in the server-centric model developers only had to maintain a single technology set, they now face different technologies on each platform they aim to support with a fat client.The ability to use the application offline requires an entire new layer of application logic to manage the propagation and merging of changes and to resolve conflicts. The only responsiblity of the server in this model is the propagation of data between clients.

Most users today carry a notebook, a smartphone and maybe even a tablet computer with them. They often want to work with the same data on different devices. Apps need to support workflows like adding some items to a Todo-Manager on a notebook and sub- sequently reviewing them on a smartphone. This implies that even simple applications that are meant for single-users have to aquire collaborative features. A single-user with multiple devices is from a technical perspective effectively collaborating with himself. Today’s applications only achieve this through data synchronization between the de- vices and a central server. If the user is mobile and does not have a reliable Internet connection he is stuck with outdated data on his smartphone. This problem can only be resolved by supporting the direct synchronization between devices. The clients can now basically act as servers themselves and manage propagation of data to other clients.The actual server does not have to disappear in this model. But like the clients it is just another node on the network. The difference is that the server node is continuously connected to the Internet and can therefore play a useful role as a fallback.Note that this only describes the extreme scenario - in most real-world applications we will see a hybrid-architecture where clients can synchronize most data directly but the server still manages security or enforces other constraints.

Building such a distributed data synchronization engine including all relevant aspects is very complex and beyond the reach of a small team of app developers. As an example the software company “Cultured Code”, which develops one of the most popular task management apps, has spent more than two years developing a generic solution without success \[6\]. It is therefore way beyond the scope of this thesis to develop a generic end-to-end solution. As described in the next section we will focus on a specific set of objectives and use cases.

## 1.2 Objectives and Approach

This thesis aims to develop patterns and tools to make the development of offline capa- ble, collaborative apps more productive.Unique to our approach is that we focus on peer-to-peer data synchronization between mobile devices with a focus on realization with open web standards.Our goal is to bring the features of distributed version control to application data.

Our guiding objectives are:

- **Offline Availability**: Enable the operation of a collaborative app with frequent network partition.
- **Synchronization Protocol**: Develop a protocol for efficient synchronization of changed data directly between devices.
- **Application Integration**: Abstract the synchronization logic to be as unintrusive as possible to an application.

A collaborative app that has to function with an unreliable network connection implies that we cannot rely on the traditional thin client model. We have to think about ways to make both data and logic available offline. The focus of this thesis is on managing offline availability and synchronization of data between collaborating devices.Being able to synchronize data directly between devices forces us to develop a distributed architecture.

Efficient synchronization means that we aim to minimize the amount of redundant data sent between devices. We have to implement methods to identify changes in application data.

Combined with the requirement to be unintrusive we exclude solutions that require the application to explicitly track changes in its code. The identification of data changes should be decoupled from the main application logic. This ensures that an upgrade of traditional applications requires minimal effort.

We will refine this set of requirements by breaking down common use cases and the evaluation of CouchDB, which is an existing solutions that supports offline-capable applications.

Otherwise important questions which are out of scope of this thesis are:

- **Security**: How can we manage access rights and encryption in a distributed architecture?
- **Device Discovery**: How can we discover devices in a network to collaborate with?
- **Data Transmission**: How is the data propagated among devices on a technical level?

## 1.3 Structure of the Thesis

The next chapter starts by describing related work and the necessary background that enables us to develop the synchronization solution.

Developing generic software that is not derived from concrete examples is doomed to fail. Chapter 3 therefore defines a concrete application scenario.The scenario is used to derive the necessary requirements in Chapter 4, which will be used to evaluate our solution.

With CouchDB we evaluate a potential existing solution to our scenario in Chapter 5. Chapter 6 finally introduces Histo, our own solution for distributed data synchronization. A discussion of our technological choices, fullfillment of requirements and a conclusion follows in the final chapter.