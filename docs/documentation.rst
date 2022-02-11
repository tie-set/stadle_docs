Documentation
=============

.. image:: ../_static/architecture_ver0-5.png

STADLE Component Specifications
*******************************

Agent
-----

At the agent side, it is expected that two independent processes, which communicate each other through local files, are running.

.. image:: ../_static/spec_agent.png

.. image:: ../_static/agent_state.png

**Client (Communication handler)**

Note that this portion is written based on the ``Client2.py`` implementation.

* The initialization brings an agent to the ``waiting_sgm`` state where the agent waits for the semi-global models (base models) for training. This change is conducted by the ``Client`` module. While being in this state, the agent will pause its training.
* The arrival of the semi-global models from an aggregator changes the agent's state to ``sg_ready``. This change is conducted by the ``Client`` module. The state is communicated to ``MLEngine`` through a local ``state`` file. At the same time, the semi-global models are saved as a binary local file. The file names and local paths can be configured through the ``config.jason`` file. Please read the `config file documentation`_.
* While being in the ``sg_ready`` and ``training`` states, ``Client`` waits for potential arrival of semi-global models. This scenario happens when its local training was too slow and the aggregator decided to aggregate other local models to create a new set of semi-global models.
  * If the new models arrives, ``Client`` changes the agent's state to ``sg_ready`` to let MLEngine to discard the current training results.
  * If not, ``Client`` waits for ``MLEngine`` finishing its training.
* The end of training is indicated by the ``sending`` state set by ``MLEngine``. Once the ``Client`` observes the state transition, it sends the trained local models stored as a binary file. After sending, it goes back to the ``waiting_sgm`` state.

**MLEngine (ML logic interface)**

Note that this portion is written based on the ``minimal_MLEngine.py`` implementation.

* When the state transits to ``sg_ready``, the ``MLEngine`` notices it reading the local ``state`` file.
* Once the ``MLEngine`` loads the semi-global models, it changes the agent's state to ``training`` and start local training, using the semi-global models as the base models. The state transition is communicated to ``Client`` through the ``state`` file.
* When the training is done, the ``MLEngine`` first checks the ``state`` file.
  * If the ``state`` file still indicates ``training``, it saves the trained local models as a binary file and changes the state to ``sending``.
  * If the ``state`` file was updated to ``sg_ready``, it discards the trained local models and goes back to the ``sg_ready`` state. This results in another ``training`` phase at the ``MLEngine``. This scenario happens when its local training was too slow and the aggregator decided to aggregate other local models to create a new set of semi-global models. The ``MLEngine`` needs to ignore the previous round of training. It may require the reduction of training time by reducing the training cycles or batch sizes.

.. _config file documentation: https://github.com/tie-set/stadle_dev/tree/master/docs/_src

Aggregator
----------

**aggregation (Computational logic)**

``Aggregator`` contains some methods related to the aggregation computation and cluster model sampling. Please refer code documents for details.

**server_th (Communication handler)**
``Server`` provides a set of communication channels to exchange models with agents and access database. Please refer code documents for details.

**state_manager (State maintenance)**
All volatile states of an agent are stored under this ``Manager``. Please refer code documents for details. The following is the brief explanation about the data structure to store models.

Model Buffers

Two model buffers are maintained to temporarily stores models sent to an aggregator.

* ``local_model_buffers``: Buffers to store local models sent from agents. Each entry is in a shape of a dictionary entry: ``'model_name' : list of local models of model_name``. The key (``'model_name'``) needs to be picked from the pre-agreed list of model names. Please refer ``LimitedDict`` specification below.
* ``cluster_model_buffers``: Buffers to store cluster models pulled from database. These models are used to synthesize semi-global models. Each entry is in a shape of a dictionary entry: ``'model_name' : list of cluster models of model_name``. The key (``'model_name'``) needs to be picked from the pre-agreed list of model names. Please refer ``LimitedDict`` specification below.

Models

Two types of synthesized models are always kept in this module.

* ``cluster_models``: A dictionary of the synthesized cluster models. Each entry is in a shape of a dictionary entry: ``'model_name' : [cluster model of model_name]``. The key (``'model_name'``) needs to be picked from the pre-agreed list of model names.
* ``semi_global_models``: A dictionary of the synthesized semi-global models. Each entry is in a shape of a dictionary entry: ``'model_name' : [semi-global model of model_name]``. The key (``'model_name'``) needs to be picked from the pre-agreed list of model names.

Note that an extra attention is needed to extract a model from these model variables because of the data structure. A model of ``model_name`` can be extracted by the following, since the value in the dictionary is a list. This structure is adopted to keep the consistency with the buffers above.

.. code-block:: python

   cluster_models['model_name'][0]
   semi_global_models['model_name'][0]

