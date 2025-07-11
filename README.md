# Luther Application Starter Kit (sandbox)

This repository contains a working starter kit for developers to modify and
specialize to their specific use case.

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/luthersystems/sandbox?quickstart=1)

## Local Setup

Once you've cloned the repo, run:

```sh
make setup
```

To download the docs and complete the local setup.

## High-level File System Structure

_Application Specific Code_: Add your specific process operations code to
the `phylum/` directory.

\_Application Starter Code: Edit the example template code in `portal/` and
`api/` to specialize for your use case.

_Platform_: The remaining files and directories are platform related code
that should not be modified.

## Component Diagram

```asciiart

                      +------------------+
                      |    FE Portal     |
                      +--------+---------+
                               |
                               v
                      +--------+---------+
                      |  Middleware API  | <---- Swagger:
                      +--------+---------+       api/swagger/oracle.swagger.json
                               |
                      +--------v---------+
                      |   Portal Service |
                      |  (./portal/)     |
                      +--------+---------+
                               |
                            JSON-RPC
                               |
                      +--------v---------+
                      |   Shiroclient    |
                      |   (gateway)      |
                      +--------+---------+
                               |
                            JSON-RPC
                               |
       +-----------------------v-----------------------+
       |       Common Operations Script (Phylum)       |
       |               ./phylum/README.md              |
       +-----------------------+-----------------------+
                               |
       +-----------------------v-----------------------+
       |  Substrate Runtime (ELPS interpreter, state)  |
       +-----------------------+-----------------------+
                               |
       +-----------------------v-----------------------+
       |     Distributed Systems Layer (Fabric)        |
       |               ./fabric/README.md              |
       +-----------------------------------------------+
```

This repo includes an end-to-end "hello world" application described below.

## Luther Documentation

