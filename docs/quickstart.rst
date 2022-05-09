Quickstart
===============

Using our STADLE platfrom is as easy as just following the 3 steps below.

STEP 1: Set up STADLE Server 
**************************************

Just go to the `stadle.ai`_ and sign up for a free account.
Then, login to your account, create a new project, and initiate an aggregator.
(Just wait for a few seconds to get the aggregator running.)

With your free account, you will be able to create one project and initiate one aggregator.

.. _stadle.ai: https://stadle.ai/

STEP 2: Install STADLE Client 
******************************************

The STADLE client can be installed using the following commands.

First upgrade the pip.

.. code-block:: python

  pip install --upgrade pip

Then, install the `stadle-client`.

.. .. code-block:: python

..  pip install stadle-client

.. If the command above is not working with your environment, please try the following command:

.. code-block:: python

  pip install --index-url http://3.110.171.230:8080 stadle_client --trusted-host 3.110.171.230 --extra-index-url https://pypi.org/simple


STEP 3: Run Local STADLE Example Codes  
******************************************

You can quickly test to run and connect any sample applications from our `STADLE examples`_ about local ML processes.
To do so, just download the local example codes from the repo.

.. _STADLE examples: https://github.com/tie-set/stadle_examples

.. code-block:: python

  git clone https://github.com/tie-set/stadle_examples.git


After downloading the sample codes, just follow the instruction of the README on how to run those applications with the STADLE client-side APIs.

For example, to run the minimal example using PyTorch, just go to `minimal_examples/pytorch` folder.
Then, modify the config files.
In particular, "aggr_ip" is the IP address that you see in the STADLE Dashcboard from your stadle.ai account, 
and "reg_port" is the port to connect shown on the STADLE dashboard as well.

Then, just run the following commands.

.. code-block:: python

  python minimal_admin_agent.py
  python minimal_fl_agent.py --agent_name <AGEMT_NAME>

You can run multiple agents with different agent names.
You will see that the information of the base ML model, aggregation process and round on the STADLE dashboard.
