Guides
======

`Minimal Example`_
******************

This sample does not have actual training. This could be used as a template 
for user implementation of ML Engine.

.. _Minimal Example: https://github.com/tie-set/stadle_dev/tree/master/stadle/prototypes/minimal

Sample Execution
----------------

# Edit the configuration file `setups/config.jason`_. The configuration details 
are explained `here`_.
# Run the following four modules as separated processes in the order of 
``pseudo_db`` -> ``server_th`` -> ``minimal_MLEngine`` -> ``client2``.

::

    python -m stadle.pseudodb.pseudo_db
    python -m stadle.aggregator.server_th
    python -m stadle.prototypes.minimal.minimal_MLEngine
    python -m stadle.agent.client2

.. _setups/config.jason: https://github.com/tie-set/stadle_dev/blob/master/setups/config.json
.. _here: https://github.com/tie-set/stadle_dev/tree/master/setups

Simulation
----------

Beta-ver: STADLE can be run for simulation on the same machine by specifying 
the port numbers for agents and aggregators.

::

    python -m stadle.pseudodb.pseudo_db [simulation_flag] [config_path]
    python -m stadle.aggregator.server_th [simulation_flag] [participation_port] [local_model_recv_port] [path_to_local_file] [config_path]
    python -m stadle.prototypes.minimal.minimal_MLEngine [simulation_flag] [path_to_local_file] [config_path]
    python -m stadle.agent.client2 [simulation_flag] [participation_port] [sg_model_recv_port] [path_to_local_file] [config_path]

Common
------

* ``simulation_flag``: 1 if it's simulation.
* ``config_path``: Path to the config file. If ``AUTO``, the config file in ``(repo root)
  /setups/config.json``. (For the consistency with older versions, it automatically 
  sets this flag to ``AUTO`` when no flag is given).

Aggregator Side
---------------

* ``participation_port``: Port number waiting for a new agent's participate 
  message.
* ``local_model_recv_port``: Port number waiting for local model uploads 
  from agents. This will be communicated to each agent who sends a participate 
  message to this aggregator via the welcome message.
* ``path_to_local_file``: Path to the local directory storing the aggr_id 
  file about a unique aggregator ID. This needs to be unique for every 
  aggregator.

Agent Side
----------

* ``participation_port``: Port number to send a participate message. 
  It needs to match with participation_port of the aggregator to which 
  the agent sends the message.
* ``sg_model_recv_port``: Port number waiting for SG models from the 
  aggregator. This will be communicated to a selected aggregator via a 
  participate message.
* ``path_to_local_file``: Path to the local directory storing the state 
  and model files. This needs to be unique for every agent (The same one for 
  one pair of MLEngine and Client2).

For example:

::

    python -m stadle.pseudodb.pseudo_db 1 AUTO
    python -m stadle.aggregator.server_th 1 50001 50002 ./data/aggr01 AUTO
    # First agent
    python -m stadle.prototypes.minimal.minimal_MLEngine 1 ./data/agent01 AUTO
    python -m stadle.agent.client2 1 50001 50003 ./data/agent01 AUTO
    # Second agent
    python -m stadle.prototypes.minimal.minimal_MLEngine 1 ./data/agent02 AUTO
    python -m stadle.agent.client2 1 50001 50004 ./data/agent02 AUTO

# Edit the configuration file ``setups/config.jason``.
# Run the following four modules as separated processes in the order of 
``pseudo_db`` -> ``server_th`` -> ``minimal_MLEngine`` -> ``client2``

`Image Classification Application`_
***********************************

This sample provides a simple example of STADLE integration with "actual" 
ML training. Please go to the `prototype directory`_ for more details.

.. _Image Classification Application: https://github.com/tie-set/stadle_dev/tree/master/stadle/prototypes/image_classification
.. _prototype directory: https://github.com/tie-set/stadle_dev/tree/master/stadle/prototypes/image_classification