Module 1: Creating and Implementing an LX iRule
================================================

In this lab we will learn how to use iRules LX with a basic example to
introduce you to the concepts of iRules LX and their configuration
objects. We will have a web app that has a web form. When we submit the
form, the page will display our POST data. As part of the lab exercise,
we will apply an LX iRule that will convert the form POST data into JSON
and change the Content-Type header.

The practicality of this use case could be when you have some type of
legacy front end service that can only POST data as a standard query
string, but a new back end service takes data as JSON. It would be
pretty impractical to use iRules TCL to perform the translation.
However, this task is trivial for iRules LX because of the power of
Node.js. We will implement an LX iRule that will accomplish this.

.. toctree::
   :maxdepth: 1
   :glob:

   lab*
