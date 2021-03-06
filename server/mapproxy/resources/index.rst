.. index::
    pair: free layer; vector layer

.. _resources:

===================
Resources reference
===================

In the section :ref:`mapproxy-configuration`, we provided VTS-Mapproxy with link
to resource configuration file in the ``[resource-backend]`` section. 
There are two possible ways, how to define resources - using configuration file
or using :ref:`resources-python`.

The sample configuration can be seen in :download:`../examples/resources.json` file
example. Generic JSON file structure looks following::

    [{
        ...
        resource1 definition
        ...
    },
    {
        ...
        resource2 definition
        ...
    },
    ...
    ]

.. _mapproxy-resource:

Resource (object)
-----------------

+ ``comment`` (string) - any comment, ignored
+ ``group`` (string) - group this resource belongs to
+ ``id`` (string) - resource identifier (withing group)
+ ``type`` (enum[:ref:`mapproxy-type`]) - data type
+ ``driver`` (:ref:`mapproxy-driver`) - data generator 
+ ``registry`` (:ref:`mapproxy-registry`) - additional local resource registry
+ ``referenceFrames`` (Object[:ref:`mapproxy-reference-frame`]) - :ref:`reference-frame`\s definition. The key value is reference to :ref:`mapproxy-ref-frame-definition`.
+ ``credits`` (array[string]) - reference to :ref:`mapproxy-credits`
+ ``definition`` (:ref:`mapproxy-definition`) - input data definition

.. _mapproxy-type:

Type
----
Mapproxy resource type

One of
^^^^^^

+ ``tms`` - tile map service (:ref:`bound-layer`)
+ ``surface`` 
+ ``geodata`` - vector free layers

.. _mapproxy-driver:

Driver
------

Read driver definition. This then corresponds to :ref:`mapproxy-definition`
configuration option.

One of
^^^^^^

+ ``tms-raster``
+ ``tms-raster-remote``
+ ``tms-patchwork``
+ ``tms-bing``
+ ``surface-spheroid``
+ ``surface-dem``
+ ``geodata-vector``
+ ``geodata-vector-tiled``

.. _mapproxy-registry:

Registry
--------

.. todo:: Registry missing

.. _mapproxy-reference-frame:

Reference frame
---------------

Definition of :ref:`lod` range and tile ranges for each :ref:`reference-frame`
for this resource. See more about configuration at
:ref:`reference-frame-configuration`.

+ ``lodRange`` (array[number, number]) - :ref:`lod` range extend
+ ``tileRange`` (array[array[number, number]]) - bounding box of tiles covering area of interest on the first LOD. Example: ``[[16, 10], [17,11]]``


.. _mapproxy-definition:

Definition
----------
Input data definition, depending on each :ref:`mapproxy-driver`, you have to
pick one of following options:

One of
^^^^^^

+ :ref:`tms-driver`
+ :ref:`surface-driver`
+ :ref:`geodata`

.. _tms-driver:

TMS Driver
----------

Tiled map service driver definition

One of
^^^^^^

+ :ref:`tms-raster`
+ :ref:`tms-raster-remote`
+ :ref:`tms-patchwork`
+ :ref:`tms-bing`

.. _tms-raster:

tms-raster
----------

Raster-based bound layer generator. Uses any raster GDAL dataset as its data
source. Supports optional data masking.

+ ``dataset`` (string) - path to GDAL dataset
+ ``mask`` (string, optional) - path to RF mask or masking GDAL dataset
+ ``format`` (string, optional) - output image format, "jpg" or "png" (defaults to "jpg")
+ ``transparent`` (boolean, optional) - Boundlayer is transparent, forces format to "png"

.. _tms-raster-remote:

tms-raster-remote
-----------------

Raster bound layer generator. Imagery is pointer to external resource via
``remoteUrl`` (a URL template). Supports optional data masking.

+ ``remoteUrl`` (:ref:`url`) - Imagery URL template.
+ ``mask`` (string) - path to RF mask or masking GDAL dataset

.. _tms-patchwork:

tms-patchwork
-------------

Simple raster bound layer generator. Generates color checkered tiles. Supports optional data masking.

* ``mask`` (string, optional) - path to RF mask or masking GDAL dataset
* ``format`` (string, optional) - output image format, "jpg" or "png" (defaults to "jpg")

.. _tms-bing:

tms-bing
--------

Bound layer generator for remote Bing data. Valid session is generated via metatada URL.

* ``metadataUrl`` (string) - Bing API metadata URL. See Bing API documentation for more info.

.. _surface-driver:

Surface driver
--------------

