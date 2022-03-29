# KorAP-Docker

KorAP consists of several components,
but they can easily be installed together using
[Docker](https://www.docker.com/).
This repository contains a recipe to install all
components needed to run KorAP on a local machine
with a single command.

In addition, all relevant tools are installed and
made available that are necessary for data conversion
and indexing of corpora in the widely used TEI-P5/i5
format for KorAP.
For different options of the tools we refer to the
respective repositories.

## Requirements

Install [docker](https://www.docker.com/) and [docker compose](https://github.com/docker/compose).

## Starting

To download, intialize and run KorAP pointing to a certain directory index
(in this example `index` in the local directory), run

```shell
$ INDEX=./index docker-compose up
```

This will make the frontend be available at
`localhost:64543`.


## Corpus Conversion

Depending on the corpus data to be indexed, it must first be converted.
In the case of a conversion from TEI p5/i5 format, the tools
required for this have already been installed with the above command.

In the following we assume that an i5 file `mycorpus.i5.xml` is
located in the local folder.

The command ...

```shell
$ docker run --rm -v ${PWD}:/data korap/kalamar tei2korapxml --input /data/mycorpus.i5.xml > mycorpus.zip
```

... will convert the i5 file into a KorAP-XML file using
[tei2korapxml](https://github.com/KorAP/KorAP-XML-TEI).

To convert the KorAP-XML archive in a second step
into individual Krill JSON, the following command ...

```shell
$ mkdir json
$ docker run --rm -u root \
  -v ${PWD}/:/kalamar/data/ korap/kalamar korapxml2krill archive \
  -z -j -1 -i /kalamar/data/mycorpus.zip -o ./data/json/
```

... will use [korapxml2krill](https://github.com/KorAP/KorAP-XML-Krill).

Depending on how the source data is designed, different parameters must be specified for the conversion.


## Index Creation

[Krill](https://github.com/KorAP/Krill)'s indexer tool can now
be used to index the json files:

```shell
$ mkdir index
$ docker run -u root --rm -v ${PWD}/:/data/ korap/kustvakt \
  Krill-Indexer.jar -c kustvakt-lite.conf \
  -i /data/json -o /data/index/
```
