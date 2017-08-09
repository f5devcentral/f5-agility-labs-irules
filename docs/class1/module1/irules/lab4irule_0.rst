:orphan:

#####################################################
Lab 4 - Stream Profile
#####################################################


Example Code to help you on your journey (Not fully functional):
------------------------------------------------------------------------------------

Stream Profile iRule – Not Fully Functional

.. code::

  when HTTP_RESPONSE {
    STREAM::disable
    STREAM::expression @Damn@Darn@
    STREAM::enable
  }
