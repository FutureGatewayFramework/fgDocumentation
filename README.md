# FutureGateway

The FutureGateway consists of a set of software components able to build, or assist existing web portals or other community oriented interfaces to become Science Gateways. In accordance to the definition of Science Gateways, the FutureGateway allow the access to distributed computing resources such as Grid, Cloud and HPC.
The idea of the FutureGateway comes from a four years experiences gained with a similar component named, Catania Science Gateway Framework. Both components have the same aim of building Science Gateways but the FutureGateway tries to overcome several limitations encountered by the first approach. In particular the following key points have been identified:

* Provide a more flexible way accessing the distributed computing services.
* Leave to the FutureGateway adopters the choice of the backward portal technology.
* Provide the most simple way to develop ScienceGateway applications.

The FutureGateway comes with a set of configurable setup scripts allowing the installation of the system on several operating system. The access to the distributed infrastructures exploits the SAGA standard which can access to different middleware using a common set of API calls. There are many different implementations of SAGA and the FutureGateway structure allows to use any of them simply providing the proper interface. It is also possible to use other systems beside JSAGA to deal with distributed infrastructures thanks to the adoption of Executor Interfaces.
The FutureGateway does not force adopters to use a particular kind of portal technology, this system could stay beside an existing portal or even assist a desktop or mobile applicationbecause it provides a set of REST APIs to interact with the distributed computing interface services. It also provides a baseline membership management including Authentication  and Authorization mechanisms that can be customized, bypasssed or switched to a special service named Portal Token Validator that delegates user membership to a dedicated web portal endpoint.

The FutureGateway is currently adopted and developed in the context of the INDIGO-datacloud project. Its PTV service is configured in order to involve INDIGO IAM system, while a dedicated executor interface have been developed in order to target the TOSCA orchestrator.



