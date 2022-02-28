Installation
===============

We support a couple of installation options such as using the source code and downloading from a PyPI server.
For now, we keep the software a private content.

STADLE Client-Side Installation
**********************************

The users basically only need to install the STADLE client-side code as TieSet usually hosts the STADLE server-side components to which the client is connected to.
The STADLE client is available on the PyPI server and can be installed using the following command:

.. code-block:: python

  pip install --index-url http://<PUBLIC-IP-ADDRESS>:8080 stadle_client --trusted-host <PUBLIC-IP-ADDRESS> --extra-index-url https://pypi.org/simple

We have already tested this on Linux x86_64 and Windows x86_64. 
Other versions will be created/fixed soon.
When you want to update the stadle client side package, just simply `pip uninstall stadle_client` and install it again.


STADLE Server-Side Installation
**********************************

The STADLE server is available on the PyPI server and can be installed using the following command:

.. code-block:: python

   pip install --index-url http://<PUBLIC-IP-ADDRESS>:8080 stadle_server --trusted-host <PUBLIC-IP-ADDRESS> --no-cache-dir https://pypi.org/simple

When you want to update the stadle client side package, just simply `pip uninstall stadle_server` and install it again.


Installation from Source
******************************* 


Linux/Ubuntu Users
------------------

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
------------------

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

Using Docker
*****************

After changing the directory to the STADLE source, build Docker compose

.. code-block:: python

   docker-compose build

Three start Docker compose

.. code-block:: python
   docker-compose up


Developers
*****************

When developing stadle, make sure to install the STADLE in develop mode. This mode allows the developer to observe the changes made to the code without installing STADLE each time an update is made to the source.

To do debug the application, use the following command.

.. code-block:: python

   python setup.py develop
   
Additionally, to include tests, install as follows.

.. code-block:: python

   pip install -e .[dev]

If the command above does not work, please try `pip install -e ."[dev]"`.

Run the test cases,

.. code-block:: python

   pytest test/

Note: If you are using the STADLE outside the source folder, make sure you ``copy`` the ``setups`` and ``prototypes`` folders to your workspace to test things out.