When sending these models, the dictionary is converted so that its entry will be a shape of 'model_name' : model of model_name (The value is a model itself not a list.) for convenience at the agent and database side.

Other Data Structure
--------------------

**LimitedDict**

A ``LimitedDict`` instance is instantiated give a list of model names. When adding an entry is attempted with a model name that is not defined in the initially given list, it refuses the addition.

.. code-block:: python

   name_list = ['name1', 'name2']
   d = LimitedDict[name_list]

STADLE Communication Protocols
******************************

.. image:: ../_static/protocols2.png

Aggregator-Agent (AGG-AGNT)
---------------------------

**participate Message**

* An agent knows the IP address and port number to join the STADLE platform through the ``config.json`` file.
* When joining the platform, an agent sends a ``participate`` message that contains its ``id``, ``models``, ``init_flag``, ``simulation_flag``, and ``exch_socket``.
  * ``models``: A dictionary of models keyed by the model names agreed on ``config.json``. The weights of models need not to be trained if ``init_flag`` is ``False``, since it is only used by an aggregator to remember the shapes of models.
  * ``init_flag``: A boolean flag to indicate if the sent model weights should be used as a base model. If it is ``True`` and there is no semi-global models ready, an aggregator sets this set of local models as the first semi-global models and send it to all agents.
  * ``simulation_flag``: ``True`` if it is a simulation run.
  * ``exch_socket``: Port number waiting for SG models from the aggregator.

**welcome Message**

* Receiving the ``participation`` message, an aggregator returns a ``welcome`` message containing ``round``, ``socket info``.
  * ``round``: A natural number that indicates the current aggregation round of the aggregator.
  * ``socket info``: Socket numbers for the agent to prepare for the future communications with the aggregator.
* An agent uses the socket information to transit to a state waiting for semi-global models.

**send_sgmodels Message**

* An aggregator sends a set of semi-global models to each agent under it with ``send_sgmodels`` messages. It contains binary representation of the dictionary of the semi-global models.
* Upon the arrival of the message, an agent starts a new round of local training after setting the semi-global models as its base models.

**upload_lmodels Message**

* After a local training phase, an agent uploads the trained local models to the aggregator via a ``upload_lmodels`` message. It contains binary representation of the dictionary of the local models.
* After sending the local models, the agent goes back to a state waiting for a new semi-global model and pauses its training.
* The aggregator stores the uploaded local models in its buffers and waits for another round of cluster model aggregation until enough number of local models are uploaded by agents.

Database-Aggregator (DB-AGG)
----------------------------

All communications between an aggregator and database are initiated by the aggregator.

**push Message**

* An aggregator send its cluster models by a ``push`` message. This message contains binary representation of a model dictionary and the cluster ID.
* Receiving the message, database stores the pair of ``(cluster id, model dictionary)`` in its storage.
* Database returns a confirmation message. Currently, this confirmation is not used at the aggregator.

**get_list Message**

* To prepare a set of cluster models for the semi-global model synthesis, an aggregator sends a ``get_list`` message.
* Database responds to it by returning a list of cluster IDs to which the database stores cluster models corresponding

**get_models Message**

* An aggregator decides, by sampling, a set of cluster models that it wants to pull for the semi-global model synthesis.
* The selected ID list is communicated by a ``get_models`` message.
* Database sends back a set of cluster models specified by the sublist of IDs in the ``get_models`` message.

`STADLE Code Documentation`_
****************************

.. _STADLE Code Documentation: https://tie-set.github.io/stadle_dev/html/

Database (PseudoDB)
-------------------

In addition to the command line instantiation (``python -m stadle.pseudodb.pseudo_db`` + command line arguments), the backend database container can be created and started within Python code.

PseudoDB::
^^^^^^^^^^^^^^^^^^^^^^^^
    PseudoDB(
        config_file: str = None,
        simulation_flag=None,
        db_server_ip_address: str = None,
        db_socket: str = None,
        db_name: str = None,
        data_path: str = None,
        db_model_path: str = None,
        db_token: str = None
    )

Constructor for PseudoDB object

**Parameters**
#. ``config_file`` - Specifies the path of the PseudoDB config file to read parameter values from, if not provided in the respective constructor parameter.  Defaults to value of ``db_server_config_path`` environmental variable (normally set to ``setups/config_db.json``) if no path is provided.
#. ``simulation_flag`` - Determines if PseudoDB should operate in simulation mode for testing, or production mode; this affects what information will be displayed at runtime.
#. ``db_server_ip_address`` - IP address of the instance running the PseudoDB.
#. ``db_socket`` - Port to be exposed for receiving messages from aggregator(s).
#. ``db_name`` - Name of DB file to be read from.
#. ``data_path`` - Path to folder containing DB file.  Will be recursively created if it does not exist.
#. ``db_model_path`` - Path to folder containing all stored model weights.
#. ``db_token`` - Token used to verify validity of connecting aggregator(s).

