<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. Skydipper API</a>
<ul>
<li><a href="#sec-1-1">1.1. Rationale</a></li>
<li><a href="#sec-1-2">1.2. Architecture overview</a>
<ul>
<li><a href="#sec-1-2-1">1.2.1. Dataset</a></li>
<li><a href="#sec-1-2-2">1.2.2. Metadata</a></li>
<li><a href="#sec-1-2-3">1.2.3. Query &amp; Converter</a></li>
<li><a href="#sec-1-2-4">1.2.4. Geostore</a></li>
</ul>
</li>
<li><a href="#sec-1-3">1.3. Microservice management</a>
<ul>
<li><a href="#sec-1-3-1">1.3.1. Microservice registration process</a></li>
<li><a href="#sec-1-3-2">1.3.2. Filtering</a></li>
<li><a href="#sec-1-3-3">1.3.3. Management API</a></li>
</ul>
</li>
<li><a href="#sec-1-4">1.4. External &amp; Managed services</a>
<ul>
<li><a href="#sec-1-4-1">1.4.1. Airflow</a></li>
<li><a href="#sec-1-4-2">1.4.2. CloudSQL</a></li>
<li><a href="#sec-1-4-3">1.4.3. Google Storage</a></li>
</ul>
</li>
<li><a href="#sec-1-5">1.5. Implementation Overview</a>
<ul>
<li><a href="#sec-1-5-1">1.5.1. Kubernetes</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</div>

# Skydipper API<a id="sec-1" name="sec-1"></a>

## Rationale<a id="sec-1-1" name="sec-1-1"></a>

Skydipper's API allows any user to ingest and process a number of
(open) datasets that are exposed through it. This is achieved with
a microservice architecture, where interfaces for different
datasets providers are registered to a centralized API gateway that
add different functionality to the software platform.

## Architecture overview<a id="sec-1-2" name="sec-1-2"></a>

The main point-of-entry to the Skydipper platform is its API
gateway, Control Tower. Control Tower is an open source project to
which Skydipper contributes, but it's also used in other software
platforms (e.g. Resource Watch API & PREP, by the World Resources
Institute, or API Highways). In addition to an API
router/gateway, Control Tower implements a management layer for
dealing with the configuration and lifecycle of the microservices
registered to it, as well as authentication and authorization of
requests.

Several microservices add some functionality already to the API:

### Dataset<a id="sec-1-2-1" name="sec-1-2-1"></a>

