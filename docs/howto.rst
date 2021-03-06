.. _guide:

======
How to
======


.. _installation:

Installation
============

With pip
--------

.. code-block:: bash

    $ pip install etcaetera

With setuptools
---------------

.. code-block:: bash

    $ git clone git@github.com:oleiade/etcaetera
    $ cd etcaetera
    $ python setup.py install


.. _usage:

Usage
=====

.. _dive:

Dive
----

A real world example is worth a thousand words

.. code-block:: python

    >>> from etcaetera.config import Config
    >>> from etcaetera.adapters import Defaults, Module, Overrides, Env, File

    # Let's create a new configuration object
    >>> config = Config()

    # And create a bunch of adapters
    >>> env_adapter = Env(keys=["MY_FIRST_SETTING", "MY_SECOND_SETTING"])
    >>> python_file_adapter = File('/etc/my/python/settings.py')
    >>> json_file_adapter = File('/etc/my_json_settings.json')
    >>> module_adapter = Module(os)
    >>> overrides = Overrides({"MY_FIRST_SETTING": "my forced value"})

    # Let's register them
    >>> config.register(env_adapter, python_file_adapter, json_file_adapter, module_adapter, overrides)

    # Load configuration
    >>> config.load()


    # And that's it
    >>> print config
    {
        "MY_FIRST_SETTING": "my forced value",
        "MY_SECOND_SETTING": "my second value",
        "FIRST_YAML_SETTING": "first yaml setting value found in yaml settings",
        "FIRST_JSON_SETTING": "first json setting value found in json settings",
        ...
    }


.. _config_object:

Config object
-------------

The config object is the central place for your whole application settings. It loads your adapters in the order you've registered them, and updates itself using it's data.

Please note that **Defaults** adapter will always be loaded first, and **Overrides** will always be loaded last.

.. code-block:: python

    >>> from etcaetera.config import Config
    >>> from etcaetera.adapters import Defaults, Module, Overrides, Env, File

    # Let's create a new configuration object
    >>> config = Config()

    # And create a bunch of adapters
    >>> env_adapter = Env(keys=["MY_FIRST_SETTING", "MY_SECOND_SETTING"])
    >>> python_file_adapter = File('/etc/my/python/settings.py')
    >>> json_file_adapter = File('/etc/my_json_settings.json')
    >>> module_adapter = Module(os)
    >>> overrides = Overrides({"MY_FIRST_SETTING": "my forced value"})

    # Let's register them
    >>> config.register(env_adapter, python_file_adapter, json_file_adapter, module_adapter, overrides)

    # Load configuration
    >>> config.load()


    # And that's it
    >>> print config
    {
        "MY_FIRST_SETTING": "my forced value",
        "MY_SECOND_SETTING": "my second value",
        "FIRST_YAML_SETTING": "first yaml setting value found in yaml settings",
        "FIRST_JSON_SETTING": "first json setting value found in json settings",
        ...
>>>>>>> release/0.2.0
    }

.. _adapters:

Adapters
--------

Adapters are the interfaces with configuration sources. They load settings from their custom source type,
and they expose them as a normalized dict to *Config* objects.

Right now, etcaetera provides the following adapters:
    * *Defaults*: sets some default settings
    * *Overrides*: overrides the config settings values
    * *Env*: extracts configuration values from system environment
    * *File*: extracts configuration values from a file. Accepted format are: json, yaml, python module file (see *File adapter* section for more details)
    * *Module*: extracts configuration values from a python module. Like in django, only uppercased variables will be matched

In a close future, etcaetera may provide adapters for:
    * *Argv* argparse format support: would load settings from an argparser parser attributes
    * *File* ini format support: would load settings from an ini file

.. _defaults:

Defaults adapter
~~~~~~~~~~~~~~~~

Defaults adapter provides your configuration object with default values.
It will always be evaluated first when ``Config.load`` method is called.
You can whether provide defaults values to *Config* as a *Defaults* object
or as a dictionary.

