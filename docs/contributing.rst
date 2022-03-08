Contributing
============

Thanks for your interest in contributing to STADLE! 
Read on to learn what would be helpful and how you could contribute to the STADLE community. 

Developers
*****************

When developing stadle, make sure to install the STADLE in develop mode. 
This mode allows the developer to observe the changes made to the code without installing STADLE each time an update is made to the source.

To do debug the application, use the following command.

.. code-block:: python

   python setup.py develop
   
Additionally, to include tests, install as follows.

.. code-block:: python

   pip install -e ."[dev]"

If the command above does not work, please try `pip install -e .[dev]`.

Run the test cases,

.. code-block:: python

   pytest test/

Note: If you are using the STADLE outside the source folder, make sure you ``copy`` the ``setups`` and ``prototypes`` folders to your workspace to test things out.


Using Docker
*****************

After changing the directory to the STADLE source, build Docker compose

.. code-block:: python

   docker-compose build

Then, start Docker compose

.. code-block:: python

   docker-compose up


Also, reach out with your issues or proposals to improve STADLE.


Bug Reports
***********

Please check/submit issues `here`_.

.. _here: https://github.com/tie-set/stadle_dev/issues


Tech Support
************

Please reach out to our technical support team via ``support@tie-set.com``.
