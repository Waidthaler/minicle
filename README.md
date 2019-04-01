# minicle v1.0.5

Minicle is Node module for easily processing command line switches and arguments.

There are scads of CLI argument processors out there, so why another one? Mostly 
because the others aim to do too much and can be a pain to use when you just 
need something quick and simple. All it does is parse CLI options. It doesn't 
handle exotic edge cases, generate usage information, validate arguments, or 
anything else.

NEW in v1.0.5: Minicle now supports GNU-style `--` end-of-switches.

## Table of Contents

* [Basic Usage](#basic-usage)
* [Git-Style Subcommands](#subcommands)
* [Todo](#todo)
* [Changelog](#changelog)

<a name="basic-usage"></a>
## Basic Usage Example
```javascript
var parse = require("minicle");

var optionMap = {
    infile:     { short: "i", vals: [ ] },
    outfile:    { short: "s", vals: [ ], max: 1 },
    verbose:    { short: "v", cnt: 0 },
    "@general": { vals: [ ] }
};

parse(optionMap);
```

The `parse` function takes `optionMap` as its sole argument, which is an object 
containing the command line switches and their parameters. It is altered in 
place.

The keys of the object represent the long form, so `infile` becomes `--infile`. 
The associated value is an object with the switch parameters. The `short` member 
specifies the short form of the switch, e.g., `short: "i"` creates `-i`. 
Switches with a `vals` member containing an array accumulate arguments.

In the case of `infile`, the user can specify `--infile` or `-i`, and any 
non-switch tokens following it will be pushed onto the `vals` array until the next
switch or the end of the CLI arguments. If the optional `max` member is provided,
it specifies a maximum number of arguments, after which further tokens are placed
in the `vals` array of the special `@general` member. If no `@general` member
exists, one will be created.

The following command lines are equivalent:

```javascript
myprog -i foo -i bar
myprog --infile foo --infile bar
myprog --infile foo bar

// will yield:

var optionMap = {
    infile:     { short: "i", vals: [ "foo", "bar" ] },
    outfile:    { short: "s", vals: [ ], max: 1 },
    verbose:    { short: "v", cnt: 0 },
    "@general": { vals: [ ] }
};
```

But if you try passing multiple arguments to `--outfile`, which has `max: 1`, 
all arguments after the first will end up in `@general`:

```javascript
myprog -o foo -o bar
myprog -o foo bar

// will yield:

var optionMap = {
    infile:     { short: "i", vals: [ ] },
    outfile:    { short: "s", vals: [ "foo" ], max: 1 },
    verbose:    { short: "v", cnt: 0 },
    "@general": { vals: [ "bar" ] }
};
```

Switches with a `cnt: 0` member instead of `vals` do not take arguments; they
simply accumulate a count of how many times they have been used. The classical
case is a verbosity switch, e.g.:

```javascript
myprog -v -v -v
myprog --verbose --verbose --verbose
myprog -vvv

// will yield:

var optionMap = {
    infile:     { short: "i", vals: [ ] },
    outfile:    { short: "s", vals: [ ], max: 1 },
    verbose:    { short: "v", cnt: 3 },
    "@general": { vals: [ ] }
};
```

Note that, as usual, short options without arguments can be combined.

<a name="subcommands"></a>
## Git-style Subcommands

Things become _slightly_ more complicated when using subcommands. The first 
argument must be either a switch or a subcommand, not a bare argument. And 
`optionMap` becomes more deeply nested:

```javascript
var optionMap = {
    add: {
        "filename":  { short: "f", vals: [ ] },
        "overwrite": { short: "o", cnt: 0 },
        "@general":  { vals: [ ] },
    },
    remove: {
        "filename":  { short: "f", vals: [ ] },
        "force":     { short: "F", cnt: 0 },
    },
    "@none": {
        "test":      { short: "t", vals: [ ], max: 1 },
        "@general":  { vals: [ ] },
    },
    "@all": {
        "dry-run": { short: "d", cnt: 0 }
    }
};
```


The following command lines then become possible:

```bash
$ myprog add --filename foo.txt -d
$ myprog remove -f foo.txt -F
$ myprog --test one two three
```

For minicle to recognize this, the `options` argument should include `subcommand: true`, e.g.

```javascript
parse(optionMap, { subcommand: true });
```

The top-level keys are the subcommands, and their associated objects are the 
same as the regular `optionMap` when not using subcommands. There are two 
additional optional top-level entries, `@none` and `@all`, both of which are 
optional. The `@none` object specifies switches that can be used when no 
subcommand is given, and `@all` specifies switches that can be used with any or 
no subcommand. The `@all` options should __not__ include a `@general` entry.

If a subcommand is used, minicle will insert it into `optionMap` as the value
of a key named `@subcommand`.

<a name="todo"></a>
## Todo

* ~~Add -- GNU-style end-of-switches flag. The `max` and `@general` elements made
  clear to me what the author of `getopts` was thinking.~~
* _Much_ more testing.
* Improved docs, more examples, maybe a tutorial. Make it as easy as possible
  for people with different learning modalities to understand.
* Update listings

<a name="changelog"></a>
## Changelog

1.0.5: Added `doubleDash` option, improved docs, tested heavily.

1.0.4: Added `@subcommand` to `optionMap` results, documented same.

1.0.3: We don't talk about this anymore.

1.0.2: Fixed bug that threw an uncaught exception when no CLI arguments were given.

1.0.1: Updated docs to include the necessary options for subcommands.

