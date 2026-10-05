# ADR 016: L4 service affilitaion

| Status  | Proposed   |
|---------|------------|
| Date    | 2026-10-05 |

## Context and Problem Statement
To archive modularization that is proposed in [RFC 008](../rfc/008-platform-mesh-modularization.md) the L4 Boundary must be clearly drawn by determining layer affiliation of each component.
Without a clear distinction for each component we run into the risk not aligning the layer boundary with the actual goal of modularizing. Since the Portal is defined to be replaceable and optional, the boundary needs to be at the API Level that the Portal leverages to fetch and modify Data.

As this ADR aims to decide only about the Layer association of service that could be L4, other services will not be discussed. This should be done in separate ADRs for L3 and L2.

## Decision Drivers
- omittability without interrupting core functionality
- coupling to Authorization-Services especially when they're absent
- support of core functionality like ressource-/service-discovery
- direct coupling of current implementation to OpenMFP, Luigi or the microfrontend-UI-Pattern in general

## Considered Options
To evaluate the affiliation of a service, this sections considers layer-association with possible options:

### Portal
The Portal should obviously be part of L4 as it is deeply coupled von OpenMFP and just represents the default implementation of a solution to operate platform mesh via a graphical user interface

### IAM UI
The IAM UI is also deeply coupled with the microfrontend-pattern and just represents part of the current ui implementation for platform mesh.

### Marketplace UI
The Marketplace UI just as the IAM UI represents a microfrontend part of the current/default ui implementation.

### extension-manager-operator
TODO

### terminal-controller-manager
TODO

### kubernetes-graphql-gateway
The kubernetes-graphql-gateway is the essential way for any interaction with the ressources provided by service-providers via HTTP.
Since the RFC calls for replaceability rather than just optionality, the graphql-schema could be defining the strong API-Contract for other UI implementations

#### gateway as L4 service
While possible, the graphql-layer could be omitted fully when operating API-only. This would then require direct kubernetes-api calls via `kubectl` and remove the active service discovery as well as automatic api-schema generation that the graphql-gateway provides.

Pro's:
- most flexible way for different ui implementations
- no GraphQL requirement
- completely omittable when API-only via `kubectl` is the desired operating-mode

Con's:
- no clear API-contract
- no explorable HTTP-API-Playground
- ressource-/service-discovery needs to be implemented

#### gateway as L4 service with separation of listener as L1
This is essentially the same variant as the 'gateway as L4 service' with the exeption of the listener being explicitly seperated from the gateway itself.
In this case we lose some flexiblity in the ui implementation and partially regain the GraphQL requirement, but at the same time simplify/implement ressource-/service-discovery out of the box.

#### gateway as L1 service
In this case the gateway (and the listener) is always included as its part of platform-mesh's core. This allows the gateway to clearly define a string API-Contract, that UI implementations (including the default implementation) can leverage and clearly seperates the UI implementation as L4 from platform-mesh's Contracts that hold in all possible modular-operating models.

Pro's:
- easiest way to provide a clear API-Boundary
- flexible interaction with data-structures and ressources in general due to composable GraphQL-queries
- explorable HTTP-API-Playground
- ressource-/service-discovery out of the box

Con's:
- fixed commitment to GraphQL as the HTTP-Query-Language
- API-only deployments still provide the full GraphQL-Functionality even when it is not required

#### gateway as L1 service with option to disable
This is essentially the same variant as the 'gateway as L1 service' the the exeption that the service, even though being considered a L1-/Core-service, can be disabled when not needed. For example in API-only operation where direct interaction with `kubectl` is acceptable/desired. This does make an API-only deployment potentially more lightweight when the GraphQL-API is not needed, but at the same time adds complexity to deployments, breaks the API-Contract that L4 would depend on if deployed later and defies the clear affiliation of the service to L1, which in turn could mean that the gateway must define its own layer, that can be omitted/replaced, which would not align with RFC 008.

### virtual-workspaces
TODO

### iam-service
TODO

## Decision Outcome
effectively ommited services in modular Setups without L4:
- Portal (as in the current openMFP implementation)
    - clearly this is the default implementation and thus should be omittable and replaceable
- IAM UI
    - clearly this is the default implementation and thus should be omittable and replaceable
- Marketplace UI
    - clearly this is the default implementation and thus should be omittable and replaceable

Services that need to be modified to be clearly affiliated with L4 or L1:
- extension-manager-operator

UI-near Services that are **not** omitted and thus should be considered as L1-Services:
- terminal-controller-manager
- kubernetes-graphql-gateway
    - this should be a L1 service that fulfills the API-Contracts defined by the GraphQL-Schemas, it should not be able to be disabled so that the defined contract always holds
- virtual-workspaces
- iam-service
