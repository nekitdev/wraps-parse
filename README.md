# `wraps-parse`

[![License][License Badge]][License]
[![Version][Version Badge]][Package]
[![Downloads][Downloads Badge]][Package]
[![Discord][Discord Badge]][Discord]

[![Documentation][Documentation Badge]][Documentation]
[![Check][Check Badge]][Actions]
[![Test][Test Badge]][Actions]
[![Coverage][Coverage Badge]][Coverage]

> *Parsing feature of wraps.*

## Installing

**Python 3.8 or above is required.**

### `pip`

Installing the library with `pip` is quite simple:

```console
$ pip install wraps-parse
```

Alternatively, the library can be installed from the source:

```console
$ pip install git+https://github.com/nekitdev/wraps-parse.git
```

Or via cloning the repository:

```console
$ git clone https://github.com/nekitdev/wraps-parse.git
$ cd wraps-parse
$ pip install .
```

### `uv`

You can add `wraps-parse` as a dependency with the following command:

```console
$ uv add wraps-parse
```

## Examples

### Format

Implementing [`ToString`][wraps_parse.format.ToString] requires defining
the [`to_string`][wraps_parse.format.ToString.to_string]
(and optionally [`to_short_string`][wraps_parse.format.ToString.to_short_string]) method:

```python
# format.py

from attrs import field, frozen
from attrs.validators import ge
from wraps_parse import ToString

is_digits = str.isdigit  # will be used later


@frozen()
class Integer(ToString):
    value: int = field(validator=ge(0))

    def to_string(self) -> str:
        return str(self.value)
```

### Simple

Implementing [`SimpleFromString`][wraps_parse.simple.SimpleFromString] requires defining
the [`from_string`][wraps_parse.simple.SimpleFromString.from_string] method returning an
[`Option[Self]`][wraps_core.option.Option]:

```python
# simple.py

from attrs import frozen
from typing_extensions import Self
from wraps_core import NULL, Option, Some
from wraps_parse import SimpleFromString

from format import Integer, is_digits


@frozen()
class SimpleInteger(SimpleFromString, Integer):
    @classmethod
    def from_string(cls, string: str) -> Option[Self]:
        return Some(cls(int(string))) if is_digits(string) else NULL
```

And using it:

```python
>>> from simple import SimpleInteger
>>> SimpleInteger.from_string("42")
Some(SimpleInteger(value=42))
>>> SimpleInteger.from_string("foo")
Null()
```

The [`parse`][wraps_parse.simple.SimpleFromString.parse] method is provided to raise
[`SimpleParseError`][wraps_parse.simple.SimpleParseError] if parsing fails:

```python
>>> from simple import SimpleInteger
>>> SimpleInteger.parse("42")
SimpleInteger(value=42)
>>> SimpleInteger.parse("foo")
Traceback (most recent call last):
  ...
wraps_parse.simple.SimpleParseError: parsing `foo` into `SimpleInteger` failed
```

### Normal

The [`FromString`][wraps_parse.normal.FromString] protocol is similar to
[`SimpleFromString`][wraps_parse.simple.SimpleFromString], except
[`Result[Self, E]`][wraps_core.result.Result] (where `E` is the error type)
is returned from [`from_string`][wraps_parse.normal.FromString.from_string] instead.

Below is the enhanced version of the code above:

```python
# normal.py

from attrs import frozen
from typing_extensions import Self
from wraps_parse import FromString
from wraps_core import Error, Ok, Result

from format import Integer, is_digits

NON_DIGIT = "non-digit character found"


@frozen()
class NormalInteger(FromString[str], Integer):  # `E` is `str`
    @classmethod
    def from_string(cls, string: str) -> Result[Self, str]:
        return Ok(cls(int(string))) if is_digits(string) else Error(NON_DIGIT)
```

And using it:

```python
>>> from normal import NormalInteger
>>> NormalInteger.from_string("13")
Ok(NormalInteger(value=13))
>>> NormalInteger.from_string("bar")
Error("non-digit character found")
```

The [`parse`][wraps_parse.normal.FromString.parse] method is provided to raise
[`ParseError`][wraps_parse.normal.ParseError] if parsing fails:

```python
>>> from normal import NormalInteger
>>> NormalInteger.parse("13")
NormalInteger(value=13)
>>> NormalInteger.parse("bar")
Traceback (most recent call last):
  ...
wraps_parse.normal.ParseError: parsing `bar` into `NormalInteger` failed (non-digit character found)
```

