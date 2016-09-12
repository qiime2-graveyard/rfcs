---
# ----------------------------------------------------------------------------
# This work is licensed under a Creative Commons Attribution 4.0 International
# License. To view a copy of this license, visit:
#
# http://creativecommons.org/licenses/by/4.0/
# ----------------------------------------------------------------------------
Feature name: tracking-provenance
Start date: 2016-09-06
Pull request: <leave empty>
Authors:
  - "@ebolyen"
Contributors:
  - "<leave empty>"
  - "<leave empty>"
  - ...
---

# Summary

The purpose of this RFC is to describe a new format for storing provenance in
the QIIME 2 archive format (`.qza`/`.qzv`) files. Currently the implementation
does not provide a robust implementation allowing the user to track the
provenance from initial import to a particular artifact. By using a directory
it is possible to store all provenance (including intermediate artifacts) and
associated metadata as concrete files. In particular this RFC describes a way
of creating a DAG representation without the use of soft/hard links which is
necessary create zip files in a cross-platform way.


# Glossary

- reproducibility: the ability to mimic a process
- replicability: the ability to exactly duplicate a process
- import: to construct an artifact from data external to the QIIME 2 framework
- provenance: the lineage of an artifact, e.g. identifying the steps taken and
  any intermediate artifacts, from first import to a particular artifact.
- DAG: [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph)
- UUID: [universally unique identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier)


# Motivation

Presently provenance can only describe the UUIDs of the parent artifacts. This
means in order to trace the provenance of any given artifact back to its first
imported artifact, one would need to retain all artifacts generated along the
way. This not only may be a large amount of data, but it limits the ability of
meta-analysis as any meta-analysis framework would need to either retain all
intermediate artifacts, or consider the artifacts of a given study to be
"imported", essentially acting as if there were no provenance.

If provenance instead retained **all** information about an artifact's lineage,
then a single `.qza` or `.qzv` file would be capable of reconstructing its own
pipeline. Furthermore this means that any derived artifacts could retain
accurate provenance by appending their own provenance to a copy of their
parent's.

Additionally, it is important to be able to distinguish potentially flawed
artifacts after creation. For example if there was a bug in one of the steps
that led to a given artifact, then that artifact and any descendants should be
recomputed. This implies we need an ability to track both the precise steps
used (including versions) and a way to track the *general* process taken. If
this was possible, then we can imagine a way to *reproduce* any artifact
automatically with the currently installed versions of software.


# Impact
The following groups are effected differently by the proposed changes:

#### QIIME 2 users

Existing artifacts will not be compatible with artifacts generated with the new
provenance machinery. It may be possible to construct the provenance proposed
from a collections of current artifacts' provenance, however supporting this
is not a priority given the instability of the archive format in general.

Additionally users of the "Artifact API" will observe a different `Provenance`
object. (Users which actually manipulate this object are considered
interface/platform developers for the purposes of this RFC.)


#### QIIME 2 plugin developers

This change will be transparent to plugin developers as the framework should be
the entity that connects provenance to an artifact. Future iterations may
expose an API to allow plugin developers to annotate additional metadata, but
it is expected that such an API would be backwards compatible.


#### QIIME 2 interface and platform developers

This is a backwards incompatible change and will require modifications to any
code which currently manipulate or view the `qiime.sdk:Provenance` type. It is
expected that the current implementation will be replaced entirely.

# Detailed design

There are two fundamental objects in describing the provenance of a given
artifact: artifacts and actions. As a graph, we can chose to represent
artifacts as nodes or edges (vice versa for actions). Because artifacts are
terminal results, we will chose artifacts to be represented as a node and
actions as an edge.

The provenance of an artifact (much like lineage) is modeled with a directed
acyclic graph (DAG). It can also be modeled as a tree, but this would require
duplication. For illustrative purposes, suppose we had four artifacts `A`, `B`,
`C`, and `D`. Consider the two equivalent representations for provenance of an
artifact named `D`, where nodes are considered artifacts and edges are actions:

```
DAG representation          Tree representation

        A                        A       A
       / \                        \     /
      B   C                        B   C
       \ /                          \ /
        D                            D
```


In the tree representation `A` must be duplicated in order to express the
provenance of `D` completely. One can imagine a sufficiently complex lineage
such that this duplication becomes problematic.

To avoid this we must have a way to alias `A` multiple times. This could be
accomplished with soft-links in a tree representation, but it isn't clear which
branch would be the source. Additionally soft-links are not supported on all
operating systems. Hard-links are not supported by the zip format. Instead we
will use a directory which will contain a flat listing of all ancestral
artifacts. This additionally avoids the path from becoming too deeply nested
when extracted.

## Filesystem view:
Below is a tree-view of a potential file-system implementation for describing
provenance:

