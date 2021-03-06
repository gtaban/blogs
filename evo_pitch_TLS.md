Hi all,

I’d like to pitch some of the ideas that have been discussed in the server working group around security. To get more information on the server API group and its goals, see https://swift.org/server-apis/. (TL;DR version is to come up with a set of foundational APIs that work cross-platform, on all platforms supported by Swift.) 

For security, we have divided the scope into SSL/TLS support and crypto support. Our first goal and the subject of this pitch is TLS. This current pitch is the result of various discussions that have taken place over the past several weeks/months and several projects by groups such as Vapor, IBM, Zewo, etc. 

Our plan is to start with the main ideas presented here and work on a Swift library that we prototype and iterate on before finalizing on a specific interface. Hopefully the ideas in this pitch are non-controversial, aside from the naming of the method and protocols (which is widely accepted as a `hard` problem).

# Problem

Since there is currently no standard Swift SSL/TLS library that is compatible on both Apple and Linux, Swift projects use their TLS library of choice (such as OpenSSL, LibreSSL, Security framework, etc). This results in:
- fragmentation of the space as well as incompatibility of project dependencies if more than one security package is needed by different modules (a project cannot have  both OpenSSL and LibreSSL in its dependency graph)
- insecurity (using an unpatched or deprecated library such as OpenSSL on macOS)
- unmaintainablity (using non-standard or non-native libraries)
- more complex code (using different APIs for each platform).

So we want to propose a standard set of protocols that define the behavior of the TLS service and how the application and the server and networking layers beneath it interact with the TLS service. What complicates this pitch is that the Swift in server space is new and none of the interfaces have yet been defined.

# Design goals

We came up with the following design goals in our solution:

- Provide a consistent and unified Swift interface so that the developer can write simple, cross-platform applications;
- Don't implement new crypto functionality and instead use existing functions in underlying libraries;
- Plug-n-play architecture which allows the developer to decide on underlying security library of choice, e.g., OpenSSL vs LibreSSL;
- Library should be agnostic of the transport mechanism (e.g., socket, etc), whilst allowing for both blocking and non-blocking connections;
- Developers should be able to use the same TLS library for both client and server applications.


# Proposal

The proposed solution basically defines a number of protocols for each interface:
- Transport management
- TLS management

A basic diagram that shows the relationship between the proposed protocols is shown below:
![alt text](https://raw.githubusercontent.com/gtaban/blogs/master/TLSServiceArchitecture.png "Architecture of TLSService modules")


The transport management protocol essentially would be a combination of a connection object (e.g., a socket pointer, a file descriptor, etc) and a connection type.

This is the connection object that gets passed to the implementation of the TLS service protocol, which also handles the read and write callbacks to the connection object.

The TLS service protocol would define a sets of methods that deal with TLS setup (certificates, server/client, etc), and TLS events (such as receiving data, encrypting and writing to connection object or reading from a connection object, decrypting and returning the data).
These methods are implemented by the TLS service which in turn uses its choice of underlying security library. As an example, the TLS service uses SecurityTransport library on Apple platform and OpenSSL on Linux. 

## How this would work

If an application requires TLS for its use case, it creates a TLS service object and configures it based on its requirements.

The application then passes the TLS service object to its lower level frameworks that deal with networking and communication. Each lower level framework maintains an optional instance variable of type TLS service protocol. If the optional variable exists, it is further passed down until it gets to the lowest level that deals with the Swift transport layer APIs (in the diagram above, this is the HTTP Management layer). When this layer creates the connection using the transport layer APIs, it assigns the TLS service object to the transport layer delegate. The Swift socket layer is then responsible for calling the TLS service protocol methods that handle the TLS functionality at the appropriate times. 


# Source Compatibility:

As mentioned, the Swift in server space is still new and the interfaces are currently under discussion. Although the proposed protocols are designed to be as non-breaking as possible, they do place several assumptions on the rest of the transport/application stack. 

- The application layer must import and instantiate a TLS service object which implements the TLS service protocol if it wants to enable TLS service.
- Every framework layer above the transport management layer but below the application layer (which includes HTTP and server frameworks) has an optional object that implements the TLS service protocol. If the TLS object exists, it is passed down to each layer below.
- The HTTP layer which sets up the transport management layer assigns the TLS object to the transport management delegate if the object exists. 
- The transport management layer which sets up the I/O communication implements the transport management protocol.
- The transport management layer which sets up the I/O communication and deals with the low level C system I/O, calls the appropriate TLS service protocol methods whenever I/O data needs to be secured.
- The long term goal for the location of the TLS service protocol is within the Foundation framework. In the short term, the protocol can live within the transport management framework.


