NVIDIA Driver & CUDA
####################

For simplicity, we install both NVIDIA driver and CUDA from the CUDA installer.

.. contents:: :depth: 2

Before Start
============

We need to install development package for building kernel modules.

::

    yum install kernel-devel dkms

.. note::
    Make sure the version of installed ``kernel-devel`` matches your current kernel version!
    
    You can check your current kernel version with ``uname -r``.
    

Install
=======

Block Nouveau
^^^^^^^^^^^^^

In ``/etc/default/grub``, add ``nouveau.modeset=0 rd.driver.blacklist=nouveau modprobe.blacklist=nouveau`` to ``GRUB_CMDLINE_LINUX``.

Then regenerate grub config files.
::

    grub2-mkconfig -o /boot/grub2/grub.cfg            # Legacy boot system
    # grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg # EFI boot system

Reboot the machine to make settings take effect.

.. hint::
    Use ``lsmod | grep nouveau`` to check if ``nouveau`` is loaded.

Driver & CUDA
^^^^^^^^^^^^^

Download CUDA installer from `NVIDIA Developer website <https://developer.nvidia.com/cuda-toolkit-archive>`_. Choose the *runfile(local)* option for offline installation.

Run the installer in silent mode. Use ``--help`` to see more install options.
::

    ./cuda_10.0.130_410.48_linux.run --silent --driver --toolkit


Post Setup
==========

Persistence Mode
^^^^^^^^^^^^^^^^

Turn on persistence mode.
::

    nvidia-smi -pm 1

