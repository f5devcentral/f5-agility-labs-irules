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

**How to Access the Labs**

You will receive instructions from your proctor on how to access the workstation in the lab.
On this workstation, you will have the following applications –

- Atom Editor – For viewing lab code with syntax highlighting. When you open up Atom, you will
  see a list of files that will be used in these labs.
- Chrome Web Browser – For testing the applications we create and BIG-IP management access.
  Links are bookmarked just below the address bar.
- Putty SSH Client – For accessing the BASH and TMSH command line of the BIG-IP. The BIG-IP
  properties have been saved to the session labeled *BIG-IP*.



.. toctree::
   :maxdepth: 1
   :glob:

   module*/module*

.. |image0| image:: /_static/class3/image1.png
   :width: 7.49583in
   :height: 3.82917in