# pbpipeline-helloworld-resources

Example SMRT Link `bundle` extension of Pipeline resources used by smrtflow and pbsmrtpipe (0.44.3) in SMRT Link **3.2.0.187627** (limited official release of 3.2.0)

The SMRT Link resource manifest is defined in `smrtlink-bundle-resources.json`

## Registered Core Resources

- Custom *Tool Contracts* are defined using Tool Contract interface defined in [pbcommand](https://github.com/PacificBiosciences/pbcommand). Examples exes are in `bin`. Then can be emitted to (static) JSON files using `$EXE --emit-tool-contract` or `$EXE emit-tool-contracts` if using the `registry` model.  
- Custom *Pipelines* are defined in `custom_pipelines.py` and can be emitted to a (static) JSON form by running `python custom_pipelines.py resolved-pipeline-templates`. 
- Custom *Pipeline Template Presets* are groupings of task options for a specific pipeline id.
- Custom *Chunk Operators* are XML mappings of how a task is scatter/gathered. See the [pbsmrtpipe docs](pbsmrtpipe.readthedocs.org) for details on the Chunking Framework

Other custom view rules and resources:

- Pipeline Template View Rules
- Report View Rules

## Requirements and Install

- [pbcommand](https://github.com/PacificBiosciences/pbcommand)
- [pbsmrtpipe](https://github.com/PacificBiosciences/pbsmrtpipe)

See the respective repos for install directions.


## Defining Custom Tool Contracts (i.e.,Tasks) and Pipelines

### Tool Contracts

Custom tasks can be defined using the PacBio Tool Contract interface. See [pbcommand](https://github.com/PacificBiosciences/pbcommand) repo for examples and the [pbcommand docs](http://pbcommand.readthedocs.io/) for more details.

Examples tasks are provided in `bin`, specifically, `hello-registry.py`. This is an example of the `quick` interface to register several tasks using a subparser-esque model. 

The static tool contract JSON files can be emitted using `bin/hello-registry.py emit-tool-contracts -o /path/to/output/tool-contracts`


### Defining Pipelines


Pipelines can be defined programmatically using python to encode the edges in the graph using a simple model using `bindings` (edges in graph).
 

Several example pipelines are defined in `custom_pipelines.py`. The static resolved pipeline templates can be emitted using `python custom_pipelines.py /path/to/resolved-pipeline-templates` (Note this requires setting up the ENV correctly. See the makefile targets `emit-pipelines` for details. This will emit static JSON files that can be loaded by both `pbsmrtpipe` and `SMRT Link Analysis Services`.


Please see [pbsmrtpipe docs]([pbsmrtpipe](http://pbsmrtpipe.readthedocs.io/en/master/)) for more detail on pipeline creation and pipeline bindings.


# Environment Setup

The registered resources that are loaded by the SMRT Link Services and pbsmrtpipe can be extended by setting environment variables.
 
- Pipelines: **PB_PIPELINE_TEMPLATE_DIR**
- Tool Contracts (JSON files in dir): **PB_TOOL_CONTRACT_DIR**
- Report View Rules (JSON files) in dir: **PB_RULES_REPORT_VIEW_DIR**
- Pipeline View Rules (XML files in dir): **PB_RULES_PIPELINE_VIEW_DIR**
- Chunk Operators (XML files in dir): **PB_CHUNK_OPERATOR_DIR**

See `setup-env.sh` for an explicit example; this can serve as a template
for other similar customizations (i.e. by setting `PROJ_DIR` differently).

## Environment Setup for Commandline (from pbsmrtpipe)

Before running the `pbsmrtpipe` exe to run a pipeline execution, run the `setup-env.sh` to set the necessary ENV variables. Once these are set you pipelines and tool contracts can accessible by showing the registered resources by running `pbsmrtpipe show-templates` and `pbsmrtpipe show-tasks`, respectively.

Note, for using in `pbsmrtpipe` from a SMRT Link install you *must* set `export SMRT_PYTHON_PASS_PATH_ENVVARS="YES"` for your custom `PATH` to be retained.

## Environment Setup and Testing for SMRT Link Services and UI

Create custom pipeline directory

####1. Stop services, if necessary

```
$SMRT_ROOT/admin/bin/services-stop
```

####2. Add custom pipeline to smrtlink install tree. 
The recommended approach is to put the custom pipeline directory into the `$SMRT_ROOT/current/addons/pipelines` directory, then add a 'current' symlink in that directory pointing to is. Something like this procedure:

```
cd $SMRT_ROOT
mkdir -p current/addons/pipelines
cp -a your-custom-pipeline-dir current/addons/pipelines
mv current/addons/pipelines/your-custom-pipeline-dir 01_current/addons/pipelines/your-custom-pipeline-dir
```
 
####3. Start services

```
$SMRT_ROOT/admin/bin/services-start
```

####4. Test that services recognize custom pipeline ids. 
Query the service with something like this:

```
curl http://SMRTLINK_HOST:SMRTLINK_SERVICES_PORT/secondary-analysis/resolved-pipeline-templates/CUSTOM_PIPELINE_ID
```

####5. Test that pbsmrtpipe is using the custom pipeline from the commandline.
Use something like this (expect zero exit status):

```
cd $SMRT_ROOT
current/smrtcmds/developer/bin/pbtestkit-multirunner --debug --nworkers 8 current/addons/pipelines/current/testkit-data/testkit.fofn
```

## Example procedure to install smrtlink and custom pipeline and run tests

```
mkdir testdir
cd testdir

# Install smrtlink (using a fairly generic install)
# For 4.0 and later (there are no restrictions on which ports you use, here we use 9110 and 9111):
smrtlink-*.run --batch --rootdir ./smrtlink --jmstype NONE --smrtlink-gui-port 9110 --smrtlink-services-port 9111

# For 3.2 (you must use services port 8081):
smrtlink-*.run --batch --rootdir ./smrtlink --jmstype NONE --smrtlink-gui-port 8080 --smrtlink-services-port 8081

# Install pbpipeline-helloworld-resources custom pipeline (using 'git clone')
mkdir -p ./smrtlink/current/addons/pipelines
cd ./smrtlink/current/addons/pipelines
git clone https://github.com/PacificBiosciences/pbpipeline-helloworld-resources
mv pbpipeline-helloworld-resources 01_pbpipeline-helloworld-resources
cd ../../../..


# Check to see if the bundled pipelines are registered by `pbsmrtpipe` exe.
# Look for 

./smrtlink/smrtcmds/bin/pbsmrtpipe show-templates | grep mk_hello_world


# Start services 
./smrtlink/admin/bin/services-start

# Note when restarting the services you might see warnings of `Failed to make request`. There is a retry mechanism on startup to make sure the services are running. The status of the services are accessible from `admin/bin/services-status`. 

# Test that services recognize all custom pipeline ids.  All these commands
# should return a valid json snipet describing the specified pipeline id.

# For 4.0 and later (note the custom 9111 SL Analysis services port).
curl http://localhost:9111/secondary-analysis/resolved-pipeline-templates/mk_hello_world.pipelines.mk_test1
curl http://localhost:9111/secondary-analysis/resolved-pipeline-templates/mk_hello_world.pipelines.mk_test2
curl http://localhost:9111/secondary-analysis/resolved-pipeline-templates/mk_hello_world.pipelines.mk_test3
curl http://localhost:9111/secondary-analysis/resolved-pipeline-templates/mk_hello_world.pipelines.dev_hello_subreadset

# For 3.2 (note the custom 8081 SL Analysis services port).
curl http://localhost:8081/secondary-analysis/resolved-pipeline-templates/mk_hello_world.pipelines.mk_test1
curl http://localhost:8081/secondary-analysis/resolved-pipeline-templates/mk_hello_world.pipelines.mk_test2
curl http://localhost:8081/secondary-analysis/resolved-pipeline-templates/mk_hello_world.pipelines.mk_test3
curl http://localhost:8081/secondary-analysis/resolved-pipeline-templates/mk_hello_world.pipelines.dev_hello_subreadset
```


## Testing Pipelines

The recommended model is to create a testkit job using a testkit.cfg file that references a small input dataset types. This integration test framework will help debug pipeline and tool contracts from the *both* the commandline as well as from the services.

Example testkit jobs are in `testkit-data`

From the commandline within your the directory that contains the testkit.cfg (e.g., dev_hello_subreadset_01)

`pbtestkit-runner testkit.cfg`

This will run `pbsmrtpipe` and to core/basic validation on the job output. See `--help` for more details.

For running all the testkit jobs in `testkit-data`:

`pbtestkit-multirunner testkit-data/testkit.fofn`


Similarly, testkit jobs can be run from the SMRT Link Services.
 
`pbtestkit-service-runner testkit.cfg --host=my-host --port=8081 --debug`

Note, this usecase is specifically for PacBio DataSet driven pipelines (i.e., pipelines driven from a SubreadSet, ReferenceSet, ...) and is *not* general to any pipeline.
 
Run all the service runnable teskit jobs in `testkit-data`:
 
`pbtestkit-service-multirunner testk-data/services-testkit.fofn --host=my-host --port=8081 --debug`


## SMRTLINK JSON Bundle Resources Spec


- PB_BUNDLE (*Required* String `[A-z0-9_\-]`) Must be a globally unique name of your bundle
- PB_PIPELINE_TEMPLATE_DIR (*Required* String) Path to resolved pipeline template(s) JSON files
- PB_PIPELINE_TEMPLATE_PRESET_DIR": (*Required* String) Path to resolved pipeline template preset(s) 
- PB_TOOL_CONTRACT_DIR (*Required* String) Path to tool contract(s) JSON files
- PB_RULES_REPORT_VIEW_DIR (*Required* String) Path to report view rules JSON files
- PB_RULES_PIPELINE_VIEW_DIR (*Required* String) Path to pipeline view rules JSON files
- PB_CHUNK_OPERATOR_DIR (*Required* String) Path to chunk operator XML files
- PB_PIPELINE_BIN_DIR (*Optional* String) Path to bin exes

All paths can be full or relative the `smrtlink-bundle-resources.json` file. Optional means the key must be provided but is null.

[See Example](https://github.com/PacificBiosciences/pbpipeline-helloworld-resources/blob/master/smrtlink-bundle-resources.json)


Note, `PB_CUSTOM_PIPELINE_PATH` is not directly supported at the commandline or in SMRT Link Analysis services.

Disclaimer
----------
THIS WEBSITE AND CONTENT AND ALL SITE-RELATED SERVICES, INCLUDING ANY DATA, ARE PROVIDED "AS IS," WITH ALL FAULTS, WITH NO REPRESENTATIONS OR WARRANTIES OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, ANY WARRANTIES OF MERCHANTABILITY, SATISFACTORY QUALITY, NON-INFRINGEMENT OR FITNESS FOR A PARTICULAR PURPOSE. YOU ASSUME TOTAL RESPONSIBILITY AND RISK FOR YOUR USE OF THIS SITE, ALL SITE-RELATED SERVICES, AND ANY THIRD PARTY WEBSITES OR APPLICATIONS. NO ORAL OR WRITTEN INFORMATION OR ADVICE SHALL CREATE A WARRANTY OF ANY KIND. ANY REFERENCES TO SPECIFIC PRODUCTS OR SERVICES ON THE WEBSITES DO NOT CONSTITUTE OR IMPLY A RECOMMENDATION OR ENDORSEMENT BY PACIFIC BIOSCIENCES.
