===============
Getting started
===============

Who should read this guide
~~~~~~~~~~~~~~~~~~~~~~~~~~

This guide is for software developers who want to deploy applications to
OpenStack clouds.

We assume that you're an experienced programmer who has not created a cloud
application in general or an OpenStack application in particular.

If you're familiar with OpenStack, this section teaches you how to program
with its components.

What you will learn
~~~~~~~~~~~~~~~~~~~

Deploying applications in a cloud environment can be very different from
deploying them in a traditional IT environment. This guide teaches you how to
deploy applications on OpenStack and some best practices for cloud application
development.

A general overview
~~~~~~~~~~~~~~~~~~

This tutorial shows two applications. The first application is a simple
fractal generator that uses mathematical equations to generate beautiful
`fractal images <http://en.wikipedia.org/wiki/Fractal>`_ . We show
you this application in its entirety so that you can compare it to the second,
more robust, application.

The second application is an OpenStack application that enables you to:

* Create and destroy compute resources. These resources are virtual
  machine instances where the Fractals application runs.
* Make cloud-related architecture decisions such as turning
  functions into micro-services and modularizing them.
* Scale available resources up and down.
* Use Object and Block storage for file and database persistence.
* Use Orchestration services to automatically adjust to the environment.
* Customize networking for better performance and segregation.
* Explore and apply advanced OpenStack cloud features.

Choose your OpenStack SDK
~~~~~~~~~~~~~~~~~~~~~~~~~

This guide focuses on how to use Python with Apache Libcloud. Anyone with a
programming background can easily read the code in this guide. Although this
guide focuses on Libcloud, you can use other languages and toolkits with the
OpenStack cloud:

============= ============= ================================================================= ====================================================
Language      Name          Description                                                       URL
============= ============= ================================================================= ====================================================
Python        Libcloud      A Python-based library managed by the Apache Foundation.
                            This library enables you to work with multiple types of clouds.   https://libcloud.apache.org
Python        OpenStack SDK A python-based library specifically developed for OpenStack.      https://github.com/stackforge/python-openstacksdk
Java          jClouds       A Java-based library. Like Libcloud, it's also managed by the     https://jclouds.apache.org
                            Apache Foundation and works with multiple types of clouds.
Ruby          fog           A Ruby-based SDK for multiple clouds.                             http://www.fogproject.org
node.js       pkgcloud      A Node.js-based SDK for multiple clouds.                          https://github.com/pkgcloud/pkgcloud
PHP           php-opencloud A library for developers using PHP to work with OpenStack clouds. http://php-opencloud.com/
NET Framework OpenStack SDK A .NET based library that can be used to write C++ applications.  https://www.nuget.org/packages/OpenStack-SDK-DotNet
              for Microsoft
              .NET
============= ============= ================================================================= ====================================================

For a list of available SDKs, see `Software Development Kits <https://wiki.openstack.org/wiki/SDKs>`_.

Future versions of this guide will show you how to use the OpenStack SDK and
languages such as Java and Ruby to complete these tasks. If you're a developer
for another toolkit that you would like this guide to include, feel free to
submit code snippets. You can also contact OpenStack Documentation team
members.

What you need
-------------

We assume that you can already access an OpenStack cloud. You must have a
project (also known as a tenant) with a minimum quota of six instances.
Because the Fractals application runs in Ubuntu, Debian, Fedora-based, and
openSUSE-based distributions, you must create instances that use one of these
operating systems.

To interact with the cloud, you must also have

.. only:: dotnet

      `OpenStack SDK for Microsoft .NET 0.9.1 or higher installed
      <https://www.nuget.org/packages/OpenStack-SDK-DotNet>`_.

      .. warning::

         This document has not yet been completed for the .NET SDK.

.. only:: fog

      `fog 1.19 or higher installed
      <http://www.fogproject.org/wiki/index.php?title=FOGUserGuide#Installing_FOG>`_
      and working with ruby gems 1.9.

      .. warning::

         This document has not yet been completed for the fog SDK.

.. only:: jclouds

    `jClouds 1.8 or higher installed <https://jclouds.apache.org/start/install>`_.

    .. warning::

       This document has not yet been completed for the jclouds SDK.

.. only:: libcloud

  `libcloud 0.15.1 or higher installed
  <https://libcloud.apache.org/getting-started.html>`_.

.. only:: node

      `a recent version of pkgcloud installed
      <https://github.com/pkgcloud/pkgcloud#getting-started>`_.

      .. warning::

         This document has not yet been completed for the pkgcloud SDK.

