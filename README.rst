backup-vm
=========

Back up your libvirt-based VMs using Borg_!

.. _Borg: https://github.com/borgbackup/borg

Features
--------

* Backup running VMs

  * Automatically creates a `COW snapshot`_ of virtual disks to avoid corruption and pivots_ them back afterwards
  * From the perspective of the VM, restoring from a live backup is like a sudden power-off

    * Chances of file corruption are still low with a `guest agent`_ installed

  * Can back up RAM contents *(Experimental)*

    * Restoring from RAM acts like normal PC hibernation

* Can back up multiple VM disks

  * Supports disk images backed by a file or a block device

* Can back up to multiple Borg repositories at once

  * Only one snapshot operation needed for multiple backups
  * Auto-answers subsequent prompts from other borg processes
  * Shows total backup progress % (even with multiple backups)

* Pass extra arguments straight to Borg on the command line

  * Different settings (e.g. compression) can be passed to each instance

.. _COW snapshot: https://wiki.libvirt.org/page/Snapshots
.. _pivots: https://wiki.libvirt.org/page/Live-disk-backup-with-active-blockcommit
.. _guest agent: https://wiki.libvirt.org/page/Qemu_guest_agent

Examples
--------

Backup
^^^^^^

Back up a virtual machine to a single Borg repo::

    backup-vm myVM myrepo::myBackup

Back up a virtual machine ``webserver`` to an onsite and an offsite Borg repository with varying compression settings::

    backup-vm webserver onsite::webserver-{now:%Y-%m-%d} --borg-args --compression lz4 offsite::webserver-{now:%Y-%m-%d} --borg-args --compression zlib,9

Back up only the system drive of a Windows VM::

    backup-vm win10 sda myrepo::win10-{now:%Y-%m-%d}

Restore
^^^^^^^

A script for automatic restoration is `in development`_; however, the backups are saved with a simple directory structure that makes manual restoration easy. Each backup has the image of each disk clearly named in the root directory (e.g. ``sda.raw``, ``hdb.qcow2``). The legacy `bash script`_ for restoring follows a similar process to what the Python version will, with the notable exception that it does not handle multiple disks.

.. _in development: https://github.com/milkey-mouse/backup-vm/issues/1
.. _bash script: https://github.com/milkey-mouse/backup-vm/blob/bash-script/restore-vm.sh

Usage
-----

::

   usage: backup-vm [-hmp] domain [disk [disk ...]] archive
       [--borg-args ...] [archive [--borg-args ...] ...]

   Back up a libvirt-based VM using borg.

   positional arguments:
   domain           libvirt domain to back up
   disk             a domain block device to back up (default: all disks)
   archive          a borg archive path (same format as borg create)

   optional arguments:
   -h, --help       show this help message and exit
   -m, --memory     (experimental) snapshot the memory state as well
   -p, --progress   force progress display even if stdout isn't a tty
   --borg-args ...  extra arguments passed straight to borg


Installation
------------

Python ≥3.4 is required, as well as the Python libvirt bindings. If possible, install them from the system package manager (``apt install python3-libvirt``); otherwise, use pip (``pip install libvirt-python``). To install the script, copy it into ``/usr/local/bin`` and optionally remove the ``.py`` extension.