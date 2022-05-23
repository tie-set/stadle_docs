Quickstart
===============

Starting using the STADLE platform is as easy as just following the 3 steps below.

STEP 1: Set up STADLE Server 
**************************************

Just go to the `stadle.ai`_ and sign up for a free account.
Then, login to your account, create a new project, and initiate an aggregator.
Just wait for a few seconds to get the aggregator running.

Please also follow the step-by-step instructions on the `Start Yout Project` page in `User Guide`.

.. NOTE:: With your free account, you will be able to create one project and initiate one aggregator. The number of agents that can be connected to the aggregator will be also limited.

.. _stadle.ai: https://stadle.ai/

STEP 2: Install STADLE Client 
******************************************

The STADLE client can be installed using the following commands.

First upgrade the pip.

.. code-block:: python

  pip install --upgrade pip

Then, install the `stadle-client`.

.. code-block:: python

  pip install stadle-client

.. If the command above is not working with your environment, please try the following command:

.. .. code-block:: python

..  pip install --index-url http://3.110.171.230:8080 stadle_client --trusted-host 3.110.171.230 --extra-index-url https://pypi.org/simple

.. NOTE:: The environment needs to be `Python 3.8.0` or newer.


STEP 3: Run Local STADLE Example Codes  
******************************************

The next phase is to utilize STADLE libraries by importing them in your local ML processes. You can quickly run and test some of the sample applications from `STADLE examples`_ in which some of the key STADLE client libraries are already connected. To do so, just download the local example codes from the repo.

.. _STADLE examples: https://github.com/tie-set/stadle_examples

.. code-block:: python

  git clone https://github.com/tie-set/stadle_examples.git


After downloading the sample codes, just follow the instruction of the README on how to run those applications with the STADLE client-side APIs.

For example, to run the minimal example using PyTorch, just go to the `"stadle_examples/minimal_examples/pytorch"` folder.
Then, modify the config files for both admin and ML agents.
In particular, `"aggr_ip”` should be the "IP Address" and `“reg_port”` should be the "Port to Connect", both shown on the STADLE dashboard.

Then, run the admin agent with the following command (In this case, the script is named minimal_admin_agent.py). The admin agent uploads the base model that defines the architecture of the ML models that will be aggregated.

.. code-block:: python

  python minimal_admin_agent.py

Then, go to the STADLE dashboard and update the page after a few seconds. You can check the name of the uploaded base model on the dashboard.
You can run multiple agents with different agent names. In this case, the name of the local ML process script is "minimal_fl_agent.py". For example, you can run the ML agents like

.. code-block:: python

  python minimal_fl_agent.py --agent_name AGEMT_NAME_01
  python minimal_fl_agent.py --agent_name AGEMT_NAME_02

On the STADLE dashboard, you will see the number of connected ML agents and downloadable recent global and local models as well as the best performing model.

You have successfully completed all the steps to conduct a STADLE project properly. The aggregation process for each round can be checked and managed on Aggregation Management page. Also, the configuration information and system status of aggregators and agents can be checked on Config Info & Settings page.

You will also be able to download the recent global and local models as well as best performing models on the STADLE dashboard.

Performance of the uploaded local ML models for each aggregation round can be tracked on the Dashboard as well as Performance Tracking page. You can monitor the learning process for each metrics of your ML models on the Performance Tracking page.
