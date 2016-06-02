```
                      ___           __  __             
                     |_ _|__ _  ___|  \/  | __ _ _ ___ 
                      | |/ _` |/ __| |\/| |/ _` | '_  \ 
                      | | (_| | (__| |  | | (_| | | | |
                     |___\__,_|\___|_|  |_|\__,_|_| |_|
                                     

```

---

**Note:**: *This tool is under development and not yet released. The documentation is still under construction and incomplete.*

---

IacMan (Infrastructure as Code Management) is a light weight toolset to
describe a complete deployment landscape as a (possibly composed) source
project with a common versioning. It acts on a basic filesystem layout and
supports life cycle operations for the described
landscape based on self-responsible deployment components using tools like
bosh, terraform, cloud foundry or comparable deployment tools.


Do some of the following problems sound familar to you?

- Your landscape consist of multiple interlinked deployments.
- Your landscape contains multiple instances of the same kind of deployment.
  (for example multiple bosh or separately deployed postgres database instances)
- You have to maintain multiple instances of the the same basic landscape.
  (for example a test and a productive landscape)
- You have to _transport_ changes from one landscape to another.
- You have to manage the alignment of multiple separately developed components
  in your landscape.
- You have to support multiple IAAS layers for your landscape.
- You have to describe deliverable versions of your landscape that can
  be installed and upgraded in foreign target environments.
- You have to deal with different kinds of deployment environments, like bosh,
  terraform and/or cloud foundry.

If at least one of the above statements matches your problems, it could be
worth to take a look at what _IacMan_ can do for you.

## Motivation

Originally this toolset was intended for maintaining a landscape of bosh
deployments. Bosh is a tool to handle the life cycle of applications
(like cloud foundry)
consisting of various jobs distributed over multiple VMs onto an IAAS 
environment. Such a deployment is based on a set of _releases_ that are
bound together by a yaml based deployment manifest  that is handed over to
bosh. While bosh is able to handle the wiring of various software releases
to for a dedicted application instance it is not able to handle the wriring
of multiple such deployments. Basically all those deployments could be
bundled together into a single deployment manifest, but this is not really
handy, because a deployment is always handled as a whole by bosh.

Therefore missing was a toolset, that was able to describe a composition of
deployments and their wiring to form a complete landscape.
Another limitation of bosh is its scope, it handles only
the deployment of VMs and their software, but requires a preset IAAS
environemt with all the neede networks, routers, load balancing and security
groups. It would be very helpful to have an environment able to this, also.
If bosh is used, for example, to maintain a cloud foundry landscape, in
addition another kind of application should be covered, cloud fonndry based
applications.

Here yacman enters the scene. It is able to describe deployment landscapes
consisting of various deployment types, the wiring of the involved component
and their deployment.

## Overview

Iacman is a shell and spiff++ lightweight toolset for describing and
maintaining deployment landscapes consistingof multiple deployments roughly
based on the idea behind bosh and bosh workspace. The central element is the
deployment that will be controlled a set of structured configuration values in
form of a yaml files. In contrast to bosh, that describes a single deployment
composed of software given by so-called releases, iacman focuses on the
composition and wiring of multiple deployments and strictly separates between
the software controlling the deployment and its configuration.

This separation leads to two related but clearly distinguished elements, the 
deployment component (or shortly component) and the deployment instance (or
shortly deployment):

- A _component_ is some kind of deplyoment _class_ describing the software to
  be deployes and the software controlling the deployment process including 
  the generation of the description/configuration of a dedicated deployment 
  based on a configuration context describing the concrete deployment instance.
- A _deployment_ represents a dedicated deployment as an instance of a component
  as a deployment class. It finally describes a set of configuration values
  that characterizes the concrete deplyoment. The deployment description
  basically consists of the formlized rules to build the the value set required
  by the component to handle a concrete deployment instance.

Both parts are bound together with the so-called _context_.

The _context_ is the generated set of configuration variables derived from
the deployment settings and passed to the component to finally handle the
deployment. It contains all the wiring and configuration information
configuring the variation points provided by the component to describe a
concrete deplyoment.

