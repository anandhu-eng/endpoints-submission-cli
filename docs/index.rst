Submission Checker
==================

CLI tool for validating MLPerf benchmark submissions.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   api

Quickstart
----------

Install::

   pip install endpoints-submission-cli

Check a submission directory::

   from submission_checker import SubmissionChecker

   report = SubmissionChecker("/path/to/submission").run()

Indices and tables
------------------

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