Surface drivers generate a meshed surface that can be used directly as a single
surface or merged into VTS storage as a remote tileset. In addition, a
:file:`freelayer.json` file is provided allowing generated surface to act as a
mesh-tiles free layer.

Common surdace driver configuration options

+ ``geoidGrid`` (string) - name of Proj.4's geoid grid file (e.g. `egm96_15.gtx`).
+ ``nominalTexelSize`` (number) - nominal resolution (meter/pixel); reported by :ref:`mapproxy-calipers`
+ ``mergeBottomLod`` (number) - Reported in generated :file:`tileset.conf`, speeds up merge with other surfaces
+ ``introspection`` (array[:ref:`introspection`] | Object[:ref:`introspection`]) - Introspection info used when using :file:`mapConfig.json` served by :ref:`mapproxy`

If there is just one TMS resource used in the introspection then the enclosing array is optional.

One of
^^^^^^

+ :ref:`surface-spheroid`
+ :ref:`surface-dem`

.. _surface-spheroid:

surface-spheroid
----------------

This driver generates meshed surface for reference frame's spheroid. If geoid
grid is provided the resulting body is in fact a geoid.

If a ``textureLayerId`` entry is present this ID is written into generated meshes as
a default bound layer to use if nothing else is mapped on the surface. Otherwise
surface is completely texture less.

+ `textureLayerId` (number, optional) - numeric bound layer ID

.. _surface-dem:

surface-dem
-----------

This driver generates a meshed surface from the supplied GDAL raster DEM/DSM/DTM
dataset.

Since GDAL raster formats are unable to safely store vertical SRS component it
cannot tell whether data are in ellipsoidal or orthometric verical datum.
Therefore by default the heights are treated as if they are above the ellipsoid
(i.e. ellipsoidal vertical datum). By providing a geoidGrid configuration option
we can specify geoid grid for the orthormetric vertical datum, i.e. to tell that
the heights store in the GDAL dataset are relative to given geoid.

Please be aware that due to such limitations the GDAL dataset's vertical system
must be compatible with reference frame's vertical system to use geoid support.
I.e. either they share the same ellipsoid or the input data are in some local
system that approximates the geoid at given place. One working example is data
in Krovak's projection that can be reinterpreted as heights above WGS84+EGM96
without any significant error.

If a ``textureLayerId`` entry is present this ID is written into generated meshes as
a default bound layer to use if nothing else is mapped on the surface. Otherwise
surface is completely texture less.

All :ref:`surface-dem` input datasets are registered in internal map lod available DEM's
under its group-id identifier and can be referenced from various
geodataresources for 2D features height config. Optionially, input dataset can be
registered in this map under an alias. See more in the geodata resources
documentation.

+ ``dataset`` (string) - path to complex dataset
+ ``mask`` (string, optional) - optional mask, generated by :ref:`mapproxy-rf-mask` tool
+ ``textureLayerId`` (number, optional) - numeric bound layer ID
+ ``heightcodingAlias`` (string, optional) - dataset is registered under given alias

.. _geodata:

Geodata
-------

Geodata drivers generate vector geographic data in the form of :ref:`free-layer`.

One of
^^^^^^

+ :ref:`geodata-vector`
+ :ref:`geodata-vector-tiled`

.. _geodata-vector:

geodata-vector
--------------

Generates monolithic free layer (geodata type) from an OGR-supported dataset
(GeoJSON, shapefile, ...). Purely 2D data are converted to full 3D data using
process called heigth coding: each 2D coordinate is extended by height read from
the accompanying DEM/DTM/DSM GDAL dataset.

Height coding DEM is in the same format a the dataset expected by :ref:`surface-dem`
driver although only its ``/dem`` part is used. This DEM can be accompanied with its
geoid grid in the same way as :ref:`surface-dem` is.

By default all layers from the source dataset are served. Optionally, layer
subset can be configured by providing list of layer names.

+ ``dataset`` (string) - path to OGR dataset
+ ``demDataset`` (string) - path to complex dem dataset
+ ``displaySize`` (string) - Nominal size of tile in pixels, e.g. 1024
+ ``geoidGrid`` (string, optional) - name of Proj.4's geoid grid file (e.g. :file:`egm96_15.gtx`)
+ ``layers`` (array[string], optional) - list of layers names, as provided by GDAL
+ ``format`` (string, optional) - output file format, so far only "geodataJson" is supported (default)
+ ``styleUrl`` (string, optional) - URL to default geodata style. If used with ``file:`` prefix, e.g.  ``file:style.json``, it's searched in ``datasets`` directory.
+ ``introspection`` (:ref:`introspection`, optional) - Extended configuration for mapConfig.json served by mapproxy

