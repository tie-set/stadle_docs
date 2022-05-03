Installation
===============

We support a couple of installation options such as using the source code and downloading from a PyPI server.
For now, the STADLE software and codes are maintained as a private content.

Installing from PyPI Server
*******************************

STADLE Client-Side Installation
------------------------------------

The users basically only need to install the STADLE client-side code as TieSet usually hosts the STADLE server-side components to which the client is connected to.
The STADLE client is available on the PyPI server and can be installed using the following command:

.. code-block:: python

  pip install stadle-client

If the command above is not working with your environment, please try the following command:

.. code-block:: python

  pip install --index-url http://3.110.171.230:8080 stadle_client --trusted-host 3.110.171.230 --extra-index-url https://pypi.org/simple

When you want to update the stadle client side package, just simply `pip uninstall stadle_client` and install it again.

.. NOTE:: We have already tested this on Linux x86_64 and Windows x86_64. Other versions will be created or fixed soon.


STADLE Server-Side Installation
------------------------------------

In case you need to install the STADLE Persistence Server and Aggregator, the packaged code is available on the PyPI server and can be installed using the following command:

.. code-block:: python

   pip install --index-url http://<PUBLIC-IP-ADDRESS>:8080 stadle_server --trusted-host <PUBLIC-IP-ADDRESS> --no-cache-dir https://pypi.org/simple

When you want to update the stadle client side package, just simply `pip uninstall stadle_server` and install it again.


Installing from Source
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
