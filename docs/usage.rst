Usage
=====

The simplest form of STADLE example is the minimal example. For a practical example, check out the image classification.

Minimal Example
---------------

This sample does not have actual training. This could be used as a template for user implementation of ML Engine.

**1. PyTorch Minimal Example**

This prototype is a minimal example designed to show how STADLE functionality can be tied into the model training process for Federated Learning, and to serve as verification that STADLE is successfully set up.

A simple two-layer ANN (created using PyTorch) is used as the model to demonstrate weight extraction and aggregation through STADLE. Components related to performing local training are left largely unimplemented for simplicity and to act as a potential template for user implementation, if applicable.

Execution
This example assumes you are running two agents. You can increase the number of agents running on the same device by specifying appropriate port numbers. Since the minimal engine is for simulation, we have included the option to start a desired number of agents by modifying ``num_agents`` in the ``minimal_engine_sim_config.json`` file. Please see the README file for the STADLE platform to see how to install the required libraries and configure STADLE components when needed.

Note that three terminals will be necessary to run all of the components - other than the admin agent, each command will start a component that runs continuously.

To run, use the following commands:

.. code-block:: python

   python -m stadle.pseudodb.pseudo_db
   python -m stadle.aggregator.server_th
   python -m prototypes.sample.minimal_pt.minimal_admin_agent
   python -m prototypes.sample.minimal_pt.minimal_engine_sim

The admin agent is used to upload the model to the aggregator(s) and database for use in the FL process.

**Evaluation**

**Performance Data
*Example performance metrics (such as accuracy) are populated with fake values, and are stored on our database. During actual usage, these are computed by the user depending on the model, data, training logic, etc.
*You can access the corresponding ``.db`` file to see the generated performance history.

**Visualization**

The same generated performance metrics can be viewed by running ``ops``, displayed on the ``performance tracking`` page.

**2. Tensorflow Minimal Example**

This prototype is a minimal example designed to show how STADLE functionality can be tied into the model training process for Federated Learning, and to serve as verification that STADLE is successfully set up.

A simple two-layer ANN (created using Keras through TensorFlow) is used as the model to demonstrate weight extraction and aggregation through STADLE. Components related to performing local training are left largely unimplemented for simplicity and to act as a potential template for user implementation, if applicable.

**Execution**

This example assumes you are running two agents. You can increase the number of agents running on the same device by specifying appropriate port numbers. Since the minimal engine is for simulation, we have included the option to start a desired number of agents by modifying ``num_agents`` in the ``minimal_engine_sim_config.json`` file. Please see the README file for the STADLE platform to see how to install the required libraries and configure STADLE components when needed.

Note that three terminals will be necessary to run all of the components - other than the admin agent, each command will start a component that runs continuously.

To run, use the following commands:

.. code-block:: python

   python -m stadle.pseudodb.pseudo_db
   python -m stadle.aggregator.server_th
   python -m prototypes.sample.minimal_tf.minimal_admin_agent
   python -m prototypes.sample.minimal_tf.minimal_engine_sim

The admin agent is used to upload the model to the aggregator(s) and database for use in the FL process.

**Evaluation**

Performance Data
*Example performance metrics (such as accuracy) are populated with fake values, and are stored on our database. During actual usage, these are computed by the user depending on the model, data, training logic, etc.
*You can access the corresponding ``.db`` file to see the generated performance history.

**Visualization**

The same generated performance metrics can be viewed by running ``ops``, displayed on the ``performance tracking`` page.

.. _minimal example: https://stadle-documentation.readthedocs.io/en/latest/usage.html#minimal-example
.. _image classification: https://stadle-documentation.readthedocs.io/en/latest/usage.html#image-classification

Image Classification
--------------------

This sample provides a simple example of STADLE integration with "actual" ML training. Please go to the prototype directory for more details.

.. image:: ../_static/architecture_ver0-5.png

.. _prototype directory: https://github.com/tie-set/stadle_dev/blob/master/prototypes/image_classification

STADLE Component Specifications
-------------------------------

**Agent**

At the agent side, it is expected that two independent processes, which communicate each other through local files, are running.

.. image:: ../_static/spec_agent.png

.. image:: ../_static/agent_state.png

**``Client`` (Communication handler)**

Note that this portion is written based on the ``Client2.py`` implementation.

