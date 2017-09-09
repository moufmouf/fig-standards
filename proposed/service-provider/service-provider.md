Service provider
================

This document describes a common interface for service providers.

The goal set by `ServiceProviderInterface` is to offer a common way for libraries to add or extend 
container entries in compatible dependency injection containers.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119][].

The word `implementor` in this document is to be interpreted as someone
implementing the `ServiceProviderInterface`.
Consumers of service providers are referred to as `containers`.

[RFC 2119]: http://tools.ietf.org/html/rfc2119

TODO: continue here
TODO: continue here
TODO: continue here
TODO: continue here
TODO: continue here

1. Specification
-----------------

### 1.1 Introduction

Frameworks often offers ways for packages to add additional features. Those packages can be called 
bundles, modules or plugins. In most framework, these packages allow the developer to add/modify 
the entries stored in the framework's container.

The goal of this PSR is to offer a solution for cross-framework modules through **standard container configuration**
via **service providers**.

### 1.2 Definitions

*    **Factory** - A callable that creates and returns a container entry. The callable takes in parameter 
     a single argument: the container (implementing the PSR-11 `ContainerInterface`). Therefore, factories have
     the following signature:
     
     ```php
     function(\Psr\Container\ContainerInterface $container) : mixed
     ```

*    **Decorators** - A callable that modifies an existing container entry. The callable takes in parameter 
     two arguments: the container (implementing the PSR-11 `ContainerInterface`) and the entry to be modified.
     It returns the new entry. Therefore, decorators have the following signature:
     
     ```php
     function(\Psr\Container\ContainerInterface $container, mixed $decoratedEntry) : mixed
     ```

*    **Service provider** - A service provider is a class that adds/modifies entries in a container.
     It does so by providing the container with a set of *factories* and *decorators*.
     
*    **Container**, **Identifier** and **Entry** have the same meaning as in PSR-11.

PSR-11 defines that a container contains "entries" (most frameworks talk about "services" that are limited to objects).
The name "service provider" was chosen instead of the more correct "entry provider" name because it is already 
wildly used by most frameworks having this notion.

### 1.3 Defining a service provider

- The `Psr\Container\ServiceProviderInterface` exposes two methods: `getFactories` and `getExtensions`.

- `getFactories` takes no parameter. It returns an array of factories, indexed by entry identifier.

- `getDecorators` takes no parameter. It returns an array of decorators, indexed by entry identifier.

### 1.4 Consuming a service provider

#### 1.4.1 Consuming in two passes

Containers consuming service providers implementing the `ServiceProviderInterface` MUST do so in 2 passes.

- On the first pass, the `getFactories` method of all service providers is called and all entries are added
  to the container
- On the second pass, the `getDecorators` method of all service providers is called **in the same order**
  and entries are modified accordingly.

The container can decide to add its own entries at any point (before, after or even in the middle of service providers
being consumed).

#### 1.4.2 Entries overloading

If an entry is defined in more than one service providers, the last service provider takes precedence.

The container MUST NOT throw an exception because an entry is overloaded.

#### 1.4.3 Decorator signature

A decorator can type-hint against the decorated entry expected type.

```php
function(\Psr\Container\ContainerInterface $container, LoggerInterface $decoratedEntry) : LoggerInterface
```

If the entry to be decorated is not of the expected type, the container MUST throw a XXX exception. (TODO: define exception type... fomr PHP 7, we could use the native exception but it's an error in PHP 5...)
TODO: question: should we define those exceptions? It is unlikely we will ever catch those after all.

#### 1.4.4 Decorating a non defined entry

If a decorator tries to decorate an entry that is not defined, `null` MUST be passed in parameter of the `$decoratedEntry` parameter.

If the decorator signature does not allow the `null` value to be passed, the container MUST throw a XXX exception. (TODO)

#### 1.4.5 Caching

Containers CAN cache the results of the calls to `getFactories` and `getDecorators`.

Implementors SHOULD ensure the service providers return stable results. Two successive calls in the same conditions to a service provider `getFactories` or `getDecorators` method should return the same results. Entry identifiers MUST NOT be generated using a function involving some random number generation.

#### 1.4.6 Performance consideration

TODO: should we say we prefer "public static" pure methods? (because they can be invoked without instantiating the service provider (easy caching))
TODO: should we define classes implementing invokable in this PSR (like the `Alias` class?)

#### 1.4.7 Returning null

If a factory or decorator returns `null`, the container MUST add a `null` entry in the container.
The container `has` method MUST return `true` for this identifier.


### 1.5 Exceptions

TODO 

2. Package
----------

The interfaces and classes described as well as relevant exceptions are provided as part of the
[psr/serviceprovider](https://packagist.org/packages/psr/serviceprovider) package.

// TODO: automatic detection using composer extra section?

3. Interfaces
-------------

<a name="service-provider-interface"></a>
### 3.1. `Psr\Container\ServiceProviderInterface`

~~~php
<?php

namespace Interop\Container;

/**
 * A service provider provides entries to a container.
 */
interface ServiceProviderInterface
{
    /**
     * Returns a list of all container entries registered by this service provider.
     *
     * - the key is the entry name
     * - the value is a callable that will return the entry, aka the **factory**
     *
     * Factories have the following signature:
     *        function(\Psr\Container\ContainerInterface $container)
     *
     * @return callable[]
     */
    public function getFactories();

    /**
     * Returns a list of all container entries extended by this service provider.
     *
     * - the key is the entry name
     * - the value is a callable that will return the modified entry
     *
     * Callables have the following signature:
     *        function(Psr\Container\ContainerInterface $container, $previous)
     *     or function(Psr\Container\ContainerInterface $container, $previous = null)
     *
     * About factories parameters:
     *
     * - the container (instance of `Psr\Container\ContainerInterface`)
     * - the entry to be extended. If the entry to be extended does not exist and the parameter is nullable, `null` will be passed.
     *
     * @return callable[]
     */
    public function getExtensions();
}
~~~
