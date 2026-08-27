# Execution Environment Vocabulary

A small RDF vocabulary that specialises `schema:RuntimePlatform` for common
scientific-software execution environments.

The vocabulary complements [CodeMeta 3](https://w3id.org/codemeta/3.0) and the
[Software Types profile](https://w3id.org/software-types). It introduces no new
properties: CodeMeta's existing top-level `runtimePlatform` property links a
software record to instances of the classes defined here.

## Model

```text
schema:SoftwareSourceCode
  schema:runtimePlatform
    schema:RuntimePlatform
      specialised by this vocabulary

schema:SoftwareSourceCode
  codemeta:isSourceCodeOf
    schema:SoftwareApplication
      classified by Software Types
```

The two branches have different purposes:

- `runtimePlatform` describes where or how the software can run.
- `isSourceCodeOf` describes the runnable products built from the source.

## Namespace

```text
https://eosc-data-commons.github.io/execution-environment/
```

Suggested prefix:

```turtle
@prefix execenv: <https://eosc-data-commons.github.io/execution-environment/> .
```

The w3id.org redirects must be registered before these identifiers are
published.

## Files

- `execution-environment.jsonld` — vocabulary in JSON-LD
- `execution-environment.ttl` — vocabulary in Turtle
- `context.jsonld` — compact JSON-LD context
- `examples/` — CodeMeta 3 examples

## Classes

```text
schema:RuntimePlatform
├── BinderEnvironment
├── ContainerEnvironment
│   ├── DockerEnvironment
│   ├── OCIEnvironment
│   └── ApptainerEnvironment
├── GalaxyEnvironment
├── CondaEnvironment
├── NixEnvironment
├── VirtualMachineEnvironment
└── HPCEnvironment
    └── SlurmEnvironment
```

The classes are not disjoint. One runtime-platform node can have multiple
types. For example, a single HPC environment can be classified as a
`SlurmEnvironment`, `CondaEnvironment` and `ApptainerEnvironment` when all
three technologies jointly constitute the environment.

Workflow languages such as CWL, WDL, Nextflow and Snakemake are not runtime
platforms. A workflow may instead declare a Docker, OCI, Conda, Apptainer,
Slurm or another runtime platform.

## CodeMeta context

```json
{
  "@context": [
    "https://w3id.org/codemeta/3.0",
    "https://w3id.org/software-types",
    "https://eosc-data-commons.github.io/execution-environment/context.jsonld",
    {
      "schema": "https://schema.org/"
    }
  ]
}
```

## One composite runtime platform

`runtimePlatform` is represented as an array. This example contains one
platform node that has three types because Slurm, Conda and Apptainer are
required together:

```json
{
  "@type": "SoftwareSourceCode",
  "name": "Example HPC application",
  "runtimePlatform": [
    {
      "@id": "#hpc-environment",
      "@type": [
        "SlurmEnvironment",
        "CondaEnvironment",
        "ApptainerEnvironment"
      ],
      "schema:hasPart": [
        { "@id": "https://example.org/job.sbatch" },
        { "@id": "https://example.org/environment.yml" },
        { "@id": "https://example.org/container.def" }
      ]
    }
  ]
}
```

## Multiple alternative runtime platforms

Several objects in the array describe distinct supported platforms:

```json
{
  "@type": "SoftwareSourceCode",
  "runtimePlatform": [
    {
      "@id": "#conda-environment",
      "@type": "CondaEnvironment"
    },
    {
      "@id": "#docker-environment",
      "@type": ["DockerEnvironment", "OCIEnvironment"]
    },
    {
      "@id": "#slurm-environment",
      "@type": "SlurmEnvironment"
    }
  ]
}
```

## Examples

- `examples/binder.jsonld`
- `examples/docker.jsonld`
- `examples/galaxy.jsonld`
- `examples/cwl.jsonld`
- `examples/slurm.jsonld`
- `examples/multiple-environments.jsonld`

Every example uses CodeMeta 3 and places `runtimePlatform` on the top-level
`SoftwareSourceCode`. `isSourceCodeOf` is used separately for the runnable
product.

## CodeMeta compatibility

CodeMeta 3 currently documents `runtimePlatform` values as text. Current
Schema.org additionally permits `schema:RuntimePlatform` objects. These
examples therefore constitute CodeMeta 3 records extended with structured
Schema.org runtime-platform values.

For consumers limited to CodeMeta's text representation, use:

```json
{
  "runtimePlatform": ["Slurm", "Conda", "Apptainer"]
}
```

The text form is less machine-actionable because it does not carry typed,
dereferenceable environment classes or links to environment definitions.

## Local validation

Validate JSON syntax:

```bash
jq empty execution-environment.jsonld context.jsonld examples/*.jsonld
```

Parse the RDF serialisations with RDFLib:

```bash
python -m pip install rdflib

python - <<'PY'
from rdflib import Graph

jsonld = Graph().parse("execution-environment.jsonld", format="json-ld")
turtle = Graph().parse("execution-environment.ttl", format="turtle")

assert set(jsonld) == set(turtle)
print(f"Validated {len(turtle)} triples")
PY
```

The examples reference the intended public context. Until the w3id.org
redirects are active, replace that URL with `../context.jsonld` for local RDF
processing.

## Scope

The vocabulary intentionally defines classes only. It does not introduce terms
for commands, entry points, providers, workflow languages, resource
requirements, detection confidence, provenance, or input/output data.

## Licence

Apache License 2.0. See `LICENSE`.
