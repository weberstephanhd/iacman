Basic Guiding Principle: Separation Of Concerns
- Modularization
- Restrict locally required global knowledge as far a possible
- Enable information separation according to realms of resposibility

Basic layout is based on a filesystem structure only.
There are no assumption how thei struture is split into SCM repositories
(if at all), but provide dedicated isolated trees according to the 
realms of responsibility.

Achieved Features:
- no symbolic links
- structure agnostic component development
  - no local knowledge of central structures
  - explicit declarative logical (no filesystem paths) definition of inter-structure relations 
    (dependencies, context, ...)
  - component local interface 
  - complete deployment context provided to component api
- separation of deployment components and deployments
  - component is independent (agnostic) of its dedicated usages in producht contexts
  - product specific wiring of deployments separated from components part of a product
  - multiple deployments (with different wiring) for the same component
- component responsible for providing life-cycle operations
  - create_exports: used for wiring deployments (defined API for shared information offered by a deployment)
  - create_manifest: create deployment manifest used for deployment based on actual context of deployment
  - deploy: actually deploy it
  - prepare: do everything on infrastructure to be ready for deployment (release/stemcell uploads for bosh)
- composition of landscapes and products based on potentially multiple separately maintained parts
  - partitioning of the product into independently maintained deployment compositions
  - extending the landscape from the pure bosh core by adding a rich set of application
    will for sure require independent releases of parts of the complete product to support
    local responsibilities
- support multi platform deployments from a single source base

