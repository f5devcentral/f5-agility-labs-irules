Introduction to iRules LX
=========================

In this class we will learn how to use iRules LX with a basic example. We
will have a web app that has a web form. When we submit the form, the
page will display our POST data. As part of the lab exercise, we will
apply an LX iRule that will convert the form POST data into JSON and
change the Content-Type header.

**Using Your Lab Environment**

You will be using Ravello for this lab. We will be working with a
Windows 7 desktop, a BIG-IP 13.0 LTM VE and a server with HTTP and SQL
services enabled. We will be using the Windows machine as our desktop
for accessing the applications on the BIG-IP.

This diagram shows the topology of the network as it is currently
configured -

|image0|

**HTTP and SSH Access to the BIG-IP**

You can SSH directly to your SSH Ravello environment with the SSH client
provided on your workstation.

.. toctree::
   :maxdepth: 1
   :glob:

   module*/module*

.. |image0| image:: /_static/class2/image1.png
   :width: 7.49583in
   :height: 3.82917in