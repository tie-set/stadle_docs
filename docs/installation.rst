Installation Instructions
==================


Server Install
**************

macOS

.. code-block:: python

   conda env create -n stadleenv -f ./setups/stadleenv.yaml

Linux

.. code-block:: python

   conda env create -n stadleenv -f ./setups/stadleenv_linux.yaml


Note: The environment has ```Python 3.7.4```. There is some known issues of ```ipfshttpclient``` with ```Python 3.7.2 and older```.


User (Agent) device
********************

.. code-block:: shell

   # macOS
   conda env create -n stadleenv -f ./setups/stadleenv.yaml
   # Linux
   conda env create -n stadleenv -f ./setups/stadleenv_linux.yaml

Database device
*******************

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

Usage
=======

Minimal Example
---------------

This sample does not have actual training. This could be used as a template for user implementation of ML Engine.

Sample Execution
1.Edit the configuration files ``setups/config_db.json``, ``setups/config_aggregator.json``, and ``setups/config_agent.json``. The configuration details are explained here.
2.Run the following four modules as separated processes in the order of ``pseudo_db`` -> ``server_th`` -> ``client`` -> ``minimal_MLEngine``. In the sample execution above, the STADLE client is runnning even the ML engine is stopped. You can reconnect ML engine by rerun the module again.

.. code-block:: shell

   # STADLE execution
   python -m stadle.pseudodb.pseudo_db
   python -m stadle.aggregator.server_th
   python -m stadle.agent.client
   # Sample ML Application execution
   python -m prototypes.minimal.minimal_MLEngine
 
 

   













