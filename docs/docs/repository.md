---
layout: docs
title: Repository API
permalink: repository
---

# Repository API

Github4s supports the [Repository API](https://developer.github.com/v3/repos/). As a result,
with Github4s, you can interact with:

- [Repositories](#repositories)
  - [Get a repository](#get-a-repository)
  - [List organization repositories](#list-organization-repositories)
  - [List user repositories](#list-user-repositories)
  - [List contributors](#list-contributors)
  - [List collaborators](#list-collaborators)
- [Commits](#commits)
  - [List commits on a repository](#list-commits-on-a-repository)
- [Contents](#contents)
  - [Get contents](#get-contents)
- [Releases](#releases)
  - [Create a release](#create-a-release)
- [Statuses](#statuses)
  - [Create a status](#create-a-status)
  - [List statuses for a specific Ref](#list-statuses-for-a-specific-ref)
  - [Get the combined status of a specific Ref](#get-the-combined-status-for-a-specific-ref)

The following examples assume the following code:

```scala mdoc:silent
import java.util.concurrent._

import cats.effect.{Blocker, ContextShift, IO}
import github4s.Github
import org.http4s.client.{Client, JavaNetClientBuilder}

import scala.concurrent.ExecutionContext.global

val httpClient: Client[IO] = {
  val blockingPool = Executors.newFixedThreadPool(5)
  val blocker = Blocker.liftExecutorService(blockingPool)
  implicit val cs: ContextShift[IO] = IO.contextShift(global)
  JavaNetClientBuilder[IO](blocker).create // use BlazeClientBuilder for production use
}

val accessToken = sys.env.get("GITHUB_TOKEN")
val gh = Github[IO](httpClient, accessToken)
```

## Repositories

### Get a repository

You can get a repository using `get`; it takes as arguments:

- the repository coordinates (`owner` and `name` of the repository).

To get a repository:

```scala mdoc:compile-only
val getRepo = gh.repos.get("47degrees", "github4s")
val response = getRepo.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the get [Repository][repository-scala].

See [the API doc](https://developer.github.com/v3/repos/#get) for full
reference.

### List organization repositories

You can retrieve the list of repositories for a particular organization using `listOrgRepos`; it
takes as arguments:

- `org`: The organization name.
- `type`: The optional type of the returned repositories, can be "all", "public", "private",
"forks", "sources" or "member", defaults to "all".
- `pagination`: Limit and Offset for pagination.

To list the repositories for an organization:

```scala mdoc:compile-only
val listOrgRepos = gh.repos.listOrgRepos("47deg")
val response = listOrgRepos.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the corresponding [List[Repository]][repository-scala].

See [the API doc](https://developer.github.com/v3/repos/#list-organization-repositories) for full
reference.

### List user repositories

You can retrieve the list of repositories for a particular user using `listUserRepos`; it
takes as arguments:

- `user`: The user name.
- `type`: The optional type of the returned repositories, can be "all", "owner" or "member", defaults to "owner".
- `pagination`: Limit and Offset for pagination.

To list the repositories for a user:

```scala mdoc:compile-only
val listUserRepos = gh.repos.listUserRepos("rafaparadela")
val response = listUserRepos.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the corresponding [List[Repository]][repository-scala].

See [the API doc](https://developer.github.com/v3/repos/#list-user-repositories) for full
reference.

### List contributors

List contributors to the specified repository,
sorted by the number of commits per contributor in descending order.

You can list contributors using `listContributors`, it takes as arguments:

- the repository coordinates (`owner` and `name` of the repository).
- `anon` Set to 1 or true to include anonymous contributors in results.

To list contributors:

```scala mdoc:compile-only
val listContributors = gh.repos.listContributors("47degrees", "github4s", Some("true"))
val response = listContributors.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the corresponding [List[User]][user-scala].

See [the API doc](https://developer.github.com/v3/repos/#list-contributors) for full
reference.

### List collaborators

List collaborators in the specified repository.

You can list collaborators using `listCollaborators`, it takes as arguments:

- the repository coordinates (`owner` and `name` of the repository).
- `affiliation`, one of `outside`, `direct`, or `all` (default `all`).
For more information take a look at [the API doc](https://developer.github.com/v3/repos/collaborators/#parameters).

```scala mdoc:compile-only
val listCollaborators = gh.repos.listCollaborators("47degrees", "github4s", Some("all"))
val response = listCollaborators.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the corresponding [List[User]][user-scala].

See [the API doc](https://developer.github.com/v3/repos/collaborators/#list-collaborators) for full
reference.

## Commits

### List commits on a repository

You can list commits using `listCommits`, it takes as arguments:

- the repository coordinates (`owner` and `name` of the repository).
- `SHA` or branch to start listing commits from. Default: the repository’s default branch (usually `master`).
- `path`: Only commits containing this file path will be returned.
- `author`: GitHub login or email address by which to filter by commit author.
- `since`: Only commits after this date will be returned. Format: "YYYY-MM-DDTHH:MM:SSZ".
- `until`: Only commits before this date will be returned. Format: "YYYY-MM-DDTHH:MM:SSZ".
- `pagination`: Limit and Offset for pagination.

To list commits:

```scala mdoc:compile-only
val listCommits =
  gh.repos.listCommits(
  "47deg",
  "github4s",
  Some("d3b048c1f500ee5450e5d7b3d1921ed3e7645891"),
  Some("README.md"),
  Some("developer@47deg.com"),
  Some("2014-11-07T22:01:45Z"),
  Some("2014-11-07T22:01:45Z"))
val response = listCommits.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the corresponding [List[Commit]][repository-scala].

See [the API doc](https://developer.github.com/v3/repos/commits/#list-commits-on-a-repository) for full
reference.

## Branches

### List branches on a repository

You can list branches using `listBranches`, it takes as arguments:

- the repository coordinates (`owner` and `name` of the repository).
- `protected` Only protected branches.

To list branches:

```scala mdoc:compile-only
val listBranches =
  gh.repos.listBranches(
  "47deg",
  "github4s")
val response = listBranches.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the corresponding [List[Branch]][repository-scala].

See [the API doc](https://developer.github.com/v3/repos/branches/#list-branches) for full reference.

## Contents

### Get contents

This method returns the contents of a file or directory in a repository.

You can get contents using `getContents`, it takes as arguments:

- the repository coordinates (`owner` and `name` of the repository).
- `path`: The content path.
- `ref`: The name of the `commit/branch/tag`. Default: the repository’s default branch (usually `master`).

To get contents:

```scala mdoc:compile-only
val getContents = gh.repos.getContents("47degrees", "github4s", "README.md", Some("heads/master"))
val response = getContents.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the corresponding [NonEmptyList[Content]][repository-scala].

See [the API doc](https://developer.github.com/v3/repos/contents/#get-contents) for full
reference.

## Releases

### Create a release

Users with push access to the repository can create a release using `createRelease`; it takes as arguments:

- the repository coordinates (`owner` and `name` of the repository).
- `tag_name`: The name of the tag.
- `name`: The name of the release.
- `body`: Text describing the contents of the tag.
- `target_commitish`: Specifies the commitish value that determines where the `Git tag` is created from.
Can be any branch or commit `SHA`. Unused if the `Git tag` already exists. Default: the repository's default branch (usually `master`).
- `draft`: true to create a draft (unpublished) release, false to create a published one. Default: false.
- `prerelease`: true to identify the release as a pre-release. false to identify the release as a full release. Default: false.

To create a release:

```scala mdoc:compile-only
val createRelease =
  gh.repos.createRelease("47degrees", "github4s", "v0.1.0", "v0.1.0", "New access token", Some("master"), Some(false), Some(false))
val response = createRelease.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the created [Release][repository-scala].

See [the API doc](https://developer.github.com/v3/repos/releases/#create-a-release) for full
reference.

## Statuses

### Create a status

You can create a status using `createStatus`; it takes as arguments:

- the repository coordinates (`owner` and `name` of the repository).
- the `SHA` of the commit for which we want to create a status.
- the state of the status we want to create (can be pending, success, failure or error).
- other optional parameters: target url, description and context.

To create a status:

```scala mdoc:compile-only
val createStatus = gh.repos.createStatus("47degrees", "github4s", "aaaaaa", "pending", None, None, None)
val response = createStatus.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the created [Status][repository-scala].

See [the API doc](https://developer.github.com/v3/repos/statuses/#create-a-status) for full
reference.

### List statuses for a specific ref

You can also list statuses through `listStatuses`; it take as arguments:

- the repository coordinates (`owner` and `name` of the repository).
- a git ref (a `SHA`, a branch `name` or a tag `name`).

To list the statuses for a specific ref:

```scala mdoc:compile-only
val listStatuses = gh.repos.listStatuses("47degrees", "github4s", "heads/master")
val response = listStatuses.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the corresponding [List[Status]][repository-scala].

See [the API doc](https://developer.github.com/v3/repos/statuses/#list-statuses-for-a-specific-ref)
for full reference.

### Get the combined status for a specific ref

Lastly, you can also get the combined status thanks to `getCombinedStatus`; it takes the same
arguments as the operation listing statuses:

```scala mdoc:compile-only
val combinedStatus = gh.repos.getCombinedStatus("47degrees", "github4s", "heads/master")
val response = combinedStatus.unsafeRunSync()
response.result match {
  case Left(e) => println(s"Something went wrong: ${e.getMessage}")
  case Right(r) => println(r)
}
```

The `result` on the right is the corresponding [CombinedStatus][repository-scala].

Note that the state of the combined status is the product of a heuristic detailed in
[the API documentation](https://developer.github.com/v3/repos/statuses/#get-the-combined-status-for-a-specific-ref).

As you can see, a few features of the repository endpoint are missing.

As a result, if you'd like to see a feature supported, feel free to create an issue and/or a pull request!

[repository-scala]: https://github.com/47degrees/github4s/blob/master/github4s/src/main/scala/github4s/domain/Repository.scala
[user-scala]: https://github.com/47degrees/github4s/blob/master/github4s/src/main/scala/github4s/domain/User.scala
