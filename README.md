[![license](https://img.shields.io/github/license/RedisJSON/RedisJSON-py.svg)](https://github.com/RedisJSON/redisjson-py/blob/master/LICENSE)
[![CircleCI](https://circleci.com/gh/RedisJSON/redisjson-py/tree/master.svg?style=svg)](https://circleci.com/gh/RedisJSON/redisjson-py/tree/master)
[![pypi](https://badge.fury.io/py/rejson.svg)](https://badge.fury.io/py/rejson)
[![PyVersions](https://img.shields.io/pypi/pyversions/rejson.svg)](https://github.com/RedisJSON/redisjson-py)
[![GitHub issues](https://img.shields.io/github/release/RedisJSON/redisjson-py.svg)](https://github.com/RedisJSON/redisjson-py/releases/latest)
[![Codecov](https://coveralls.io/repos/github/RedisLabs/rejson-py/badge.svg?branch=master)](https://coveralls.io/github/RedisLabs/rejson-py?branch=master)
[![Known Vulnerabilities](https://snyk.io/test/github/RedisJSON/redisjson-py/badge.svg?targetFile=pyproject.toml)](https://snyk.io/test/github/RedisJSON/redisjson-py?targetFile=pyproject.toml)

# RedisJSON Python Client

[![Forum](https://img.shields.io/badge/Forum-RedisJSON-blue)](https://forum.redislabs.com/c/modules/redisjson)
[![Discord](https://img.shields.io/discord/697882427875393627?style=flat-square)](https://discord.gg/QUkjSsk)

## Deprecation notice

As of [redis-py 4.0.0](https://pypi.org/project/redis/4.0.0) this library is deprecated. It's features have been merged into redis-py. Please either install it [from pypy](https://pypi.org/project/redis) or [the repo](https://github.com/redis/redis-py).

--------------------------------

rejson-py is a package that allows storing, updating and querying objects as
JSON documents in a [Redis](https://redis.io) database that is extended with the
[ReJSON module](https://github.com/redislabsmodules/rejson). The package extends
[redis-py](https://github.com/andymccurdy/redis-py)'s interface with ReJSON's
API, and performs on-the-fly serialization/deserialization of objects to/from
JSON.

## Installation

```bash
$ pip install rejson
```

## Development

1. Create a virtualenv to manage your python dependencies, and ensure it's active.
   ```virtualenv -v venv```
2. Install [pypoetry](https://python-poetry.org/) to manage your dependencies.
   ```pip install --user poetry```
3. Install dependencies.
   ```poetry install```

[tox](https://tox.readthedocs.io/en/latest/) runs all tests as its default target. Running *tox* by itself will run unit tests. Ensure you have a running redis, with the module loaded.

## Usage example

```python
   from rejson import Client, Path

   rj = Client(host='localhost', port=6379, decode_responses=True)

   # Set the key `obj` to some object
   obj = {
       'answer': 42,
       'arr': [None, True, 3.14],
       'truth': {
           'coord': 'out there'
       }
   }
   rj.jsonset('obj', Path.rootPath(), obj)

   # Get something
   print 'Is there anybody... {}?'.format(
       rj.jsonget('obj', Path('.truth.coord'))
   )

   # Delete something (or perhaps nothing), append something and pop it
   rj.jsondel('obj', Path('.arr[0]'))
   rj.jsonarrappend('obj', Path('.arr'), 'something')
   print '{} popped!'.format(rj.jsonarrpop('obj', Path('.arr')))

   # Update something else
   rj.jsonset('obj', Path('.answer'), 2.17)

   # And use just like the regular redis-py client
   jp = rj.pipeline()
   jp.set('foo', 'bar')
   jp.jsonset('baz', Path.rootPath(), 'qaz')
   jp.execute()

   # If you use non-ascii character in your JSON data, you can add the no_escape flag to JSON.GET command
   obj_non_ascii = {
     'non_ascii_string': 'hyvää'
   }
   rj.jsonset('non-ascii', Path.rootPath(), obj_non_ascii)
   print '{} is a non-ascii string'.format(rj.jsonget('non-ascii', Path('.non_ascii_string'), no_escape=True))
```

## Encoding/Decoding

rejson-py uses Python's [json](https://docs.python.org/2/library/json.html).
The client can be set to use custom encoders/decoders at creation, or by calling
explicitly the [setEncoder()](./API.md#setencoder) and
[setDecoder()](./API.md#setencoder) methods, respectively.

The following shows how to use this for a custom class that's stored as
a JSON string for example:

```python

   from json import JSONEncoder, JSONDecoder
   from rejson import Client

   class CustomClass(object):
       "Some non-JSON-serializable"
       def __init__(self, s=None):
           if s is not None:
               # deserialize the instance from the serialization
               if s.startswith('CustomClass:'):
                   ...
               else:
                   raise Exception('unknown format')
           else:
               # initialize the instance
               ...

       def __str__(self):
           _str = 'CustomClass:'
           # append the instance's state to the serialization
           ...
           return _str

       ...

   class CustomEncoder(JSONEncoder):
       "A custom encoder for the custom class"
       def default(self, obj):
           if isinstance(obj, CustomClass):
               return str(obj)
           return json.JSONEncoder.encode(self, obj)

   class TestDecoder(JSONDecoder):
       "A custom decoder for the custom class"
       def decode(self, obj):
           d = json.JSONDecoder.decode(self, obj)
           if isinstance(d, basestring) and d.startswith('CustomClass:'):
               return CustomClass(d)
           return d

   # Create a new instance of CustomClass
   obj = CustomClass()

   # Create a new client with the custom encoder and decoder
   rj = Client(encoder=CustomEncoder(), decoder=CustomDecoder())

   # Store the object
   rj.jsonset('custom', Path.rootPath(), obj))

   # Retrieve it
   obj = rj.jsonget('custom', Path.rootPath())
```

## API

As rejson-py exposes the same methods as redis-py, it can be used as a drop-in
replacement. On top of Redis' core commands, the client also adds ReJSON's
vocabulary and a couple of helper methods. These are documented in the
[API.md](API.md) file, which can be generated by running:

```bash
$ python gendoc rejson > API.md
```

For complete documentation about ReJSON's commands, refer to [ReJSON's website](http://rejson.io).

## License

[BSD 2-Clause](https://github.com/RedisLabs/rejson-py/blob/master/LICENSE)
