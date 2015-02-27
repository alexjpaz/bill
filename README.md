Bill
===
_Continuous deployment production environments, built on Docker, CoreOS, etcd and fleet._

Bill is a pluggable in-house service platform with a PaaS-like workflow.

![Screenshot](https://raw.githubusercontent.com/alexjpaz/bill/master/docs/images/more-awesomer-screenshot.png)

## Features
* Beautiful web UI
* Run anywhere (Vagrant, public cloud or bare metal)
* No special code required in your services
* Built for Continuous Deployment
* Zero-downtime deployments
* Service discovery
* Same workflow from dev to production
* Easy environments

## Components
* Web front-end - A beautiful UI for configuring and monitoring your services.
* Service directory - A catalog of your services and their configuration.
* Scheduler - Deploys services onto the platform.
* Orchestrator - REST API used by the web front-end; presents a unified subset of functionality from Scheduler, Service Directory, Fleet and Etcd.
* Centralised monitoring and logging.

### Service Directory
This is a database of all your services and their configuration (e.g. environment variables, data volumes, port mappings and the number of instances to launch). Ultimately this information will be reduced to a set of systemd unit files (by the scheduler) to be submitted to Fleet for running on the cluster.
This service has a REST API and is backed by a database (LevelDB).

### Scheduler
This service receives HTTP POST commands to deploy services that are defined in the service directory. Using the service data from the directory it will render unit files and run them on the CoreOS cluster using Fleet. A history of deployments and associated config is also available from the scheduler.

For each service the scheduler will deploy a container for the service and an announce sidekick container.

### Orchestrator
This is a service that ties all of the other services together, providing a single access-point for the front-end to interface with. Also offers a web socket endpoint for realtime updates to the web front-end.

### Web Front-End
A beautiful and easy-to-use web UI for managing your services and observing the health of your cluster. Built in Ember.

### Monitoring and Logging
Currently cAdvisor is used for monitoring, and there is no centralised logging. WIP.

## Installation

Bill's Docker repositories are hosted at Quay.io, but they are public so you don't need any credentials.

You will need to install `fleetctl` and `etcdctl`. On OS/X you can install both with brew:
```
$ brew install etcdctl fleetctl
```

### Vagrant

Clone this repository and run the following from the root directory of this repository:

```
$ ./scripts/install-vagrant.sh
```

This will bring up a three-node CoreOS Vagrant cluster and install Bill on it. Note that it may take 10 minutes or more to complete.

For extra debug output, run with `DEBUG=1` environment variable set.

If you already have a Vagrant cluster running and want to reinstall the units, use:

```
$./script/reinstall-units-vagrant.sh
```

To interact with the units in the cluster via Fleet, just specify the URL to Etcd on one of your hosts as a parameter to Fleet. e.g.:

```
$ fleetctl -strict-host-key-checking=false -endpoint=http://172.17.8.101:4001 list-units
```

You can also SSH into one of the VMs and run `fleetctl` from there:

```
$ cd coreos-vagrant
$ vagrant ssh core-01
```

...however bear in mind that Fleet needs to SSH into the other VMs in order to perform operations that involve calling down to systemd (e.g. `journal`), and for this you need to have SSHd into the VM running the unit in question. For this reason you may find it simpler (albeit more verbose) to run `fleetctl` from outside the CoreOS VMs.

### DigitalOcean

Bill has been tested on Digital Ocean but there isn't currently an install script for it. It shouldn't take much, just be sure to edit the BILL_DNSIMPLE_* values in `digitalocean/user-data`. Stay tuned...

## Tests

There is an integration test that brings up a CoreOS Vagrant cluster, installs Bill and then runs a contrived service on it and verifies that it works:

```
$ cd test
$ ./integration.sh
```

Each bill repository (service directory, orchestrator, scheduler) has tests that run on bill-ci.yld.io (in StriderCD), triggered by a Github webhook.

## Bill Repositories

The various components of Bill are spread across several repositories:
* [Orchestrator](https://github.com/yldio/bill-orchestrator)
* [Service Directory](https://github.com/yldio/bill-service-directory)
* [Scheduler](https://github.com/yldio/bill-scheduler)
* [Web](https://github.com/yldio/bill-web)
* [HAProxy](https://github.com/yldio/bill-haproxy)
