Installation
============

We only support installing from the source as we keep the software a private content.

Linux/Ubuntu Users
******************

Please follow the installation steps for ``server``, ``agent`` and ``database``.

**Install Pre-requisites**

**Create Virtual Environment**

.. code-block:: python

   python -m venv ENVSTADLE
   source ENVSTADLE/bin/activate

**Build Package**

First upgrade pip,

.. code-block:: python

   pip install --upgrade pip

Then build the wheel,

.. code-block:: python

   python setup.py bdist_wheel

**Install Wheel**

.. code-block:: python

   python -m pip install dist/stadle-*-py3-none-any.whl \
               --extra-index-url https://test.pypi.org/simple \
               --no-cache-dir

MacOS Users
***********

Please follow the installation steps for ``server``, ``agent`` and ``database``.

**Create Virtual Environment**

.. code-block:: python

   python -m venv ENVSTADLE
   source ENVSTADLE/bin/activate

**Build Package**

First upgrade pip,

.. code-block:: python

   pip install --upgrade pip

Then build the wheel,

.. code-block:: python

   python setup.py bdist_wheel

**Install Wheel**

.. code-block:: python

   python -m pip install dist/stadle-*-py3-none-any.whl \
               --extra-index-url https://test.pypi.org/simple \
               --no-cache-dir

Developers
********************

When developing stadle, make sure to install the STADLE in develop mode. This mode allows the developer to observe the changes made to the code without installing STADLE each time an update is made to the source.

To do debug the application, use the following command.

.. code-block:: python

   python setup.py develop
   
Additionally, to include tests, install as follows.

.. code-block:: python

   pip install -e .[dev]

Run the test cases,

.. code-block:: python

   pytest test/

Note: If you are using the STADLE outside the source folder, make sure you ``copy`` the ``setups`` and ``prototypes`` folders to your workspace to test things out.