.. only:: openstacksdk

    the OpenStack SDK installed.

    .. warning::

       This document has not yet been completed for the OpenStack SDK.

.. only:: phpopencloud

    `a recent version of php-opencloud installed <http://docs.php-opencloud.com/en/latest/>`_.

    .. warning::

       This document has not yet been completed for the php-opencloud SDK.

You need the following information that you can obtain from your cloud
provider:

* auth URL
* user name
* password
* project ID or name (projects are also known as tenants)
* cloud region

You can also download the OpenStack RC file from the OpenStack dashboard. Log
in to the Horizon dashboard and click :guilabel:`Project->Access &
Security->API Access->Download OpenStack RC file`. If you choose this route,
be aware that the "auth URL" doesn't include the path. For example, if your
:file:`openrc.sh` file shows:

.. code-block:: bash

        export OS_AUTH_URL=http://controller:5000/v2.0

the actual auth URL will be

.. code-block:: python

        http://controller:5000


How you'll interact with OpenStack
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this tutorial, you interact with your OpenStack cloud through one of the
SDKs listed in "Choose your OpenStack SDK." The code snippets in this
initial version of the guide assume that you're using Libcloud.

.. only:: fog

    .. literalinclude:: ../samples/fog/getting_started.rb
        :start-after: step-1
        :end-before: step-2

.. only:: libcloud

    To try it, add the following code to a Python script (or use an
    interactive Python shell) by calling :code:`python -i`.

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-1
        :end-before: step-2

.. only:: openstacksdk

    .. code-block:: python

      from openstack import connection
      conn = connection.Connection(auth_url="http://controller:5000/v3",
                                   user_name="your_auth_username",
                                   password="your_auth_password", ...)


.. note:: Because the tutorial uses the :code:`conn` object,
          make sure that you always have one handy.

.. only:: libcloud

    .. note:: If you receive the
              :code:`libcloud.common.types.InvalidCredsError: 'Invalid
              credentials with the provider'` exception while trying to run
              one of the following API calls, double-check your credentials.

    .. note:: If your provider does not support regions, try a
              blank string ('') for the `region_name`.

Flavors and images
~~~~~~~~~~~~~~~~~~

To run your application, you must launch an instance. This instance serves as
a virtual machine.

To launch an instance, you choose a flavor and an image. The flavor represents
the size of the instance, including the number of CPUs and amount of RAM and
disk space. An image is a prepared OS installation from which you clone your
instance. When you boot instances in a public cloud, larger flavors can be
more expensive than smaller ones in terms of resources and monetary cost.

To list the images that are available in your cloud, run some API calls:

.. only:: fog

    .. literalinclude:: ../samples/fog/getting_started.rb
        :start-after: step-2
        :end-before: step-3

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-2
        :end-before: step-3

    You should see a result something like:

    .. code-block:: python

        <NodeImage: id=2cccbea0-cea9-4f86-a3ed-065c652adda5, name=ubuntu-14.04, driver=OpenStack  ...>
        <NodeImage: id=f2a8dadc-7c7b-498f-996a-b5272c715e55, name=cirros-0.3.3-x86_64, driver=OpenStack  ...>

You can also get information about available flavors:

.. only:: fog

    .. literalinclude:: ../samples/fog/getting_started.rb
        :start-after: step-3
        :end-before: step-4

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-3
        :end-before: step-4

    This code produces output like:

    .. code-block:: python

        <OpenStackNodeSize: id=1, name=m1.tiny, ram=512, disk=1, bandwidth=None, price=0.0, driver=OpenStack, vcpus=1,  ...>
        <OpenStackNodeSize: id=2, name=m1.small, ram=2048, disk=20, bandwidth=None, price=0.0, driver=OpenStack, vcpus=1,  ...>
        <OpenStackNodeSize: id=3, name=m1.medium, ram=4096, disk=40, bandwidth=None, price=0.0, driver=OpenStack, vcpus=2,  ...>
        <OpenStackNodeSize: id=4, name=m1.large, ram=8192, disk=80, bandwidth=None, price=0.0, driver=OpenStack, vcpus=4,  ...>
        <OpenStackNodeSize: id=5, name=m1.xlarge, ram=16384, disk=160, bandwidth=None, price=0.0, driver=OpenStack, vcpus=8,  ...>


Your images and flavors will be different, of course.

