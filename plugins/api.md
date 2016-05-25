
Plugins can be used to predefine behaviour for components of a dedicated
type.

## Plugin Definition

For every type there is a folder with the same name.

It might contain action implementations in the `actions` folder
and/or a flat `component.yml`.
The component declaration is merged (with spiff++) with the component 
declaration found in the dedicated component to generate the effective
declaration. Typically here the manifest file name is predefined.

Every component action might be defaulted by the plugin. The plugin's action
is only used if no explicit action is defined in the component or deployment.

## Actions

Every action (as usual) is always called via the iacman tool. It gets a first
parameter specifying the file containing the context data to be used for the
actual execution. There might be additional arguments provided by the original
command line.

Additionally the environment contains some settings that might be useful for
the execution of the action. They describe the deployment environment:

- `IAC_QUIET`: if non-empty additional output should be suppressed
- `IAC_DEPL_STATE`: directory intended to store the state of the actual deployment
- `IAC_DEPL_INST`: directory of the actual deployment instance
- `IAC_DEPL_GEN`: generation directory to be used for the actual deployment
- `IAC_DEPL_NAME`: name of the actual deployment
- `IAC_DEPL_COMP_NAME`: name of the component of the actual deployment
- `IAC_DEPL_COMP`: directory of the component of the actual deployment
- `IAC_LS_ROOT`: landscape root directory
- `IAC_ACTION_CREATE_MANIFEST`: effective action to create the manifest

There are several actions used or explicitly supported by the iacman tool.
- `prepare`: prepare the deployment. So explicit one-time actions required
             to deploy the actual deployment instace. For example: 
   * creating cerificates
   * creating keys
   * uploading software (for example bosh stemcells or releases)
- `create_exports`: create the export data for the actual deployment instance.
                    It contains all data provided by the deployment to be used
                    by other deployments.
- `create_manifest`: create the actual deploymenr manifest used to control
                     the deployment of the actual deployment instance. It
                     should incorporate the configuration provided by the 
                     actual deployment context.
- `deploy`: execute the deployment of the actual deployment instance.
            Do everything required to be done to provide a fully-functional
            deployment.

Additionally there might be other actions depending on the kind of component
or deployment. They all can be called via `iac action`.