Introspection can be used to serve ``mapConfig`` where geodata are show with some surface which in turn can have its own introspection configuration.

.. _geodata-vector-tiled:

geodata-vector-tiled
--------------------

Generates tiled geodata (geodata-tiles type) from pre-tiled data. Input tiling
must match reference frame's space division, at least in one of its nodes. For
example, OSM tiles in pseudomerc projection can be used in ``webmerc-projected`` and
``webmerc-unprojected`` :ref:`reference-frame`\s and in the ``pseudomerc`` subtree in in
``melown2015`` reference frame.

Configuration is the same as for :ref:`geodata-vector` driver but input interpretation
is different and served data are different.

Geodata's metatiles are generated purely from heightcoding GDAL dataset.


+ ``dataset`` (:ref:`url`) - Option definition.dataset is a OGR dataset path/URL template that is expanded for each requested tile before opening and processing.


.. _introspection:

Introspection
-------------

Introspection is extending configuration for :ref:`mapproxy` served :file:`mapConfig.json`
(only when browsing is enabled).

+ ``position`` (:ref:`position`) - VTS position in JSON/python format
+ ``tms`` (:ref:`resource-reference`) - bound layer(s) mapped on the surface, see below 
+ ``geodata`` (:ref:`resource-reference`) - ``group`` and ``id`` reference to free layer resource
+ ``surface`` (:ref:`resource-reference`) - reference to existing surface resource

.. _resource-reference:

Resource reference
------------------
Whenever you need to define reference to existing resource

+ ``group`` (string) - group part of TMS resource identifier
+ ``id`` (string) - ID part of TMS resource identifier

.. _position:

Position
--------
In VTS, the general position format is called an objective position. Position
can be defined from ``object`` or ``subject`` perspective. Example
configuration::

    ["obj",472205.0,5555763.0,"fix",229.0,-10,-57,0,75,80]

where

* element 0 is string "object"
* elements 1-2 are XY components position of the center of orbit in navigation
  SRS
* element 3 is either "fix" or "float"
* element 4 is either Z component of the center of orbit in navigation SRS (if
  element 3 is "fix") or its AGL (if element 3 is "float")
* elements 5-7 are NED based Euler angles (yaw, pitch, and roll) of the
  direction of view, measured at the center of orbit
* element 8 is vertical extent of camera view, measured at the center of orbit
  at physical SRS units
* element 9 is vertical FOV, measured in degrees.
* As a special case, the value of element 8 may be 0, indicating that the
  projection is orthographic.

A slightly less general position format, named subjective position, is defined as a similar 10-tuple::

    ["subj",472201.0,5555739.0,"fix",266.4,-10,-57,0,44.7,80]

where

* element 0 is string "subj"
* elements 1-2 are XY components of the center of perspectivity in navigation
  SRS
* element 3 is either "fix" or "float"
* elements 4 is either Z component of the center of perspectivity in navigation
  SRS (if element 3 is "fix") or its AGL (if element 3 is "float")
* elements 5-7 are NED based Euler angles of the direction of view, measured at
  the center of perspectivity
* element 8 is distance from the center of perspectivity to the center of orbit,
  and
* element 9 (optional) is vertical FOV, measured in degree

For getting optimal position definition of your dataset,
:ref:`mapproxy-calipers` should be used.

.. _url:

URL template
------------
This is documented elsewhere but as a convenience we provide URL template
expansion documentation here.

Each tile has its global and local ``tileId``. For simple reference frames (like
``webmerc-projected``) global and local identifers are the same.

For complex reference frames (``melown2015``, ``earth-qsc``) global identifier
is from tile tree root, i.e. from 0-0-0. Local identifier is tile identifier
relative to its reference frame subtree.

For example (in ``melown2015``):
    + tile with global ID 10-256-256 is in the pseudomerc subtree with root at
      1-0-0 and its local ID is 9-256-256
    + tile with global ID 10-768-512 is in the steres-wgs84 subtree with root at
      1-1-0 and its local ID is also 9-256-256

Available expansion strings. Only some make sense for templates used in
:ref:`mapproxy`.

+ ``{lod}`` - global tile LOD
+ ``{x}`` - global tile X index
+ ``{y}`` - global tile Y index
+ ``{loclod}`` - local tile LOD
+ ``{locx}`` - local tile X index
+ ``{locy}`` - local tile Y index
+ ``{sub}`` - sub-tile identifier (e.g. submesh index in atlas image)'
+ ``{alt(1,2,3,4)}`` - exands to one of given strings
+ ``{ppx}`` - tile's old PP space X index (makes sense only in ppspace)
+ ``{ppy}`` - tile's old PP space Y index (makes sense only in ppspace)

