Documentation
=============


Client API Documentation
**************************


AdminAgent
-----------

.. py:attribute:: stadle.AdminAgent

    Class of Admin Agent to register initial base models.


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

.. function:: stadle.BasicClient.send_trained_model(model)

    Extract weights from locally-trained model and send weights to aggregator.

    :param model: Locally-trained model to extract weights from.
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


Config File Documentation
**************************

The users of the STADLE basocally care about the agent side configulation in order to connect the agent side ML application to the STADLE aggregator.

Configuration of Agent
------------------------

This JSON file, for example `config_agent.json` file, is read by STADLE agents for initial setup.
Here is the sample content of the JSON file.

.. code-block::
    :linenos:

    {
        "agent_name": "default_agent"
        "model_path": "./data/agent",
        "local_model_file_name": "lms.binaryfile",
        "semi_global_model_file_name": "sgms.binaryfile",
        "state_file_name": "state",
        "aggr_ip": "localhost",
        "reg_socket": "8765",
        "init_weights_flag": 1,
        "token": "stadle12345",
        "simulation": "False",
        "exch_socket": "0000"
    }

- `agent_name`: A unique name of the agent that users can define.
  - e.g. `default_agent`
- `model_path`: A path to a local director in the agent machine to save local models and some state info. 
  - e.g. `./data/agent`
- `local_model_file_name`: A file name to save local models in the agent machine. 
  - e.g. `lms.binaryfile`
- `semi_global_model_file_name`: A file name to save the latest semi-global models in the agent machine. 
  - e.g. `sgms.binaryfile`
- `state_file_name`: A file name to store the agent state in the agent machine.
  - e.g. `state`
- `aggr_ip`: An aggregator IP address for agents to connect.
  - e.g. `localhost`, `123.456.789`
- `reg_socket`: A socket number used by agents to join an aggregator for the first time.
  - e.g. `8765`
- `init_weights_flag`: A flag used for initializing weights.
  - e.g. `1`
- `token`: A token that is used for registration process of agents. Agents need to have the same token to be registered in the STADLE system.
  - e.g. `stadle12345`
- `simulation`: A flag used to enable a simulation mode.
  - e.g. `True`
- `exh_socket`: A socket number used to upload local models to an aggregator from an agent. Agents will get to know this socket from the communications with an aggregator.
  - e.g. `7890`


Configuration of Persistence Server
------------------------------------

This JSON file, for example `config_db.json`, is read by STADLE persistence server for initial setup.
Here is the sample content of the JSON file.

.. code-block::
    :linenos:

    {
        "db_ip": "localhost",
        "db_socket": 9017,
        "db_data_path": "./db",
        "db_name": "sample_data",
        "db_model_path": "./db/sample_models",
        "db_token": "Stadledb123$%",
        "simulation": "False"
    }

- `db_ip`: An DB IP address
  - e.g. `localhost`
- `db_socket`: A socket number used between DB and an aggregator.
  - e.g. `9017`
- `db_data_path`: A path to the database directory.
  - e.g. `./db`
- `db_name`: Name of database. If the same database name is called, STADLE reuse the databasem, otherwise creating a new db.
  - e.g. `sample_data`
- `db_model_path`: A path to the directory in which AI models are stored.
  - e.g. `./db/sample_models`
- `simulation`: A flag used to enable a simulation mode.
  - e.g. `True`


Configuration of Aggregator
-------------------------

This JSON file, for example `config_aggregator.json`, is read by STADLE aggregators for initial setup.
Here is the sample content of the JSON file.

.. code-block::
    :linenos:

    {
        "aggr_name": "default_aggregator"
        "aggr_ip": "localhost",
        "project_id": "default_project_id",
        "reg_socket": 8765,
        "recv_socket": 4321,
        "exch_socket": 7890,
        "db_ip": "0.0.0.0",
        "db_socket": 9017,
        "round_interval": 5,
        "sample_size": 1,
        "is_sampling": 1,
        "aggregation_threshold": 0.7,
        "aggr_data_path": "./data/agg",
        "aggr_token": "stadle12345",
        "db_token": "Stadledb123$%",
    }


- `aggr_name`: A unique name of the aggregator that users can define.
  - e.g. `default_aggregator`
- `aggr_ip`: An aggregator IP address
  - e.g. `localhost`
- `project_id`: A project ID to which an aggregator is assigned.
  - e.g. `default_project_id`
- `reg_socket`: A socket number used by agents to join an aggregator for the first time.
  - e.g. `8765`
- `recv_socket`: A socket number used to send back semi global models to an agent from an aggregator. Agents will get to know this socket from the communications with an aggregator.
  - e.g. `4321`
- `exch_socket`: A socket number used to upload local models to an aggregator from an agent. Agents will get to know this socket from the communications with an aggregator.
  - e.g. `7890`
- `db_ip`: IP address of DB instance. Used to connect to DB in which aggregators' info is saved.
  - e.g. `localhost`
- `db_socket`: A socket number used between DB and an aggregator.
  - e.g. `9017`
- `round_interval`: Period of time after which an agent check if there are enough number of models to start an aggregation step. (Unit: seconds)
  - e.g. `5`
- `sample_size`: The number of cluster models used by an aggregator when it synthesizes semi global models.
  - MUST BE LESS THAN the total number of clusters
  - e.g. `1`
- `is_sampling`: Boolean flag that indicates if an aggregator uses a sampling synthesis. Sampling is on if `1`. All cluster models are used if it is set to `0`.
  - e.g. `1`
- `aggregation_threshold`: The percentage over the number of local models required to start an aggregation step. If it is `0.7`, for example, 70% of the active agents need to upload the local ML models to start the aggregation.
  - e.g. `1.0`, `0.7`
- `aggr_data_path`: A path to aggregators data such as their IDs. If multiple aggregators are running, each path needs to be identical.
  - e.g. `./data/agg`
- `aggr_token`: A token that is used for registration process of agents. Agents need to have the same token to be registered in the STADLE system.
  - e.g. `stadle12345`
- `db_token`: A token that is used for registring an aggregator into the STADLE database. Aggregators need to have the same token to be registered in the STADLE Database.
  - e.g. `Stadledb123$%`
