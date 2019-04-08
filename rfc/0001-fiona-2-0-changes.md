RFC 1: A plan for Fiona 2.0
===========================

* Request for comments: 1
* Author: Sean Gillies, sean.gillies@gmail.com
* Date: 2019-04-08

## Abstract

It's a good time to begin work on a new major version of Fiona. Removing support for obsolete versions of Python and GDAL will cut maintenance costs and improved design will set the project up for more growth.

## Introduction

Instead of working on another backwards compatible minor version of Fiona (1.9), the Fiona team shall begin working on what would be a new major version with some breaking changes.

The planned changes are:

* Dropping support for Python versions < 3.5.
* Dropping support for GDAL versions < 2.1.
* Replacing the existing coordinate referencing dicts with a new CRS class (as in Rasterio).
* Replacing the `"geometry"` in `feature["geometry"]` with a Cython extension class that could enhance serialization performance. Geometry objects would become immutable.

## Details

The Python project will no longer support Python versions < 3.5 after the end of 2019. Python 3.4 has been retired. We can stop testing with 3.4 and stop building wheels for 3.4. Python 2.7 will not be retired until Jan 1, 2020, but Numpy and many associated projects have already decided to drop support for Python 2.7 in new versions starting Jan 1, 2019: https://python3statement.org/. Fiona can follow suit. Not only do we cut the cost of testing and building wheels for 2.7, we can eliminate the compat module and version-dependent dependencies.

TODO: details about supported GDAL versions.

TODO: CRS details

TODO: Geometry details

## Considerations

TODO: mitigation of pain for users, effects on wheel-building infrastructure, conda, etc.

## References

https://python3statement.org/
