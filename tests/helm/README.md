# Helm template tests

## Running the tests

```sh
make test
```

The tests are written in Go, which is the only requirement.
If you run these from here, be sure to update helm dependencies first by running

```sh
make -C ../../ helm-dependency-update
```

## Adding new golden file tests

Aside from normal Go tests, we have a number of golden file tests.
Golden file tests are automatically discovered - any file matching `*.input.yaml`
will be picked up as a golden file input, and the framework will expect a corresponding
`*.output.yaml` to exist in the same directory.

The files can be placed in any subdirectory, including nested ones. The only requirement is
that they should be placed in the same one.

### Generating the golden output file

Writing the output files by hand is painful. In most circumstances, if you have an input
file, you want to generate the output using Helm, and then verify it for correctness.

This can be done by running:

```bash
helm template deploy/helm/sumologic/ \
  --namespace sumologic \
  --set sumologic.accessId='accessId' \
  --set sumologic.accessKey='accessKey' \
  -f test_name.input.yaml \
  -s ${path_to_template} >test_name.output.yaml
```

Some aspects of the output files would change very frequently if stored literally.
For example, the Chart version appears as a label on all resources. To avoid updating
it with every release, we replace it in output templates.

You'll need to modify the output files in the following ways:

- replace all instances of the release name (`collection` by default) with `RELEASE-NAME`
- replace all instances of the chart version with `%CURRENT_CHART_VERSION%`
- replace all checksum annotation (`config/checksum`) values with `%CONFIG_CHECKSUM%`
