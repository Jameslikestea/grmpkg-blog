---
title: "Redesigning for Git"
date: 2022-03-08T20:48:42Z
draft: false
---

Okay... It's been a while since I last posted... And a lot has changed with the project.

In the last update I spoke about using S3 and federating the identity with AWS cognito, basically building a true cloud native solution, for a true cloud native langage. However, what I was finding was that I just could not get the performance that I needed out of the cloud native solution and I was concerned with the vgo limit on zipped packages. Due to the prior reasons I have redesigned the architecture and this blog post oes through the changes.

## Git, it's a love hate relationship

Anyone whose used Go before knows that `go get` has a very strong integration with the git (and specifically GitHub.) Git is a fantstic VCS, and I wanted to utilise the already strong integration with the VCS system that's built into Go.

Git is great, but what we're trying to achieve does not fully fit the functionality of Git. Git is designed for regular updates and for collaborative workflows, this is not what we're trying to do. We want to build something that maintains exact, persistent and immutable versions of libraries. If we allow for anyone (who's authorised) to overwrite or delete branches and tags, then we've not met our objectives of the project.

So, with that in mind, we can set out some basic requirements of the software.

| Requirement  | Description                                                                                                                                                     |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Immutability | Tags should never be overwritten by reference, this does mean that we can't delete blobs, trees or commits                                                      |
| Storage      | We need some way to persist our objects and tags. We need to ensure that our persistence layer is performant and ensure consistency                             |
| Security     | One thing that is of paramount importance is the security of the software. We will use OAuth2.0 providers for authentication and OPA Policies for authorization | 

These obviously only provide some very high level requirements, so let's jump into some more specific requirements and design decisions.

### How the application will route data

Ultimately the application is going to take data and Git objects from a client, and persist those into a database of choosing. This dataflow allows for fast persistence of large volumes of requests whilst still maintaining a secure posture.

![Diagram showing component architecture](/2022-03-08-redesign-for-git-architecture.jpg)

## Choosing a persistence layer

As mentioned above, the persistence layer is super important for this application. It forms our knowledge base for the entire application for both internal data and git data.

For the above reasons, I have decided to start with a focus on Cassandra compatible interfaces, specifically [ScyllaDB](https://www.scylladb.com/), with the intention that this could also be deployed against Amazon Keyspaces. There are several reasons for this choice, but it is mainly due to gocqlx's great integration and the speed of the database.

Due to Go's interface design this will be really easy to expand out to other storage solutions in the future such as S3, MySQL or Oracle. Perhaps some of these will be community lead.

### Storing internal data

What we need to understand about a Git server, is that the objects are stored with reference to their hashes. This storage structure kind of resembles a Hash Map or a Key Value store, which is how we will setup the storage.

As we have this prebuilt Hash Map/Key Value store, we can also use that to store internal data. The way we can do this is to store the full object against a hash of the lookup value, for example, a namespace will have user info, permissions etc stored against it, but it will likely be stored against a hash of the name.

User accounts will likely be similar (using OAuth2.0) we can store the users details, against a hash of the User ID. I guess what I'm trying to say, is that we can represent everything as a Git object, our most simple form of storage.

## Policy Engines

So as I mentioned in the architecture, we will have a policy engine to make a lot of our decisions. We're going to use [Open Policy Agent](https://www.openpolicyagent.org/) to create our policies. We can use this to make fast decisions about the data that we're provided with, this enables us to enforce our business processes on the repositories in a timely manner.

Policies that we might want to enforce are things such as, repository name constraints, RBAC permissions for multi-tenancy, namespace ownership and push/pull permissions.

## Summary

To summarise, there is a lot of change in the project at the moment as I redesign and, subsequently rewrite a lot of the architecture. I think that by providing this as a "tag-only" git service, we open the software up to a lot more flexibility such as moving into other languages or ecosystems.