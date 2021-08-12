Usage
=====

The simplest form of STADLE example is the `minimal example`. For a practical example, check out the `image classification`.

.. _minimal example: https://stadle-documentation.readthedocs.io/en/latest/usage.html#minimal-example
.. _image classification: https://stadle-documentation.readthedocs.io/en/latest/usage.html#image-classification

Minimal Example
***************

This sample does not have actual training. This could be used as a template for user implementation of ML Engine.


PyTorch Minimal Example
--------------------------

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

* Performance Data
  * Example performance metrics (such as accuracy) are populated with fake values, and are stored on our database. During actual usage, these are computed by the user depending on the model, data, training logic, etc.
  * You can access the corresponding ``.db`` file to see the generated performance history.

**Visualization**

The same generated performance metrics can be viewed by running ``ops``, displayed on the ``performance tracking`` page.

Tensorflow Minimal Example
-----------------------------

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
* Example performance metrics (such as accuracy) are populated with fake values, and are stored on our database. During actual usage, these are computed by the user depending on the model, data, training logic, etc.
* You can access the corresponding ``.db`` file to see the generated performance history.

**Visualization**

The same generated performance metrics can be viewed by running ``ops``, displayed on the ``performance tracking`` page.

Image Classification
********************

This sample provides a simple example of STADLE integration with "actual" ML training. Please go to the `prototype directory` for more details.

.. _prototype directory: https://github.com/tie-set/stadle_dev/blob/master/prototypes/image_classification
