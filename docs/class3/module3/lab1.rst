Asynchronous Programming
------------------------

Test and Review the Existing Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this lab we will be working with the virtual server (10.0.0.22) &
workspace named *ilxlab3*. The plugin and TCL iRule are already assigned
to the virtual server. To start off we have a web application that
displays a list of users in a database. This web app is configured on
our BIG-IP at the URL http://10.0.0.22/.

SQL Database Lookup
~~~~~~~~~~~~~~~~~~~

In this lab we are simply going to view some log statements into the
Node.js and look at the order they appear in the log file. First we will
review the sql query method in our extension code highlighted below:

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 4-18

   // Add a method
   ilx.addMethod('get_users', function(req, res) {
     // Perform the query from pool
     sqlPool.query(
       'SELECT id, name, grp FROM users_db.users ORDER BY id;',
       function(err, rows) {
         if (err) {
           // MySQL query failed for some reason, send a 2 back to TCK
           console.error('Error with query: ', err.message);
           return res.reply(2);
         }

         // Check array length from sql
         if (rows.length)
           res.reply([0, rows]);
         else
           res.reply(1); // if 0 return 1 to the Tcl iRule to show no matching records
       }
     );
   });

You will notice that the function has 2 arguments, the first being the
text of the actual query. Because this method is asynchronous, the
second argument is the callback function that will get executed when the
query answer is received by Node.js.

To demonstrate asynchronous behavior, we will put logging statements
before and after the query method as such:

**Code Step 1**

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 4, 13, 22

   // Add a method 
   ilx.addMethod('get_users', function(req, res) {
     // Perform the query from pool
     console.log('Starting SQL query');
     sqlPool.query(
       'SELECT id, name, grp FROM users_db.users ORDER BY id;',
       function(err, rows) {
         if (err) {
           // MySQL query failed for some reason, send a 2 back to TCK
           console.error('Error with query: ', err.message);
           return res.reply(2);
         }
         console.log('There are', rows.length,'records in the DB.');
   
         // Check array length from sql
         if (rows.length)
           res.reply([0, rows]);
         else
           res.reply(1); // if 0 return 1 to the Tcl iRule to show no matching records
       }
     );
     console.log('SQL query finished.');
   });
   
Make sure to use the TMSH plugin restart command after you reload the
workspace. Now tail the log contents of the log file with the following
BASH command and then refresh the ilxlab3 web page:

``# tail -f /var/log/ilx/Common.ilxlab3_pl.mysql``

What do you notice about the order of the log statements?

Now let’s make the following changes to the node.js as seen below.

**Code Step 2**

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 20, 23

   // Add a method
   ilx.addMethod('get_users', function(req, res) {
     // Perform the query from pool
     console.log('Starting SQL query');
     sqlPool.query(
       'SELECT id, name, grp FROM users_db.users ORDER BY id;',
       function(err, rows) {
         if (err) {
           // MySQL query failed for some reason, send a 2 back to TCK
           console.error('Error with query: ', err.message);
           return res.reply(2);
         }
         console.log('There are', rows.length,'records in the DB.');

         // Check array length from sql
         if (rows.length)
           res.reply([0, rows]);
         else
           res.reply(1); // if 0 return 1 to the Tcl iRule to show no matching records
         console.log('SQL query is really finished.');
       }
     );
     console.log('Function call is finished.');
   });

Use the TMSH plugin restart command after you reload the workspace. Now
tail the log contents of the log file again and then refresh the ilxlab3
web page. You will see that they are in the right order. The callback
function is executed much later because I/O responses take much longer.

But you might ask, how much later is the callback function executing? To
answer that question, lets add some more code:

**Code Step 3**

.. code-block:: javascript
   :linenos:
   :emphasize-lines: 5, 21

   // Add a method
   ilx.addMethod('get_users', function(req, res) {
     // Perform the query from pool
     console.log('Starting SQL query');
     var start = Date.now();
     sqlPool.query(
       'SELECT id, name, grp FROM users_db.users ORDER BY id;',
       function(err, rows) {
         if (err) {
           // MySQL query failed for some reason, send a 2 back to TCK
           console.error('Error with query: ', err.message);
           return res.reply(2);
         }
         console.log('There are', rows.length,'records in the DB.');

         // Check array length from sql
         if (rows.length)
           res.reply([0, rows]);
         else
           res.reply(1); // if 0 return 1 to the Tcl iRule to show no matching records
         console.log('SQL query is really finished, time:', Date.now() - start, 'msec');
       }
     );

     console.log('Function call is finished.');
   });

Use the TMSH plugin restart command after you reload the workspace. Now
tail the log contents of the log file again and then refresh the ilxlab3
web page. Most likely, you are seeing that the time logged is in the
order of tens of milliseconds. As you saw from the I/O time table in the
presentation, this is an eternity compared to reads from local memory.

