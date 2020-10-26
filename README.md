# FutureGateway Framework

[![Documentation Status](https://readthedocs.org/projects/futuregateway-framework/badge/?version=latest)](https://futuregateway-framework.readthedocs.io/en/latest/?badge=latest)

The FutureGatewayFramework (FG) consists of a set of software components able to build, or assist existing web portals, or other community oriented interfaces to become 'Science Gateways'. In accordance to the definition of Science Gateways, the FutureGateway allows the access to distributed computing resources such as Grid, Cloud and HPC, preserving security and ensuring users accountability.
The idea of the FutureGateway Framework comes from many years of experience gained in supporting scientific users communities to exploit federated services, thus improve their research quality. In that period several key common factors have been identified among different comminity projects and they are listed below:

- Provide the most possible flexible way accessing the distributed computing services.
- Do not force the user community to adopt a particular technology especially those related to their final User Interfaces.
- Provide the simplest way to develop ScienceGateway applications.

The FutureGateway does not force adopters to use a particular kind of portal technology, this system could stay beside an existing portal or even assist a desktop or mobile application because it provides a set of REST APIs to interact with the distributed computing interface services. It also provides different ways to users Authentication and Authorization mechanisms that can be customized, bypasssed or switched to a special service named Portal Token Validator that delegates user membership to a dedicated web portal endpoint.

The FGF comes with a set of configurable setup scripts allowing the installation of the system on several operating system. The access to the distributed infrastructures is provided by custmomizable modules named ExecutorInterfaces (EIs).

## Executor Interfaces

- **Grid and Cloud Engine** The most generic _EI_ is a component that ueses the [Grid and Cloud engine][gnceng] a software libray which uses [JSAGA][jsaga] component to exploit the [SAGA][saga] standard to access different kind of middleware, using a common set of java API calls.

- **Tosca IDC** This _EI_ has been developed during the [indigo-dc][indigodc] project and it uses a set of REST API calls to interact with the INDIGO-Tosca ochestrator developed in the context of the same project.

[infn]: https://www.infn.it
[infnct]: https://www.ct.infn.it
[indigo-dc]: https://www.indigo-datacloud.eu
[eosc-hub]: https://www.eosc-hub.eu
[fgf]: https://github.com/FutureGatewayFramework
[fg]: https://github.com/indigo-dc/fgDocumentation
[inveniordmpm]: https://indico.cern.ch/event/854421/page/18559-general-information
[infnoar]: https://www.openaccessrepository.it
[gnceng]: https://csgf.readthedocs.io/en/latest/grid-and-cloud-engine/docs/
[jsaga]: http://software.in2p3.fr/jsaga/latest-release/
[saga]: https://www.ogf.org/documents/GFD.90.pdf
