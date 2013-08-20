PSR-DI Meta Document
====================

1. Summary
----------

The purpose of the PSR-DI is to provide interoperable dependency injection containers.
The ultimate goal is to allow an MVC framework to use any dependency injection containers that would fit its needs.


2. Why Bother?
--------------

Today, most MVC frameworks are tied to one DI container (or service locator). The MVC framework is the
first component called by the web-server when a request is received. It usually instanciates the DI container
that comes with it and uses that container to get the controller and any instances that are tied to the controller.

Therefore, it is not often not possible to use a third party DI container with a MVC framework. Conversely,
it is not possible for a MVC framework developer to let the user choose which DI container he wants to use.
The implementor of the MVC framework is making the choice for the user.

Pros:

* Drastically reduces coupling between MVC frameworks and DI Containers

Cons:

* ???

3. Scope
--------

## 3.1 Goals

The goals stated below could be fulfilled by different PSRs.

* **Goal 1**: Provide interoperable DI Containers and reduce coupling between DI containers and MVC frameworks

* **Goal 2**: Multiple active DI containers. We could have multiple DI containers (with a different set of features)
  speaking to each other. The goal would be to enable one DI container to fetch an instance that it has no
  knowledge of in another DI container.

* **Goal 3**: A package might be able to add its own instances to a DI containers. When installing a package,
  this package might declare its own instances in the active DI container.

## 3.2 Non-Goals

* Enforce features in any implementation of a DI Container: this document MUST NOT enforce implementation
  of any particular features (aliasing, context, autowiring, configuration file...) in DI containers. Every DI container 
  implementor should be free to choose its own implementation.

4. Approaches
-------------

### 4.1 Introduction

As Larry Garfield pointed out, we are working on Depedency Injection interoperability.
We do not want to work on a service locator common interface.

DI Containers and service locators are the 2 sides of a same coin. In both, you do provide an
instance identifier (a string) and the service/container returns an object.

They are however  different in the way they are used. If you use a container as a service locator,
your are drastically increasing the dependency of your code (on all your project).

A DI container should actually only be used by the component bootstraping the project (that is
the MVC framework if we are talking about a web app, or the "App" class if we are dealing
with command-line scripts).

Regarding this document, this means that establishing a common interface for all DI containers
is not enough. This would be enough to get interoperable service locators, but we
also need to have a simple way for an MVC framework or an App to discover which DI containers
are available.


### 4.2 Chosen Approach

[A discussion on the mailing list](https://groups.google.com/forum/#!topic/php-fig/nJYwn2cYirk) 
started by trying to find what is common between any DI container.

This discussion (along [Matthieu Napoli's GIST](https://gist.github.com/mnapoli/6159681)) has demonstrated 
that there are a common set of function names that can be used:

- `get` to retrieve objects
- `has` to check objects existence

There are a number of additional features that have been talked about:

- listing all instances
- having many DI containers talk to each other
- having a common interface to declare instances
- having a common way for a package to declare its instances
- ...

We propose to sort all these requested features in different work groups. Each group might or might not
lead to a PSR.

#### 4.2.1 **Group 1**: DIC - MVC framework interoperability

**Step 1**: 

We already know what is common between DI containers. What we do not know yet is **what
a MVC framework needs from the DI Container**.

For instance, Mouf's MVC framework (splash), needs to be able to retrieve all instances
of the container implementing the `ControlerInterface` interface. So it requires a
search capability (or at least a possibility to iterate over declared instances)

Does Symfony requires context support? Does Zend MVC requires something else?
By studying common routing libraries, we might be able to see if the features required
by the MVC frameworks are covered by the interface offered.

If (and only if) the set of functions used by MVC frameworks can be reduced to a common
interface that could be implemented in most DIC, then we can move on to step 2. 

**Step 2**

This step is about finding for an MVC framework to find a DIC. It is optionnal, but it 
might be great. Just imagine: you install one DI container you like. You install about
any MVC framework. The framework is clever enough to detect your DI container and to
fetch your controller from it. Wouldn't that be great?

This is however a difficult discussion. This might mean building a kind of a "service locator" to
locate DI containers, which is a bit awkward. This also means at some point playing with
a "global" or "static" service locator, which might not please everybody.

Also, this problem shares a lot of common questions with **Group 2** (interoperability between
DI containers).

Group 1 work will be summarized in [this meta document](di-mvc-interop-meta.md).

#### 4.2.2 **Group 2**: interoperability between DI containers

Let's assume I want to use Zend DI container for my project.
And I want to use Mouf's Splash MVC framework. Splash requires the DI Container to 
be iterable over all the instances it contains. But Zend DI container cannot list my instances 
(it doesn't know how to iterate over instances).

This could be a dead-end. But not so if we can run several DI containers together and if they can
share instances. In this case, I could just add another DI container for Splash (for instance
Mouf DI container), and keep my non controller-related instances in Zend DI container.

For this to be possible, DI containers must know each other.

This can either be done via chaining, or via registering to a common DI container locator.

Group 2 work will be summarized in [this meta document](inter-di-interop-meta.md).

#### 4.2.3 **Group 3**: DI containers configuration interoperability

A number of packages could come with "default settings", and therefore "default instances".
This work group might want to see how those packages could declare in a simple way
a common set of instances that might be created in the DI container.

This is a tricky question. First of all, some DI containers heavily rely on annotations,
or on auto binding (Zend DI for instance). Therefore, it might not be always possible to declare
instances. Also, should a package declare its own instances? Does this imply an install mechanism?
 


5. People
---------

### 5.1 Editor(s)

* 

### 5.2 Sponsors

* 
* 

### 5.3 Contributors

* David Negrier

6. Votes
--------

* **Entrance Vote: ** tbd
* **Acceptance Vote:** tbd

7. Relevant Links
-----------------

_**Note:** Order descending chronologically._

* [Comparison of container interfaces](https://groups.google.com/forum/#!topic/php-fig/nJYwn2cYirk)
* [DI Containers usage comparison](https://gist.github.com/mnapoli/6159681)