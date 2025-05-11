Documentation
=============


Client API Documentation
**************************


BasicClient
-----------

.. function:: stadle.BasicClient(config_file: str = None,\
                 simulation_flag=True,\
                 aggregator_ip_address: str = None,\
                 reg_port: str = None,\
                 exch_port: str = None,\
                 model_path: str = None,\
                 agent_running: bool = True)

    Create BasicClient using passed-in parameters or parameters from config file (passed-in parameters take priority),
    used to connnect to a STADLE aggregator and begin participation in FL process

    :param config_file: Specifies the path of the aggregator config file to read parameter values from, if not provided in the respective constructor parameter. Defaults to value of agent_config_path environmental variable (normally set to setups/config_agent.json) if no path is provided.
    :param simulation_flag: Determines if client should operate in simulation mode for testing, or production mode; simulation mode uses the default aggregator token and displays debug information at runtime.
    :param aggregator_ip_address: Address of the aggregator instance to connect to.
    :param reg_port: Port to be used to create port for registering through aggregator.
    :param model_path: Path to folder used for local storage (client state, id, local and sg models).
    :param agent_running: Flag to determine if agent should actively participate in model exchange with aggregator.

    :return: Configured BasicClient object

.. function:: stadle.BasicClient.send_trained_model(model, perf_values)

    Extract weights from locally-trained model and send weights to aggregator.

    :param model: Locally-trained model to extract weights from.
    :param perf_values: A dictionary containing key-value pairs for different performance metrics to be displayed in STADLE Ops.  Metric names may only contain slashes, alphanumerics, underscores, periods, dashes, and spaces.  Metric values must be numeric.
    :return: False if new aggregated model was received during local training process (nothing sent in this case), True otherwise

.. function:: stadle.BasicClient.send_metrics(model_id, perf_values)

    Send metric values to be associated with the model corresponding to model_id.

    :param model_id: ID of model associated with metrics
    :param perf_values: A dictionary containing key-value pairs for different performance metrics to be displayed in STADLE Ops.  Metric names may only contain slashes, alphanumerics, underscores, periods, dashes, and spaces.  Metric values must be numeric.

.. function:: stadle.BasicClient.wait_for_sg_model()

    Blocking function that waits to receive the aggregated model from the aggregator.

    :return: Model object with aggregated weights from previous round, model ID associated with model

.. function:: stadle.BasicClient.set_bm_obj(model)

    Set container model object in IntegratedClient for use when converting to/from agnostic format.

    :param model: Used as a container to store aggregated model weights (for ease of use in local training).

.. function:: stadle.BasicClient.disconnect()

    Disconnect client and exit from FL process participation.


Config File Documentation
*************************

Configuration of Agent
----------------------

This JSON file (e.g., `config_agent.json`) is read by STADLE admin and ML agents for initial setup.
Here is a sample content of the JSON file:

.. code-block:: json
    :linenos:

    {
        "agent_name": "default_agent",
        "model_path": "./data/agent",
        "local_model_file_name": "lms.binaryfile",
        "semi_global_model_file_name": "sgms.binaryfile",
        "state_file_name": "state",
        "aggr_ip": "localhost",
        "reg_port": "8765",
        "init_weights_flag": 1,
        "token": "stadle12345",
        "simulation": "False",
        "base_model": {
            "model_name": "YOLOv11 Model",
            "model_fn_src": "yolo_model",
            "model_fn": "get_model",
            "model_format": "PyTorch"
        }
    }

Field Descriptions
~~~~~~~~~~~~~~~~~~

- **`agent_name`**: A unique name of the agent that users can define.
  - *Example:* `default_agent`

- **`model_path`**: A path to a local directory in the agent machine to save local models and some state info.
  - *Example:* `./data/agent`

- **`local_model_file_name`**: A file name to save local models in the agent machine.
  - *Example:* `lms.binaryfile`

- **`semi_global_model_file_name`**: A file name to save the latest semi-global models in the agent machine.
  - *Example:* `sgms.binaryfile`

- **`state_file_name`**: A file name to store the agent state in the agent machine.
  - *Example:* `state`

- **`aggr_ip`**: An aggregator IP address for agents to connect.
  - *Example:* `localhost`, `123.456.789`

- **`reg_port`**: A port number used by agents to join an aggregator for the first time.
  - *Example:* `8765`

- **`init_weights_flag`**: A flag used for initializing weights.
  - *Example:* `1`

- **`token`**: A token used for the registration process of agents. Agents need to have the same token to be registered in the STADLE system.
  - *Example:* `stadle12345`

- **`simulation`**: A flag used to enable simulation mode.
  - *Example:* `True`

- **`base_model`**: Information used when loading the base model object.
  - **`model_name`**: Name to be associated with the base model object.
  - **`model_fn_src`**: Module containing the function that returns the model object. This can refer to a local file or an installed module.
  - **`model_fn`**: Name of the model class or function to instantiate
  - **`model_fn_args`**: Arguments passed as kwargs to model_fn (optional)
  - **`model_format`**: Type of model loaded by the `model_fn` function.


Client CLI Documentation
*************************

Upload Model Command
--------------------

The `stadle upload-model` command is used to upload a model to the STADLE system based on configuration specified in a JSON file.

**Usage:**

.. code-block:: bash

    stadle upload-model --config_path <path_to_config_file>

**Arguments:**

- **`--config_path`** (required):  
  Path to the JSON configuration file (e.g., `config_agent.json`) that contains agent and model information.

**Example:**

.. code-block:: bash

    stadle upload-model --config_path config_agent.json

This command reads the specified JSON configuration, loads the base model using the information in **`base_model`**, and uploads the base model and associated information to STADLE.