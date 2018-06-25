# FutureGateway
The FutureGateway (FG) consists of a set of software components able to build, or assist existing web portals, or other community oriented interfaces to become 'Science Gateways'. In accordance to the definition of Science Gateways, the FutureGateway allows the access to distributed computing resources such as Grid, Cloud and HPC, preserving security and ensuring users accountability.
The idea of the FutureGateway framework comes from many years of experience gained in supporting scientific users communities to exploit federated services, thus improve their research quality. In that period several key common factors have been identified among different comminity projects and they are listed below:

* Provide the most possible flexible way accessing the distributed computing services.
* Do not force the user community to adopt a particular technology especially those related to their final User Interfaces.
* Provide the simplest way to develop ScienceGateway applications.

The FG comes with a set of configurable setup scripts allowing the installation of the system on several operating system. The access to the distributed infrastructures is provided by custmomizable modules named ExecutorInterfaces (EIs). One of the exsisting EIs exploits the SAGA standard which can access to different middleware using a common set of API calls. There are many different implementations of SAGA and the FutureGateway structure allows to use any of them simply providing the proper EI.
The FutureGateway does not force adopters to use a particular kind of portal technology, this system could stay beside an existing portal or even assist a desktop or mobile application because it provides a set of REST APIs to interact with the distributed computing interface services. It also provides different ways to users Authentication  and Authorization mechanisms that can be customized, bypasssed or switched to a special service named Portal Token Validator that delegates user membership to a dedicated web portal endpoint.

