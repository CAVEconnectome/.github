# CAVE: Connectome Annotation Versioning Engine

CAVE is a computational infrastructure for immediate and reproducible connectome analysis in up-to petascale datasets (\~ $1mm^3$) while proofreading and annotating is ongoing. CAVE consists of a number of services and database management systems that are listed below.  It was initially developed as a collaboration between the electron microscopy group at the [Allen Institute for Brain Science](https://alleninstitute.org/division/brain-science/) and [Sebastian Seung's lab ](https://seunglab.org/) at Princeton University. It is most fully described in a [pre-print available here](https://www.biorxiv.org/content/10.1101/2023.07.26.550598v1), though aspects of the PyChunkedGraph service were described also in earlier works ([Dorkenwald et al. 2022a](https://elifesciences.org/articles/76120#abstract) and [Dorkenwald et al. 2022b](https://www.nature.com/articles/s41592-021-01330-0)).  It is use by several large scale published electron microscopy efforts, including the [MICrONS project](https://microns-explorer.org), [flywire](https://ngl.flywire.ai), [fanc](https://www.biorxiv.org/content/10.1101/2022.12.15.520299v1.full), and [h01](https://h01-release.storage.googleapis.com/landing.html), and several unpublished datasets.

## Micro-Services

#### Info 
A service for storing basic metadata about data, presently datastack, aligned_volume, permission_group and table_mapping. [Repository](https://github.com/seung-lab/AnnotationFrameworkInfoService)

#### Auth: Authentication and authorization 
A service for controlling authorization on top of authentication provided by Google OAuth. [Repository](https://github.com/seung-lab/middle_auth)

We deploy a second version of auth as "sticky" auth which has stick_sessions turned on. This is useful for certain operations that require sticky sessions, such as the flask-admin interface, whereas the other pods for auth do not require it and can distribute traffic more easily. There are no code base changes, only deployment changes.

#### Schema
A service for storing/serving presently supported annotation schemas.  Note this is also a library that is utilized by AnnotationEngine and MaterializationEngine to generate database schemas. [Repository](https://www.github.com/seung-lab/EMAnnotationschemas)

#### Jsonstateserver
A service for posting and retrieving json states. Intended to provide support for neuroglancer link shortening. Backed by Google Datastore (future version to place json in bucket storage). [Repository](https://github.com/seung-lab/NeuroglancerJsonServer)

#### AnnotationEngine
A service for creating, reading and updating annotations on aligned volumes, independent of segmentation. Backed by postgreSQL and postGIS. [Repository](https://github.com/CAVEconnectome/AnnotationEngine)

#### MaterializationEngine
A service to keep a set of annotations up to date with a segmentation. Shared codebase also contains an API to query these annotations, and a worker to process these annotations.  Backed by postgreSQL and postGIS, workflow powered by celery with a redis broker. [Repository](https://github.com/CAVEconnectome/MaterializationEngine)

#### Pychunkedgraph (PCG)
A service to dynamically proofread and track a supervoxel graph of a segmentation.  Stores graph in a dynamic oct-tree backed by Google BigTable. Abbreviation PCG. Transmits activity via google pub-sub, presently consumed by Meshing and L2cache. [Repository](https://github.com/seung-lab/PyChunkedGraph)

#### Meshing
A service to track, serve and regenerate meshes from a dynamic segmentation (shared codebase with the PCG).  Has a worker component as well, driven by google pubsub. [Repository](https://github.com/seung-lab/PyChunkedGraph)

#### L2Cache
A service to track, serve and regenerate summary statistics of “level 2” fragments of segmentation in the PCG. Has a worker component for calculating statistics,driven by google pub-sub. [Repository](https://github.com/seung-lab/PCGL2Cache)

#### Guidebook
A flask app service for creating neuroglancer links with annotation points to guide proofreaders to tips and branches. Guidebook utilizes PCG aware skeletonization procedures via pcg_skel (see below). [Repository](https://github.com/CAVEconnnectome/guidebook)

#### Dash
A flask app for serving dash apps that are protected by auth. Currently only serving dash-connectivity-viewer apps, provides interactive analysis UI of connectivity and cell types, along with dynamic neuroglancer link generation. [Repository](https://github.com/CAVEconnectome/dash_on_flask), [Repository](https://github.com/ceesem/dash-connectivity-viewer)

#### ProofreadingProgress
An app designed for FlyWire to track proofreading progress. [Repository](https://github.com/seung-lab/ProofreadingProgress)

#### Proofreading Management
TBD. [Repository](https://github.com/seung-lab/ProofreadingManagement) (not public and not generalized past flywire)

#### Proxy Map
This is a deprecated service for redirecting traffic to a google bucket which is not publicly readable. Requests are rerouted to a configurable set of google buckets, with an authorization header that is attached by the server to allow the user to get access to the bucket. The resulting data is simply forwarded back to the user. Connections to these endpoints are protected by the authorization service and so access can be limited in this way. Allows neuroglancer communication. [Repository](https://github.com/seung-lab/middle_auth_proxy) (not public repo)

## Access Tools

#### CaveClient
Python library for accessing various microservice endpoints. [Repository](https://github.com/seung-lab/CAVEclient), [Documentation](https://caveclient.readthedocs.io/)

#### MeshParty
Python library that wraps around cloudvolume to provide analysis and visualization functionality on top of meshes and skeleton. [Repository](https://github.com/sdorkenw/MeshParty), [Documentation](https://meshparty.readthedocs.io/)

#### Pcg_skel
Library for running skeletonization that integrates the chunkedgraph level2 graph, and the l2cache. [Repository](https://github.com/CAVEconnectome/pcg_skel), [Documentation](https://github.com/CAVEconnectome/pcg_skel/blob/master/README.md)


## CAVE-adjacent Tools

These tools integrate with CAVE and are useful for CAVE users. They are developed and maintained by members of the community and not the CAVE team.

#### CloudVolume
Python library for accessing images, segmentation, meshes and skeletons from precomputed and CAVE (i.e. graphene protocol) sources. [Repository](https://github.com/seung-lab/cloud-volume), [Documentation](https://github.com/seung-lab/cloud-volume/blob/master/README.md)

#### Navis
Python library wrapped around many other libraries that gives analysis tools for analyzing neurons. Integrates with cloud-volume and caveclient. Focused mostly on fly neurons, but provides general blender integration for example. [Repository](https://github.com/navis-org/navis), [Documentation](https://navis.readthedocs.io/en/latest/index.html)

## Deployment

CAVE uses kubernetes for deployment. We developed a bash script based system of templates and config files to deploy CAVE. [Repository](https://github.com/seung-lab/CAVEdeployment), [Instructions](https://docs.google.com/document/d/1C110QZEWdE4NnAIR7HLStKhff3d4Ha60VJ_W09akj3c/edit?usp=sharing)
