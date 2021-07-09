Installation Instructions
=========================


Server Install
**************

**macOS**

.. code-block:: python

   conda env create -n stadleenv -f ./setups/stadleenv.yaml

**Linux**

.. code-block:: python

   conda env create -n stadleenv -f ./setups/stadleenv_linux.yaml


Note: The environment uses ```Python 3.7.4```. There are known issues for ```ipfshttpclient``` with ```Python 3.7.2 and older```.


User (Agent) device
********************

.. code-block:: shell

   # macOS
   conda env create -n stadleenv -f ./setups/stadleenv.yaml
   # Linux
   conda env create -n stadleenv -f ./setups/stadleenv_linux.yaml

Database device
***************

.. code-block:: shell

   # macOS
   conda env create -n stadleenv -f ./setups/stadleenv.yaml
   # Linux
   conda env create -n stadleenv -f ./setups/stadleenv_linux.yaml

Be sure to activate the virtual environment. 

.. code-block:: shell

   # macOS
   conda activate stadleenv
   # Linux
   source activate stadleenv