The _wiring_ describes the flow of derived configuration information
from one component to another. For example, if the deployment of a bosh
director should report monitoring information to a graphite enpoint the
director deployment must get information about the graphite endpoint, that is
typically provided by another deployment. This kind of wiring is done
in the deployment. Therefore a deployment may configure _dependencies_ to
other deployments. On the other side every deployment may provide _exports_
describing reusable configuration information used by other deployments to
build their deployment context. These exports build the contract of a deployment
that can be used by others. This is definately not the deployment manifest,
it is an explicitly designed set of configuration information, that should be
kept as stable as possible during the evolution of a deployment to clearly
decouple multiple deployments. Iacman analyses the dependencies of a
deployment and provides access to the appropriate export information as part
of the context.

The task of iacman now is to control these generation processes. The context
is generated based on formal settings of a deployment. Therefore the deployment
configures dependencies to other deployments and configuration stubs containing
rules and values to finally generate the context. Dependencies are resolved to
the export information of the referenced deployments.

The rule engine used to merge and evaluate all the involved rule and
configuration files is spiff++, an improved version of the well-known tool in
the bosh context. The framework gathers the relevant stub files for spiff
according to the actual landscape description, executed a merge and passes the
resulting context to the actions to be executed.

A component provides so-called _actions_ that finally implement component
specific procedures to fulfill certain tasks, like export creation, manifest
creation or the final deployment. All those actions are called with the 
aggregated context to provide the deployment specific configuration set.

The general process looks like follows:
```

  deployment <..dependencies.. deployment definition ....> component
      |                                  |                    |
      |                                  |                    |
      |                                  V                    |
      |                         gathered config stubs         |
      |                                  |                    |
      |                                  V                    V
      V
    exports  -----------------> generated context -------> actions ---> deployment
                                                              /\
                                                             /  \
                                                            |    |
                                                            V    V
                                                       exports  manifest  

```

## Basic Scenarios

### Singleton landscapes

### Separation between Landscape instance and landscape template

## The toolset

Iacman as toolset supports two usage scenarios:
- It provides a shell library environment for accessing the landscape model,
  generating the context and executing actions. This library can be used
  to implement own tools.
- It provides a Command Line Interface (`iac`) to interactively manage
  a deployment landscape.

### The Command Line Interface

The basic command to work with iacman is the command `iac`. It works in
any directory of a landscape filesystem structure. Therefore it automatically
identifies the actual position in the landcape, the element represented by this
location and the complete landscape structure.
 
The command offers various sub commands to work on the actual element, any
other element or the complete landscape.

The general syntax is

<center>`iac {<options>} <sub command> {<command arguments>}`</center>

Additionally there are several shortcuts of the form `iac<sub>` (for exaple iacls).

Any subcommand may be abbreviated (for example `manifest` -> `man`). The
following sub commands are supported:

- `ls [<dir>]`
  list the logical landscape structure or the specified sub structure.

- `show [[component|deployment|module|landscape] <element name>]`
  show details for a given element. If no element is specified the element
  represented by the current working directory is used. If no element type
  is specified the element, regardless of its type, with the given name is 
  shown. Thereby deployments are preferred.

- `directory [deployment|component|module|config|state|gen|landscape]`
  print the selected directory path of the given element in the actual
  landscape. This command can be used by any tool called in the landscape
  structure to determine dedicated locations.
  * `deployment`: the directory of the deployment definition
  * `config`: the directory of the deployment configuration in the landscape
  * `state`: the directory of the deployment state in the landscape
  * `gen`: the directory of the private generation directory of the deployment
  * `component`: the directory of the component (of the given deployment)
  * `module`: the root directory of the actual module
  * `landscape`: the root directory of the landscape
  The above keys may be abbreviated.

- `context [<deployment name>]`
  generate the context for the given deployment.

-  `manifest [<deplyoment name>]`
   generate the deployment manifest for the given deployment.

-  `exports [<deplyoment name>]`
   generate the export contract for the given deployment.

- `order [<deployment name>|all]`
   determine a valid deployment order for the given deployment(s).

-  `action <action name>`
   call named action for the given deployment. The deployment can be specified
   via the option `-d`.

-  `prepare [<deplyoment name>]`
   call prepare action for the given deployment.

-  `deploy [<deplyoment name>]`
   call deploy action for the given deployment.

Any non-existent sub command name will be interpreted as action name and `iacman`
tries to call the appropriate action


## Descriptive Elements

### Components

### Plugins

### Deployments

#### Deployment Definitions

#### Deployment Configurations

## Dependencies and Cycles