```
provenance/
├── action.yml
├── execution.yml
├── identity.yml
├── metadata/
|   └── param_name.csv
└── artifacts/
    ├── 822af83c-073f-4d08-8689-faefac0cf528/
    │   ├── action.yml
    │   ├── execution.yml
    |   ├── identity.yml
    │   └── metadata/
    ├── cc4f1879-f725-48fe-8198-e67e8878eed0/
    │   ├── action.yml
    │   ├── execution.yml
    |   ├── identity.yml
    │   └── metadata/
    └── d15fa588-fdd2-4150-927a-5ed8afdaaa06/
        ├── action.yml
        ├── execution.yml
        ├── identity.yml
        └── metadata/
            ├── bar.csv
            └── foo.csv
```

#### provenance/
A directory dedicated to recording the provenance of the artifact
(as shown in the above filesystem tree). This would exist at the root level of
a `.qza`/`.qzv` archive.

#### action.yml
A YAML file which indicates what was run. This file is in principle concerned
only with *reproducibility*. In other words, runtime and other ephemeral
characteristics are not considered, only the intended action.

```yaml
type: method  # or "visualizer" (identical) or "import" (handled in next sect.)
plugin: q2-example-plugin
version: 2!2017.09.0+2.g1076c97
action: some_action
# How are citations represented here? They are directly related to the action,
# so this file seems like the appropriate place to put them.
inputs:
 - input_a: 822af83c-073f-4d08-8689-faefac0cf528
 - input_b: cc4f1879-f725-48fe-8198-e67e8878eed0
parameters:
 - param_name: !metadata param_name.csv
 - complex:
    a: 10
    b: 15
    c: 20
 - other:
    - 1
    - 10
    - 100
 - color: !color FF0000
```

In the case of `parameters` custom YAML tags are used so that an interpreter
does not need to load `q2-example-plugin` in order to determine the primitive
types used by `some_action`. In other words, it can identify that `param_name`
takes metadata (of some variety) and so it can find the values in the
`metadata/` directory (described below). Similarly it can determine that
`color` took a `Color` primitive type (even though it never loaded
`q2-example-plugin` to determine that). This is important as we shouldn't
expect the consumer of provenance to have the same installation as the producer
of the provenance (especially if you consider that provenance may span multiple
studies and installations).

Outputs are omitted as that represents (at least) the current artifact. Sibling
relationships can be determined by comparing either the provenance of
`action.yml` or the provenance of `execution.yml` depending on the strictness
of definition required. For example if you were interested in sibling-ship as
defined by being produced by the same parents (but at potentially different
times), then only the structure of the provenance (`action.yml`) is
interesting. If the definition depends on siblings being members of the same
execution event, then `execution.yml` (defined below) is more appropriate.

##### Handling Import

Importing represents the base-case of our provenance. QIIME 2 must rely on the
user to accurately describe what the data they have is. In an ideal world these
are limited to raw sequence reads, but in practice an artifact of *any* type
may be imported from arbitrary data.

When an artifact is imported for the first time, it isn't the result of an
*action* (as defined by the framework), but instead of a compilation of
transformers. Given some source data, they convert it to an artifact. In
principle it shouldn't matter which transformers were invoked, only the source
data.

```yaml
type: import
format: SomeKindOfFormat  # This is a stronger statement than just the manifest
manifest:
    - name: somefile.txt
      md5sum: 5273810cbc42c66bffd88cc442ef6519
    - name: relative/otherfile.txt
      md5sum: 95eeb5826aebb1aab33c0579787ae13e
```

By storing the md5sums of an import it becomes possible to demonstrate that a
given file is a member of the provenance for all derived artifacts. This only
verifies the identity of a file. To understand how it was used, the name of a
file should be considered in the context of the format. This means that if a
file changes name over time, we can still understand the context in which it
was used, independent of its current name.

The name is unnecessary for single file imports, how do we want to handle this?
There is potential to leak system information here as the name of the file
is not constrained by a directory format.

#### execution.yml
A YAML file which describes the execution context. This file is in principle
concerned with *replicability*. In other words, the exact runtime configuration
is relevant.

```yaml
# A UUID which represents an execution event, nothing more
uuid: 93062a51-bab2-4e27-9d8b-b86029450db0
runtime:
    # Do we care about milliseconds?
    start: <timestamp (seconds)>
    end: <timestamp (seconds)>
# Is it necessary to identify hosts? Different hosts may have different
# (hopefully compatible) versions.
transformers:
    # A singular `input` (matching `output`) is used in the case of import.
    inputs:
        input_a:
            - plugin: q2-some-plugin
              version: 2!2017.09.0
              from: ArtifactDirectoryFormat
              to: SomeIntermediateView
            - plugin: q2-other-plugin
              version: 2!2017.09.0
              from: SomeIntermediateView
              to: SomeOtherView
        input_b:
            - plugin: q2-other-plugin
              version: 2!2017.09.0
              from: AnotherDirectoryFormat
              to: AnotherView
    output:
        - plugin: q2-some-plugin
          version: 2!2017.09.0
          from: SomeView
          to: ArtifactDirectoryFormat

versions:
  python: 3.5.1
  qiime: 2!2017.09.0
  # <dump contents of pip freeze>

```