Choose an image and flavor for your instance. You need about 1GB RAM, 1 CPU,
and a 1GB disk. This example uses the Ubuntu image with the :code:`m1.small`
flavor, which are safe choices. In subsequent tutorial sections in this guide,
you must change the image and flavor IDs to correspond to the image and flavor
that you choose.

If the image that you want is not available in your cloud, you can usually
upload one depending on your cloud's policy settings. For information about
how to upload images, see
`obtaining images <http://docs.openstack.org/image-guide/content/ch_obtaining_images.html>`_.

Set the image and size variables to appropriate values for your cloud. We'll
use these variables in later sections.

First, tell the connection to get a specifed image by using the ID of the
image that you picked in the previous section:

.. only:: fog

    .. literalinclude:: ../samples/fog/getting_started.rb
        :start-after: step-4
        :end-before: step-5

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-4
        :end-before: step-5

    You should see output something like this:

    .. code-block:: python

         <NodeImage: id=2cccbea0-cea9-4f86-a3ed-065c652adda5, name=ubuntu-14.04, driver=OpenStack  ...>

Next, tell the script which flavor you want to use:

.. only:: fog

    .. literalinclude:: ../samples/fog/getting_started.rb
        :start-after: step-5
        :end-before: step-6

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-5
        :end-before: step-6

    You should see output something like this:

    .. code-block:: python

        <OpenStackNodeSize: id=3, name=m1.medium, ram=4096, disk=40, bandwidth=None, price=0.0, driver=OpenStack, vcpus=2,  ...>

Now, you're ready to launch the instance.

Launch an instance
~~~~~~~~~~~~~~~~~~

Use your selected image and flavor to create an instance.

.. only:: libcloud

    .. note:: The following instance creation example assumes that you have a
              single-tenant network. If you receive the 'Exception: 400 Bad
              Request Multiple possible networks found, use a Network ID to be
              more specific' error, you have multiple-tenant networks. You
              must add a `networks` parameter to the `create_node` call. See
              :doc:`/appendix` for details.

Create the instance.

.. note:: Your SDK might call an instance a 'node' or 'server'.

.. only:: fog

    .. literalinclude:: ../samples/fog/getting_started.rb
        :start-after: step-6
        :end-before: step-7

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-6
        :end-before: step-7

    You should see output something like:

    .. code-block:: python

       <Node: uuid=1242d56cac5bcd4c110c60d57ccdbff086515133, name=testing, state=PENDING, public_ips=[], private_ips=[], provider=OpenStack ...>

.. only:: openstacksdk

    .. code-block:: python

       args = {
           "name": "testing",
           "flavorRef": flavor,
           "imageRef": image,
       }
       instance = conn.compute.create_server(**args)

If you list existing instances:

.. only:: fog

    .. literalinclude:: ../samples/fog/getting_started.rb
        :start-after: step-7
        :end-before: step-8

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-7
        :end-before: step-8

The new instance appears.

.. only:: libcloud

    .. code-block:: python

       <Node: uuid=1242d56cac5bcd4c110c60d57ccdbff086515133, name=testing, state=RUNNING, public_ips=[], private_ips=[], provider=OpenStack ...>

.. only:: openstacksdk

    .. code-block:: python

       instances = conn.compute.list_servers()
       for instance in instances:
           print(instance)

Before you move on, you must do one more thing.

Destroy an instance
~~~~~~~~~~~~~~~~~~~

Cloud resources such as running instances that you no longer use can cost
money. Destroy cloud resources to avoid unexpected expenses.

.. only:: fog

    .. literalinclude:: ../samples/fog/getting_started.rb
        :start-after: step-8
        :end-before: step-9

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-8
        :end-before: step-9


If you list the instances again, the instance disappears.

Leave your shell open to use it for another instance deployment in this
section.

Deploy the application to a new instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that you know how to create and destroy instances, you can deploy the
sample application. The instance that you create for the application is
similar to the first instance that you created, but this time, we'll briefly
introduce a few extra concepts.

.. note:: Internet connectivity from your cloud instance is required
          to download the application.

When you create an instance for the application, you'll want to give it a bit
more information than you supplied to the bare instance that you just created
and destroyed. We'll go into more detail in later sections, but for now,
simply create the following resources so that you can feed them to the
instance:

