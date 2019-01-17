# CustomArgParse

This is a custom argument parser for Python. The parser essentially extends the `argparse` python package to provide support for providing options via configuration files.

## Compatibility
The tool has been written for Ubuntu and tested on both Python v2.7 and Python v3.5. It is also expected to be compatible with higher versions of python too.

## Dependencies
Requires python2 or python3 and the `argparse` package:

```
pip install argparse
```

## Usage

This can be imported as a utility package in any Python script which requires command-line arguments and also arguments from a configuration file. 

The package contains the `CustomArgumentParser` class which provides `parse_args` and `parse_known_args` methods just like those of the standard `argparse` package. But, the command-line arguments to the program must supply a `--configfile` argument with the location of the configuration file. The configuration file should have a python dictionary named `config` which contains more arguments. Any key inside `config` or inside any nested dictionary in `config` must be of type `string`. The `CustomArgumentParser` class methods then parse the command-line arguments, the config file arguments and merge them with the following override priority:
```
Command-line > configuration file > defaults
```
Specifically, the command-line arguments can override elements of the `config` dictionary in the configuration file. They can even modify only certain inner elements of nested dictionaries in the `config` dictionary by using the dot `(.)` operator.

Both `parse_args` and `parse_known_args` methods return the same outputs as their complements in the `argparse` package, except that the returned namespaces also contain all arguments from the supplied configuration file. 

The package also provides two custom datatypes called `pytuple` and `pylist` which allow: (i) having lists and tuples as values for keys in the `config` dictionary, and (ii) specifying lists and tuples as command-line arguments. Note that lists and tuples can already be specified by using `nargs` in `argparse`. These custom datatypes provide convenient alternatives for the same (but see **Note 4** below). Another additional type `pybool` for `bool` data is also provided (see **Note 5** below).
Finally, the package provides a function called `args_to_dict` which can be used to convert the returned namespace into a dictionary format.

## Example

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

```python
## Main program: main.py ##
if __name__ == '__main__':
    # Create a custom argument parser
    parser = CustomArgumentParser(description='Custom Argument Parser')

    # Add arguments (including a --configfile)
    parser.add_argument('-c', '--configfile')
    parser.add_argument('--model', type=pylist)
    parser.add_argument('--epochs', default=1, type=int)

    # Test parse_known_args()
    args, unknown = parser.parse_known_args()
    print(args)

    # Test args_to_dict
    print('Expanded args: \n{}'.format(args_to_dict(args, expand=True)))
## End: main.py ##
```

```bash
# Command from the command-line
python main.py -c config.py --model '[5, mode, 35.6]' --epochs 5 --arg1 3 --arg2.obj2 bar [other arguments added to the parser]
```

The example shows how one can specify a configuration file with `--configfile` argument, add a `pylist` type argument which can accept lists with elements of type `int`, `float` or `str` (see **Note 4** below) and other regular arguments like `--epochs` too. Optionally one can also override some of the arguments in the configuration file like `--arg1` or even nested arguments like `--arg2.obj2` with the dot `(.)` operator.

## Notes

**Note 1**: The module infers the data type of arguments added from `config`, by the values provided for those arguments in `config`. However, if an argument in `config` is expected to accept values of multiple different types, the module cannot infer this and hence only allows values of the inferred type for such arguments.

**Note 2**: If the value of an argument in `config` is `None`, the module does not infer its type to be `NoneType` but rather as `str` and hence allows overriding that argument with string values from the command-line.

**Note 3**: It is recommended to not add arguments with the same name to both `config` and the CustomArgumentParser object, since the arguments from both sources will be combined internally. If an argument with the same name is added to both `config` and the CustomArgumentParser object, the argument will have its data type inferred from the value provided in `config`, however the value retained will be that provided via the command-line (unless it is skipped, in which case the value from `config` is retained). In such cases, if the value provided via the command-line cannot be interpreted as the data type inferred from `config`, an error occurs.

**Note 4**: The `pylist` and `pytuple` datatypes are more flexible than specifying lists/tuples with `nargs` since they allow elements of different types in the list/tuple, unlike `nargs` which requires the same type for each element. Currently `pylist` and `pytuple` only support a combination of `int`, `float` and `str` type elements. A `pylist`/`pytuple` type argument is specified by a comma-separated list of attributes which can additionally be enclosed in round or square brackets. The commas should not be separated by spaces on the command-line unless the full comma-separated list (along with brackets, if any) is enclosed in single quotes. A cast to `int` is attempted on each element of the comma-separated list, failing which a cast to `float` is attempted, failing which the element is retained as a string.

**Note 5**: The `pybool` datatype allows proper processing of boolean variables from configuration files. If specified as the datatype of a command-line argument, it additionally allows using capital/small variants of `no`, `false`, `f`, `n` and `0` in place of False and `yes`, `true`, `t`, `y`, `1` in place of True.