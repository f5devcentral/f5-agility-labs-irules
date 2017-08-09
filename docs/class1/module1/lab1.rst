#########################################################
Lab 1 - Create an iRule that Parses URI to Route Traffic
#########################################################


Creating your first HTTP iRule that traffic based upon the value of the Host header.
------------------------------------------------------------------------------------
- dvwa.f5lab.com – dvwa_pool_http
- peruggia.f5lab.com – peruggia_http_pool
- wackopicko.f5lab.com – wackopicko_http_pool

.. IMPORTANT::
   - Estimated completion time: 10 minutes

#. Open Chrome Browser
#. Enter https://bigip1 into the address bar and hit Enter
#. Login with username: admin password: admin
#. Click Local Traffic -> iRules  -> iRules List
#. Click Create button
#. Enter Name of URI_Routing_iRule
#. Enter your code
#. Click Finished
#. Click Local Traffic -> Virtual Servers -> Virtual Server List
#. Click on http_irules_vip
#. Click on the Resources tab
#. Click Manage button for the iRules section

   .. image:: /_static/class1/iRulesManage.png
      :width: 800

#.  Click on URI_Routing_iRule from the Available box and click the << button, thus moving it to the Enabled box.
#.  Click the Finished button
#.  Open a new tab in Chrome
#.  Enter http://dvwa.f5lab.com/ and ensure you get there
#.  Now enter http://peruggia.f5lab.com/ and ensure you get to the app
#.  Finally, enter http://wackopicko.f5lab.com/  and ensure you can get to that app


.. HINT::
   `If you need a basic hint here is some example code: <../../class1/module1/irules/lab1irule_0.html>`__

   Here is a link to DevCentral: https://devcentral.f5.com/wiki/iRules.HTTP__host.ashx

   If you are really stuck, here is what we are looking for:

   #. `When HTTP_Request comes in <../../class1/module1/irules/lab1irule_1.html>`__
   #. `Evaluate the HTTP_host name  <../../class1/module1/irules/lab1irule_2.html>`__
   #. `If it matches send it to the correct pool. <../../class1/module1/irules/lab1irule_3.html>`__
   #. `Loop through all the host names you want to match on and continue to direct to the correct pools. <../../class1/module1/irules/lab1irule_4.html>`__
   #. `Now you should have enough to understand and the majority of code to create the iRule.  If not here is the complete iRule. <../../class1/module1/irules/lab1irule_99.html>`__