.. _mapproxy-ref-frame-definition:

Reference frame
---------------
.. todo:: Reference frame configuration

.. _mapproxy-credits:

Credits
-------
+ ``id`` (number) - Identification
+ ``notice`` (string) - What you want to appear on the map. Several placeholders can be defined (see lower) using ``{PLACEHOLDER}`` syntax.
+ ``url`` (string, optional) Optional URL

Following placeholders can be used in the ``notice`` configuration options and
will be replaced by corresponding signs.

+ ``{copy}`` Will add copyright sign
+ ``{Y}`` Will add current year


.. _srs-configuration:

SRS configuration
-----------------

:ref:`srs`\s are defined in :file:`srs.json` (see `source`_)
is dictionary, where each layer is identified by key and following options:

+ ``comment`` (string) - Description string
+ ``srsDef`` (string)-  `Proj4 SRS definition <http://proj4.org>`_, e.g. you can use `epsg.io <http://epsg.io>`_ service to get it done.::

        "srsDef": "+proj=qsc +units=m +datum=WGS84 +lat_0=90 +lon_0=0 +wktext",
+ ``type`` (string) -  One of
    + ``cartesian``
    + ``geographic``
    + ``projected``

*Optional parameters*:

+ ``srsModifiers`` (Array(string)) - Modification options: ``adjustVertical``
+ ``periodicity`` (Object) - Defined by ``type`` and ``period``::

        "periodicity" : { "type" : "X", "period": 40075016.685578 }
+ ``geoidGrid`` (Object) `Geoid grid <https://en.wikipedia.org/wiki/Geoid#/media/File:Geoid_height_red_blue_averagebw.png>`_ can be attached as JPEG encoded file with ``extents, valueRange`` and ``srsDefEllps`` example::

        "geoidGrid": {
            "extents": {"ll": [-2009979, 3000861], "ur": [2999421, 8260731]},
            "valueRange": [-17.6, 67.3],
            "definition": "geoidgrid/utm33n-va-geoidgrid.jpg",
            "srsDefEllps" : "+proj=utm +zone=33 +datum=WGS84 +no_defs"
        }

.. _reference-frame-configuration:

Reference frame configuration
-----------------------------

:ref:`reference-frame` is defined by SRSs, LODs, extentds and other parameters.
Reference ranges are stored as *Array* (not dictionary, compared to previously
described data structures).
Basic reference frames are defined in :file:`referenceframes.json` in
:ref:`registry`.

+ ``version`` (number) - Version nuber

.. todo:: what does version mean

``id`` (string) - Unique identifier
``description`` (string) - Longer descriptive text
``model`` (:ref:`ref-frame-model`) - Definition of ``physicalSRS``, ``navigationSrs`` and ``publicSrs`` as reference to :ref:`srs-configuration`
``division`` (:ref:`division`) - Division defintion

.. _ref-frame-model:

Model
-----
Reference frame model (see :ref:`reference-frame-configuration` ``model``
option.

+ ``physicalSrs`` (string) - reference to :ref:`srs-configuration`. See more in
  :ref:`physical-srs`
+ ``navigationSrs`` (string) - reference to :ref:`srs-configuration`. See more
  in :ref:`navigation-srs`
+ ``publicSrs`` (string) - reference to :ref:`srs-configuration`. See more in
  :ref:`public-srs`

.. _division:

Division
--------

+ ``extents`` (ref:`extent-configuration`) - Bounding box
+ ``heightRange`` (array(number)) - low and top bounds of heights (above the ellipsoid), where the reference frame makes sense
+ ``nodes`` (Array) - Definition of nodes for various lod ranges and positions, see :ref:`nodes-configuration`

.. _nodes-configuration:

Nodes
-----

Following options can define reference frame node:

+ ``id`` (Object) - It's dictionary with keys ``lod`` (level od detail) and ``position`` (tile
    position with the tile grid).
+ ``srs`` (String) - Reference to :ref:`srs-configuration`
+ ``extents`` (ref:`extent-configuration`) - Bounding box
+ ``partitioning`` (string|:ref:`extent-configuration`) - Either you can use the keyword ``bisection`` or you can use binary keys and

.. _extent-configuration:

Extent
------
Bounding box configuration

+ ``ll`` (array(number)) - minx, miny
+ ``ur`` (array(number)) - maxx, maxy

.. _source: https://github.com/Melown/vts-registry/blob/master/registry/registry/boundlayers.json

.. _resources-python:

Python configuration
--------------------

.. todo:: more about python config file
