RFC 1: Changes for Fiona 2.0
============================

* Request for comments: 1
* Author: Sean Gillies, sean.gillies@gmail.com
* Date: 2019-04-08

## Abstract

It's a good time to begin work on a new major version of Fiona. Removing support for obsolete versions of Python and GDAL will cut maintenance costs and improved design will set the project up for more growth.

## Introduction

Instead of working on another backwards compatible minor version of Fiona (1.9), the Fiona team shall begin working on what would be a new major version with some breaking changes.

The planned changes are:

* Dropping support for Python versions < 3.5.
* Dropping support for GDAL versions < 2.2.
* Replacing the existing coordinate referencing dicts with a new CRS class (as in Rasterio).
* Replacing the `"geometry"` in `feature["geometry"]` with a Cython extension class that could enhance serialization performance. Geometry objects would become immutable.

## Details

The Python project will no longer support Python versions < 3.5 after the end of 2019. Python 3.4 has been retired. We can stop testing with 3.4 and stop building wheels for 3.4. Python 2.7 will not be retired until Jan 1, 2020, but Numpy and many associated projects have already decided to drop support for Python 2.7 in new versions starting Jan 1, 2019: https://python3statement.org/. Fiona can follow suit. Not only do we cut the cost of testing and building wheels for 2.7, we can eliminate the compat module and backported dependencies such as argparse, ordereddict, enum34, and mock.

Many Fiona users get their distributions from PyPI and Conda-forge, which provide very recent versions of GDAL, 2.3 or 2.4. Many others get theirs from Ubuntu-GIS, which provides GDAL 2.1 in the stable PPA and GDAL 2.2 in the unstable PPA for Xenial (16.04), the oldest supported LTS release. The Fiona project has no regular input from other packagers downstream. Cutting off support for GDAL versions < 2.2 lightens the project build matrix and allows the deletion of `fiona/_shim1.*`, `fiona/_shim2.*`, and many lines of code in setup.py.

Copying Rasterio's CRS class to fiona.crs will allow Fiona to change to WKT as the canonical representation of spatial reference systems and gain accuracy and precision. CRS objects would be immutable, which is a breaking change. The CRS class would be a Cython extension class with a OGRSpatialRefenceH object at its core.

Representing spatial features and geometries as GeoJSON-like dicts has served Fiona well, but many of the project's users are looking for a faster representation. Replacing, e.g., `{"type": "Polygon", "coordinates": [[[...]]]}` with a Cython extension object based on a OGRGeometryH object will permit serialization to WKB or GeoJSON as needed and also faster read/filter/write pipelines that could completely bypass the GeoJSON serialization that has been standard until now.

## Considerations

It would be appropriate to warn users about upcoming changes for CRS and geometry. Doing so is somewhat complicated by the fact that we're currently using standard dicts for each. We may be faced with replacing the dicts with a custom, mutable dict-like object so that we can raise a DeprecationWarning when its `__setitem__` method is called. Discussion of whether this warrants another minor release before 2.0 is warranted.

## References

https://python3statement.org/
