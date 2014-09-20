# ymlconfig

## Introduction

ymlconfig is a small convenience library for working with .yml configuration files. It has two features not present in the stock PyYAML library.

1. **!format** custom tag support for doing string.format style substitution
2. dict objects are converted to [Bunch](http://github.com/dsc/bunch) objects before return.

## Motivation

Having configuration metadata in a single human-readable file which users can easily customize to their particular circumstances makes things like project build and deployment work more smoothly and consistenly.

### YAML

I find YAML files to be very easily readable, but capable of handling sophisticated use-cases like nested structures. The alternatives seem weaker:
- **.ini**: Too simple a format, no nested structure support.
- **json**: Lack of comments and too many brackets are not human-friendly.
- **xml**: You're kidding, right?

### Bunch

With configuration files the code often knows what keys it is going to access. I find reading and writing:

``` python
SetDb(config.dbSettings.serverurl)
```
easier than:
``` python
SetDb(config["dbSettings"]["serverurl"])
```

### **!format** tag
One thing missing in YAML is support for doing value substitution string formatting. The **!format** custom tag adds that. For example

``` yaml
url: &mainurl 'example.com'
searchSettings:
   # override this if you don't run search on the main server
   serviceUrl: !format
      format: 'https://{url}/search'
      url: *mainurl
```

## Example

With this simple [fabric](http://www.fabfile.org/) task.
``` python
from fabric.api import task
import ymlconfig

@task
def DoSomething(configPath = '~/project/config.yml', **kwargs):
   config = ymlconfig.load_file(configPath)
   print("Doing: {action} {target}.".format(config.somethingToDo))
```
And *~/project/config.yml* containing:
``` yaml

mainactivity: &defaultActivity 'buy food'
where: &target 'supermarket'

somethingToDo:
    action: !format
        format: '{reason} so {what}'
        reason: 'I'm hungry'
        what: *defaultActivity
    target: !format
        format: '{operator} {target}'
        operator: 'at'
        target: *target
```
``` Shell
$ fab DoSomething
Doing: I'm hungry so buy food at supermarket.

$ fab DoSomething:reason='broke',what='get money',target='the bank')
Doing: I'm broke so get money at the bank.
```
## Reference

### **!format** YAML Tag

The **!format** custom YAML tag provides two extensions to standard PyYAML parsing. The contents of the node must be a mapping, and must contain a **format** member, which must be a string. It operates as follows.

First, any optional kwargs that were passed to the load function are substituted for any matching items in the node.

Second, python string.format is called on the format member string passing all the members of the node object as named arguments.

For example with the following data:

``` yaml
username: &name 'Fred'
value: !format
    format: '{name} likes {food}.'
    name: *name
    food: 'chocolate'
```

``` python
print(ymlconfig.load_data(data).value)
>>> Fred likes chocolate.

print(ymlconfig.load_data(data, food='carrots').value)
>>> Fred likes carrots.
```

### **ymlconfig.load(yml_data, **kwargs):**

**yml_data**: yml data to be parsed
**returns**: Bunch() object con

In addition to the standard PyYAML parsing, the !format custom tag is supported.

### **ymlconfig.load_file(path, **kwargs):**

A thin wrapper around **ymlconfig.load**, that opens the file at **path** and passes it's data to **ymlconfig.load**.