Installation Instructions
==================


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

**Sample Execution**

1. Edit the configuration files ``setups/config_db.json``, ``setups/config_aggregator.json``, and ``setups/config_agent.json``. The configuration details are explained here.

2. Run the following four modules as separated processes in the order of ``pseudo_db`` -> ``server_th`` -> ``client`` -> ``minimal_MLEngine``. In the sample execution above, the STADLE client is runnning even the ML engine is stopped. You can reconnect ML engine by rerun the module again.

.. code-block:: shell

   # STADLE execution
   python -m stadle.pseudodb.pseudo_db
   python -m stadle.aggregator.server_th
   python -m stadle.agent.client
   # Sample ML Application execution
   python -m prototypes.minimal.minimal_MLEngine
 
3. Or you can simply runthe following three modules as separated processes in the order of ``pseudo_db`` -> ``server_th`` -> ``minimal_MLEngine_template`` where the ML engine template utilizes the libraries provided by ``stadle.agent.client.py``.

.. code-block:: shell

   # STADLE execution
   python -m stadle.pseudodb.pseudo_db
   python -m stadle.aggregator.server_th
   # Sample ML Application execution with STADLE client
   python -m prototypes.minimal.minimal_MLEngine_template
   

Also, here is how to run the STADLE Ops.

.. code-block:: shell

   python ops/routes.py
   
Simulation
----------

Beta-ver: STADLE can be run for simulation on the same machine by specifying different port numbers for agents and aggregators.

.. code-block:: shell

   python -m stadle.pseudodb.pseudo_db [simulation_flag] [config_path]
   python -m stadle.aggregator.server_th [simulation_flag] [participation_port] [local_model_recv_port] [path_to_local_file] [config_path]
   python -m prototypes.minimal.minimal_MLEngine_template [simulation_flag] [participation_port] [sg_model_recv_port] [path_to_local_file] [config_path]

**Common**

* ``simulation_flag``: 1 if it's simulation
* ``config_path``: Path to the config file. If ``AUTO``, the config file in (``repo root)/setups/config.json``. (For the consistency with older versions, it automatically sets this flag to ``AUTO`` when no flag is given.)

**Aggregator side**

* ``participation_port``: Port number waiting for a new agent's participate message
* ``local_model_recv_port``: Port number waiting for local model uploads from agents. This will be communicated to each agent who sends a participate message to this aggregator via the welcome message.
* ``path_to_local_file``: Path to the local directory storing the ``aggr_id`` file about a unique aggregator ID. This needs to be unique for every aggregator.

**Agent side**

* ``participation_port``: Port number to send a participate message. It needs to match with ``participation_port`` of the aggregator to which the agent sends the message.
* ``sg_model_recv_port``: Port number waiting for SG models from the aggregator. This will be communicated to a selected aggregator via a participate message.
* ``path_to_local_file``: Path to the local directory storing the ``state`` and model files. This needs to be unique for every agent (The same one for one pair of ``MLEngine`` and ``Client2``).

**For example:**

.. code-block:: shell

   python -m stadle.pseudodb.pseudo_db 1 AUTO
   python -m stadle.aggregator.server_th 1 50001 50002 ./data/aggr01 AUTO
   # First agent
   python -m prototypes.minimal.minimal_MLEngine_template 1 50001 50003 ./data/agent01 AUTO
   # Second agent
   python -m prototypes.minimal.minimal_MLEngine_template 1 50001 50004 ./data/agent02 AUTO
   

1. Edit the configuration files if necessary.

2. Run the modules above as separated processes in the order of ``pseudo_db`` -> ``server_th`` -> ``minimal_MLEngine_template``.

Another way of doing simulation with many nodes of clients is as follows:

.. code-block:: shell

   python -m stadle.pseudodb.pseudo_db
   python -m stadle.aggregator.server_th
   python -m prototypes.minimal.minimal_MLEngine_template_sim [NUMBER_OF_AGENTS]
   

Please replace the [NUMBER_OF_AGENTS] with a number such as 10. You may have to increase the number of files that can be opened by the system. In macOS, for example, you can change the setting with ulimit command such as "ulimit -n 2048".


Image Classification Application
--------------------------------

This sample provides a simple example of STADLE integration with "actual" ML training. Please go to the prototype directory for more details.

.. image:: ../_static/STADLE architecture.png
    :width: 200px
    :align: center
    :height: 100px
    :alt: alternate text













