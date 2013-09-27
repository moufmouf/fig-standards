**Step A**: Common object retrieval interface
=============================================

This document is a working group related to [dependency injection containers interoperability](dependency-injection-meta.md).

Thanks to [Matthieu Napoli's work](https://gist.github.com/mnapoli/6159681), we already know what is common between DI containers. What we do not know yet is **what
a typical MVC framework needs from the DI Container**.

TODO: maybe we could copy matthieu's GIST here, for reference. It would be easier to fork / modify / pull.


MVC frameworks requirement list
-------------------------------

In order to have interoperable DI containers, we must first know how those DI containers are used
by their primary users: MVC frameworks.

The goal of this chapter is to list the requirements of various MVC frameworks, regarding DI containers.

When we have that list, we can very easily see if most MVC frameworks share a common list of requested features or not.
If they share such a set of requested features, we can move on an work on a common interface.
If they do not share a set of requested features, all hope is not lost. We can still work on [*step C* on DI containers interoperability](step-c-inter-di-interop-meta.md),
in order to be able to have several DI containers that can cooperate in the same application. 

MVC frameworks are listed alphabetically.

**TODO: enrich the list of MVC framework**

###[Silex (micro framework)](http://silex.sensiolabs.org/)

Silex is directly extending Pimple (the Silex `Application` class is a subclass of `Pimple`).
The only thing required by Pimple is **to be able to fetch an instance by name**. This is done
using the `ArrayAccess` interface.

Note: Pimple/Silex is quite easy to extend. Although it is Pimple and Silex are deeply integrated,
it is almost trivial to add a fallback DI container. Here is 
[an exemple integrating Silex with the Mouf DI framework](http://mouf-php.com/packages/mouf/mvc.silex-mouf/README.md).

Note 2: migrating Pimple to use an interface with a `get` method is problematic. Indeed, Silex (that **extends** Pimple) is already
using the `get` method to register routes. (This occurs because Silex is extending Pimple instead of aggregating Pimple) 

###[Symfony 2 router](http://symfony.com/doc/current/cookbook/controller/service.html)

In Symfony 2, you can declare a controller as a "service". In this case, the controller will be
instanciated using the service locator (=DI container).

Therefore, **the Symfony 2 router does only require a service locator with a "get" method to run**.

Proof of concept: Here is a [package that enable extending the Symfony service locator with additional
fallback containers](https://github.com/thecodingmachine/interop.symfony.di.git). The additional 
containers must implement the ContainerInterface. With this ContainerInterface alone, we are able
to:

- declare the controller in any other container
- the Symfony router can detect the controller in the third-party container and instanciate it


###[Splash (Mouf's MVC framework)](http://mouf-php.com/packages/mouf/mvc.splash/index.md)

Splash is requiring the DIC to be searchable, for a given type.
Indeed, it will scan all the instances available in the DIC to see which are implementing the `ControllerInterface` interface.

So at best, Splash needs a `findByType($type)` method to find instances of a given type.
A simple *list of all instances available* could be enough though, since Splash can do the analysis by itself.

**TODO: enrich the list of MVC framework**

Naming of the interface
-----------------------

The name of the interface should be studied with care.

Here are the names that have been proposed so far:

`ContainerInterface`:

*Comment by Matthieu Napoli*: A container is usually doing more than simply providing a get/has method, therefore, this
name might not be the best.

`ServiceLocator`: 

*Comment by Matthieu Napoli*: A framework will use a container as a service locator, and that's OK.
A user however shouldn't. (that why also I think the interface should be named ServiceLocatorInterface).

*Comment by David Négrier*: A DI container might not only serve "service". You can store other kind of
objects if you want (menu items, widgets...), therefore, I would avoid the term "Service" in the name
if the class.

**TODO**: add more names, ask the mailing list about a good name.

Behaviour if instance not found
-------------------------------

Currently, most DI containers are throwing exceptions if the instance does not exist ([10 out of 12 are throwing exceptions](https://gist.github.com/mnapoli/6159681)).
Still, we might wonder if this is the best way to handle things. Indeed, if we want to chain DI containers (see
[Proposition 2 by Marco Pivetta here](step-c-inter-di-interop-meta.md)), it would be easier to return "null".

Returning an exception is great if we are using the DI container as a service locator (for instance if
we are using the DI container inside a controller), but we should not be using this service this way, it is
an anti-pattern.

There is also the possibility to [use the decorator pattern to modify that behaviour](https://github.com/jeremeamia/acclimate#container-decorators), 
just like Jeremy Lindblom is doing in Acclimate. 