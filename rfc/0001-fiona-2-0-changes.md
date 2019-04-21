RFC 1: Changes for Fiona 2.0
============================

* Request for comments: 1
* Author: Sean Gillies, sean.gillies@gmail.com
* Date: 2019-04-08

## Abstract

It's a good time to begin work on a new major version of Fiona. Removing support for obsolete versions of Python and GDAL will cut maintenance costs and improved design will set the project up for more growth.

## Introduction

Instead of working on another backwards compatible minor version of Fiona, the Fiona team shall begin working on what would be a new major version with some breaking changes.

The planned changes are:

* Dropping support for Python versions < 3.5.
* Dropping support for GDAL versions < 2.2.
* Replacing the existing coordinate referencing dicts with a new CRS class (as in Rasterio).
* Replacing the `"geometry"` in `feature["geometry"]` with a Cython extension class that could enhance interoperability with shapely. Geometry objects would become immutable.
* Replacing the feature dict with a Cython extension class that could improve dataset filter and translation performance.

## Details

The Python project will no longer support Python versions < 3.5 after the end of 2019. Python 3.4 has been retired. We can stop testing with 3.4 and stop building wheels for 3.4. Python 2.7 will not be retired until Jan 1, 2020, but Numpy and many associated projects have already decided to drop support for Python 2.7 in new versions starting Jan 1, 2019: https://python3statement.org/. Fiona can follow suit. Not only do we cut the cost of testing and building wheels for 2.7, we can eliminate the compat module and backported dependencies such as argparse, ordereddict, enum34, and mock.

Many Fiona users get their distributions from PyPI and Conda-forge, which provide very recent versions of GDAL, 2.3 or 2.4. Many others get theirs from Ubuntu-GIS, which provides GDAL 2.1 in the stable PPA and GDAL 2.2 in the unstable PPA for Xenial (16.04), the oldest supported LTS release. The Fiona project has no regular input from other packagers downstream. Cutting off support for GDAL versions < 2.2 lightens the project build matrix and allows the deletion of `fiona/_shim1.*`, `fiona/_shim2.*`, and many lines of code in setup.py.

Copying Rasterio's CRS class to fiona.crs will allow Fiona to change to WKT as the canonical representation of spatial reference systems and gain accuracy and precision. CRS objects would be immutable, which is a breaking change. The CRS class would be a Cython extension class with a OGRSpatialRefenceH object at its core.

Representing spatial features and geometries as GeoJSON-like dicts has served Fiona well, but there are drawbacks. The friction of converting between Fiona and Shapely geometries using GeoJSON-like dicts as an intermediate format slows down projects like GeoPandas; the "well known binary" (WKB) format could reduce this friction. There are also situations where no intermediate geometry format is needed at all, such as in the following example of filtering a vector dataset using a bounding box.

```python
with fiona.open("input") as input, fiona.open("output", "w", input.schema) as output:
    output.writerecords(input.filter(*bbox))
```

Replacing the geometry in `feature["geometry"]` with a Cython extension class based on an OGRGeometryH object can address the first of these drawbacks. Serialization to WKB or GeoJSON would be done lazily, only as needed. Replacing the feature dict with another Cython extension class based on an OGRFeatureH object can address the second drawback. Making the new geometry class immutable will be one of the keys to keeping its implementation simple.

## Considerations

Dropping support for Python 2.7 must be advertised in various places: the project discussion group, developer blogs, Twitter, gdal-dev. There are no other special considerations.

Dropping support for GDAL 1.x - 2.1.x must also be advertised in the same places. GDAL 2.2 is currently the baseline version and will remain the baseline version for Fiona 2.0. There are no other special considerations.

It would be appropriate to warn users about upcoming changes for CRS, geometry, and feature objects. The semantic version spec 
recommends at least one minor version release with the deprecation warnings in place before breaking changes are released: https://semver.org/#how-should-i-handle-deprecating-functionality. This minor version release would be 1.9.0.

What are the deprecations?

- CRS and geometry objects will be immutable: there will be no `__setitem__` or `__setattr__`.
- CRS, geometry, and feature objects will no longer be dicts, or subclass dict. This has a number of consequences, such as no more default JSON serialization.

Warning about these deprecations will be somewhat complicated. We are faced with replacing the standard dicts in 1.9 with custom, dict-like classes so that we can raise DeprecationWarnings in `__setitem__`, `__setattr__`, `__instancecheck__`, and `__subclasscheck__`.

## References

https://python3statement.org/
https://semver.org