This microservice exposes the concept of 'dataset', a generic
entity that abstracts any source of data that is possible to use
in the Skydipper API. Actual implementations of dataset types will
need to expose a consistent interface that matches well with
\`dataset\` expects: a \`query\` endpoint, that receives ad-hoc SQL
queries; a \`fields\` endpoint, that provides information about the
dataset columns (for tabular datasets) and about the coverage axis
and dimensions(for raster-type datasets); plus a \`tiles\` endpoint,
that exposes slippy map tiles if the dataset where to have a
geographic representation (i.e. a map).

Several types of datasets are already implemented on the platform
or are in development:

1.  Google Earth Engine

    Google Earth Engine is a product by Google that exposes a
    multi-petabyte catalog of satellite imagery and other geospatial
    datasets, with a backend with capabilities to perform
    planetary-level analysis.

2.  Document adapter

    Allows for the upload and download of data in CSV and JSON
    format. Elastic Search backed.

3.  COG

    Cloud Optimized Geotiffs (COGs) are regular GeoTiff files (the
    most common open format used for raster data in GIS packages)
    that can be partially requested with HTTP GET range
    requests. This enables a cloud-optimized approach to dealing with
    raster data. Implementation is based in the COG driver by GDAL
    (Geospatial Data Abstraction Library), a currently-maintained
    library for dealing with geographical data in both raster and
    vector format.

4.  Carto

    Carto is a market leader managed geodatabase in the cloud.

### Metadata<a id="sec-1-2-2" name="sec-1-2-2"></a>

An important aspect of dealing with open data is dealing with its
metadata information, its integrity in time and being able to
properly reference datasets. The metadata microservice allows to
register metadata to particular dataset in several languages with
a common interface. Potentially, metadata can be attributed to
several other entities present in the Skydipper API.

### Query & Converter<a id="sec-1-2-3" name="sec-1-2-3"></a>

This pair of microservices exposes the query endpoint, to which
the dataset adapters have to conform, and provide the adapters
with a SQL abstract syntax tree so that data manipulation comes
from an unified interface.

### Geostore<a id="sec-1-2-4" name="sec-1-2-4"></a>

Geostore is a geoJSON database that in addition to storing
arbitrary (e.g. user uploaded, generated experimental designs)
vector layers following the GeoJSON specification, it can provide
structured data &#x2013; i.e. an endpoint providing country and region
administrative boundaries after the country iso codes.

## Microservice management<a id="sec-1-3" name="sec-1-3"></a>

### Microservice registration process<a id="sec-1-3-1" name="sec-1-3-1"></a>

Microservices that want to be exposed through the Skydipper API
need to expose an /api/info endpoint that returns a JSON object
with an array at its root named \`endpoints\`. Each element of the
array represents an endpoint to be exposed through Skydipper, done
with three elements: \`path\`, that represents the final exposed
path; a \`method\` that will represent the HTTP method that this
endpoint accepts (more than one method can be provided by
registering the same endpoint with another method), and a
\`redirect\` object, its attributes being the \`path\` and \`method\` of
the internal microservice.

### Filtering<a id="sec-1-3-2" name="sec-1-3-2"></a>

Filters can be supplied in the \`info\` endpoint to disambiguate
between microservices that expose the same endpoint.

### Management API<a id="sec-1-3-3" name="sec-1-3-3"></a>

There are several endpoints to check the state of the
microservices integrated in the API.

## External & Managed services<a id="sec-1-4" name="sec-1-4"></a>

### Airflow<a id="sec-1-4-1" name="sec-1-4-1"></a>

Airflow is a python-oriented ETL tool. Initially a development by
the data engineering team of Airbnb, its stewardship is now of the
Apache Foundation. It's under active development and is currently
used by many technology companies. It's purpose is to orchestrate
data workflows, and does so by providing the user with the
capabilities to run arbitrary code (python-based or otherwise) in
many computing backends. This code is orchestrated by means of
configuration-as-code files that specify dependencies as Directed
Acyclic Graphs (DAGs) of tasks.

Google offers since recently a managed version of Airflow in its
Google Cloud platform, integrated with its Storage and Kubernetes
products. It's here where much of the ML model training logic will
be expressed: in conjunction with the CloudSQL (see below)
service, a queueing system will be implemented that coordinates
data collection from experimental designs, performs the necessary
dataprep, compiles data into packages, and trains and deploys ML
models

### CloudSQL<a id="sec-1-4-2" name="sec-1-4-2"></a>

CloudSQL is the managed SQL offering by Google Cloud. It offers
fully managed instances of either MySQL and PostgreSQL. It offers
encryption for moving and at rest data; automated provisioning,
scaling and backups; plus some extensions, like PostGIS, a leading
geographical information manager based on Postgres.

### Google Storage<a id="sec-1-4-3" name="sec-1-4-3"></a>

Google Storage is the object store integrated in the Google Cloud
platform. If offers a variety of storage modes, with a tradeoff
between high access speed with low latency and storage cost.

## Implementation Overview<a id="sec-1-5" name="sec-1-5"></a>

### Kubernetes<a id="sec-1-5-1" name="sec-1-5-1"></a>

1.  Baseline configuration

    Skydipper is implemented in a managed Kubernetes cluster in the
    Google Cloud platform, although its architecture is pre-adapted
    to run on any platform that supports Kubernetes &#x2013; either a
    locally managed cluster or any other cloud platform.
    
    Initially, the platform runs in a cluster with three nodes
    without autoscaling, placed in the Western Europe region. It has
    six virtual cpus available and a combined memory of 22.50
    Gb. This provisioned capacity is exclusively for API operations
    &#x2013; model compiling, training and ingestion are offloaded to a
    workflow manager (Airflow) that has different capacity
    requirements, both in its baseline and in peak loads.

2.  Autoscaling and workload management

    No autoscaling procedure is needed in this state of development
    of Skydipper, but mechanisms to raise its capacity in case of
    high loads is contemplated as a part of the Kubernetes Engine
    configuration. Any database type that is deployed in the cluster
    should be run in an ad-hoc pool to better partition resources,
    particularly with some 'resource-hungry' services such as
    MongoDB.

3.  Continuous integration

    An instance of Jenkins deployed in the cluster offers continuous
    integration of the deployments made to the API. Jenkins deals
    with monitoring code changes in Skydipper's repositories,
    performs automated testing on the codebase, and if these succeed,
    deploys the changes to the cluster and performs some other
    ancillary tasks such as uploading docker manifests to DockerHub.
