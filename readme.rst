Cypher
======

A virtual machine configuration for the NPR News Apps team. 

**Work in progress, still experimental**

Current issues: Many tools assume they're running on localhost, including our OAuth sign-in configuration. NPM has issues with symlinks for some modules. Forwarded ports may conflict with local toolchains.

About the name
--------------

'A cypher can be "built" virtually anywhere at any time: all that is required is a group of dancers. It does not require a stage, an audience, a roof, a dance floor, or even a designated block of time. The cypher's very informality and transience are part of its power; it appears when and where it is needed, then melts away.'

-- *Foundation: B-boys, B-girls, and Hip-Hop Culture in New York*, Joseph Schloss

Goals
-----

* Allow non-admin users on any platform to use NPR tools and workflows.
* Use NPR's legacy UNIX-only toolkits on Windows.
* Provide a stable development environment on MacOS.
* More directly match the live EC2 environments used for long-running news apps.
* Set up and tear down disposable VMs for quick experiments.

Not goals
---------

* Creating a secure environment
* Deploying application-specific containers
* Automating our EC2 instances

Getting started
---------------

This project is built on top of Vagrant and VirtualBox, both of which you should have installed, and which will require admin permission.

Clone this repo, navigate to it in a shell, and run `vagrant up` to start the VM (guest) for the first time. It will download a Linux operating system image, and then spend some time updating and installing dependencies. When it's done, it will return you to your shell. This can take some time the first time around, but will be much faster on repeat.

Once it's up, you can run the following commands to interface with the machine:

* ``vagrant suspend`` - put the VM to sleep (like closing the lid on a laptop).
* ``vagrant halt`` - power down the VM completely.
* ``vagrant up`` - power the machine on.
* ``vagrant ssh`` - connect to the machine to run commands.
* ``vagrant reload`` - restart the VM, re-running the Vagrantfile to update config changes.

Extra provisioning
------------------

When creating or starting a virtual machine, you can add a ``--provision-with=XXXXX`` flag to install libraries for specific tasks

* ``gdal`` - installs GDAL and other geospatial command-line tools

Basic usage
-----------

There's one rule we want to keep in mind: tools and applications run on the VM, but data stays on the host. Following this rule allows us to greatly simplify workflows, and makes it much easier to create or destroy VMs without losing work.

By default, the Vagrantfile will configure your VM to load the user's home directory as ``~/host``. You should clone repositories from GitHub to your preferred folder *on the host* and then access them from the VM. You can still source environment variables from ``workinprivate`` this way.

To access the guest, it's easiest to simply ``vagrant ssh`` into the machine to run commands. You can also reach the guest in a browser or other network services using a local IP of 192.168.187.187. Services running on ports 8000 and 7777 are forwarded to localhost, as well.

Example
-------

Let's say I'm setting up our liveblog to run on a Windows machine (which isn't normally possible) by running the tool in the guest.

1. Clone the repo using GitHub Deskop for Windows
#. Run ``vagrant up`` to start the VM.
#. Log in with ``vagrant ssh``.
#. The liveblog requires MongoDB - ``sudo apt-get install mongodb`` to install it.
#. Set up a virtualenv and install pip dependencies as usual.
#. Run ``fab text.get_liveblog`` and ``fab app:7777`` to start the local server.
#. Load ``192.168.187.187:7777`` in a browser to view the liveblog preview.

Best practices
--------------

If you have personal credentials that don't live in ``workinprivate/env.sh``, like AWS tokens, it can be useful to put them into a file that both host and guest can source from their bash config. For example, I keep a separate ``~/.secrets`` file that you can source from ``.bashrc`` on both host and guest machines. This split is also useful for giving presentations, so that you can open and update Bash without broadcasting tokens to the world.

You can open an SSH session to ``vagrant@localhost:2222`` using the client of your choice, if you want--the private keys are in this repo in both .pem (Linux/OS X) and .ppk (Windows) format. If you use AppleScript to open terminals for a project, you can adapt them by adding a line before any panel commands reading ``write text "ssh -i path/to/private_key.pem vagrant@localhost:2222"``.

It may be helpful to edit your machine's hostfile (``windows/system32/drivers/etc/hosts`` on Windows, ``etc/hosts`` on Linux/OS X) and add an alias for the guest IP (I use ``vagrant.local``).