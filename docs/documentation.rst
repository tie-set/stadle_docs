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

`Client API Documentation`_
****************************

BasicClient
-----------

.. function:: stadle.BasicClient(config_file: str = None,\
                 simulation_flag=True,\
                 aggregator_ip_address: str = None,\
                 reg_socket: str = None,\
                 exch_socket: str = None,\
                 model_path: str = None,\
                 agent_running: bool = True)

    Create BasicClient using passed-in parameters or parameters from config file (passed-in parameters take priority),
    used to connnect to a STADLE aggregator and begin participation in FL process

    :param config_file: Specifies the path of the aggregator config file to read parameter values from, if not provided in the respective constructor parameter. Defaults to value of agent_config_path environmental variable (normally set to setups/config_agent.json) if no path is provided.
    :param simulation_flag: Determines if client should operate in simulation mode for testing, or production mode; simulation mode uses the default aggregator token and displays debug information at runtime.
    :param aggregator_ip_address: IP address of the aggregator instance to connect to.
    :param reg_socket: Port to be used to create socket for registering through aggregator.
    :param exch_socket: *Deprecated*
    :param model_path: Path to folder used for local storage (client state, id, local and sg models).
    :param agent_running: Flag to determine if agent should actively participate in model exchange with aggregator.

    :return: Configured BasicClient object

.. function:: stadle.BasicClient.send_trained_model(model, perf_values)

    Extract weights from locally-trained model and send weights to aggregator.

    :param model: Locally-trained model to extract weights from.
    :param perf_values: A dictionary containing key-value pairs for different performance metrics to be displayed in STADLE Ops.  Valid keys are {'performance','accuracy','loss_training','loss_valid','loss_test','f_score','reward'}.
    :return: False if new aggregated model was received during local training process (nothing sent in this case), True otherwise

.. function:: stadle.BasicClient.wait_for_sg_model()

    Blocking function that waits to receive the aggregated model from the aggregator.

    :return: Model object with aggregated weights from previous round.

.. function:: stadle.BasicClient.set_bm_obj(model)

    Set container model object in IntegratedClient for use when converting to/from agnostic format.

    :param model: Used as a container to store aggregated model weights (for ease of use in local training).

.. function:: stadle.BasicClient.disconnect()

    Disconnect client and exit from FL process participation.


IntegratedClient
----------------

.. function:: stadle.IntegratedClient(config_file: str = None,\
                 simulation_flag=True,\
                 aggregator_ip_address: str = None,\
                 reg_socket: str = None,\
                 exch_socket: str = None,\
                 model_path: str = None,\
                 agent_running: bool = True)

    Create IntegratedClient using passed-in parameters or parameters from config file (passed-in parameters take priority),
    used to connnect to a STADLE aggregator and begin participation in FL process.

    :param config_file: Specifies the path of the aggregator config file to read parameter values from, if not provided in the respective constructor parameter. Defaults to value of agent_config_path environmental variable (normally set to setups/config_agent.json) if no path is provided.
    :param simulation_flag: Determines if client should operate in simulation mode for testing, or production mode; simulation mode uses the default aggregator token and displays debug information at runtime.
    :param aggregator_ip_address: IP address of the aggregator instance to connect to.
    :param reg_socket: Port to be used to create socket for registering through aggregator.
    :param exch_socket: *Deprecated*
    :param model_path: Path to folder used for local storage (client state, id, local and sg models).
    :param agent_running: Flag to determine if agent should actively participate in model exchange with aggregator.

    :return: Configured IntegratedClient object

.. function:: stadle.IntegratedClient.set_training_function(fn, train_data, **kwargs)

    Pass model training function, data, and associated arguments to the IntegratedClient for use during local training.

    Model training function must take model, data, and keys of kwargs as arguments.  It must also return the trained
    model and a training performance metric (float value).

    :param fn: Function to perform model training using train_data and kwargs.
    :param train_data: Data object provided to training function during FL process.
    :param **kwargs: Additional required arguments for training function, passed to the function each time it is called.

.. function:: stadle.IntegratedClient.set_cross_validation_function(fn, cross_validation_data, **kwargs)

    Pass model validation function, data, and associated arguments to the IntegratedClient for use during FL process.

    Model validation function must take model, data, and keys of kwargs as arguments.  It must also return two performance
    metrics (float values).

    :param fn: Function to perform model training using cross_validation_data and kwargs.
    :param cross_validation_data: Data object provided to validation function during FL process.
    :param **kwargs: Additional required arguments for validation function, passed to the function each time it is called.

.. function:: stadle.IntegratedClient.set_testing_function(fn, test_data, **kwargs)

    Pass model test function, data, and associated arguments to the IntegratedClient for use at end of FL process.

    Model test function must take model, data, and keys of kwargs as arguments.  It must also return two performance
    metrics (float values).

    :param fn: Function to perform model training using test_data and kwargs.
    :param test_data: Data object provided to validation function during FL process.
    :param **kwargs: Additional required arguments for test function, passed to the function when it is called.

.. function:: stadle.IntegratedClient.set_termination_function(fn, **kwargs)

    Pass agent termination function and associated arguments to the IntegratedClient for use in managing the FL process.

    :param fn: Function to determine if agent should stop participation and disconnect.  Must return either True or False.
    :param **kwargs: Required arguments for termination function, passed to the function each time it is called.

.. function:: stadle.IntegratedClient.set_bm_obj(model)

    Set container model object in IntegratedClient for use when converting to/from agnostic format.

    :param model: Used as a container to store aggregated model weights (for ease of use in local training).

.. function:: stadle.IntegratedClient.start()

    Start FL process defined by functions passed to IntegratedClient.  STADLE then manages both the client-side and server-side of FL.


`Config File Documentation`_
****************************
