---
title: Map
description: An iterable container implementation
layout: default
project_type: php
package_name: dhii/map
package_url: https://packagist.org/packages/dhii/map
---

**Package**: [`{{ page.package_name }}`][package-url]  
**Standards**: [PSR-11][], [Dhii Collection][], [Dhii Iterator][]

## Details
The primary purpose of this package is to provide a concrete implementation of an iterable container. Because maps implement `ContainerInterface`, they can be used to retrieve values in an interoperable way. At the same time, because all [Dhii Iterator][] implementations are `Traversable`, it is possible to retrieve all values of a map, which is useful for serialization, making it possible to cache an entire map. By extending a map, it could even be possible to generate the values in a map, such as from definitions, or from another data source, like a database or a REST API.

Using a factory, it is possible to produce a hierarchy of maps, thus forming a tree. Because in this tree maps contain other maps, the whole tree can also be processed in a generic, standard way - such as for serialization. For example `CountableMapFactory`, given an arbitrary tree of [iterables][iterable], can produce a hierarchy of maps. This is the approach that is used by [Dhii Config][].

[PSR-11]:                       https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md
[Dhii Collection]:              https://github.com/Dhii/collections-interface
[Dhii Iterator]:                https://packagist.org/packages/dhii/iterator-interface
[iterable]:                     https://github.com/Dhii/normalization-helper-base/blob/develop/src/NormalizeIterableCapableTrait.php
[package-url]:                  {{ page.package_url }}
