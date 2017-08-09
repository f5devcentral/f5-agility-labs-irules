NPM and Exception Handling
--------------------------

Test and Review the Existing Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this lab we will be working from the file ilxlab2\_steps.js. You will
be **cutting and pasting** code from this file as directed. We will be
working with the virtual server (10.0.0.21) & workspace named *ilxlab2*
which already has some base code in it. The plugin has already been created
and the TCL iRule are already assigned to the virtual server.

To start off we have a web application with a web form that we enter
some information into and submit. The response of the POST will show our
form data and “Content-Type” header. This web app is configured on our
BIG-IP at the URL http://10.0.0.21/ilxlab2/. Now lets look at the web
app in the browser. Here is the example of the web form:

|image6|

While we may never have a real use case with JSON in a web form, doing
this allows us to use a web browser for the lab rather than having to
use command line tools.

Without modifying any text in the form, pressing the submit button
should result in this:

|image7|

Push the back button and modify the JSON text to so that it will be
invalid:

|image8|

Then press submit and you should see this:

|image9|

This error came from our webserver. You can tell this because the
webpage has the logo, header and horizontal rules. Later in this lab we
will see errors from the BIG-IP that will be text only.

Now, go back to the web form and click refresh to clear the incorrect
JSON.

iRules LX Code Update Behavior
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Because the way iRules LX transitions new generations of an LX Plugin,
when we make changes to the code we will not see these changes in our
original browser window because it is using the same TCP connection and
is being serviced by the previous version of the LX plugin. To force the
BIG-IP to give us the new generation of the LX plugin, we will be
running the following TMSH command after every workspace change:

.. admonition:: TMSH

   restart ilx plugin <plugin_name> immediate

Error Logs
~~~~~~~~~~

In lab exercises from here on out we will need to view the output of
STDOUT/STDERR of our Node.js processes. By default, it will be directed
to the log files /var/log/ltm. Starting in version 13.0, we introduced a
new feature to iRules LX allowing for STDOUT/STDERR to be sent to a
dedicated log file for each Node.js process for an extension within an
LX Plugin.

The extension ``ilxlab2_ext`` of plugin ``ilxlab2_pl`` is already
configured with the following settings so we can make the logs of the
lab easier to find -

+---------------------+-------------+----------------------------------------------------+
| Setting             | New Value   | Reason                                             |
+=====================+=============+====================================================+
| Concurrency Mode    | Single      | Keep logs for all connections in a single file.    |
+---------------------+-------------+----------------------------------------------------+
| iRules LX Logging   | Checked     | Will make extension send logs to dedicated file.   |
+---------------------+-------------+----------------------------------------------------+

