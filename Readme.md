# KorAP-Docker

The [KorAP Corpus Analysis Platform](http://korap.ids-mannheim.de/)
consists of several independent components,
but they can easily be installed together using
[Docker](https://www.docker.com/).
This repository contains a recipe to install all
components needed to run KorAP on a local machine
with a single command.

In addition, all relevant tools are installed and
made available that are necessary for data conversion
and indexing of corpora in the widely used TEI-P5
([I5](https://www.ids-mannheim.de/en/digspra/corpus-linguistics/projects/corpus-development/ids-text-model/))
format for KorAP.
For different options of the tools we refer to the
respective repositories.


## Requirements

Install [docker](https://www.docker.com/) and
[docker compose](https://github.com/docker/compose) (>= v2).

## Starting

To get KorAP running, an index is required.
For testing, there is a test index available as a docker image. Just run

```shell
INDEX='example-index' docker-compose -p korap --profile=lite --profile=example up
```

to start the example image and the service with Linux
(See [here](#Windows) for more information on Windows).

Otherwise it's possible to download the sample index provided by
[Kustvakt](https://github.com/KorAP/Kustvakt/tree/master/sample-index).
To download, intialize and run KorAP pointing to that index folder
(in this example stored in the `index` folder in the local directory),
run

```shell
INDEX=./index docker-compose -p korap --profile=lite up
```

This will make the frontend be available at
`localhost:64543`.

To use your own index, please follow the instructions
on [Corpus Conversion](#corpus-conversion) first.

To run the service with a user management system, create a file `$(pwd)/data/kalamar.production.conf` containing the following configuration:

```perl
{
    Kalamar => {
        plugins  => ['Auth']
    },
    'Kalamar-Auth' => {
        client_file => '/kalamar/super_client_info'
    }
}
```


Then start the service with

```shell
INDEX=./index docker-compose -p korap --profile=full up
```

Login with `user1` and `password1`. To change authentication settings, see the `/kusvakt/ldap` folder inside the docker container and Kustvakt's [LDAP Settings Wiki](https://github.com/KorAP/Kustvakt/wiki/LDAP-Setting) for documentation.

## Corpus Conversion

In order to create an index based on existing
corpus data, some conversion steps are usually
necessary.
In the case of a conversion from TEI P5
([I5](https://www.ids-mannheim.de/en/digspra/corpus-linguistics/projects/corpus-development/ids-text-model/)) format,
the tools required for this have already been installed
with the command above.

In the following we take the open part of the
[Dortmunder Chatkorpus 2.2](https://www.uni-due.de/germanistik/chatkorpus/)
(Beißwenger & Storrer 2008) as an example to build an index.

The file is located at `example/dck-part1.i5.xml`.

The command ...

```shell
docker run --rm \
  -v ${PWD}/example:/data:z korap/kalamar:latest-conv \
  tei2korapxml \
  --inline-tokens '!cmc#morpho' \
  --input /data/dck-part1.i5.xml > dck.zip
```

... will convert the i5 file into a
[KorAP-XML](https://github.com/KorAP/KorAP-XML-Krill#about-korap-xml)
file using
[tei2korapxml](https://github.com/KorAP/KorAP-XML-TEI).

This format is designed to add further arbitrary annotations
to the primary data. In this example, however, we will stick
with the inline annotations that the example corpus already
contains and will make available later under the label `cmc`.

To convert the KorAP-XML archive in a second step
into individual [Krill](https://github.com/KorAP/Krill) compatible
JSON files, the following command ...

```shell
mkdir json
```

```shell
docker run --rm -u root \
  -v ${PWD}:/kalamar/data:z korap/kalamar:latest-conv\
  korapxml2krill archive \
  -z \
  -i /kalamar/data/dck.zip \
  --jobs -1 \
  --token 'cmc#morpho' \
  --base-paragraphs 'DeReKo#Structure' \
  --base-sentences 'DeReKo#Structure' \
  -o ./data/json/
```

... will use [korapxml2krill](https://github.com/KorAP/KorAP-XML-Krill).

Depending on how the source data is designed,
different parameters must be specified for the conversion.

Here, the inline token annotation is used as the basis for
word tokenization, and the included document structure is 
used for default annotation of sentence and paragraph boundaries.


## Index Creation

[Krill](https://github.com/KorAP/Krill)'s indexer tool can now
be used to index the JSON files:

```shell
mkdir index
```

```shell
docker run -u root --rm -v ${PWD}:/data:z korap/kustvakt \
  Krill-Indexer.jar -c /kustvakt/kustvakt-lite.conf \
  -i /data/json -o /data/index/
```

After that, the index can be loaded with the aforementioned
call and is searchable via the browser.

## Windows

Windows with Powershell requires environment variables to pass in a different way.
In addition the `PWD` variable is not set beforehand. To run, e.g., the KorAP one-liner
with Windows, you have to start

```powershell
$env:INDEX='example-index'; $env:PWD='.'; docker-compose -p korap --profile=lite --profile=example up

```

## Development and License

**Authors**: [Nils Diewald](https://www.nils-diewald.de/), Harald Lüngen, Marc Kupietz

Copyright (c) 2022-2024, [IDS Mannheim](https://www.ids-mannheim.de/), Germany

KorAP-Docker is published under the BSD-2 License.

The example corpus corresponds to the *release part* of the
[Dortmunder Chatkorpus 2.2](https://www.uni-due.de/germanistik/chatkorpus/)
as prepared by
[DeReKo](https://www.ids-mannheim.de/digspra/kl/projekte/korpora/).
The corpus is released under the [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) License.
Legal restrictions may arise from data protection legislation.


## Bibliography

Beißwenger, Michael / Storrer, Angelika (2008):
Corpora of Computer-Mediated Communication.
In: Anke Lüdeling & Merja Kytö (Eds): *Corpus Linguistics. An International Handbook.*
Volume 1. Berlin. New York (Handbooks of Linguistics and Communication Science 29.1),
pp. 292--308.
