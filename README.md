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

## Notes

**Note 1**: The module infers the data type of arguments added from `config`, by the values provided for those arguments in `config`. However, if an argument in `config` is expected to accept values of multiple different types, the module cannot infer this and hence only allows values of the inferred type for such arguments.

**Note 2**: If the value of an argument in `config` is `None`, the module does not infer its type to be `NoneType` but rather as `str` and hence allows overriding that argument with string values from the command-line.

**Note 3**: It is recommended to not add arguments with the same name to both `config` and the CustomArgumentParser object, since the arguments from both sources will be combined internally. If an argument with the same name is added to both `config` and the CustomArgumentParser object, the argument will have its data type inferred from the value provided in `config`, however the value retained will be that provided via the command-line (unless it is skipped, in which case the value from `config` is retained). In such cases, if the value provided via the command-line cannot be interpreted as the data type inferred from `config`, an error occurs.