# interlok-version-catalog

This project is the shared dependency version source for Interlok and the published platform used by downstream projects.
 
It provides two related things:

- a Gradle version catalog in [gradle/libs.versions.toml](gradle/libs.versions.toml)
- a Java platform/BOM published from [build.gradle](build.gradle)

The purpose is to centralise shared dependency versions in one place and let downstream repos consume the platform instead of repeating version strings in many build files.

## Why this project exists

Interlok is used across many repositories. Over time, the same dependency families appear in multiple modules and repos:

- Interlok modules such as `interlok-core`, `interlok-common`, `interlok-client`, `interlok-stax`, `interlok-json`
- shared runtime libraries such as `slf4j`, `log4j2`, `jetty`, `junit`, `mockito`, `derby`, `xstream`, and others

Without a central version source, each project starts to drift and upgrades become noisy, repetitive, and hard to reason about.

This project gives us:

- a single source of truth for shared versions
- consistent upgrades across Interlok projects
- simpler downstream Gradle files
- one place to manage version changes and Dependabot updates
- a published dependency platform that downstream repos can consume safely

## What is in the catalog

The catalog covers the common shared dependencies used across the Interlok ecosystem, including:

- Interlok core modules
- supporting Interlok libs and runtimes
- logging and Jetty components
- JUnit and Mockito
- JAXB/Jakarta libraries
- database drivers and pool libraries
- ActiveMQ and other messaging dependencies
- core Apache Commons dependencies

The exact list is in [gradle/libs.versions.toml](gradle/libs.versions.toml).

## How the catalog works

The TOML file declares both the versions and the coordinates that use them.

Example:

```toml
[versions]
slf4j = "2.0.17"

[libraries]
slf4jApi = { module = "org.slf4j:slf4j-api", version.ref = "slf4j" }
```

The platform project publishes those entries as dependency constraints, so downstream builds can depend on a library without repeating a version.

Example:

```gradle
dependencies {
  constraints {
    api libs.slf4jApi
    api libs.junitJupiterApi
  }
}
```

This is the pattern behind the BOM: consumers choose a dependency, and Gradle resolves the version from the platform rather than from a literal in the build script.

## How to use it in a downstream project

A downstream project should consume the published platform like this:

```gradle
dependencies {
  implementation platform("com.adaptris:interlok-version-catalog:5.0-SNAPSHOT")

  implementation "com.adaptris:interlok-core"
  implementation "com.adaptris:interlok-common"
  implementation "org.slf4j:slf4j-api"
}
```

This keeps the version policy in the catalog rather than in each repo.

## How to add a dependency

To add a new shared dependency to the catalog:

1. add the version to the `[versions]` table
2. add the module mapping to `[libraries]`
3. add the library to the platform constraints in [build.gradle](build.gradle)

Example:

```toml
[versions]
myLibrary = "1.2.3"

[libraries]
myLibrary = { module = "com.example:my-library", version.ref = "myLibrary" }
```

```gradle
dependencies {
  constraints {
    api libs.myLibrary
  }
}
```

## How to remove a dependency

To remove a dependency from the catalog:

1. remove the library entry from `[libraries]`
2. remove the version from `[versions]` if it is no longer used
3. remove the corresponding `api libs.*` constraint from [build.gradle](build.gradle)
4. verify no other dependency still references it

This keeps the catalog and the published BOM aligned.

## How to update a version

To update a shared dependency:

1. change the version in `[versions]`
2. leave the library coordinate unchanged unless the artifact itself changed
3. validate the catalog project with Gradle
4. publish the updated platform

Example:

```toml
[versions]
slf4j = "2.0.20"
```

That single change will flow to any downstream repo using the platform.

## How to publish the catalog

This project is intended to be published as an artifact to the shared Nexus repository so other Interlok repos can consume it.

Typical flow:

1. update the shared version catalog as needed
2. verify the project builds cleanly
3. publish the catalog/platform artifact to the repository
4. downstream repos can then consume the published platform

## closing comment

This project is intentionally small and opinionated: one catalog file, one published BOM/platform, and one place to manage shared versions.

The benefit is lower maintenance cost, less drift, and easier dependency updates for the wider Interlok ecosystem.