Check out the [docs](https://docs.luthersystems.com) on our website.

Run `make init-docs` and the docs are available [here](./docs/README.md).

## Getting Started

### Codespaces

This repository can be used in the cloud using Github Codespaces. You may fork
it into your own organization to use your organization's subscription to Github
and the feature and apply the running costs to your spending limits, or you may
contact Luther about receiving permission to use our subscription.

To use codespaces:

- Select the Code pane on the repository main page, select the Codespaces tab,
  and select "New codespace".
- The minimum machine size (2 core, 4GB RAM, 32 GB storage) is preferred.
- Wait for initialization to complete; this will take less than 5 minutes.

### MacOS

On MacOS you can use the commands, using [homebrew](https://brew.sh/):

```bash
brew install make git go wget jq
brew install --cask docker
```

_IMPORTANT:_ Make sure your `docker --version` is >= 20.10.6 and
`docker-compose --version` is >= 1.29.1.

If you are not using `brew`, make sure xcode tools are installed:

```bash
xcode-select --install
```

### Ubuntu

If you are running Ubuntu 20.04+ you can use the commands to install the dependencies:

```bash
sudo apt update && sudo apt install make jq zip gcc python3-pip golang-1.16
```

Install docker using the official [steps](https://docs.docker.com/engine/install/ubuntu/).

Install docker-compose:

```bash
sudo pip3 install docker-compose
```

Make sure your [user has permissions](https://docs.docker.com/engine/install/linux-postinstall/)
to run docker.

See [this](https://dev.luthersystemsapp.com/ubuntu20_04-sandbox-install.sh)
script for the exact steps to install the dependencies on a fresh Ubuntu 20.04
instance.

## Build On Your Machine

Clone this repo:

```bash
git clone https://github.com/luthersystems/sandbox.git
```

Run `make` to build all the services:

```bash
make
```

## "Hello World" Application

This repo includes a small application for managing an insurance claim. It
serves a [JSON API](api/srvpb/v1/oracle.swagger.json) that provides endpoints to:

1. create a new insurance claim
2. add a claimant to the claim
3. retrieve claim details

> To simplify the sandbox, we have omitted authentication which we handle
> using [lutherauth](https://docs.luthersystems.com/luther/application/modules/lutherauth).
> Authorization is implemented at the application layer over tokens issued by
> lutherauth.

### Running the Application

First we'll run the sample application with a local instance of the Luther
platform (gateway, common operations script, and a fabric network).
Run `make up` to bring up a local docker network running the application and
platform containers.

```bash
make up
```

After this completes successfully run `docker ps` which lists the running
containers. The REST/JSON API is accessible from your localhost on port 8080
which can be spot-tested using cURL and jq:

```bash
curl -v http://localhost:8080/v1/health_check | jq .
```

With the containers running we can also run the end-to-end integration tests.
Once the tests complete `make down` will cleanup all the containers.

```bash
make integration
make down
```

Running `docker ps` again will show all the containers have been removed.

### Directory Structure

Overview of the directory structure:

| Directory          | Description                                                              |
| ------------------ | ------------------------------------------------------------------------ |
| `build/`           | Temporary build artifacts (do not check into Git)                        |
| `common.config.mk` | User-defined settings and overrides across the project                   |
| `api/`             | API specification and artifacts (see `api/README.md`)                    |
| `compose/`         | Docker Compose configurations used during testing and `make` targets     |
| `fabric/`          | Local distributed systems network configuration (not used in Codespaces) |
| `portal/`          | REST/JSON middleware service and app logic                               |
| `phylum/`          | Business logic written in Common Operations Script (ELPS)                |
| `scripts/`         | Helper scripts for the build process                                     |
| `tests/`           | End-to-end tests using `martin`, Luther’s e2e testing tool               |

### Developing the application

The API is defined using protobuf objects and service definitions under the
`api/` directory. Learn more about how the API is defined and the data model
definitions by reading the sandbox API's [documentation](api/).

The application API is served by the "oracle", which interfaces with the Luther
platform. Learn more about the design of the oracle and how to extend its
functionality by reading the sandbox oracle's
[documentation](portal/).

The oracle interacts with the core business logic that is defined by the
"phylum", [elps](https://github.com/luthersystems/elps) code that defines an
application's business rules. Learn more about writing phyla by reading the
sandbox phylum's [documentation](phylum/).

### Testing Modifications

There are 3 main types of tests in this project:

1. Phylum _unit_ tests. These tests exercise business rules and logic around
   storage of smart contract data model entities. More information about
   writing and running unit tests can be found in the phylum
   [documentation](phylum/README.md).
   See the example [claims test](./phylum/claim_test.lisp).

2. Oracle _functional_ tests. These tests exercise API endpoints and their
   connectivity to the phylum application layer. More information about writing
   and running functional tests can be found in the oracle
   [documentation](portal/README.md).

3. End-To-End _integration_ tests. These tests use the `martin` tool. These
   tests exercise realistic end-user functionality of the oracle REST/JSON APIs
   using [Postman](https://www.postman.com/product/api-client/) under the hood.
   More information about writing and running integration tests can be found in
   the test [documentation](tests/README.md)

After making some changes to the phylum's business logic, the oracle middleware,
or the API it is a good idea to test those changes. The quickest integrity
check to detect errors in the application is to run the phylum unit tests and
API functional tests from the phylum and oracle directories respectively. This
can be done easily from the application's top level with the following command:

```bash
make test
```

Instead of running the above command the phylum and oracle can be tested
individually with the following commands:

```bash
make phylumtest
make oraclegotest
```

If these tests pass then one can move on to run the end-to-end integration tests
against a real network of docker containers. As done in the Getting Started
section, this will require running `make up` to create a network and `make
integration` to actually run the tests.

> `make mem-up` is a faster alternative suitable for development.

```bash
make up
make integration
```

### Over-The-Air (OTA) Update

During development application, particularly if developing an application with a
UI, phylum bugs may be discovered while the application is running (i.e. `make
up`). After fixing bugs in the local phylum code, redeploy the code onto the
running fabric network with the following shell command:

```bash
(cd fabric && make init)
```

> This command rebuilds and re-installs the business logic on the running
> Fabric network without restarting the containers.

This uses the OTA Update module to immediately install the new business logic on
to the fabric network. The upgrade here is done the same way devops engineers
would perform the application upgrade when running the platform on production
infrastructure.

### Rapid Development and Live Reloading in an In-Memory Simulation

Constantly running a local instance of the Luther platform can consume a lot of
computer resources and running `make up` and `make down` frequently is time
consuming. Instead of running the complete platform it can be simulated
locally, in-memory. Running an in-memory version of the platform is much faster
and less resource intensive. In contrast to running the real platform which is
done with `make up` running the application with an in-memory platform is done
with the `make mem-up` command.

```bash
make mem-up
make integration
```

Running `docker ps` at this point will show that only the application
oracle/middleware is running. Beyond starting fast and consuming fewer
resources, the in-memory platform also features _live code reloading_ where any
phylum code changes will immediately be reflected in the running application.

If integration tests fail after making modifications you can diagnose them by
reading the test output and comparing that with the application logs which are
found by running the following command:

```bash
docker logs sandbox_oracle
```

As with running the real platform, the oracle docker container and the in-memory
platform are cleaned up by running the command:

```bash
make down
```

## Distributed Systems

See [fabric/README.md](./fabric/README.md) for an overview of the distributed systems
architecture that powers the platform.

## Advanced Usage

Once you're comfortable with the basic development workflow, checkout the
more advanced features.

### Application tracing (OpenTelemetry)

There is support for tracing of the application and the Luther platform using
the OpenTelemetry protocol. Each can optionally be configured by setting an
environment variable to point at an OTLP endpoint (e.g. a Grafana agent). When
configured, trace spans will be created at key layers of the stack and delivered
to the configured endpoint.

You can test tracing locally by setting the following env variables:

```bash
export SANDBOX_ORACLE_OTLP_ENDPOINT=http://tempo:4317
export SHIROCLIENT_GATEWAY_OTLP_TRACER_ENDPOINT=http://tempo:4317
export CHAINCODE_OTLP_TRACER_ENDPOINT=http://tempo:4317
```

And bringing up observability:

```
make observability-up
```

Login to the local [grafana](http://localhost:3000/) with username: admin,
password: admin.

Click "Explore" and select "Tempo" as the data source. Query `{}` to list
all traces.

#### ELPS trace spans

Phylum endpoints defined with `defendpoint` will automatically receive a span
named after the endpoint. Other functions in the phylum can be traced by adding
a special [ELPS](https://github.com/luthersystems/elps) doc keyword:

```lisp
(defun trace-this ()
  "@trace"
  (slow-function1)
  (slow-function2))
```

Custom span names are also supported as follows:

```
"@trace{ custom span name }"
```

### Run Distributed Systems Explorer

To examine a graphical UI for the transactions and blocks and look at
the details of the work the sandbox network has done, build the
Explorer. With the full network running, run:

```bash
make explorer
```

This creates a web app which will be visible on `localhost:8090`. The default
login credentials are username: `admin`, password `adminpw`. Bringing up the
network should produce some transactions and blocks, and `make integration` will
generate more activity, which can be viewed in the web app.

If the `make` command fails, or if the Explorer runs but no new activity is
detected, it has most likely failed to authenticate; run

```bash
make explorer-clean
make explorer-up
```

To wipe out the pre-existing database and recreate it empty, then re-build the
Explorer. This will reconnect it to the current network.

## Platform Releases

See [Latest Platform Releases](https://docs.luthersystems.com/deployment/release-notes).
