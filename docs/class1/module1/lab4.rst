#####################################################
Lab 4 – Stream Profile
#####################################################

Create a Stream Profile to change the body of the DVWA site

.. IMPORTANT::
  •	Estimated completion time: 10 minutes

#. Open Chrome Browser
#. Enter https://bigip1 into the address bar and hit Enter
#. Login with username: admin password: admin
#. Click Local Traffic -> iRules  -> iRules List
#. Click Create button
#. Enter Name of Stream_iRule
#. Enter Your Code
#. Click Finished
#. Click Local Traffic -> Virtual Servers -> Virtual Server List
#. Click on https_irules_vip
#. In the Configuration section ensure it says Advanced in the drop down
#. Go to the Stream Profile section and select stream
#. Scroll to the bottom and click the Update button
#. Click on the Resources tab
#. Click Manage button for the iRules section

   .. image:: /_static/class1/iRulesManage.png
      :width: 800

#. What should you do here?
#. Click the Finished button
#. Open the Firefox browser
#. Enter https://dvwa.f5lab.com  and ensure you get there and it is HTTPS and that the word “Damn” is replaced with “Darn”

.. HINT::

  Basic Hint
  `if you need a hint here is some example code: <../../class1/module1/irules/lab4irule_0.html>`__

  Link to DevCentral: https://devcentral.f5.com/wiki/iRules.STREAM.ashx

  If you are really stuck, here is what we are looking for:

  #. `When HTTP_Request comes in <../../class1/module1/irules/lab4irule_1.html>`__
  #. `Second we need to disable both encoding the stream profile for the request <../../class1/module1/irules/lab4irule_2.html>`__
  #. `When HTTP_RESPONSE comes back <../../class1/module1/irules/lab4irule_3.html>`__
  #. `Next we need to change our stream matching string and turn on the stream profile again. <../../class1/module1/irules/lab4irule_4.html>`__
  #. `Now you should have enough to understand and the majority of code to create the iRule.  If not here is the complete iRule. <../../class1/module1/irules/lab4irule_99.html>`__
