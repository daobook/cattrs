# cattrs

<a href="https://pypi.python.org/pypi/cattrs"><img src="https://img.shields.io/pypi/v/cattrs.svg"/></a>
<a href="https://github.com/python-attrs/cattrs/actions?workflow=CI"><img src="https://github.com/python-attrs/cattrs/workflows/CI/badge.svg"/></a>
<a href="https://catt.rs/en/latest/?badge=latest"><img src="https://readthedocs.org/projects/cattrs/badge/?version=latest" alt="Documentation Status"/></a>
<a href="https://github.com/python-attrs/cattrs"><img src="https://img.shields.io/pypi/pyversions/cattrs.svg" alt="Supported Python versions"/></a>
<a href="https://codecov.io/gh/python-attrs/cattrs/"><img src="https://codecov.io/gh/python-attrs/cattrs/branch/master/graph/badge.svg"/></a>
<a href="https://github.com/psf/black"><img src="https://img.shields.io/badge/code%20style-black-000000.svg"/></a>

---

**cattrs** is an open source Python library for structuring and unstructuring
data. _cattrs_ works best with _attrs_ classes, dataclasses and the usual
Python collections, but other kinds of classes are supported by manually
registering converters.

Python has a rich set of powerful, easy to use, built-in data types like
dictionaries, lists and tuples. These data types are also the lingua franca
of most data serialization libraries, for formats like json, msgpack, yaml or
toml.

Data types like this, and mappings like `dict` s in particular, represent
unstructured data. Your data is, in all likelihood, structured: not all
combinations of field names or values are valid inputs to your programs. In
Python, structured data is better represented with classes and enumerations.
_attrs_ is an excellent library for declaratively describing the structure of
your data, and validating it.

When you're handed unstructured data (by your network, file system, database...),
_cattrs_ helps to convert this data into structured data. When you have to
convert your structured data into data types other libraries can handle,
_cattrs_ turns your classes and enumerations into dictionaries, integers and
strings.

Here's a simple taste. The list containing a float, an int and a string
gets converted into a tuple of three ints.

```python
>>> import cattrs

>>> cattrs.structure([1.0, 2, "3"], tuple[int, int, int])
(1, 2, 3)
```

_cattrs_ works well with _attrs_ classes out of the box.

```python
>>> from attrs import frozen
>>> import cattrs

>>> @frozen  # It works with non-frozen classes too.
... class C:
...     a: int
...     b: str

>>> instance = C(1, 'a')
>>> cattrs.unstructure(instance)
{'a': 1, 'b': 'a'}
>>> cattrs.structure({'a': 1, 'b': 'a'}, C)
C(a=1, b='a')
```

Here's a much more complex example, involving `attrs` classes with type
metadata.

```python
>>> from enum import unique, Enum
>>> from typing import Optional, Sequence, Union
>>> from cattrs import structure, unstructure
>>> from attrs import define, field

>>> @unique
... class CatBreed(Enum):
...     SIAMESE = "siamese"
...     MAINE_COON = "maine_coon"
...     SACRED_BIRMAN = "birman"

>>> @define
... class Cat:
...     breed: CatBreed
...     names: Sequence[str]

>>> @define
... class DogMicrochip:
...     chip_id = field()  # Type annotations are optional, but recommended
...     time_chipped: float = field()

>>> @define
... class Dog:
...     cuteness: int
...     chip: Optional[DogMicrochip] = None

>>> p = unstructure([Dog(cuteness=1, chip=DogMicrochip(chip_id=1, time_chipped=10.0)),
...                  Cat(breed=CatBreed.MAINE_COON, names=('Fluffly', 'Fluffer'))])

>>> print(p)
[{'cuteness': 1, 'chip': {'chip_id': 1, 'time_chipped': 10.0}}, {'breed': 'maine_coon', 'names': ('Fluffly', 'Fluffer')}]
>>> print(structure(p, list[Union[Dog, Cat]]))
[Dog(cuteness=1, chip=DogMicrochip(chip_id=1, time_chipped=10.0)), Cat(breed=<CatBreed.MAINE_COON: 'maine_coon'>, names=['Fluffly', 'Fluffer'])]
```

