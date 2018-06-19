# CustomArgParse

This is a custom argument parser for Python v3.5 and above. The parser essentially extends the `argparse` python package to provide support for providing options via configuration files.

## Compatibility
The tool has been written for Ubuntu and tested on Python v3.5 and above but may be compatible with other versions of python too.

## Dependencies
Requires python3 and the `argparse` package:

```
pip install argparse
```

## Usage

This can be imported as a utility package in any Python script which requires command-line arguments and also arguments from a configuration file. 

The package contains the `CustomArgumentParser` class which provides `parse_args` and `parse_known_args` methods just like those of the standard `argparse` package. But, the command-line arguments to the program must supply a `-c` or a `--configfile` argument with the location of the configuration file. The configuration file should have a python dictionary named `config` which contains more arguments. Any key inside `config` or inside any nested dictionary in `config` must be of type `string`. The `CustomArgumentParser` class methods then parse the command-line arguments, the config file arguments and merge them with the following override priority:
```
Command-line > configuration file > defaults
```
Specifically, the command-line arguments can override elements of the `config` dictionary in the configuration file. They can even modify only certain inner elements of nested dictionaries in the `config` dictionary by using the dot(.) operator, e.g.:

```python
## Configuration file: config.py ##
config = {
    'arg1': 4,
    'arg2': {
        'obj1': [3,4],
        'obj2': 'foo'
    }
}
## End: config.py ##
```

```bash
python3 <main_program.py> -c config.py --arg1 3 --arg2.obj2 bar [other arguments added to the parser]
```

Both `parse_args` and `parse_known_args` methods return the same outputs as their complements in the `argparse` package, except that the returned namespaces also contain all arguments from the supplied configuration file. Finally, the package provides a function called `args_to_dict` which can be used to convert the returned namespace into a dictionary format.