### Note

Please note that [`SimpleFromString`][wraps_parse.simple.SimpleFromString] and
[`FromString`][wraps_parse.normal.FromString] are intentionally incompatible with each other.

One can implement either, not both.

Also, when implementing both parsing and formatting, make sure the methods implemented are
inverses of each other.

## Documentation

You can find the documentation [here][Documentation].

## Support

If you need support with the library, you can send us an [email][Email]
or refer to the official [Discord server][Discord].

## Changelog

You can find the changelog [here][Changelog].

## Security Policy

You can find the Security Policy of `wraps-parse` [here][Security].

## Contributing

If you are interested in contributing to `wraps-parse`, make sure to take a look at the
[Contributing Guide][Contributing Guide], as well as the [Code of Conduct][Code of Conduct].

## License

`wraps-parse` is licensed under the MIT License terms. See [License][License] for details.

[Email]: mailto:support@nekit.dev

[Discord]: https://nekit.dev/chat

[Actions]: https://github.com/nekitdev/wraps-parse/actions

[Changelog]: https://github.com/nekitdev/wraps-parse/blob/main/CHANGELOG.md
[Code of Conduct]: https://github.com/nekitdev/wraps-parse/blob/main/CODE_OF_CONDUCT.md
[Contributing Guide]: https://github.com/nekitdev/wraps-parse/blob/main/CONTRIBUTING.md
[Security]: https://github.com/nekitdev/wraps-parse/blob/main/SECURITY.md

[License]: https://github.com/nekitdev/wraps-parse/blob/main/LICENSE

[Package]: https://pypi.org/project/wraps-parse
[Coverage]: https://codecov.io/gh/nekitdev/wraps-parse
[Documentation]: https://nekitdev.github.io/wraps-parse

[Discord Badge]: https://img.shields.io/discord/728012506899021874
[License Badge]: https://img.shields.io/pypi/l/wraps-parse
[Version Badge]: https://img.shields.io/pypi/v/wraps-parse
[Downloads Badge]: https://img.shields.io/pypi/dm/wraps-parse

[Documentation Badge]: https://github.com/nekitdev/wraps-parse/workflows/docs/badge.svg
[Check Badge]: https://github.com/nekitdev/wraps-parse/workflows/check/badge.svg
[Test Badge]: https://github.com/nekitdev/wraps-parse/workflows/test/badge.svg
[Coverage Badge]: https://codecov.io/gh/nekitdev/wraps-parse/branch/main/graph/badge.svg

[wraps_core.option.Option]: https://nekitdev.github.io/wraps-core/reference/option#wraps_core.option.Option
[wraps_core.result.Result]: https://nekitdev.github.io/wraps-core/reference/result#wraps_core.result.Result

[wraps_parse.simple.SimpleFromString]: https://nekitdev.github.io/wraps-parse/reference/simple#wraps_parse.simple.SimpleFromString
[wraps_parse.simple.SimpleParseError]: https://nekitdev.github.io/wraps-parse/reference/simple#wraps_parse.simple.SimpleParseError
[wraps_parse.simple.SimpleFromString.from_string]: https://nekitdev.github.io/wraps-parse/reference/simple#wraps_parse.simple.SimpleFromString.from_string
[wraps_parse.simple.SimpleFromString.parse]: https://nekitdev.github.io/wraps-parse/reference/simple#wraps_parse.simple.SimpleFromString.parse

[wraps_parse.normal.FromString]: https://nekitdev.github.io/wraps-parse/reference/normal#wraps_parse.normal.FromString
[wraps_parse.normal.ParseError]: https://nekitdev.github.io/wraps-parse/reference/normal#wraps_parse.normal.ParseError
[wraps_parse.normal.FromString.from_string]: https://nekitdev.github.io/wraps-parse/reference/normal#wraps_parse.normal.FromString.from_string
[wraps_parse.normal.FromString.parse]: https://nekitdev.github.io/wraps-parse/reference/normal#wraps_parse.normal.FromString.parse

[wraps_parse.format.ToString]: https://nekitdev.github.io/wraps-parse/reference/format#wraps_parse.format.ToString
[wraps_parse.format.ToString.to_string]: https://nekitdev.github.io/wraps-parse/reference/format#wraps_parse.format.ToString.to_string
[wraps_parse.format.ToString.to_short_string]: https://nekitdev.github.io/wraps-parse/reference/format#wraps_parse.format.ToString.to_short_string
