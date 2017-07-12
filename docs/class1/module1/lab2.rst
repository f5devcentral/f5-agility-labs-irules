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

   .. figure:: ./images/iRulesManage.png
      :width: 1200

   |

#. Click on Header_Log_Strip_iRule from the Available box and click the << button, thus moving it to the Enabled box, your first and now second iRule should be in the Enabled box.
#. Click the Finished button
#. Open the Firefox browser
#. Click the 3 horizontal line button on the far right of the address bar
#. Click on HTTPFox and toggle on
#. Click the Start button on the HTTPFox window
#. Enter http://dvwa.f5lab.com/  and ensure you get there
#. Now enter http://wackopicko.f5lab.com/
#. Finally, enter http://peruggia.f5lab.com/ and ensure you can get to that app
#. Look at the headers for each of your requests. Did you log them all? What is the value of the Server header?

.. HINT::
   `if you need a hint here is some example code: <../../_sources/class1/module1/irules/lab2irule.rst.txt>`__
   
   Link to DevCentral: https://devcentral.f5.com/wiki/iRules.HTTP__header.ashx
