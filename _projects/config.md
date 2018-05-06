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
This package provides concrete implementation of configuration. One way in which this is achieved is in the form of a [Dhii Map][] hierarchy, such as the `DereferencingConfigMap`. Values can be retrieved from the hierarchy via the [Dhii Config][] standard by providing a path - a list of delimited container keys, e.g. `path/to/config`. Because this implementation is also a map, it is also iterable, and therefore serializeable. A multi-level config can easily be created from an arbitrary hierarchy of iterables with `DereferencingConfigMapFactory`

### Reference Tokens
Another useful feature of `DereferencingConfigMap` is its ability to de-reference tokens in stringable values of the config. If the string value for a path contains tokens in the format `${key}`, those tokens will be replaced at the time of retrieval with values that correspond to their keys. The keys can generally be of any format acceptable by [PSR-11][] `ContainerInterface#get()`. The value will be retrieved from the container that is assigned to the `DereferencingConfigMap`, and that can be any PSR-11 container, including another `DereferencingConfigMap`. This makes it possible for config values to include other config values, which is very helpful in centralizing certain data, and avoiding repetition. If no container is specified for de-referencing, `DereferencingConfigMap` instances will use themselves to de-reference tokens. Because all config nodes produced by `DereferencingConfigMapFactory` are instances of `DereferencingConfigMap`, de-referencing will work even when retrieving values from nested config nodes.

This approach has a caveat, however: `DereferencingConfigMap` requires the container used for de-referencing to be passed during construction time, which means that it is not possible to assign a parent `DereferencingConfigMap` node as the de-referencing container for a child node - the parent is also read-only, and this introduces a chicken-egg problem. The problem is solved by adding another non-config container at the top of the hierarchy, such as a [composite container][dhii/composite-container] implementation, which at the same time allows creating a list of configs, where configs will "override" other configs in reverse order without the need to be merged into one. To be able to add configs after creation, use a writable list, such as [`AddCapableOrderedList`]. Also, in order for tokens in multi-level configs to be de-referenced correctly, the **top-most** config instance must be passed to **all** child instances below; this is already done by `DereferencingConfigMapFactory`.

## Examples
### Single Layer
```php
use Dhii\Config\DereferencingConfigMap;

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

### Multi Layer
```php
use Dhii\Collection\AddCapableOrderedList;
use Dhii\Data\Container\CompositeContainer;
use Dhii\Config\DereferencingConfigMap;
use Dhii\Config\DereferencingConfigMapFactory;
use Dhii\Config\ConfigFactoryInterface;
use Zend\Stdlib\ArrayUtils;

$data = [
    'db'                    => [
        'table_prefix'          => 'wp_',
        "tables"                => [
            'posts'                 => [
                'name'                  => '${db/table_prefix}posts',
                'engine'                => 'InnoDB'
            ],
            'postmeta'              => [
                'name'                  => '${db/table_prefix}postmeta',
                'engine'                => 'MyISAM'
            ],
        ],
    ],
];

// The configs list
$configs = new AddCapableOrderedList();
// The container for all configurations
$topConfig = new CompositeContainer($configs);
// The top config will now be used for de-referencing by all config nodes made by the factory
$configFactory = new DereferencingConfigMapFactory($topConfig);
// Create the config hierarchy with the data - objects work just as well
$config = $configFactory->make([ConfigFactoryInterface::K_DATA => (object) $data]);
// Add the config to list - without this, de-referencing will not work properly with multiple levels
$configs->add($config);

$tablePrefix = $config->get('db/table_prefix'); // wp_
$postsTableName = $config->get('db/tables/posts/name'); // wp_posts

// Retrieve a nested configuration node
$postmetaTableConfig = $config->get('db/tables/postmeta');
// https://github.com/zendframework/zend-stdlib/blob/master/src/ArrayUtils.php#L216
$postmetaTableValues = ArrayUtils::iteratorToArray($postmetaTableConfig);
/*
[
    'name'                  => 'wp_postmeta',
    'engine'                => 'MyISAM'
]
*/
// De-referencing still works due to common top-most parent
$postmetaTableName = $postmetaTableConfig->get('name'); // wp_postmeta

$allValues = ArrayUtils::iteratorToArray($config);
/*
[
    'table_prefix'          => 'wp_',
        'tables'                => [
            'posts'                 => [
                'name'                  => 'wp_posts',
                'engine'                => 'InnoDB'
            ],
            'postmeta'              => [
                'name'                  => 'wp_postmeta',
                'engine'                => 'MyISAM'
            ],
        ],
    ],
]
*/
```

[PSR-11]:                                       https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md
[Dhii Iterator]:                                https://packagist.org/packages/dhii/iterator-interface
[Dhii Config]:                                  https://packagist.org/packages/dhii/config-interface
[Dhii Map]:                                     {% link _projects/map.md %}
[dhii/composite-container]:                     https://packagist.org/packages/dhii/composite-container
[`AddCapableOrderedList`]:                      https://github.com/Dhii/set/blob/develop/src/AddCapableOrderedList.php
[package-url]:                                  {{ page.package_url }}
