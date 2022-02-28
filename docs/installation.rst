Installation
===============

We support a couple of installation options from the source or PyPI server.
For now, we keep the software a private content.

STADLE Client-Side Installation
**********************************

The users basically only need to install the STADLE client-side code and connect to the STALDE server that is hosted by TieSet for you.

The STADLE client is available on the PyPI server and can be installed using the following command:

.. code-block:: python

  pip install --index-url http://xxx.xxx.xxx.xxx:8080 stadle_client --trusted-host xxx.xxx.xxx.xxx --extra-index-url https://pypi.org/simple

We have already tested this on Linux x86_64 and Windows x86_64. Other versions will be created/fixed soon.
When you update the stadle client side package, please `pip uninstall stadle_client` and do install it again.



STADLE Server-Side Installation
**********************************

The STADLE server is available on the PyPI server and can be installed using the following command:

.. code-block:: python
  pip install --index-url http://<PUBLIC-IP-ADDRESS>:8080 stadle_server --trusted-host <PUBLIC-IP-ADDRESS> --no-cache-dir <PackageName>


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

If the command above does not work, please try `pip install -e ."[dev]"`.

Run the test cases,

.. code-block:: python

   pytest test/

Note: If you are using the STADLE outside the source folder, make sure you ``copy`` the ``setups`` and ``prototypes`` folders to your workspace to test things out.
