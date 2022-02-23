Deployment
============

This section covers the requisite steps for integrating STADLE with a basic deep learning training process.
Please refer to :ref:`Installation` to set up the virtual environment used to run STADLE.

Running Server-Side STADLE Components
=====================================

There are two main STADLE-side components that must be running to manage a federated learning process.

The first is the **persistence server** - this component is in charge of managing the model parameters and other
FL-specific data generated while using STADLE.  With the STADLE virtual environment active, the persistence server
can be started with the following command:

::

	stadle persistence-server
The persistence server can be configured with a config file by including the path to the file as an argument:

::

	stadle persistence-server --config_file /path/to/config/file.json
Specific parameters can also be set using command line arguments - refer to :ref:`Config File Documentation` for details
on the config file parameters, and run ``stadle persistence-server --help`` to see the accepted command line arguments for the persistence server.


The second STADLE-side component is the **aggregator** - as the name suggests, this component is in charge of collecting
and aggreagting the models it receives from the clients performing local training to produce cluster models.  It then
communicates with the persistence server to aggregate a sample of cluster models and produce a semi-global model, which is
sent back to the clients for the next round of local training.  Similarly, the aggregator can be started with the following
command:

::

	stadle aggregator --config_file /path/to/config/file.json
Specific parameters can be set using command line arguments - refer to :ref:`Config File Documentation` for details
on the config file parameters, and run ``stadle aggregator --help`` to see the accepted command line arguments for the aggregator.

