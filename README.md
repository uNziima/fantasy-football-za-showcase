# Fantasy Football ZA

Fantasy football for the South African Premier Soccer League (PSL). Players pick a squad of real PSL footballers and score points from how those footballers perform each match week.

> **This is a showcase repository for a product in active development.** The source code, scoring values and pricing model are private. This page shows how the system is engineered. **Code walkthroughs are available on request for interviews.**

## Why it exists

Fantasy football is hugely popular in South Africa, but almost all of it is built around the English Premier League. Fantasy Football ZA is built for the local league, local fans and local data costs.

## Architecture

```
┌─────────────────────────────────────────────┐
│  ffza-app                                   │
│  HTTP routes (Javalin), configuration,      │
│  data provider client, persistence          │
│                    │                        │
│                    ▼ depends on             │
│  ┌───────────────────────────────────────┐  │
│  │  ffza-scoring                         │  │
│  │  Scoring engine and value types       │  │
│  │  No dependencies. No database access. │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

| Module | Depends on | Contains |
|---|---|---|
| `ffza-scoring` | nothing | The scoring engine and its value types |
| `ffza-app` | `ffza-scoring` and a small set of libraries | Everything else |

## Engineering decisions

- **Scoring is a pure function, enforced by the build.** The requirements say scoring must depend only on its inputs. Instead of relying on a convention, the scoring module has no compile-time dependencies and a Java module descriptor that reads nothing. Importing database code into it fails the build.
- **The build pins the Java version.** Maven's enforcer fails the build at the `validate` phase on anything other than Java 21, before a single file is compiled.
- **Page weight has a budget.** A guard in `mvn verify` fails the build if any page exceeds its size budget, and prints the measured size on every run. South African mobile data is expensive, so this is a product requirement, not a nice-to-have.
- **Secrets never leave the config file.** Configuration is gitignored, and values are never logged, returned by an endpoint or included in an error message. Only key names are ever reported.
- **Tests do not need a database.** A fresh checkout builds and tests with `mvn clean verify` and no manual setup.
- **Deployment is scripted but not automated.** A systemd unit runs the app as a dedicated unprivileged user, and a deploy script copies a built artifact to a host that already exists. It never creates infrastructure and never accepts credentials.

## Documentation written before code

- Functional and non-functional requirements, with an amendment log
- Architecture: hosting, libraries, modules, routes and budgets
- Data model
- Field tests of candidate football data providers

## Status

In active development. Build, module boundaries, configuration and health endpoint are in place. Hosting is planned on Oracle Cloud.

## Tech stack

Java 21 (Eclipse Temurin) · Maven (multi-module, JPMS) · Javalin · systemd · Oracle Cloud (planned)

## Author

**Ulikhaya Mazibuko** · [ulikhayamazibuko.com](https://ulikhayamazibuko.com) · [LinkedIn](https://www.linkedin.com/in/ulikhaya-mazibuko-43623b261) · ulikhayamazibuko@gmail.com