.. code-block:: python

    >>> from etcaetera.adapter import Defaults

    # Defaults adapter provides default configuration settings
    >>> defaults = Defaults({"ABC": "123"})
    >>> config = Config(defaults)

    >>> print config
    {
        "ABC": "123"
    }


.. _overrides:

Overrides adapter
~~~~~~~~~~~~~~~~~

The Overrides adapter overrides *Config* object values with it's own values.
It will always be evaluated last when the ``Config.load`` method is called.

.. code-block:: python

    >>> from etcaetera.adapter import Overrides

    # The Overrides adapter helps you set overriding configuration settings.
    # When registered over a Config objects, it will always be evaluated last.
    # Use it if you wish to force some config values.
    >>> overrides_adapter = Overrides({"USER": "overrided value"})
    >>> config = Config({
        "USER": "default_value",
        "FIRST_SETTING": "first setting value"
    })

    >>> config.register(overrides_default)
    >>> config.load()

    >>> print config
    {
        "USER": "overrided user",
        "FIRST_SETTING": "first setting value"
    }


.. _env:

Env adapter
~~~~~~~~~~~

The Env adapter loads settings from your system environement.
It should come with a list of keys to fetch. If you don't provide the keys yourself,
the parent *Config* object will automatically provide it's own.

.. code-block:: python

    >>> from etcaetera.adapter import Env

    # You can provide keys to be fetched by the adapter at construction
    >>> env = Env(keys=["USER", "PATH"])

    # Or whenever you call load over it. They will be merged
    # with those provided at initialization.
    >>> env.load(keys=["PWD"])

    >>> print env.data
    {
        "USER": "user extracted from environment",
        "PATH": "path extracted from environment",
        "PWD": "pwd extracted from environment"
    }


.. _file:

File adapter
~~~~~~~~~~~~

The File adapter will load the configuration settings from a file.
Supported formats are json, yaml and python module files. Every key-value pairs
stored in the pointed file will be loaded in the *Config* object it is registered to.


Python module files
```````````````````

The Python module files should be in the same format as the Django settings files. Only uppercased variables
will be loaded. Any python data structures can be used.

*Here's an example*

*Given the following settings.py file*

.. code-block:: bash

    $ cat /my/settings.py
    FIRST_SETTING = 123
    SECOND_SETTING = "this is the second value"
    THIRD_SETTING = {"easy as": "do re mi"}
    ignored_value = "this will be ignore"

*File adapter output will look like this*:

.. code-block:: python

    >>> from etcaetera.adapter import File

    >>> file = File('/my/settings.py')
    >>> file.load()

    >>> print file.data
    {
        "FIRST_SETTING": 123,
        "SECOND_SETTING": "this is the second value",
        "THIRD_SETTING": {"easy as": "do re mi"}
    }

Serialized files (aka json and yaml)
````````````````````````````````````

*Given the following json file content*:

.. code-block:: bash

    $ cat /my/json/file.json
    {
        "FIRST_SETTING": "first json file extracted setting",
        "SECOND_SETTING": "second json file extracted setting"
    }

*The File adapter output will look like this*:

.. code-block:: python

    >>> from etcaetera.adapter import File

    # The File adapter awaits on a file path at construction.
    # All you have to do then, is to let the magic happen
    >>> file = File('/my/json/file.json')
    >>> file.load()

    >>> print file.data
    {
        "FIRST_SETTING": "first json file extracted setting",
        "SECOND_SETTING": "second json file extracted setting"
    }


.. _module:

Module adapter
~~~~~~~~~~~~~~

The Module adapter will load settings from a python module. It emulates the django
settings module loading behavior, so that every uppercased locals of the module is matched.

**Given a mymodule.settings module looking this**:

.. code-block:: python

    MY_FIRST_SETTING = 123
    MY_SECOND_SETTING = "abc"

**Loaded module data will look like this**:

.. code-block:: python

    >>> from etcaetera.adapter import Module

    # It will extract all of the module's uppercased local variables
    >>> module = Module(mymodule.settings)
    >>> module.load()

    >>> print module.data
    {
        MY_FIRST_SETTING = 123
        MY_SECOND_SETTING = "abc"
    }

