# KIM-PROPERTY utility module

[![Build Status](https://travis-ci.org/openkim/kim-property.svg?branch=master)](https://travis-ci.org/openkim/kim-property)
[![Python package](https://github.com/openkim/kim-property/workflows/Python%20package/badge.svg)](https://github.com/openkim/kim-property/actions)
[![Windows Build status](https://ci.appveyor.com/api/projects/status/iu5se53hqyaigiur?svg=true)](https://ci.appveyor.com/project/yafshar/kim-property)
[![Anaconda-Server Badge](https://img.shields.io/conda/vn/conda-forge/kim_property.svg)](https://anaconda.org/conda-forge/kim_property)
[![PyPI](https://img.shields.io/pypi/v/kim-property.svg)](https://pypi.python.org/pypi/kim-property)
[![License](https://img.shields.io/badge/license-CDDL--1.0-blue)](LICENSE.CDDL)

The objective is to make it as easy as possible to convert a script (for
example a [LAMMPS](https://lammps.sandia.gov/) script) that computes a
property to a KIM Test.

This utility module has 5 modes:

1- **[Create](#Create)**\
    Take as input the property instance ID and property definition name and
    create initial property instance data structure. It checks and indicates
    whether the property definition exists in [OpenKIM](https://openkim.org/).

2- **[Destroy](#Destroy)**\
    Delete a previously created property instance ID.

3- **[Modify](#Modify)**\
    Incrementally build the property instance by receiving keys with
    associated arguments. It can "append" and add to a key's existing array
    argument.

4- **[Remove](#Remove)**\
    Remove a key.

5- **[Dump](#Dump)**\
    Take as input the generated instance and a filename, validate instance
    against the property definition and either issues an error or writes the
    instance out to file in edn format. Final validation should make sure
    all keys/arguments are legal and all required keys are provided.

## Create

Creating property instances::

````py
    >>> kim_property_create(1, 'cohesive-energy-relation-cubic-crystal')
    '[{"property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal" "instance-id" 1}]'

    >>> str = kim_property_create(1, 'cohesive-energy-relation-cubic-crystal')

    >>> kim_property_create(2, 'atomic-mass', str)
    '[{"property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal" "instance-id" 1} {"property-id" "tag:brunnels@noreply.openkim.org,2016-05-11:property/atomic-mass" "instance-id" 2}]'

    >>> str = kim_property_create(2, 'atomic-mass', str)
    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
        }
        {
            "property-id" "tag:brunnels@noreply.openkim.org,2016-05-11:property/atomic-mass"
            "instance-id" 2
        }
    ]
````

A property instance is stored in a subset of the KIM-EDN format as described in [KIM Property Instances](https://openkim.org/doc/schema/properties-framework).
Each property instance must contain the `property-id` and `instance-id`.
`kim_property` utility module can create a new property instance, using a KIM property ID (e.g. `tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal`) or only a KIM property name (e.g `cohesive-energy-relation-cubic-crystal`) where, it internally will use the correct KIM property ID.
Both use cases are depicted above.

## Destroy

Destroying property instances::

````py
    >>> obj = '[{"property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal" "instance-id" 1}]'

    >>> kim_property_destroy(obj, 1)
    '[]'

    >>> obj = '[{"property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal" "instance-id" 1} {"property-id" "tag:brunnels@noreply.openkim.org,2016-05-11:property/atomic-mass" "instance-id" 2}]'

    >>> kim_property_destroy(obj, 2)
    '[{"property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal" "instance-id" 1}]'
````

## Modify

Modifying (setting) property instances::

````py
    >>> str = kim_property_create(1, 'cohesive-energy-relation-cubic-crystal')
    >>> str = kim_property_modify(str, 1,
                "key", "short-name",
                "source-value", "1", "fcc",
                "key", "species",
                "source-value", "1:4", "Al", "Al", "Al", "Al",
                "key", "a",
                "source-value", "1:5", "3.9149", "4.0000", "4.032", "4.0817", "4.1602",
                "source-unit", "angstrom", "digits", "5")
    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
            "short-name" {
                "source-value" [
                    "fcc"
                ]
            }
            "species" {
                "source-value" [
                    "Al"
                    "Al"
                    "Al"
                    "Al"
                ]
            }
            "a" {
                "source-value" [
                    3.9149
                    4.0
                    4.032
                    4.0817
                    4.1602
                ]
                "source-unit" "angstrom"
                "digits" 5
            }
        }
    ]
````

For cases where there are multiple keys or a key receives an array of values
computed one at a time, the `kim_property_modify` can be called multiple
times and append values to a given key.

````py
    >>> str = kim_property_create(1, 'cohesive-energy-relation-cubic-crystal')
    >>> str = kim_property_modify(str, 1,
                "key", "short-name",
                "source-value", "1", "fcc",
                "key", "species",
                "source-value", "1:4", "Al", "Al", "Al", "Al",
                "key", "a",
                "source-value", "1:5", "3.9149", "4.0000", "4.032", "4.0817", "4.1602",
                "source-unit", "angstrom", "digits", "5")
    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
            "short-name" {
                "source-value" [
                    "fcc"
                ]
            }
            "species" {
                "source-value" [
                    "Al"
                    "Al"
                    "Al"
                    "Al"
                ]
            }
            "a" {
                "source-value" [
                    3.9149
                    4.0
                    4.032
                    4.0817
                    4.1602
                ]
                "source-unit" "angstrom"
                "digits" 5
            }
        }
    ]
    >>> str = kim_property_modify(str, 1,
                "key", "basis-atom-coordinates",
                "source-value", "2", "1:2", "0.5", "0.5")

    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
            "short-name" {
                "source-value" [
                    "fcc"
                ]
            }
            "species" {
                "source-value" [
                    "Al"
                    "Al"
                    "Al"
                    "Al"
                ]
            }
            "a" {
                "source-value" [
                    3.9149
                    4.0
                    4.032
                    4.0817
                    4.1602
                ]
                "source-unit" "angstrom"
                "digits" 5
            }
            "basis-atom-coordinates" {
                "source-value" [
                    [
                        0.0
                        0.0
                        0.0
                    ]
                    [
                        0.5
                        0.5
                        0.0
                    ]
                ]
            }
        }
    ]
    >>> str = kim_property_modify(str, 1,
                "key", "basis-atom-coordinates",
                "source-value", "3", "1:3", "0.5", "0.0", "0.5",
                "key", "basis-atom-coordinates",
                "source-value", "4", "2:3", "0.5", "0.5")

    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
            "short-name" {
                "source-value" [
                    "fcc"
                ]
            }
            "species" {
                "source-value" [
                    "Al"
                    "Al"
                    "Al"
                    "Al"
                ]
            }
            "a" {
                "source-value" [
                    3.9149
                    4.0
                    4.032
                    4.0817
                    4.1602
                ]
                "source-unit" "angstrom"
                "digits" 5
            }
            "basis-atom-coordinates" {
                "source-value" [
                    [
                        0.0
                        0.0
                        0.0
                    ]
                    [
                        0.5
                        0.5
                        0.0
                    ]
                    [
                        0.5
                        0.0
                        0.5
                    ]
                    [
                        0.0
                        0.5
                        0.5
                    ]
                ]
            }
        }
    ]

    >>> str = kim_property_modify(str, 1,
                "key", "cohesive-potential-energy",
                "source-value", "1:5", "3.324", "3.3576", "3.3600", "3.3550", "3.3260",
                "source-std-uncert-value", "1:5", "0.002", "0.0001", "0.00001", "0.0012", "0.00015",
                "source-unit", "eV",
                "digits", "5")

    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
            "short-name" {
                "source-value" [
                    "fcc"
                ]
            }
            "species" {
                "source-value" [
                    "Al"
                    "Al"
                    "Al"
                    "Al"
                ]
            }
            "a" {
                "source-value" [
                    3.9149
                    4.0
                    4.032
                    4.0817
                    4.1602
                ]
                "source-unit" "angstrom"
                "digits" 5
            }
            "basis-atom-coordinates" {
                "source-value" [
                    [
                        0.0
                        0.0
                        0.0
                    ]
                    [
                        0.5
                        0.5
                        0.0
                    ]
                    [
                        0.5
                        0.0
                        0.5
                    ]
                    [
                        0.0
                        0.5
                        0.5
                    ]
                ]
            }
            "cohesive-potential-energy" {
                "source-value" [
                    3.324
                    3.3576
                    3.36
                    3.355
                    3.326
                ]
                "source-std-uncert-value" [
                    0.002
                    0.0001
                    1e-05
                    0.0012
                    0.00015
                ]
                "source-unit" "eV"
                "digits" 5
            }
        }
    ]
````

### Note

Variables which are introduced with a specified extent of either an empty vector `[]` or `[1]`, are scalars.
Scalars can be set by calling the `kim_property_modify` for the first time.
Any number of other call of `kim_property_modify` does not change the set scalar variables.

For example:

````py
    >>> str = kim_property_create(1, 'cohesive-energy-relation-cubic-crystal')
    >>> str = kim_property_modify(str, 1,
                "key", "space-group",
                "source-value", "Immm")

    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
            "space-group" {
                "source-value" "Immm"
            }
        }
    ]
````

Calling the `kim_property_modify` again does not change the set scalar variable.

````py
    >>> str = kim_property_modify(str, 1,
                "key", "space-group",
                "source-value", "P6_3/mmc")

    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
            "space-group" {
                "source-value" "Immm"
            }
        }
    ]

````

To update these values with the newly computed or obtained value, one has to [remove](#Remove) the key and [modify or set](#Modify) the new value again.

````py
    >>> str = kim_property_remove(str, 1,
                "key", "space-group")

    >>> str = kim_property_modify(str, 1,
                "key", "space-group",
                "source-value", "P6_3/mmc")

    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
            "space-group" {
                "source-value" "P6_3/mmc"
            }
        }
    ]

````

## Remove

Removing (a) key(s) from a property instance::

````py
    >>> str = kim_property_create(1, 'cohesive-energy-relation-cubic-crystal')
    >>> str = kim_property_modify(str, 1,
                "key", "short-name",
                "source-value", "1", "fcc",
                "key", "species",
                "source-value", "1:4", "Al", "Al", "Al", "Al",
                "key", "a",
                "source-value", "1:5", "3.9149", "4.0000", "4.032", "4.0817", "4.1602",
                "source-unit", "angstrom", "digits", "5",
                "key", "basis-atom-coordinates",
                "source-value", "2", "1:2", "0.5", "0.5",
                "key", "basis-atom-coordinates",
                "source-value", "3", "1:3", "0.5", "0.0", "0.5",
                "key", "basis-atom-coordinates",
                "source-value", "4", "2:3", "0.5", "0.5",
                "key", "cohesive-potential-energy",
                "source-value", "1:5", "3.324", "3.3576", "3.3600", "3.3550", "3.3260",
                "source-std-uncert-value", "1:5", "0.002", "0.0001", "0.00001", "0.0012", "0.00015",
                "source-unit", "eV",
                "digits", "5")

    >>> str = kim_property_remove(str, 1, "key", "a", "source-unit")
    >>> str = kim_property_remove(str, 1, "key", "cohesive-potential-energy", "key", "basis-atom-coordinates")

    >>> obj = kim_edn.loads(str)
    >>> print(kim_edn.dumps(obj, indent=4))
    [
        {
            "property-id" "tag:staff@noreply.openkim.org,2014-04-15:property/cohesive-energy-relation-cubic-crystal"
            "instance-id" 1
            "short-name" {
                "source-value" [
                    "fcc"
                ]
            }
            "species" {
                "source-value" [
                    "Al"
                    "Al"
                    "Al"
                    "Al"
                ]
            }
            "a" {
                "source-value" [
                    3.9149
                    4.0
                    4.032
                    4.0817
                    4.1602
                ]
                "digits" 5
            }
        }
    ]
````

## Dump

After validating the generated instance against the property definition,
serializes it to a [KIM-EDN](https://github.com/openkim/kim-edn#kim-edn)
formatted stream to `fp` (a `.write()`-supporting file-like object or a name
string to open a file).

The validation makes sure all keys/arguments are legal and all required keys
are provided.

````py
    >>> str = kim_property_create(1, 'cohesive-energy-relation-cubic-crystal')
    >>> str = kim_property_modify(str, 1,
                "key", "short-name",
                "source-value", "1", "fcc",
                "key", "species",
                "source-value", "1:4", "Al", "Al", "Al", "Al",
                "key", "a",
                "source-value", "1:5", "3.9149", "4.0000", "4.032", "4.0817", "4.1602",
                "source-unit", "angstrom", "digits", "5",
                "key", "basis-atom-coordinates",
                "source-value", "2", "1:2", "0.5", "0.5",
                "key", "basis-atom-coordinates",
                "source-value", "3", "1:3", "0.5", "0.0", "0.5",
                "key", "basis-atom-coordinates",
                "source-value", "4", "2:3", "0.5", "0.5",
                "key", "cohesive-potential-energy",
                "source-value", "1:5", "3.324", "3.3576", "3.3600", "3.3550", "3.3260",
                "source-std-uncert-value", "1:5", "0.002", "0.0001", "0.00001", "0.0012", "0.00015",
                "source-unit", "eV",
                "digits", "5")
    >>> kim_property_dump(str, "results.edn")
````

or

````py
    >>> str = kim_property_create(1, 'cohesive-energy-relation-cubic-crystal')
    >>> str = kim_property_modify(str, 1,
                "key", "short-name",
                "source-value", "1", "fcc",
                "key", "species",
                "source-value", "1:4", "Al", "Al", "Al", "Al",
                "key", "a",
                "source-value", "1:5", "3.9149", "4.0000", "4.032", "4.0817", "4.1602",
                "source-unit", "angstrom", "digits", "5",
                "key", "basis-atom-coordinates",
                "source-value", "2", "1:2", "0.5", "0.5",
                "key", "basis-atom-coordinates",
                "source-value", "3", "1:3", "0.5", "0.0", "0.5",
                "key", "basis-atom-coordinates",
                "source-value", "4", "2:3", "0.5", "0.5",
                "key", "cohesive-potential-energy",
                "source-value", "1:5", "3.324", "3.3576", "3.3600", "3.3550", "3.3260",
                "source-std-uncert-value", "1:5", "0.002", "0.0001", "0.00001", "0.0012", "0.00015",
                "source-unit", "eV",
                "digits", "5")
    >>> obj = kim_edn.loads(str)
    >>> kim_property_dump(obj, "results.edn")
````

or

````py
    >>> str = kim_property_create(1, 'cohesive-energy-relation-cubic-crystal')
    >>> str = kim_property_modify(str, 1,
                "key", "short-name",
                "source-value", "1", "fcc",
                "key", "species",
                "source-value", "1:4", "Al", "Al", "Al", "Al",
                "key", "a",
                "source-value", "1:5", "3.9149", "4.0000", "4.032", "4.0817", "4.1602",
                "source-unit", "angstrom", "digits", "5",
                "key", "basis-atom-coordinates",
                "source-value", "2", "1:2", "0.5", "0.5",
                "key", "basis-atom-coordinates",
                "source-value", "3", "1:3", "0.5", "0.0", "0.5",
                "key", "basis-atom-coordinates",
                "source-value", "4", "2:3", "0.5", "0.5",
                "key", "cohesive-potential-energy",
                "source-value", "1:5", "3.324", "3.3576", "3.3600", "3.3550", "3.3260",
                "source-std-uncert-value", "1:5", "0.002", "0.0001", "0.00001", "0.0012", "0.00015",
                "source-unit", "eV",
                "digits", "5")
    >>> with open("results.edn", 'w') as fp:
            kim_property_dump(str, fp)
````

## Installing kim_property

### Requirements

You need Python 3.6 or later to run `kim_property`. You can have multiple Python
versions (2.x and 3.x) installed on the same system without problems.

To install Python 3 for different Linux flavors, macOS and Windows, packages
are available at\
[https://www.python.org/getit/](https://www.python.org/getit/)

### Using pip

**pip** is the most popular tool for installing Python packages, and the one
included with modern versions of Python.

`kim_property` can be installed with `pip`:

```sh
pip install kim_property
```

#### Note

* Depending on your Python installation, you may need to use `pip3` instead of `pip`.

```sh
pip3 install kim_property
```

* Depending on your configuration, you may have to run `pip` like this:

```sh
python3 -m pip install kim_property
```

### Using pip (GIT Support)

`pip` currently supports cloning over `git`

```sh
pip install git+https://github.com/openkim/kim-property.git
```

* For more information and examples, see the [pip install](https://pip.pypa.io/en/stable/reference/pip_install/#id18) reference.

### Using conda

**conda** is the package management tool for Anaconda Python installations.

Installing `kim_property` from the `conda-forge` channel can be achieved by adding
`conda-forge` to your channels with:

```sh
conda config --add channels conda-forge
```

Once the `conda-forge` channel has been enabled, `kim_property` can be installed
with:

```sh
conda install kim_property
```

It is possible to list all of the versions of `kim_property` available on your
platform with:

```sh
conda search kim_property --channel conda-forge
```

## Copyright

Copyright (c) 2020, Regents of the University of Minnesota.\
All Rights Reserved

## Contributing

Contributors:\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Yaser Afshar
