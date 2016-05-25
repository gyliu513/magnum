..
   This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Magnum and Open DC/OS Integration
=================================

Launchpad Blueprint:

https://blueprints.launchpad.net/magnum/+spec/mesos-dcos

Open DC/OS [1]_ is a distributed operating system based on the Apache Mesos
distributed systems kernel. It enables the management of multiple machines as
if they were a single computer. It automates resource management, schedules
process placement, facilitates inter-process communication, and simplifies
the installation and management of distributed services. Its included web
interface and available command-line interface (CLI) facilitate remote
management and monitoring of the cluster and its services.

Open DC/OS now supports both docker containerizer and mesos containerizer,
the mesos containerizer support both docker and AppC image spec, the mesos
containerizer can manage docker containers well even docker daemon is not
running.

End user can install Open DC/OS with different ways, such as vagrant, cloud,
local etc. For cloud, the Open DC/OS only support AWS now, end user can deploy
a DC/OS cluster quickly with a tempalte. But for local install, there are many
steps to install a Open DC/OS cluster.

Problem Description
===================

Containers are the first citizen in Magnum, but the concept of container is
not only limited in docker container, but also others, such as AppC, linux
container etc. Magnum now mainly focusing on docker container management
integration, it can now work well with Mesos, Kuernetes and Swarm. But most
of those COE are focusing on docker management and they cannot manage other
containers, such as AppC, linux container etc.

Currently, Magnum provides limited support for Mesos Bay as there is only one
framework named as Marathon running on top of Mesos. Compared with Open DC/OS,
the current Mesos Bay lack the following features:

1. App Store for application management. The Open DC/OS has a universe to
   to provide app store functions.

2. Different container technology support. The Open DC/OS support different
   container technologies, such as docker, AppC etc, and may introduce OCI
   support in future, introducing Open DC/OS Bay can enable Magnum support
   more container technologies.

3. Better external storage integration. The Open DC/OS is planning to introduce
   docker volume isolator support in next release, the docker volume isolator
   is leveraging docker volume driver API to integrate with 3rd party
   distributed storage platforms, such as OpenStack Cinder, GlusterFS, Ceph etc.

4. Better network management. The Open DC/OS is planning to introduce CNI
   network isolator in next release, the CNI network isolator is leveraging CNI
   technolgies to manage network for containers.

5. Loosely coupled with docker daemon. The Open DC/OS can work well for docker
   container even if docker daemon is not running. The docker daemon now have
   some issues in large scale cluster, decouple docker daemon but still can
   enable end user get some docker features in large scale cluster.


Proposed Changes
================

We propose extending Magnum as follows.

1. Adding a new coe type named as dcos.

2. Leverage mesos-slave-flags [3]_ to customize Open DC/OS.

     magnum baymodel-create --name dcosbaymodel \
                            --image-id fedora-21-atomic-5 \
                            --keypair-id testkey \
                            --external-network-id 1hsdhs88sddds889 \
                            --dns-nameserver 8.8.8.8 \
                            --flavor-id m1.small \
                            --docker-volume-size 5 \
                            --coe dcos\
                            --network-driver flannel \
                            --labels isolation=docker/volume,launcher=linux \
                                     ,image_providers=docker,appc

  Magnum will validate the labels together with the driver specified before
  creating the bay and will return an error if the validation fails.

  Magnum will continue to CRUD bays in the same way:

     magnum bay-create --name dcosbay --baymodel dcosbaymodel --node-count 1

3. Keep the old Mesos Bay and add a new Open DC/OS Bay. Once the Open DC/OS Bay
   is stable, deprecate the Mesos Bay. 

4. Update unit and functional tests to support Open DC/OS Bay.

5. Preserve the user experience by ensuring that any operation on Open DC/OS
   Bay will be identical between a COE deployed by Magnum and a COE deployed
   by other methods.


REST API Impact
---------------

There will be no REST API exposed from Magnum for end user to operate Open
DC/OS, end user can logon to Open DC/OS dashboard or call Open DC/OS REST
API directly to manage the containers or the applications.

Implementation
==============

Assignee(s)
-----------

Primary assignee:

- Guang Ya Liu (jay-lau-513)

Other contributors:

- Qun Wang (wangqun)
- Gao Jin Cao


Work Items
----------

1. Build VM image for Open DC/OS Bay.
2. Add Heat template for Open DC/OS Bay.
3. Add Open DC/OS Bay montior for auto scaling.
4. Document how to use the Open DC/OS Bay.

Dependencies
============

1. This blue print will focus on running on Open DC/OS in CentOS 7.1.

2. Depend on blueprint

https://blueprints.launchpad.net/magnum/+spec/mesos-slave-flags

Testing
=======

Each commit will be accompanied with unit tests. There will also be
functional tests which will be used as part of a cross-functional gate
test for Magnum.

Documentation Impact
====================

The Magnum Developer Quickstart document will be updated to support the
Open DC/OS Bay introduced by this document. Additionally, background
information on how to use the Open DC/OS Bay will be included.

References
==========

.. [1] https://dcos.io/docs/1.7/overview/what-is-dcos/
.. [2] https://dcos.io/install/
.. [3] https://blueprints.launchpad.net/magnum/+spec/mesos-slave-flags
