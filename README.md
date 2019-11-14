
# Usage Examples for FhirProto
This repository contains examples of how to use the <strong>FhirProto</strong> platform at [github.com/google/fhir](github.com/google/fhir). This repo contains a <code>generate-synthea.<span></span>sh</code> script for using [Synthea](https://github.com/synthetichealth/synthea) to create a synthetic FHIR JSON dataset, and then shows some examples of parsing, printing, validating, profiling and querying. Some of these examples are left intentionally incomplete, to leave exercises to go along with this guide.
## Setting Up Bazel

FhirProto uses [Bazel](https://bazel.build/) as it’s dependency management/build tool. This is a declarative build system used by Google, Tensorflow, and many others. Installation is pretty simple: follow the steps [here](https://docs.bazel.build/versions/master/install.html) to download and run the install script. Pro-tip: make sure not to drop the `--user` flag when running the script.
## Setting Up the Example Repository

Next, we’ll clone the example repository into git directory. If you don’t have a directory you already use for git code, `~/git` is perfectly reasonable:
```
mkdir ~/git
cd ~/git
```
Then, clone this repo using

```
git clone https://github.com/google/fhir-example.git
cd fhir-example
```
  
Next, we’ll generate a synthetic FHIR JSON dataset, using [Synthea](https://github.com/synthetichealth/synthea), via the `generate-synthea.sh` script. We’ll need a workspace directory for this dataset - here we’ll use `~/fhirdata/`

```
WORKSPACE=~/fhirdata
./generate-synthea.sh $WORKSPACE
```
This will create two data directories:

-   `$WORKSPACE/bundles/` contains 1000 patient bundles, each in its own JSON file
-   `$WORKSPACE/ndjson/` contains the output of running [SplitBundle](https://github.com/google/fhir/blob/master/java/src/main/java/com/google/fhir/examples/SplitBundleMain.java) on each of these bundles.
    

SplitBundle reads all the generated bundles, and generates an NDJSON file for each resource type present in the bundles. For instance, `Patient.fhir.ndjson` contains an NDJSON where each line is a JSON patient record found in the bundles, and `Observation.fhir.ndjson` contains a line for each observation found in the bundles. In addition, it generates two files for use with [Analytic SQL-on-FHIR](https://github.com/FHIR/sql-on-fhir/blob/master/sql-on-fhir.md): a `Patient.schema.json` file containing the analytic schema for each resource, and a `Patient.analytic.ndjson` file containing the resources printing according to the analytic schema. For more on this, see the <strong>Analytic Printing and BigQuery</strong> section.

At this point, we can validate that our bazel environment is set up correctly and our dataset is generated by running a simple test example:
```
bazel build //cc:ParsePatients
bazel-bin/cc/ParsePatient $WORKSPACE
```

This should parse all 1000 patients we generated into <strong>FhirProto</strong>, and print one out as an example.
## Add Proto Generation scripts to your bin
Finally, generating custom profiles and protos makes use of a couple of scripts defined by the FhirProto library. To add these to your `bin`, run
```
wget -O ~/bin/generate_protos_utils.sh https://raw.githubusercontent.com/google/fhir/v0.5.0/bazel/generate_protos_utils.sh && \
    wget -O ~/bin/generate_protos.sh https://raw.githubusercontent.com/google/fhir/v0.5.0/bazel/generate_protos.sh && \
    wget -O ~/bin/generate_definitions_and_protos.sh https://raw.githubusercontent.com/google/fhir/v0.5.0/bazel/generate_definitions_and_protos.sh && \
    chmod +x ~/bin/generate_protos.sh && chmod +x ~/bin/generate_definitions_and_protos.sh
```
