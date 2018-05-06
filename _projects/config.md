---
title: Config
description: A standard hierarchical configuration implementation
layout: default
project_type: php
package_name: dhii/config
package_url: https://packagist.org/packages/dhii/config
---

Package: [`{{ page.package_name }}`][package-url]

Standards:
- [PSR-11][]
- [Dhii Config][]
- [Dhii Map][]

## Details
### Interop
This package provides concrete implementation of configuration. One way in which this is achieved is in the form of a [Dhii Map][] hierarchy, such as the `DereferencingConfigMap`. Values can be retrieved from the hierarchy via the [Dhii Config][] standard by providing a path - a list of container keys. Because this implementation is also a map, it is therefore iterable, and therefore serializeable. A multi-level config can easily be created from an arbitrary hierarchy of iterables with `DereferencingConfigMapFactory`

### Reference Tokens
Another useful feature of `DereferencingConfigMap` is its ability to de-reference tokens in stringable values of the config. If the string value for a path contains a token in the format `${key}`, it will be replaced at the time of retrieval with a value that corresponds to that key. The key can generally be of any format acceptable by PSR-11 `ContainerInterface#get()`. The value will be retrieved from the container that is assigned to the `DereferencingConfigMap`, and that can be any PSR-11 container, including another `ConfigInterface` implementation - such as another `DereferencingConfigMap`. This makes it possible to reference config values from other values. By default, `DereferencingConfigMap` instances will use themselves to de-reference tokens. Because all config nodes produced by `DereferencingConfigMapFactory` are instances of `DereferencingConfigMap`, de-referencing will work even when retrieving values from nested config nodes.

This approach has a caveat, however: `DereferencingConfigMap` requires the container used for de-referencing to be passed during construction time, which means that it is not possible to assign a parent node as the de-referencing container for a child node: the parent is also read-only, and this introduces a chicken-egg problem. In order for tokens in multi-level configs to be de-referenced correctly, the top-most config instance must be passed to **all** child instances below; this is already done by`DereferencingConfigMapFactory`. The problem is solved by adding another non-config container at the top of the hierarchy, such as a [composite container][dhii/composite-container] implementation.

## Examples
### Single Layer
```php
$data = [
    'first_name'    => 'John',
    'last_name'     => 'Smith',
    'full_name'     => '${first_name} ${last_name}'
];

// Passing `null` as the dereferencing container causes the config to use itself for dereferencing
$config = new DereferencingConfigMap($data, null);

$firstName = $config->get('first_name'); // John
$fullName = $config->get('full_name'); // John Smith

$allValues = iterator_to_array($config);
/*
[
    'first_name'    => 'John',
    'last_name'     => 'Smith',
    'full_name'     => 'John Smith'
]
*/
```

[PSR-11]:                       https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md
[Dhii Iterator]:                https://packagist.org/packages/dhii/iterator-interface
[Dhii Config]:                  https://packagist.org/packages/dhii/config-interface
[Dhii Map]:                     {% link _projects/map.md %}
[dhii/composite-container]:     https://packagist.org/packages/dhii/composite-container
[package-url]:                  {{ page.package_url }}
