===============
Getting Started
===============

Lab Components
==============

The following table lists the virtual appliances in the lab along with their networks and credentials to use.

.. list-table::
    :widths: 20 40 40
    :header-rows: 1
    :stub-columns: 1

    * - **System Type**
      - **Networks**
      - **Credentials**

    * - NGINX+
      - Management: 10.1.1.4
        Internal: 10.1.10.11
      - SSH keys
    * - MyApplication Server
      - Management: 10.1.1.8
        Internal: 10.1.10.21
      - None
    * - NGINX Controller
      - Management: 10.1.1.7
      - SSH      


Starting the Lab
================

*Insert instructions here to access UDF*

Using NGINX with Docker
=======================

.. code-block:: shell

  docker run --rm -it nginx njs

.. code-block:: none

  interactive njs 0.3.9

  v.<Tab> -> the properties and prototype methods of v.
  type console.help() for more information

  >> 2+2
  4
  >> function hi(msg) {console.log(msg)}
  undefined
  >> hi("Hello world")
  'Hello world'
  undefined
  >> globalThis
  global {
   njs: njs {
    version: '0.3.9'
   }, ...

Downloading Lab Files
=====================

.. code-block:: shell

  git clone https://github.com/xeioex/njs-examples
  cd njs-examples
  ls *

  README.rst

  conf:
  complex_redirects.conf		file_io.conf			hello.conf			jwt.conf			stream
  decode_uri.conf			gen_hs_jwt.conf			join_subrequests.conf		secure_link_hash.conf		subrequests_chaining.conf

  njs:
  complex_redirects.js	file_io.js		hello.js		jwt.js			stream
  decode_uri.js		gen_hs_jwt.js		join_subrequests.js	secure_link_hash.js	subrequests_chaining.js