Consider unstructured data a low-level representation that needs to be converted
to structured data to be handled, and use `structure`. When you're done,
`unstructure` the data to its unstructured form and pass it along to another
library or module. Use [attrs type metadata](http://attrs.readthedocs.io/en/stable/examples.html#types)
to add type metadata to attributes, so _cattrs_ will know how to structure and
destructure them.

- Free software: MIT license
- Documentation: https://catt.rs
- Python versions supported: 3.7 and up. (Older Python versions, like 2.7, 3.5 and 3.6 are supported by older versions; see the changelog.)

## Features

- Converts structured data into unstructured data, recursively:

  - _attrs_ classes and dataclasses are converted into dictionaries in a way similar to `attrs.asdict`, or into tuples in a way similar to `attrs.astuple`.
  - Enumeration instances are converted to their values.
  - Other types are let through without conversion. This includes types such as
    integers, dictionaries, lists and instances of non-_attrs_ classes.
  - Custom converters for any type can be registered using `register_unstructure_hook`.

- Converts unstructured data into structured data, recursively, according to
  your specification given as a type. The following types are supported:

  - `typing.Optional[T]`.
  - `typing.List[T]`, `typing.MutableSequence[T]`, `typing.Sequence[T]` (converts to a list).
  - `typing.Tuple` (both variants, `Tuple[T, ...]` and `Tuple[X, Y, Z]`).
  - `typing.MutableSet[T]`, `typing.Set[T]` (converts to a set).
  - `typing.FrozenSet[T]` (converts to a frozenset).
  - `typing.Dict[K, V]`, `typing.MutableMapping[K, V]`, `typing.Mapping[K, V]` (converts to a dict).
  - _attrs_ classes with simple attributes and the usual `__init__`.

    - Simple attributes are attributes that can be assigned unstructured data,
      like numbers, strings, and collections of unstructured data.

  - All _attrs_ classes and dataclasses with the usual `__init__`, if their complex attributes have type metadata.
  - `typing.Union` s of supported _attrs_ classes, given that all of the classes have a unique field.
  - `typing.Union` s of anything, given that you provide a disambiguation function for it.
  - Custom converters for any type can be registered using `register_structure_hook`.

_cattrs_ comes with preconfigured converters for a number of serialization libraries, including json, msgpack, bson, yaml and toml.
For details, see the [cattr.preconf package](https://catt.rs/en/stable/preconf.html).

## Additional documentation and talks

- [On structured and unstructured data, or the case for cattrs](https://threeofwands.com/on-structured-and-unstructured-data-or-the-case-for-cattrs/)
- [Why I use attrs instead of pydantic](https://threeofwands.com/why-i-use-attrs-instead-of-pydantic/)
- [cattrs I: un/structuring speed](https://threeofwands.com/why-cattrs-is-so-fast/)
- [Python has a macro language - it's Python (PyCon IT 2022)](https://www.youtube.com/watch?v=UYRSixikUTo)

## Credits

Major credits to Hynek Schlawack for creating [attrs](https://attrs.org) and its predecessor,
[characteristic](https://github.com/hynek/characteristic).

_cattrs_ is tested with [Hypothesis](http://hypothesis.readthedocs.io/en/latest/), by David R. MacIver.

_cattrs_ is benchmarked using [perf](https://github.com/haypo/perf) and [pytest-benchmark](https://pytest-benchmark.readthedocs.io/en/latest/index.html).

This package was created with [Cookiecutter](https://github.com/audreyr/cookiecutter) and the [`audreyr/cookiecutter-pypackage`](https://github.com/audreyr/cookiecutter-pypackage) project template.