*The initialization brings an agent to the ``waiting_sgm`` state where the agent waits for the semi-global models (base models) for training. This change is conducted by the ``Client`` module. While being in this state, the agent will pause its training.
*The arrival of the semi-global models from an aggregator changes the agent's state to ``sg_ready``. This change is conducted by the ``Client`` module. The state is communicated to ``MLEngine`` through a local ``state`` file. At the same time, the semi-global models are saved as a binary local file. The file names and local paths can be configured through the ``config.jason`` file. Please read the config file documentation.
*While being in the ``sg_ready`` and ``training`` states, ``Client`` waits for potential arrival of semi-global models. This scenario happens when its local training was too slow and the aggregator decided to aggregate other local models to create a new set of semi-global models.
**If the new models arrives, ``Client`` changes the agent's state to ``sg_ready`` to let MLEngine to discard the current training results.
**If not, ``Client`` waits for ``MLEngine`` finishing its training.
*The end of training is indicated by the ``sending`` state set by ``MLEngine``. Once the ``Client`` observes the state transition, it sends the trained local models stored as a binary file. After sending, it goes back to the ``waiting_sgm`` state.

**``MLEngine`` (ML logic interface)**

Note that this portion is written based on the ``minimal_MLEngine.py`` implementation.

*When the state transits to ``sg_ready``, the ``MLEngine`` notices it reading the local ``state`` file.
*Once the ``MLEngine`` loads the semi-global models, it changes the agent's state to ``training`` and start local training, using the semi-global models as the base models. The state transition is communicated to ``Client`` through the ``state`` file.
*When the training is done, the ``MLEngine`` first checks the ``state`` file.
**If the ``state`` file still indicates ``training``, it saves the trained local models as a binary file and changes the state to ``sending``.
**If the ``state`` file was updated to ``sg_ready``, it discards the trained local models and goes back to the ``sg_ready`` state. This results in another ``training`` phase at the ``MLEngine``. This scenario happens when its local training was too slow and the aggregator decided to aggregate other local models to create a new set of semi-global models. The ``MLEngine`` needs to ignore the previous round of training. It may require the reduction of training time by reducing the training cycles or batch sizes.

**Aggregator**

**``aggregation`` (Computational logic)**

``Aggregator`` contains some methods related to the aggregation computation and cluster model sampling. Please refer code documents for details.

**``server_th`` (Communication handler)**
``Server`` provides a set of communication channels to exchange models with agents and access database. Please refer code documents for details.

**``state_manager`` (State maintenance)**
All volatile states of an agent are stored under this ``Manager``. Please refer code documents for details. The following is the brief explanation about the data structure to store models.

Model Buffers

Two model buffers are maintained to temporarily stores models sent to an aggregator.

*``local_model_buffers``: Buffers to store local models sent from agents. Each entry is in a shape of a dictionary entry: ``'model_name' : list of local models of model_name``. The key (``'model_name'``) needs to be picked from the pre-agreed list of model names. Please refer ``LimitedDict`` specification below.
*``cluster_model_buffers``: Buffers to store cluster models pulled from database. These models are used to synthesize semi-global models. Each entry is in a shape of a dictionary entry: ``'model_name' : list of cluster models of model_name``. The key (``'model_name'``) needs to be picked from the pre-agreed list of model names. Please refer ``LimitedDict`` specification below.

Models
Two types of synthesized models are always kept in this module.

*``cluster_models``: A dictionary of the synthesized cluster models. Each entry is in a shape of a dictionary entry: ``'model_name' : [cluster model of model_name]``. The key (``'model_name'``) needs to be picked from the pre-agreed list of model names.
*``semi_global_models``: A dictionary of the synthesized semi-global models. Each entry is in a shape of a dictionary entry: ``'model_name' : [semi-global model of model_name]``. The key (``'model_name'``) needs to be picked from the pre-agreed list of model names.

Note that an extra attention is needed to extract a model from these model variables because of the data structure. A model of ``model_name`` can be extracted by the following, since the value in the dictionary is a list. This structure is adopted to keep the consistency with the buffers above.

.. code-block:: python

   cluster_models['model_name'][0]
   semi_global_models['model_name'][0]

When sending these models, the dictionary is converted so that its entry will be a shape of 'model_name' : model of model_name (The value is a model itself not a list.) for convenience at the agent and database side.

**Other Data Structure**

**LimitedDict**

A ``LimitedDict`` instance is instantiated give a list of model names. When adding an entry is attempted with a model name that is not defined in the initially given list, it refuses the addition.

.. code-block:: python

   name_list = ['name1', 'name2']
   d = LimitedDict[name_list]

.. _config file documentation: https://github.com/tie-set/stadle_dev/tree/master/docs/_src

