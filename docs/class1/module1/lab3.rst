#####################################################
Lab 3 – HTTP to HTTPS Redirect
#####################################################

Create an HTTP to HTTPS redirect. Additionally, when traffic goes to the HTTPS side the app selection should still work as well as the header stripping.

.. IMPORTANT::
  •	Estimated completion time: 20 minutes


#. Open Chrome Browser
#. Enter https://bigip1 into the address bar and hit Enter
#. Login with username: admin password: admin
#. Click Local Traffic -> iRules  -> iRules List
#. Click Create button
#. Enter Name of HTTP_to_HTTPS_iRule
#. Enter Your Code
#. Click Finished
#. Click Local Traffic -> Virtual Servers -> Virtual Server List
#. Click on http_irules_vip
#. Click on the Resources tab
#. Click Manage button for the iRules section

   .. image:: /_static/class1/iRulesManage.png
      :width: 800

#. What should you do here now that we are doing HTTPS?
#. Click the Finished button
#. Open the Firefox browser
#. Click the 3 horizontal line button on the far right of the address bar
#. Click on HTTPFox (and add it as an plugin), and toggle on - alternatively use Chrome developer tools.
#. Click the Start button on the HTTPFox window (if using HTTPFox)
#. Enter http://dvwa.f5lab.com/  and ensure you get there and it is HTTPS
#. Now enter http://wackopicko.f5lab.com/ and ensure you get there via HTTPS
#. Finally, enter http://peruggia.f5lab.com/ and ensure you can get to that app via HTTPS
#. Look at the headers for each of your requests. Did you log them all? What is the value of the Server header? None of this should have changed since the last lab.

.. HINT::

  Basic Hint
  `if you need a hint here is some example code: <../../class1/module1/irules/lab3irule_0.html>`__

  Link to DevCentral: https://devcentral.f5.com/wiki/iRules.HTTP__redirect.ashx

  If you are really stuck, here is what we are looking for:

  #. `When HTTP_Request comes in <../../class1/module1/irules/lab3irule_1.html>`__
  #. `Redirect from HTTP to HTTPS <../../class1/module1/irules/lab3irule_2.html>`__
  #. `Now you should have enough to understand and the majority of code to create the iRule.  If not here is the complete iRule. <../../class1/module1/irules/lab3irule_99.html>`__
