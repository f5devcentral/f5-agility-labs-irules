Application Flow Control with iRules 
====================================

This class covers the following topics: 

- HTTP Protocol Review
- HTTP Request Side Overview
- HTTP Response Side Overview
- HTTP Related Events
- HTTP Headers
- STREAM Command
- HTTP Payload Capture and Manipulation (If time permits)
- SSL::profile (If time permits)

Expected time to complete: **1.25 hours**

.. NOTE::
  All work for this lab will be performed exclusively from the Windows
  jumphost. No installation or interaction with your local system is
  required.


Lab Components
~~~~~~~~~~~~~~

The following table lists the Credentials for all components:

.. list-table::
    :widths: 20 40 40
    :header-rows: 1

    * - **Component**
      - **VLAN/IP Address(es)**
      - **Credentials**
    * - BigIP admin
      - **Management:** ``bigip1``
      - ``admin``/``admin.F5demo.com``
    * - Bigip root
      - **Management:** - ``bigip1``
      - ``root``/``default.F5demo.com``
    * - Jumphost
      - **Jumphost:** ``Windows Jumpbox``
      - ``external_user``/``P@ssw0rd!``



.. toctree::
   :maxdepth: 1
   :glob:

   module*/module*
