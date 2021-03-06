---
layout: page
title: Developing the Avatica Go Client
permalink: /develop/avatica-go.html
---

<!--
{% comment %}
Licensed to the Apache Software Foundation (ASF) under one or more
contributor license agreements.  See the NOTICE file distributed with
this work for additional information regarding copyright ownership.
The ASF licenses this file to you under the Apache License, Version 2.0
(the "License"); you may not use this file except in compliance with
the License.  You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
{% endcomment %}
-->

* TOC
{:toc}

## Issues

To file issues, please use the [Calcite JIRA](https://issues.apache.org/jira/projects/CALCITE/issues) and select `avatica-go`
as the component.

## Updating protobuf definitions

To update the procotol buffer definitions, update `AVATICA_VER` in `gen-protobuf.bat` and `gen-protobuf.sh` to match
the version you want to generate protobufs for and then run the appropriate script for your platform.

## Live reload during development

It is possible to reload the code in real-time during development. This executes the test suite every time a `.go` or
`.mod` file is updated. The test suite takes a while to run, so the tests will not complete instantly, but live-reloading
during development allows us to not have to manually execute the test suite on save.

### Set up
1. Install [docker](https://docs.docker.com/install/) and [docker-compose](https://docs.docker.com/compose/install/).

2. From the root of the repository, run `DEV=true docker-compose up --build`.

## Testing

The test suite takes around 4 minutes to run if you run both the Avatica HSQLDB and Apache Phoenix tests.

### Easy way
1. Install [docker](https://docs.docker.com/install/) and [docker-compose](https://docs.docker.com/compose/install/).

2. From the root of the repository, run `docker-compose up --build --abort-on-container-exit`.

### Manual set up
1. Install [Go](https://golang.org/doc/install).

For Go 1.10 and below, install the dependencies (skip these steps if using Go 1.11 and above):
1a. Install [dep](https://github.com/golang/dep): `go get -u github.com/golang/dep/cmd/dep`

1b. Install dependencies by running `dep ensure -v` from the root of the repository.

2. The test suite requires access to an instance of Avatica running HSQLDB and an instance of Apache Phoenix running the
Phoenix Query Server.

You should then set the `HSQLDB_HOST` and `PHOENIX_HOST` environment variables. For example:
{% highlight shell %}
HSQLDB_HOST: http://hsqldb:8765
PHOENIX_HOST: http://phoenix:8765
{% endhighlight %}

3. To select the test suite, export `AVATICA_FLAVOR=HSQLDB` for Avatica HSQLDB or `AVATICA_FLAVOR=PHOENIX` for Phoenix.

4. Then run `go test -v ./...` from the root of the repository to execute the test suite.

## Releasing

### Preparing for release
1. If you have not set up a GPG signing key, set one up by following these [instructions](https://www.apache.org/dev/openpgp.html#generate-key).

2. If this release is a new major version (we are releasing 4.0.0 vs the current version 3.0.0), update the version in the
import path in `go.mod`. The import paths in the various sample code snippets should also be updated.

3. Since we need to support Go modules, tags must be prefixed with a `v`. For example, tag as `v3.1.0` rather than `3.1.0`.

### Build the release
1. Tag the release. For example, if releasing `vX.Y.Z` and this is `rc0`, execute: `git tag vX.Y.Z-rcN` on the master branch.

2. Push the tag to git: `git push origin vX.Y.Z-rcN`

2. From the root of the repository, run `./make-release-artifacts.sh`. You will be asked to select the tag to build release
artifacts for. The latest tag is automatically selected if no tag is selected. The release artifacts will be placed in a
folder named for the release within the `dist/` folder.

### Check the release before uploading
The name of the release folder must be in the following format: `apache-calcite-avatica-go-X.Y.Z-rcN`. The version must 
include release candidate identifiers such as `-rc0`, if they are present.

The files inside the release folder must have any release candidate identifiers such as `-rc1` removed, even if the
release is a release candidate. `src` must also be added to the filename.

For example, if we are uploading the `apache-calcite-avatica-go-3.0.0-rc1` folder, the files must be named 
`apache-calcite-avatica-go-3.0.0-src.tar.gz`. Note the inclusion of `src` in the filename.

The tar.gz must be named `apache-calcite-avatica-go-X.Y.Z-src.tar.gz`. 

There must be a GPG signature for the tar.gz named: `apache-calcite-avatica-go-X.Y.Z-src.tar.gz.asc`

There must be a SHA256 hash for the tar.gz named: `apache-calcite-avatica-go-X.Y.Z-src.tar.gz.sha256`

### Uploading release artifacts to dev for voting
`svn` must be installed in order to upload release artifacts.

1. Check out the Calcite dev release subdirectory: `svn co "https://dist.apache.org/repos/dist/dev/calcite/" calcite-dev`.

2. Move the release folder under `dist/` into the `calcite-dev` folder.

3. Add the new release to the svn repository: `svn add apache-calcite-avatica-go-X.Y.Z-rcN`. Remember to change the folder name to the
correct release in the command.

4. Commit to upload the artifacts: `svn commit -m "apache-calcite-avatica-go-X.Y.Z-rcN" --username yourapacheusername --force-log`
Note the use of `--force-log` to suppress the svn warning, because the commit message is the same as the name of the directory.

### Send an email to the Dev list for voting:

Send out the email for voting:
{% highlight text %} 
To: dev@calcite.apache.org
Subject: [VOTE] Release apache-calcite-avatica-go-X.Y.Z (release candidate N)

Hi all,

I have created a build for Apache Calcite Avatica Go X.Y.Z, release candidate N.

Thanks to everyone who has contributed to this release.
<Further details about release.> You can read the release notes here:
https://github.com/apache/calcite-avatica-go/blob/XXXX/site/_docs/go_history.md

The commit to be voted upon:
https://gitbox.apache.org/repos/asf/calcite-avatica-go/commit/NNNNNN

Its hash is XXXX.

The artifacts to be voted on are located here:
https://dist.apache.org/repos/dist/dev/calcite/apache-calcite-avatica-go-X.Y.Z-rcN/

The hashes of the artifacts are as follows:
src.tar.gz.sha256 XXXX

Release artifacts are signed with the following key:
https://people.apache.org/keys/committer/francischuang.asc

Please vote on releasing this package as Apache Calcite Avatica Go X.Y.Z.

The vote is open for the next 72 hours and passes if a majority of
at least three +1 PMC votes are cast.

[ ] +1 Release this package as Apache Calcite Go X.Y.Z
[ ]  0 I don't feel strongly about it, but I'm okay with the release
[ ] -1 Do not release this package because...


Here is my vote:

+1 (binding)

Francis
{% endhighlight %}

After vote finishes, send out the result:
{% highlight text %} 
Subject: [RESULT] [VOTE] Release apache-calcite-avatica-go-X.Y.Z (release candidate N)
To: dev@calcite.apache.org

Thanks to everyone who has tested the release candidate and given
their comments and votes.

The tally is as follows.

N binding +1s:
<names>

N non-binding +1s:
<names>

No 0s or -1s.

Therefore I am delighted to announce that the proposal to release
Apache Calcite Avatica Go X.Y.Z has passed.

Thanks everyone. We’ll now roll the release out to the mirrors.

Francis
{% endhighlight %}

### Promoting a release after voting
`svn` must be installed in order to upload release artifacts.

NOTE: Only official releases that has passed a vote may be uploaded to the release directory.

1. Check out the Calcite release directory: `svn co "https://dist.apache.org/repos/dist/release/calcite/" calcite-release`.

2. Copy the release into the `calcite-release` folder. Remember to check the name of the release's folder to ensure that it is in
the correct format.

3. Add the release to the svn repository: `svn add apache-calcite-avatica-go-X.Y.Z`. Remember to change the folder name to the
correct release in the command.

4. Commit to upload the artifacts: `svn commit -m "Release apache-calcite-avatica-go-X.Y.Z" --username yourapacheusername`.

5. Tag the final release in git and push it:

{% highlight bash %}
git tag vX.Y.Z X.Y.Z-rcN
git push origin vX.Y.Z
{% endhighlight %}

6. After 24 hours, announce the release by sending an announcement to the [dev list](https://mail-archives.apache.org/mod_mbox/calcite-dev/)
and [announce@apache.org](https://mail-archives.apache.org/mod_mbox/www-announce/).

An example of the announcement could look like:
{% highlight text %}
Subject: [ANNOUNCE] Apache Calcite Avatica Go X.Y.Z released
To: dev@calcite.apache.org

The Apache Calcite team is pleased to announce the release of Apache Calcite Avatica Go X.Y.Z.

Avatica is a framework for building database drivers. Avatica
defines a wire API and serialization mechanism for clients to
communicate with a server as a proxy to a database. The reference
Avatica client and server are implemented in Java and communicate
over HTTP. Avatica is a sub-project of Apache Calcite.

The Avatica Go client is a Go database/sql driver that enables Go
programs to communicate with the Avatica server.

Apache Calcite Avatica Go X.Y.Z is a minor release of Avatica Go
with fixes to the import paths after enabling support for Go modules.

This release includes updated dependencies, testing against more
targets and support for Go Modules as described in the release notes:

    https://calcite.apache.org/avatica/docs/go_history.html#vX-Y-Z

The release is available here:

    https://calcite.apache.org/avatica/downloads/avatica-go.html

We welcome your help and feedback. For more information on how to
report problems, and to get involved, visit the project website at

    https://calcite.apache.org/avatica

Francis Chuang, on behalf of the Apache Calcite Team
{% endhighlight %}