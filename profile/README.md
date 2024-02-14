# CAVE: Connectome Annotation Versioning Engine

CAVE is a computational infrastructure for immediate and reproducible connectome analysis in up-to petascale datasets (\~$1mm^3$) while proofreading and annotating is ongoing. CAVE consists of a number of services and database management systems:

## Micro-Services

### Info 
A service for storing basic metadata about data, presently datastack, aligned_volume, permission_group and table_mapping. https://github.com/seung-lab/AnnotationFrameworkInfoService

### Auth: Authentication and authorization 
A service for controlling authorization on top of authentication provided by Google OAuth.https://github.com/seung-lab/middle_auth 

We deploy a second version of auth as "sticky" auth which has stick_sessions turned on. This is useful for certain operations that require sticky sessions, such as the flask-admin interface, whereas the other pods for auth do not require it and can distribute traffic more easily. There are no code base changes, only deployment changes.

### Schema
A service for storing/serving presently supported annotation schemas.  Note this is also a library that is utilized by AnnotationEngine and MaterializationEngine to generate database schemas.  https://www.github.com/seung-lab/EMAnnotationschemas

### Jsonstateserver
A service for posting and retrieving json states. Intended to provide support for neuroglancer link shortening. Backed by Google Datastore (future version to place json in bucket storage). 
https://github.com/seung-lab/NeuroglancerJsonServer

### AnnotationEngine
A service for creating, reading and updating annotations on aligned volumes, independent of segmentation. Backed by postgreSQL and postGIS. 
https://github.com/seung-lab/AnnotationEngine

### MaterializationEngine
A service to keep a set of annotations up to date with a segmentation. Shared codebase also contains an API to query these annotations, and a worker to process these annotations.  Backed by postgreSQL and postGIS, workflow powered by celery with a redis broker.
https://github.com/seung-lab/MaterializationEngine

### Pychunkedgraph (PCG)
A service to dynamically proofread and track a supervoxel graph of a segmentation.  Stores graph in a dynamic oct-tree backed by Google BigTable. Abbreviation PCG. Transmits activity via google pub-sub, presently consumed by Meshing and L2cache. 
https://github.com/seung-lab/PyChunkedGraph

### Meshing
A service to track, serve and regenerate meshes from a dynamic segmentation (shared codebase with the PCG).  Has a worker component as well, driven by google pubsub.
https://github.com/seung-lab/PyChunkedGraph

### L2Cache
A service to track, serve and regenerate summary statistics of “level 2” fragments of segmentation in the PCG. Has a worker component for calculating statistics,driven by google pub-sub.
https://github.com/seung-lab/PCGL2Cache

### Guidebook
A flask app service for creating neuroglancer links with annotation points to guide proofreaders to tips and branches. https://github.com/AllenInstitute/guidebook
Utilizes PCG aware skeletonization procedures https://github.com/AllenInstitute/pcg_skel

### Dash
A flask app for serving dash apps that are protected by auth.  https://github.com/fcollman/dash_on_flask.
Currently only serving dash-connectivity-viewer apps, provides interactive analysis UI of connectivity and cell types, along with dynamic neuroglancer link generation https://github.com/ceesem/dash-connectivity-viewer

### ProofreadingProgress
An app designed for FlyWire to track proofreading progress
https://github.com/seung-lab/ProofreadingProgress

### Proofreading Management
TBD. https://github.com/seung-lab/ProofreadingManagement (not public and not generalized past flywire)

### Proxy Map
This is a deprecated service for redirecting traffic to a google bucket which is not publicly readable. Requests are rerouted to a configurable set of google buckets, with an authorization header that is attached by the server to allow the user to get access to the bucket. The resulting data is simply forwarded back to the user. Connections to these endpoints are protected by the authorization service and so access can be limited in this way. Allows neuroglancer communication. 
https://github.com/seung-lab/middle_auth_proxy (not public repo)

## Access Tools

### CaveClient
Python library for accessing various microservice endpoints. 
Repo: https://github.com/seung-lab/CAVEclient
Docs: https://caveclient.readthedocs.io/

### MeshParty
Python library that wraps around cloudvolume to provide analysis and visualization functionality on top of meshes and skeleton.
Repo: https://github.com/sdorkenw/MeshParty
Docs: https://meshparty.readthedocs.io/

### Pcg_skel
Library for running skeletonization that integrates the chunkedgraph level2 graph, and the l2cache. 
Repo: https://github.com/AllenInstitute/pcg_skel
Docs: https://github.com/AllenInstitute/pcg_skel/blob/master/README.md


## CAVE-adjacent Tools

These tools integrate with CAVE and are useful for CAVE users. They are developed and maintained by members of the community and not the CAVE team.

### CloudVolume
Python library for accessing images, segmentation, meshes and skeletons from precomputed and CAVE (i.e. graphene protocol) sources. 
Repo: https://github.com/seung-lab/cloud-volume
Docs: https://github.com/seung-lab/cloud-volume/blob/master/README.md

### Navis
Python library wrapped around many other libraries that gives analysis tools for analyzing neurons. Integrates with cloud-volume and caveclient. Focused mostly on fly neurons, but provides general blender integration for example. 
Repo: https://github.com/navis-org/navis
Docs: https://navis.readthedocs.io/en/latest/index.html

