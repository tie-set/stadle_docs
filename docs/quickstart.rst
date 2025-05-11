Quickstart
==========

Getting started with the STADLE platform is as easy as following the three steps below.

STEP 1: Set up the STADLE Server
********************************

Visit `stadle.ai`_ and sign up for a free account.  
After logging in, create a new project and launch an aggregator.  
It only takes a few seconds for the aggregator to initialize.

Be sure to follow the step-by-step instructions on the **Start Your Project** page in the **User Guide**.

.. note::

   With a free account, you can create one project and launch a single aggregator.  
   The number of agents that can connect to this aggregator is also limited.

.. _stadle.ai: https://stadle.ai/

STEP 2: Install the STADLE Client
*********************************

Install the STADLE client using the following commands.

First, upgrade pip:

.. code-block:: bash

    pip install --upgrade pip

Then, install the ``stadle-client`` library:

.. code-block:: bash

    pip install stadle-client

.. note::

   STADLE requires Python version 3.8.0 or newer.

STEP 3: Run a Local STADLE Example
**********************************

Now it's time to use STADLE in your own machine learning workflows.  
You can quickly test the platform using sample applications provided in the `STADLE examples`_ repository, which already includes the key STADLE client libraries.

Clone the example repository:

.. _STADLE examples: https://github.com/tie-set/stadle_examples

.. code-block:: bash

    git clone https://github.com/tie-set/stadle_examples.git

After downloading the examples, refer to the README for instructions on running the applications with STADLE’s client-side APIs.

For example, to run the minimal PyTorch-based example, navigate to the ``stadle_examples/minimal_examples/pytorch`` folder.  
Edit the agent config file to set the appropriate values:

- ``aggr_ip``: the **IP Address** shown on your STADLE project dashboard  
- ``reg_port``: the **Port to Connect** as displayed on the dashboard

Then, upload the base model using the following command:

.. code-block:: bash

    stadle upload-model --config_path <path_to_config_file>

After the upload is complete, visit your STADLE project dashboard to verify that the base model has been registered.

You can now run multiple agents, differentiated by their names.  
For instance, with the FL agent script ``minimal_fl_agent.py``, run the agents as follows:

.. code-block:: bash

    python minimal_fl_agent.py --agent_name AGENT_NAME_01
    python minimal_fl_agent.py --agent_name AGENT_NAME_02

Your STADLE dashboard will display the number of connected ML agents and show the models that have been uploaded or aggregated.

Congratulations! You’ve successfully completed all the steps to launch a STADLE project.  
You can monitor and manage the aggregation process on the **Aggregation Management** page, and view system configurations and status under **Config Info & Settings**.