**Returns**
``PseudoDB`` object with parameters either set from constructor or read from config file.


start_db_server()::
^^^^^^^^^^^^^^^^^^^
    start_db_server()

Opens communication socket and starts DB handler to process requests from connected aggregators



Aggregator (Server)
-------------------

In addition to the command line instantiation (``python -m stadle.aggregator.server_th`` + command line arguments), aggregators can be created and started within Python code.

Server::
^^^^^^^^^^^^^^^^^^^^^^^^
    Server(
        config_file=None,
        simulation_flag=None,
        aggregator_ip_address=None,
        dp_server_ip_address=None,
        aggregation_threshold=None,
        round_interval=None,
        sample_size=None,
        is_sampling=None,
        project_id=None,
        aggr_token=None,
        db_token=None,
        reg_socket=None,
        recv_socket=None,
        aggr_data_path=None,
        exch_socket=None
    )

Constructor for Server object (aggregator)

**Parameters**
#. ``config_file`` - Specifies the path of the aggregator config file to read parameter values from, if not provided in the respective constructor parameter.  Defaults to value of ``aggregate_config_path`` environmental variable (normally set to ``setups/config_aggregator.json``) if no path is provided.
#. ``simulation_flag`` - Determines if aggregator should operate in simulation mode for testing, or production mode; simulation mode uses the default DB token and displays debug information at runtime.
#. ``aggregator_ip_address`` - IP address of this aggregator instance.
#. ``db_server_ip_address`` - IP address of the PseudoDB instance to connect to.
#. ``aggregation_threshold`` - Threshold value of ``(number of received models)/(number of connected agents)`` that must be reached before the round ends and aggregation begins.
#. ``round_interval`` - Delay between subsequent polling messages/aggregation condition checks.
#. ``sample_size`` - Number of cluster models to be sampled when creating semi-global model.
#. ``is_sampling`` - If cluster models should be sampled, or all used when creating semi-global model.
#. ``project_id`` - ID of project that the aggregator is assigned to.
#. ``aggr_token`` - Token used to verify validity of connecting clients.
#. ``db_token`` - Token used to connect to PseudoDB.
#. ``reg_socket`` - Port exposed for clients to use for registration through the aggregator.
#. ``recv_socket`` - Port exposed for clients to communicate with aggregator.
#. ``aggr_data_path`` - Path to folder used for local storage (currently storing aggregator id).
#. ``exch_socket`` - *Deprecated*

**Returns**
``Server`` object with parameters either set from constructor or read from config file.

start_fl_server()::
^^^^^^^^^^^^^^^^^^^^^^^^
    start_fl_server()

Initiates communication with PseudoDB (polling + requests), and opens socket to receive messages from connected clients.

show_server_th_states()::
^^^^^^^^^^^^^^^^^^^^^^^^^^^
    show_server_th_states()

Displays current value of aggregator parameters through ``logging`` module.


Client
--------

The Client object acts as an interface between the MLEngine code produced by the user and STADLE functionality.
It can be used to manage the federated training process at various levels of complexity.

Client::
^^^^^^^^^^^^^^^^^^
    Client(
        config_file: str = None,
        simulation_flag=None,
        aggregator_ip_address: str = None,
        reg_socket: str = None,
        exch_socket: str = None,
        model_path: str = None,
        agent_running: bool = False
    )

Constructor for Client object

**Parameters**
#. ``config_file`` - Specifies the path of the aggregator config file to read parameter values from, if not provided in the respective constructor parameter.  Defaults to value of ``agent_config_path`` environmental variable (normally set to ``setups/config_agent.json``) if no path is provided.
#. ``simulation_flag`` - Determines if client should operate in simulation mode for testing, or production mode; simulation mode uses the default aggregator token and displays debug information at runtime.
#. ``aggregator_ip_address`` - IP address of the aggregator instance to connect to.
#. ``reg_socket`` - Port to be used to create socket for registering through aggregator.
#. ``exch_socket`` - *Deprecated*
#. ``model_path`` - Path to folder used for local storage (client state, id, local and sg models).
#. ``agent_running`` - Flag to determine if agent should actively participate in model exchange with aggregator.

**Returns**
``Client`` object with parameters either set from constructor or read from config file.

set_exch_active()::
^^^^^^^^^^^^^^^^^^^
    set_exch_active(
        exch_active_flag: bool
    )

``exch_active`` - Sets ``exch_active`` flag in client, which determines if client should be considered 'active' by aggregator for aggregation purposes.