* A key pair. To access your instance, you must import an SSH public key into
  OpenStack to create a key pair. OpenStack installs this key pair on the new
  instance. Typically, your public key is written to :code:`.ssh/id_rsa.pub`. If
  you do not have an SSH public key file, follow
  `these instructions <https://help.github.com/articles/generating-ssh- keys/>`_ first.
  We'll cover these instructions in depth in :doc:`/introduction`.

.. only:: fog

    .. warning:: This section has not been completed.

.. only:: libcloud

    In the following example, :code:`pub_key_file` should be set to
    the location of your public SSH key file.

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-9
        :end-before: step-10

    ::

       <KeyPair name=demokey fingerprint=aa:bb:cc... driver=OpenStack>

* Network access. By default, OpenStack filters all traffic. You must create
  a security group and apply it to your instance. The security group allows HTTP
  and SSH access. We'll go into more detail in :doc:`/introduction`.

.. only:: fog

    .. literalinclude:: ../samples/fog/getting_started.rb
        :start-after: step-10
        :end-before: step-11

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-10
        :end-before: step-11

* Userdata. During instance creation, you can provide userdata to OpenStack to
  configure instances after they boot. The cloud-init service applies the
  userdata to an instance. You must pre-install the cloud-init service on your
  chosen image. We'll go into more detail in :doc:`/introduction`.

.. only:: fog

    .. warning:: This section has not been completed.

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-11
        :end-before: step-12

Now, you can boot and configure the instance.

Boot and configure an instance
------------------------------

Use the image, flavor, key pair, and userdata to create a instance. After you
request the instance, wait for it to build.

.. only:: fog

    .. warning:: This section has not been completed.

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-12
        :end-before: step-13

When the instance boots, the `ex_userdata` variable value instructs the
instance to deploy the Fractals application.

Associate a floating IP for external connectivity
-------------------------------------------------

We'll cover networking in detail in :doc:`/networking`.

To see the application running, you must know where to look for it. By
default, your instance has outbound network access. To make your instance
reachable from the Internet, you need an IP address. By default in some cases,
your instance is provisioned with a publicly rout-able IP address. In this
case, you'll see an IP address listed under `public_ips` or `private_ips` when
you list the instances. If not, you must create and attach a floating IP
address to your instance.

.. only:: fog

    .. warning:: This section has not been completed.

.. only:: libcloud

    Use :code:`ex_list_floating_ip_pools()` and select the first floating IP
    address pool. Allocate this pool to your project and attach it to your
    instance.

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-13
        :end-before: step-14

.. todo:: remove extra blank line after break

    You should see the floating IP output to the command line:

    ::

        <OpenStack_1_1_FloatingIpAddress: id=4536ed1e-4374-4d7f-b02c-c3be2cb09b67, ip_addr=203.0.113.101, pool=<OpenStack_1_1_FloatingIpPool: name=floating001>, driver=<libcloud.compute.drivers.openstack.OpenStack_1_1_NodeDriver object at 0x1310b50>>

    You can then attach it to the instance:

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-14
        :end-before: step-15

Run the script to start the deployment.

Access the application
----------------------

Deploying application data and configuration to the instance can take some
time. Consider enjoying a cup of coffee while you wait. After the application
deploys, you can visit the awesome graphic interface at the following link
using your preferred browser.

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
        :start-after: step-15

.. note:: If you do not use floating IPs, substitute another IP address as appropriate

.. figure:: images/screenshot_webinterface.png
    :width: 800px
    :align: center
    :height: 600px
    :alt: screenshot of the webinterface
    :figclass: align-center

Next steps
~~~~~~~~~~

Don't worry if these concepts are not yet completely clear. In
:doc:`/introduction`, we explore these concepts in more detail.

* :doc:`/scaling_out`: Learn how to scale your application
* :doc:`/durability`: Learn how to use Object Storage to make your application durable
* :doc:`/block_storage`: Migrate the database to block storage, or use
  the database-as-a-service component
* :doc:`/orchestration`: Automatically orchestrate your application
* :doc:`/networking`: Learn about complex networking
* :doc:`/advice`: Get advice about operations
* :doc:`/craziness`: Learn some crazy things that you might not think to do ;)

.. todo:: List the next sections here or simply reference introduction.

Complete code sample
~~~~~~~~~~~~~~~~~~~~

The following file contains all of the code from this section of the
tutorial. This comprehensive code sample lets you view and run the code
as a single script.

Before you run this script, confirm that you have set your authentication
information, the flavor ID, and image ID.

.. only:: libcloud

    .. literalinclude:: ../samples/libcloud/getting_started.py
       :language: python