The precise contents of this file are less clear, but the goal is to describe
information that **should not** be necessary to reproduce an analysis. That
being said if there was a bug in one of the dependencies then the `versions` is
necessary to distinguish flawed artifacts after the fact. These versions serve
a different purpose than the plugin version in `action.yml`, which is used to
pinpoint a particular method which was invoked, in other words, the API
targeted.

#### identity.yml
This file contains the contents of `metadata.yml` from the artifact's root
directory. As of the time of this writing, that looks like:

```yaml
uuid: 0e61e6af-d49e-4aac-8173-23634f3b4c91
type: Some[SemanticType]
format: SomeDirectoryFormat
# Should there be a version here to indicate the version of archive being
# used? It may be possible to invoke different handlers for each node depending
# on version.
```

Provenance is omitted.

#### metadata/
A directory containing CSV-formatted files named according to the parameter
they correspond to. This directory may be empty, but it must be present.

**example.csv**
```csv
,column
id1, foo
id2, bar
id3, baz
```

What should the index be called? There are cases where there will be:
no columns, one column, many columns; but an index is always assumed to exist.
The current example does not use a name for the index.

#### artifacts/
A directory containing ancestral artifacts which are themselves directories
identified by UUID. This directory is only present in the root of the
provenance directory. The `inputs` fields of `action.yml` is assumed to
correspond to a directory in this directory.


### Creating new Provenance

To create a new artifact, the framework would take all input artifacts of an
action and perform the following steps:

1. merge contents of inputs' `provenance/artifacts/` directories. Duplicate
   UUIDs are assumed to be the same artifact.
2. for each input, create a new directory named by the respective artifact's
   UUIDs
    - copy contents into newly created directory, excluding the `artifacts`
      directory.
3. create `action.yml`, `execution.yml`, `identity.yml`, and `metadata/` in the
   root of `provenance/` for the new artifact as defined by the current action.


### Impact on the archive format
The current `metadata.yml` file in the root of an artifact's archive would no
longer contain a provenance field (there is now an entire directory for it
called `provenance/`).

Furthermore the `VERSION` file will need to be bumped to indicate an new
archive format version.


### `Provenance` object API
The implementation and API of the `Provenance` object is not defined by this
RFC, but adaptations should be made to it as necessary.


# How We Teach This

Ideally tools will be created to visualize the provenance of an artifact. A
workshop could then have an exercise in which users follow an analysis where
they view the provenance at each step. They could even exchange artifacts and
observe that the provenance exists whether they made the artifact or not.

Another good application of provenance would be to generate a "pipeline"
automatically. This could be easily demonstrated in a tutorial/workshop and
would quickly demonstrate the power of provenance in the artifacts.

It may be desired that the `provenance/` directory includes a `README.md` which
contains a description of the format as well (perhaps this finalized RFC?).


# Drawbacks

Existing artifacts will be incompatible with new artifacts.

This proposal defines a custom format for distributed provenance which could
require adaptation for other software. Other software which provides
distributed provenance is not known the author at the time of this writing.


# Alternatives

It would also be possible to store this data relationally (sqlite) or as a
single file (some DAG format). However that would require special
representations of metadata which would make consumption harder. Additionally
the format would no longer be self-descriptive.

Using a trivially recursive definition would be simpler to implement, however
it would cause excessive duplication. As an example, see the following which
does not contain duplication:

```
provenance/
├── artifact.yml
└── parents/
    └── 1b720f60-1f55-4868-82fc-fb45ef08922b/
        ├── artifact.yml
        └── parents/
            ├── 61e41163-22b4-4dc5-87b7-ef8b81e5f8a2/
            │   └── artifact.yml
            └── 8f70be04-75ca-4fc1-8deb-7c30cf69de3b/
                └── artifacts.yml
```

With only 4 nodes it is already excessively nested and hard to read. Any shared
provenance would only exacerbate this problem.

It may be possible to store artifacts as edges in the provenance, but it isn't
clear how terminal node would be handled in that case.


# Unresolved questions

How would non-framework metadata be encoded, for example suppose there was an
interface or even a plugin which wanted to encode additional (and potentially
round-trippable) information in an artifacts provenance?

Some ideas might include:

 * Having an `extras.yml` file or equivalent
 * Having an `extras/` directory or equivalent
 * Some combination of the above
 * Permitting files which are not otherwise reserved for a purpose
   (extra data by convention).
