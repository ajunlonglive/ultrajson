# UltraJSON

[![PyPI version](https://img.shields.io/pypi/v/ujson.svg?logo=pypi&logoColor=FFE873)](https://pypi.org/project/ujson)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/ujson.svg?logo=python&logoColor=FFE873)](https://pypi.org/project/ujson)
[![PyPI downloads](https://img.shields.io/pypi/dm/ujson.svg)](https://pypistats.org/packages/ujson)
[![GitHub Actions status](https://github.com/ultrajson/ultrajson/workflows/Test/badge.svg)](https://github.com/ultrajson/ultrajson/actions)
[![codecov](https://codecov.io/gh/ultrajson/ultrajson/branch/main/graph/badge.svg)](https://codecov.io/gh/ultrajson/ultrajson)
[![DOI](https://zenodo.org/badge/1418941.svg)](https://zenodo.org/badge/latestdoi/1418941)
[![Code style: Black](https://img.shields.io/badge/code%20style-Black-000000.svg)](https://github.com/psf/black)

UltraJSON is an ultra fast JSON encoder and decoder written in pure C with bindings for
Python 3.7+.

Install with pip:

```sh
python -m pip install ujson
```

## Usage

May be used as a drop in replacement for most other JSON parsers for Python:

```pycon
>>> import ujson
>>> ujson.dumps([{"key": "value"}, 81, True])
'[{"key":"value"},81,true]'
>>> ujson.loads("""[{"key": "value"}, 81, true]""")
[{'key': 'value'}, 81, True]
```

### Encoder options

#### encode_html_chars

Used to enable special encoding of "unsafe" HTML characters into safer Unicode
sequences. Default is `False`:

```pycon
>>> ujson.dumps("<script>John&Doe", encode_html_chars=True)
'"\\u003cscript\\u003eJohn\\u0026Doe"'
```

#### ensure_ascii

Limits output to ASCII and escapes all extended characters above 127. Default is `True`.
If your end format supports UTF-8, setting this option to false is highly recommended to
save space:

```pycon
>>> ujson.dumps("åäö")
'"\\u00e5\\u00e4\\u00f6"'
>>> ujson.dumps("åäö", ensure_ascii=False)
'"åäö"'
```

#### escape_forward_slashes

Controls whether forward slashes (`/`) are escaped. Default is `True`:

```pycon
>>> ujson.dumps("http://esn.me")
'"http:\\/\\/esn.me"'
>>> ujson.dumps("http://esn.me", escape_forward_slashes=False)
'"http://esn.me"'
```

#### indent

Controls whether indentation ("pretty output") is enabled. Default is `0` (disabled):

```pycon
>>> ujson.dumps({"foo": "bar"})
'{"foo":"bar"}'
>>> print(ujson.dumps({"foo": "bar"}, indent=4))
{
    "foo":"bar"
}
```

## Benchmarks

*UltraJSON* calls/sec compared to other popular JSON parsers with performance gain
specified below each.

### Test machine

Linux 5.0.0-1032-azure x86_64 #34-Ubuntu SMP Mon Feb 10 19:37:25 UTC 2020

### Versions

- CPython 3.8.2 (default, Feb 28 2020, 14:28:43) [GCC 7.4.0]
- nujson    : 1.35.2
- orjson    : 2.6.1
- simplejson: 3.17.0
- ujson     : 2.0.2

|                                                                               | ujson      | nujson     | orjson     | simplejson | json       |
|-------------------------------------------------------------------------------|-----------:|-----------:|-----------:|-----------:|-----------:|
| Array with 256 doubles                                                        |            |            |            |            |            |
| encode                                                                        |     22,082 |      4,282 |     76,975 |      5,328 |      5,436 |
| decode                                                                        |     24,127 |     34,349 |     29,059 |     14,174 |     13,822 |
| Array with 256 UTF-8 strings                                                  |            |            |            |            |            |
| encode                                                                        |      3,557 |      2,528 |     24,300 |      3,061 |      2,068 |
| decode                                                                        |      2,030 |      2,490 |        931 |        406 |        358 |
| Array with 256 strings                                                        |            |            |            |            |            |
| encode                                                                        |     39,041 |     31,769 |     76,403 |     16,615 |     16,910 |
| decode                                                                        |     25,185 |     24,287 |     34,437 |     32,388 |     27,999 |
| Medium complex object                                                         |            |            |            |            |            |
| encode                                                                        |     10,382 |     11,427 |     32,995 |      3,959 |      5,275 |
| decode                                                                        |      9,785 |      9,796 |     11,515 |      5,898 |      7,200 |
| Array with 256 True values                                                    |            |            |            |            |            |
| encode                                                                        |    114,341 |    101,039 |    344,256 |     62,382 |     72,872 |
| decode                                                                        |    149,367 |    151,615 |    181,123 |    114,597 |    130,392 |
| Array with 256 dict{string, int} pairs                                        |            |            |            |            |            |
| encode                                                                        |     13,715 |     14,420 |     51,942 |      3,271 |      6,584 |
| decode                                                                        |     12,670 |     11,788 |     12,176 |      6,743 |      8,278 |
| Dict with 256 arrays with 256 dict{string, int} pairs                         |            |            |            |            |            |
| encode                                                                        |         50 |         54 |        216 |         10 |         23 |
| decode                                                                        |         32 |         32 |         30 |         20 |         23 |
| Dict with 256 arrays with 256 dict{string, int} pairs, outputting sorted keys |            |            |            |            |            |
| encode                                                                        |         46 |         41 |            |          8 |         24 |
| Complex object                                                                |            |            |            |            |            |
| encode                                                                        |        533 |        582 |            |        408 |        431 |
| decode                                                                        |        466 |        454 |            |        154 |        164 |

## Build options

For those with particular needs, such as Linux distribution packagers, several
build options are provided in the form of environment variables.

### Debugging symbols

#### UJSON_BUILD_NO_STRIP

By default, debugging symbols are stripped on Linux platforms. Setting this
environment variable with a value of `1` or `True` disables this behavior.

### Using an external or system copy of the double-conversion library

These two environment variables are typically used together, something like:

```sh
export UJSON_BUILD_DC_INCLUDES='/usr/include/double-conversion'
export UJSON_BUILD_DC_LIBS='-ldouble-conversion'
```

Users planning to link against an external shared library should be aware of
the ABI-compatibility requirements this introduces when upgrading system
libraries or copying compiled wheels to other machines.

#### UJSON_BUILD_DC_INCLUDES

One or more directories, delimited by `os.pathsep` (same as the `PATH`
environment variable), in which to look for `double-conversion` header files;
the default is to use the bundled copy.

#### UJSON_BUILD_DC_LIBS

Compiler flags needed to link the `double-conversion` library; the default
is to use the bundled copy.
