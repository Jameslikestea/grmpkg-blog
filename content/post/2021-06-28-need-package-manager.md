---
title: "The need for a package manager"
date: 2021-06-28T13:58:52+01:00
draft: false
---

## Dependency Management

The Go programming language, like many other languages, allows for developers to import 3rd party libraries. These libraries are not in the standard library but anyone with the link can install them.

Over time the number of 3rd party libraries grows and grows and needs to be managed in a way that is accessible for developers and makes libraries with a given purpose easy to find.

## The mutability problem

As Go uses Git and Github to manage a vast quantity of libraries there is a problem that is presented. As dependency trees get deeper and deeper, especially with the introduction of Go Modules, if and when libraries get phased out they can cause problems throughout the whole tree. This is not something that is just hypothetical, it happened a few years ago in npm, the node package manager, which you can read about on [the register](https://www.theregister.com/2016/03/23/npm_left_pad_chaos/).

Mutability is fine for dependency trees with a depth of 1 because you control the imports and can therefor remove it, thus fixing your code for the next release. However for trees that are deeper, this is not necessarily the case, because you do not control projects that you depend on, if they lose their dependencies, you too lose your full tree, breaking builds, testing and deployments.

## Introducing vGo and the Go Proxy

To help introduce some determinism in the Go build systems, Go did introduce `GOPROXY` which would help with caching and immutability, you can read more about how `GOPROXY` works in this [JFrog article](https://jfrog.com/blog/why-goproxy-matters-and-which-to-pick/).

Although this helps with the problem, this is designed for use in enterprise or corporate environments. Also of note is the `proxy.golang.org` which enables public caching of modules, this is great but is still backed by git and can miss rarely used modules.

Over the next few instalments of the blog, we'll start building the following:

- A packager to zip a package
- An API for uploading the packages
- An API compatible with the GOPROXY spec

So stay tuned for that the next posts.