To see these settings for yourself, click on the *ilxlab2\_pl* LX plugin
and then click on the *ilxlab2\_ext* extension. The dedicated log files
can be found in the directory */var/log/ilx/* and will be named in the
format *<partition_name>.<plugin_name>.<ext_name><tmm_id_if_dedicated_mode>*.
Here are some examples of file names -

.. code-block:: console

   Common.ilxlab2_pl.ilxlab2_ext
   Common.ilxlab99_pl.some_ext0
   Common.ilxlab99_pl.some_ext1

Exception Handling
~~~~~~~~~~~~~~~~~~

Good software development incorporates exception handling into the code.
Without it, our programs would simply crash when there is an uncaught
exception. On iRules TCL, the TCL interpreter crashes for an uncaught
exception, but the worst consequence is that a single client connection
is reset.

Because Node.js in iRules LX is external from TMM, a crash is much more
serious. Any connection being serviced by that Node.js process will get
reset and all state for any outstanding RPC calls will be lost. A crash
triggered from a single function call has the potential to reset
hundreds or even thousands of connections on the BIG-IP. Also, any new
connections that are trying to establish while Node.js is rebooting
could also be reset.

Therefore, it is imperative that we learn proper exception handling.

Handle Errors in JavaScript
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Right now the LX workspace code does not have any function call that can
throw an exception, but we would like to add more functionality to it.
Here is the addMethod function that we have in the Node.js code:

.. code-block:: javascript
   :linenos:

   ilx.addMethod('jsonParse', function (req, res) {
     // Extract JSON from POST data
     var postData = qs.parse(req.params()[0]).JSON;

     // Send data back to TCL
     res.reply(postData);
   });


All we are doing is extracting the form input box labeled “JSON”. But we
would like to insert more data into the JSON that we send to the
application. In order to do that, we must first parse the JSON to a JS
object, then stringify it again. Go to the *code\_instructions* and
complete **code step 1** (remember to copy and paste). The ILX addMethod code should look like this
after you are done (changes are highlighted) -

**Code Step 1**

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 4, 6

   ilx.addMethod('jsonParse', function (req, res) {
     // Extract JSON from POST data
     var postData = qs.parse(req.params()[0]).JSON;
     var jsonData = JSON.parse(postData);

     res.reply(JSON.stringify(jsonData));
   });


Save and reload the workpsace. Now submit some invalid JSON in the form
like we did earlier. You will see an text only error like this:

|image10|

This error is coming from the iRules TCL code in our “catch” of the ILX
call. If we look at the logs we will see the following:

..code-block:: console

  # tail -1 /var/log/ltm
  Jul 11 16:02:15 bigip1 err tmm1[14567]: Rule /Common/ilxlab2_pl/json_parse <HTTP_REQUEST_DATA>: Client - 10.0.0.  10, ILX failure: ILX timeout.     invoked from within "ILX::call $handle jsonParse [HTTP::payload]" ``

  # tail -1 /var/log/ltm
  Jul 11 16:02:15 bigip1 err tmm1[14567]: Rule /Common/ilxlab2_pl/json_parse <HTTP_REQUEST_DATA>: Client - 10.0.0.  10, ILX failure: ILX timeout.     invoked from within "ILX::call $handle jsonParse [HTTP::payload]"

The log file for the extension should have some entries similar to this:

.. code-block:: console

   # tail -20 /var/log/ilx/Common.ilxlab2_pl.ilxlab2_ext
   Jul 11 16:02:12 pid[15201] undefined:5
   Jul 11 16:02:12 pid[15201] randomtext
   Jul 11 16:02:12 pid[15201] ^
   Jul 11 16:02:12 pid[15201] SyntaxError: Unexpected token w
   Jul 11 16:02:12 pid[15201]     at Object.parse (native)
   Jul 11 16:02:12 pid[15201]     at Object.jsonParse (/var/sdm/plugin_store/plugins/:Common:   ilxlab2_pl_62102_2/extensions/ilxlab2_ext/index.js:13:23)
   Jul 11 16:02:12 pid[15201]     at ILXClient.<anonymous> (/var/sdm/plugin_store/plugins/:Common:   ilxlab2_pl_62102_2/extensions/ilxlab2_ext/node_modules/f5-nodejs/lib/ilx_server.js:100:46)
   <--------------Rest of output truncated -------------->

As you can see, our bad JSON threw an exception that crashed the Node.js
process which caused an ILX timeout in TCL. This is the stack track for our exception.

To prevent Node.js from crashing we need to put JSON.parse in a try/catch block. Perform
code step 2 on the workspace to do this. The Node function should end up like this –

**Code Step 2**

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 4-9

   ilx.addMethod('jsonParse', function (req, res) {
     // Extract JSON from POST data
     var postData = qs.parse(req.params()[0]).JSON;
     try {
       var jsonData = JSON.parse(postData);
     } catch (err) {
       console.log('Error with JSON.parse: ' + err.message);
       return; // Stop processing this function
     }

     res.reply(JSON.stringify(jsonData));
   });

Save and reload the workspace. Now if you try bad JSON again, you will still
get the same error on the web browser, but we will not crash the Node.js
process. Doing a tail of the log files again, you will see an error message
similar to this:

``Jul 11 16:14:55 pid[15456] Error with JSON.parse: Unexpected token w``

**Note**: Try/catch is only for synchronous functions. Most asynchronous
functions handle exceptions/errors in the callback function or with
event handlers and vary greatly from one module to the next. You will
have to consult the documentation for the module you wish to use.

RPC Status Return Value
^^^^^^^^^^^^^^^^^^^^^^^

While try/catch did help to prevent the Node process from crashing, the
error the client received does not help them very much. It would be
better if we could give some more info to the client via iRules TCL, but
TCL does not know about the issue that happen with Node.js. Therefore,
we should return some type of status to TCL if it the RPC to Node fails.

One way we can accomplish this is by the return of multiple values from
Node.js. Our first value could be some type of RPC status value (say an
RPC error value) and the rest of the value(s) could be our result from
the RPC. It is quite common in programming to make an error value would
be 0 if everything was okay but would be an integer to indicate a
specific error code.

For this next step, we will make changes to both Node and TCL to create
the error communication between Node and TMM. Perform code step 3a and 3b
on the workspace. This is what the Node method and the TCL
*HTTP\_REQUEST\_DATA* event should look like after you make the changes:

**Code Step 3 Node.js**

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 8, 11

   ilx.addMethod('jsonParse', function (req, res) {
     // Extract JSON from POST data
     var postData = qs.parse(req.params()[0]).JSON;
     try {
       var jsonData = JSON.parse(postData);
     } catch (err) {
       console.log('Error with JSON.parse: ' + err.message);
       return res.reply(1);
     }

     res.reply([0, JSON.stringify(jsonData)]);
   });

As you can see in the res.resply function, we can return multiple values
back to TCL if we put an array as the argument. TCL will then see these
values returned as a TCL list.

**Code Step 3 TCL**

.. code-block:: tcl
   :linenos:
   :emphasize-lines: 10-21

   when HTTP_REQUEST_DATA {
       # Send data to Node.js
       set handle [ILX::init " ilxlab2_pl" "ilxlab2_ext"]
       if {[catch {ILX::call $handle jsonParse [HTTP::payload]} result]} {
         log local0.error  "Client - [IP::client_addr], ILX failure: $result"
         HTTP::respond 400 content "<html>There has been an error.</html>"
         return
       }

       if {[lindex $result 0] > 0} {
         # What is our error code?
         switch [lindex $result 0] {
           1 { set error_msg "Invalid JSON"}
         }
         HTTP::respond 400 content "<html>The following error occured: $error_msg</html>"
       } else {
         #Replace Content-Type header and POST payload
         HTTP::header replace "Content-Type" "application/json"
         HTTP::payload replace 0 $cl [lindex $result 1]
       }
   }

Here we are checking the value of index 0 of the TCL list to see if it is
greater than zero. Based upon what that value is we can tailor our return
message back to the client. What we have done is allowed Node.js to
communicate specific errors that we define back to the client. You would
never want to send back all errors because stack traces could reveal
sensitive data about your iRule.

Save and reload the workspace. Now when you submit invalid JSON in the
browser you should see an error like this –

|image11|

Now that we have the exception handling taken care of, lets add some
more functionality to this iRule. We mentioned a little while ago we
would like to add some more data to the JSON that gets sent to the
server.

Let’s say we wanted to insert random data to act as some type of nonce.
In code step 4 let’s use the crypto module to insert the random text.
This code snippet will show what all the node.js code should look like
after this step:

**Code Step 4**

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 5, 21

   'use strict'; // Just for best practices
   // Import modules here
   var f5 = require('f5-nodejs');
   var qs = require('querystring');
   var crypto = require('crypto');

   // Create an ILX server instance
   var ilx = new f5.ILXServer();

   // This method will transform POST data into JSON
   ilx.addMethod('jsonParse', function (req, res) {
     // Extract JSON from POST data
     var postData = qs.parse(req.params()[0]).JSON;
     try {
       var jsonData = JSON.parse(postData);
     } catch (err) {
       console.log('Error with JSON.parse: ' + err.message);
       return res.reply(1);
     }

     jsonData.token = crypto.randomBytes(8).toString('hex');
     res.reply([0, JSON.stringify(jsonData)]);
   });

   ilx.listen();

Save and reload the workspace.

**Note**: This is not really a proper use of a cryptographic nonce, it
is just to show how we can extend functionality with Node.js.

Now this time, send valid JSON text via the web form and we should see a
result like this:

|image12|

You can see our token has been added to the JSON.

This concludes the exception handling exercise.

Installing Packages with NPM
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can install modules from NPM when you want to get extra
functionality that is not provided with the built in Node.js modules.
NPM and the active community around it is one of the primary reasons
that Node.js was chosen for iRules LX.

We have a use case requiring us to do syntax validation of an email
address that is in the JSON text from a web form. We won’t be checking
if the email address itself is a working address, just that the syntax
is in the correct form. We will download a package from NPM to handle
the this.

Installing the Validator Module from NPM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The first thing we must do is install a NPM module for validating email
addresses. We will accomplish this with the *validator* module. To
install the module into the workspace, we need to access the BASH prompt
of our BIG-IP, then ``cd`` into the workspace directory and run the
commands:

.. code-block:: console

   [root@localhost] # cd /var/ilx/workspaces/Common /ilxlab2/extensions/ilxlab2_ext/
   [root@localhost] # npm install validator --save
   validator@6.1.0 node_modules/validator
   [root@localhost] # ls node_modules/
   f5-nodejs  validator


The ``--save`` option saves the module to the package.json file
dependencies as shown here in the workspace:

|image13|

Using the Validator Module
^^^^^^^^^^^^^^^^^^^^^^^^^^

To use this module, we must import it into out Node.js code and
then call it. In code step 5, we will “require” the module in
Node.js, then put some code that will validate if our email address
has the proper format. We will also need to add some extra code to TCL
to hand 2 more error conditions that email validation brings. The first
check ensures that the email value is in our JSON,  the second uses the
validator module to validate the syntax of the email address. Here is
what the code will look like once you are finished:

**Code Step 5 Node.js**

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 6, 22, 23

   'use strict' // Just for best practices
   // Import modules here
   var f5 = require('f5-nodejs');
   var qs = require('querystring');
   var crypto = require('crypto');
   var validator = require('validator');

   // Create an ILX server instance
   var ilx = new f5.ILXServer();

   // This method will transform POST data into JSON
   ilx.addMethod('jsonParse', function (req, res) {
     // Extract JSON from POST data
     var postData = qs.parse(req.params()[0]).JSON;
     try {
       var jsonData = JSON.parse(postData);
     } catch (err) {
       console.log('Error with JSON.parse: ' + err.message);
       return res.reply(1);
     }

     if (! ('email' in jsonData)) return res.reply(2); //
     if (! validator.isEmail(jsonData.email)) return res.reply(3);
     postData.token = crypto.randomBytes(8).toString('hex')
     res.reply([0, JSON.stringify(jsonData)]);
   });

   ilx.listen();

You will notice that we check first for the existence of the email property
in the JSON and then check if the string in the JSON is valid. If you
attempted to only do the email validation but the email property was not
present, this would throw an exception for a missing property in the JS
object and crash Node.


**Code Step 5 TCL**

.. code-block:: tcl
   :linenos:
   :emphasize-lines: 14, 15

   when HTTP_REQUEST_DATA {
       # Send data to Node.js
       set handle [ILX::init "json_parser_pl" "parser_ext"]
       if {[catch {ILX::call $handle jsonParse [HTTP::payload]} result]} {
         log local0.error  "Client - [IP::client_addr], ILX failure: $result"
         HTTP::respond 400 content "<html>There has been an error.</html>"
         return
       }

       if {[lindex $result 0] > 0} {
         # What is our error code?
         switch [lindex $result 0] {
           1 { set error_msg "Invalid JSON"}
           2 { set error_msg "Property \"email\" missing from JSON."}
           3 { set error_msg "Property \"email\" not a valid email address."}
         }
         HTTP::respond 400 content "<html>The following error occured: $error_msg</html>"
       } else {
         #Replace Content-Type header and POST payload
         HTTP::header replace "Content-Type" "application/json"
         HTTP::payload replace 0 $cl [lindex $result 1]
       }
   }

Both the email property presence check and invalid email error get an
error code that we pass over to TCL to give the client a useable error
message. Now we can test these error conditions.

Save and reload the workspace. Go to your browser and remove the email
property and trailing comma from the password property like so:

|image14|

When you press submit, you should see an error like this:

|image15|

Now go back to the form and refresh the web form back
to normal. Now remove the “@” symbol the email address:

|image16|

Then submit the form and you should see the following:

|image17|

.. |image6| image:: /_static/class3/image7.png
   :width: 3.35047in
   :height: 2.74171in
.. |image7| image:: /_static/class3/image8.png
   :width: 3.61001in
   :height: 2.08705in
.. |image8| image:: /_static/class3/image9.png
   :width: 3.15999in
   :height: 2.42508in
.. |image9| image:: /_static/class3/image10.png
   :width: 4.44534in
   :height: 1.55393in
.. |image10| image:: /_static/class3/image11.png
   :width: 5.28966in
   :height: 0.88318in
.. |image11| image:: /_static/class3/image12.png
   :width: 3.67347in
   :height: 0.59130in
.. |image12| image:: /_static/class3/image13.png
   :width: 3.42615in
   :height: 2.18037in
.. |image13| image:: /_static/class3/image14.png
   :width: 5.63090in
   :height: 1.78672in
.. |image14| image:: /_static/class3/image15.png
   :width: 2.58703in
   :height: 2.41944in
.. |image15| image:: /_static/class3/image16.png
   :width: 5.17619in
   :height: 0.60586in
.. |image16| image:: /_static/class3/image17.png
   :width: 2.75043in
   :height: 2.37327in
.. |image17| image:: /_static/class3/image18.png
   :width: 5.45094in
   :height: 0.40864in
