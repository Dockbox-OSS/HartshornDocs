# Hartshorn Framework Documentation
Hartshorn makes it easy to create Java enterprise applications through service-driven development. It provides everything you need to develop lightweight, platform agnostic applications. As of Hartshorn 4.0, Hartshorn requires JDK 16+. Hartshorn, and all its official modules are open source and can be found on [:fontawesome-brands-github: GitHub](https://github.com/GuusLieben/Hartshorn).

## Definition of 'Hartshorn Framework'
Hartshorn is an umbrella term for the Hartshorn Framework and all official [modules](./modules/modules.md). Hartshorn Framework itself is a service- and dependency management framework which compliments [JSR 330](https://www.jcp.org/en/jsr/detail?id=330), but does not embrace the Java EE specification. You can read more about Hartshorn's core technologies and principles in the [Core technologies topic](./core/cdi.md).

## Philosophy
As with any framework, it is important that you know not only what it does but also what principles it follows. Hartshorn follows a range of technical [core principles](./core/cdi.md), as well as a specific design philosophy. Here are the guiding principles of the Hartshorn Framework:  

* Extensibility and flexibility. Hartshorn puts an emphasis on enabling developers to extend and switch between implementations at every level, without requiring changes to your code.
* Allow for diverse usages. Hartshorn allows for a large amount of flexibility, by providing a template for you to develop your application in. 
* High standards for code quality. Hartshorn aims to use modern language features in combination with a meaningful and future-proof API to allow developers to intuitively use the framework.

## Getting started
If you are just getting started with Hartshorn, you'll want to view [the 'Getting Started' guides](./getting-started/setup.md). The provided guides use Hartshorn's application bootstrapper which enables you to get started quickly.