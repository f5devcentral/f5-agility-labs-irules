#####################################################
Lab 2 – Log and Change Headers
#####################################################

Your iRule should log all request headers and all response headers and should remove the response header “Server”.

.. ATTENTION::
  Extra Credit: Change it to look like IIS.

.. IMPORTANT::
  •	Estimated completion time: 15 minutes

#. Open Chrome Browser
#. Enter https://bigip1 into the address bar and hit Enter
#. Login with username: admin password: admin
#. Click Local Traffic -> iRules  -> iRules List
#. Click Create button
#. Enter Name of Header_Log_Strip_iRule
#. Enter Your Code
#. Click Finished
#. Click Local Traffic -> Virtual Servers -> Virtual Server List
#. Click on http_irules_vip
#. Click on the Resources tab
#. Click Manage button for the iRules section

   .. image:: /_static/class1/iRulesManage.png
      :width: 800

#. Click on Header_Log_Strip_iRule from the Available box and click the << button, thus moving it to the Enabled box, your first and now second iRule should be in the Enabled box.
#. Click the Finished button
#. Open the Firefox browser
#. Click the 3 horizontal line button on the far right of the address bar
#. Install HTTPFox and toggle on, or use developer tools in Chrome to view headers
#. Click the Start button on the HTTPFox window (if using HTTPFox)
#. Enter http://dvwa.f5lab.com/  and ensure you get there
#. Now enter http://wackopicko.f5lab.com/
#. Finally, enter http://peruggia.f5lab.com/ and ensure you can get to that app
#. Look at the headers for each of your requests. Did you log them all? What is the value of the Server header?


.. HINT::

  Basic Hint
  `if you need a hint here is some example code: <../../class1/module1/irules/lab2irule_0.html>`__

  Link to DevCentral: https://devcentral.f5.com/wiki/iRules.HTTP__header.ashx

  If you are really stuck, here is what we are looking for:

  #. `When HTTP_Request comes in <../../class1/module1/irules/lab2irule_1.html>`__
  #. `Log the headers from the HTTP_REQUEST <../../class1/module1/irules/lab2irule_2.html>`__
  #. `When HTTP_RESPONSE comes back <../../class1/module1/irules/lab2irule_3.html>`__
  #. `Log the response headers <../../class1/module1/irules/lab2irule_4.html>`__
  #. `Now remove the HTTP::header named Server <../../class1/module1/irules/lab2irule_5.html>`__
  #. `Now you should have enough to understand and the majority of code to create the iRule.  If not here is the complete iRule. <../../class1/module1/irules/lab2irule_99.html>`__
