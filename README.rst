* Install all below requirements
* After generating inventory, add bastion ansible_connection=local
* Setup machines by running debops common stuff
* provision my environment


=====================================================
Using Debops with a multi-machine Vagrantfile
=====================================================

This project is an example for using Debops with a multi-machine
Vagrantfile. It uses the Vagrant provisioner plugin Vai_ to generate
the inventory-file. Provision is then actually done using `debops` as
usual.

The Vagrant Ansible provisioner is not used at all, because Vai does a
much better job for our needs. (A former version of this example used
the Vagrant Ansible provisioner, but have been much more complicated, see
below.)


Requirements
==============

* Ansible
* Vagrant 1.6 or newer (1.5 may work, too; 1.4 does not)
* Vai_, a Vagrant provisioner plugin (see below for installation)
* `debops` (of course ;-)


Quick Start
===========

* Install the Vagrant provisioning plugin Vai_::

    vagrant plugin install vai

* Fire up Vagrant: ``vagrant up``

  This will create two virtual machine `web` and `db` and generate the
  inventory file in ``ansible/inventory``.

* Run::

    ANSIBLE_SSH_ARGS="-o UserKnownHostsFile=/dev/null" debops ./simpletest.yml

  This will run a simple playbook testing if files have been found and
  variables have been set up as correctly as expected. You should get
  "okay" for all tasks.

  Please note: Using this ``ANSIBLE_SSH_ARGS=...`` is optional. But it
  avoids cluttering your known_hosts with keys of your ever-changing
  vagrant VMs. *Absolutely do not use this for your production servers!*

Now you can use debops as usual (mind ``ANSIBLE_SSH_ARGS=...`` :-). It
will automatically include the host definitions auto-generated by
vagrant. If for some reason debops resp. Ansible does not find the
inventory, you may safely run ``vagrant provision`` to regenerate it.


How it works
==============

The `Vai` provisioner generates a inventory file which is setting up
host, port, private key file and the remote user as required by
Vagrant. Ansible resp. debops will read the information an know how to
connect to the Vagrant VM.



Adopting to your needs
=========================

In short:

* Adopt the paths in the Vagrantfile to match your desired directory
  layout.

* Set up your host- and group-vars in ``ansible/inventory`` as usual.

* Define your groups in the Vagrantfile and/or in an inventory-file
  (e.g. ``ansible/inventory/groups``, the actual name of the file does
  not matter).

  In this example we do both: ``secondGroup`` is defined using the
  `Vagrantfile` and ``firstGroup`` is defined in an inventory-file.

* Add more machines to the ``Vagrantfile`` as you need.

* When (re-) starting machines the inventory file will be regenerated
  by the Ansible provisioner.



Why using `Vai` instead of the `Ansible` provisioner
=====================================================

A former version of this example used the official Vagrant Ansible
provisioner. But this has be much more complicated, inflexible and
insecure.

The Vagrant Ansible provider was only used to generate the
inventory-file. But this inventory-file did not contain all required
information, so the remote user and the private key file have had been
defined in :file:`.debops.cfg`.

Additionally the Vagrant file was required to contain code for
creating a dummy playbook (which the Ansible provisioner requires) and
sym-linking the inventory-file into the inventory.

So as `Vai` came up, we decided to switch to it.


Reasoning for why we formerly did it that way
----------------------------------------------

We tried a lot off different setups, but the only one working
reasonable is to *not* provision from within the Vagrantfile, but use
the Ansible provisioner solely to generate the inventory file. Here is
why:

When setting up the Ansible provisioner as shown in the Vagrant and
Ansible documentation, ``ansible-playbook`` will be run once for each
machine, one machine after each other. Parallel multi-machine
provisioning is not possible this way.

When passing ``ansible.limit = 'all'`` to the Ansible provisioner (as
described in the docs, too) Vagrant will run ``ansible-playbook`` for
*all* boxes, but so many times as you have defined boxes.

Working around this is hard and the results are not satisfying.

If you intimately read the docs, you will find that the Ansible
provider is only attached to the last machine. And indeed: Now when
setting ``ansible.limit ='all'`` `vagrant` will run
``ansible-playbook`` once and only once on all machines. (Side-note:
If you want to define the machines using a loop, you have to be very
careful doing this right, see
`<https://github.com/mitchellh/vagrant/issues/1784#issuecomment-62460418
this comment>`_)

But this again has a downside: Limiting provisioning to a single
machines (``vagrant provision [vm-name]``) does not work for all
machines except the last. This is because the provisioner is only
attached to the last machine.

* About the vagrant Ansible provisioner:

  - If `ansible.inventory_path` is set, the provider will not
    generate an inventory file. You will have to take care of this by
    yourself.
  - The path in `ansible.inventory_path`, if given, must already exist.
  - The executable is hard-coded to `ansible-playbook`.



Alternative setup
=====================

Defining groups in the Vagrantfile
-------------------------------------

The Ansible provisioner supports defining groups in the Vagrantfile.
You may do this, if you like, but we do not recommend it, because we
think it make things more complicated.


Using a hand crafted inventory
-------------------------------

If for some reason you prefer to craft the inventory yourself (instead
of letting vagrant generate it) you can completely remove the Ansible
provisioner from the Vagrantfile. It's sole purpose is to generate the
inventory-file.

Please note that when using a hand-crafted inventory. you will have to
take care of the actual configuration of the machines changing. E.g.
IP-ports may change if other machines are running, too.


.. _Vai: https://github.com/MatthewMi11er/vai

..
 Local Variables:
 mode: rst
 ispell-local-dictionary: "american"
 End:
