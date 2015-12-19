# Container Configuration Meta Document

## 1. Introduction

TODO

## 2. Why bother?

Modules (aka packages or bundles) are widespread in modern frameworks. Unfortunately each framework has its own convention and tools for writing them. This PSR is a step towards helping developers write modules that can work in any framework.

This PSR focuses on letting modules register container entries. That is necessary so that modules can expose services to users.

## 3. Scope

### 3.1. Goals

The goal of this PSR is really to make sure that containers can read a common format or use a common API so that module authors can write only one package instead of one package per framework.

To reach this goal, the problem can be split into several sub-parts:

- **Defining a minimum set of features that each container should have**: what features a container MUST support (for instance: tagging, instantiation using factories, extending existing services, etc...)
- **Defining a common format**: this PSR must decide if interoperability is performed at the container entry descriptor level (interfaces for objects describing services), or at the file level, in which case a common file format must be decided (JSON / XML / YML / NEON, etc...)

### 3.2. Non-goals

The goal is not to standardize the way containers work. Each container should keep its own file format, API, specific features, etc... Containers implementing this PSR should only be able to support one additional format / API, and the minimum set of features defined in this PSR.

The way a framework can automatically **discover the container definitions** of a package (whether they are in PHP files or config files) is **out of scope** of this PSR.

## 4. Approaches

### 4.1. The format

TODO

### 4.2. Features

TODO
