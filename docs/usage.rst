Usage
=====

This section outlines the required steps to integrate STADLE into a basic deep learning training process.  
Please refer to the :ref:`Quickstart` section to set up the client environment for connecting to STADLE.  
You can also download the sample code, which already includes STADLE integration, from `here`_.

.. _here: https://github.com/tie-set/stadle_examples

STADLE Aggregator Functionalities
*********************************

STADLE aggregators can be configured through the `stadle.ai`_ dashboard, as explained in the Quickstart and User Guide.

.. _stadle.ai: https://stadle.ai/

Creating a Project
------------------

Once you create an account, the first step is to create a new project from the **Overview** page.  
Each project corresponds to one AI model architecture (e.g., VGG16).  
To federate different AI models, you must create separate projects—since the model architecture must remain consistent across all agents connected to a given aggregator.

.. note::

   A free account allows the creation of only one project.

Initiating an Aggregator
------------------------

After creating a project, you can initiate one or more aggregators from the **Overview** page.  
If you want to set up decentralized aggregation with multiple aggregators, you can launch several aggregator instances within the same project to enable the synthesis of Semi-Global Models (SG Models).

.. note::

   A free account allows you to initiate only one aggregator.

Downloading Models
------------------

You can download the latest:

- Global ML models  
- Local models  
- Best-performing models  

These are accessible from your STADLE project dashboard.

Completing the Current Round
----------------------------

This feature allows you to force the aggregator to complete the current round of aggregation.  
Normally, an aggregator waits to collect a sufficient number of local models before proceeding.  
However, using the **Complete Current Round** option, you can manually trigger aggregation even if the collection threshold has not been met.

Aggregation Threshold
---------------------

The **Aggregation Threshold** determines the proportion of local models required from active agents to proceed with aggregation.  
For example, a threshold of `0.7` means that 70% of the active agents must submit their models for aggregation to occur.

Agent Timeout
-------------

This feature disconnects inactive agents based on a timeout interval.  
If an agent is unresponsive for a user-defined number of seconds, it is marked as **TIMEOUT** and excluded from the aggregation process.  
If the agent reconnects, it will be included again.

- A timeout of `0` disables this functionality.

Aggregation Method Selection
----------------------------

While **FedAvg** is the default aggregation algorithm, STADLE supports a variety of methods to suit different applications:

- FedAvg  
- Geometric Median  
- Coordinate-Wise Median  
- Krum  
- Krum Averaging

Choose the method that best aligns with your model’s robustness and convergence requirements.

Synthesize Semi-Global Models
-----------------------------

STADLE supports decentralized aggregation by allowing multiple aggregators to work together to produce **Semi-Global Models (SG Models)**.  
This decentralized structure enables scalable and efficient global model synthesis across distributed clusters.

Aggregation Management
----------------------

On the **Aggregation Management** page, you can monitor:

- Current aggregation round  
- Maximum number of connectable agents  
- Number of active agents  
- Number of local models required for aggregation  
- Number of models already collected

Performance Tracking
--------------------

Track the performance of local ML models on both the **Dashboard** and the **Performance Tracking** page.  
Metrics are recorded for each aggregation round to help you monitor training progress and model accuracy.

Stopping & Restarting Aggregators
---------------------------------

Aggregators can be stopped or restarted from the **Config Info & Settings** page.  
Their status will update to **INACTIVE** or **ACTIVE** based on the action performed.


Client-side STADLE Integration
*******************************

This section will cover the process of integrating STADLE with existing PyTorch code used to train a CNN on the CIFAR-10
dataset.

Local Training Code
-------------------

The following is a breakdown of the PyTorch code serving as the example DL process:

.. code-block::
	:linenos:

	import sys

	import torch
	import torch.nn as nn
	import torch.optim as optim
	import torchvision
	import torchvision.transforms as transforms

	from vgg import VGG
This section imports ``sys`` and the requisite PyTorch libraries for future use.  In addition, a predefined VGG model is imported from
the model definition file.

.. code-block::
	:linenos:

	transform_train = transforms.Compose([
	    transforms.RandomCrop(32, padding=4),
	    transforms.RandomHorizontalFlip(),
	    transforms.ToTensor(),
	    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010)),
	])

	transform_test = transforms.Compose([
	    transforms.ToTensor(),
	    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010)),
	])

	trainset = torchvision.datasets.CIFAR10(
	    root='data', train=True, download=True, transform=transform_train)
	trainloader = torch.utils.data.DataLoader(
	    trainset, batch_size=64, shuffle=True, num_workers=2)

	testset = torchvision.datasets.CIFAR10(
	    root='data', train=False, download=True, transform=transform_test)
	testloader = torch.utils.data.DataLoader(
	    testset, batch_size=64, shuffle=False, num_workers=2)
This section loads in the CIFAR-10 dataset (downloading it if necessary) and applies the transforms to each image to help
augment the dataset for robust training.

.. code-block::
	:linenos:

	device = 'cuda'

	num_epochs = 200
	lr = 0.001
	momentum = 0.9

	model = VGG('VGG16').to(device)

	criterion = nn.CrossEntropyLoss()
	optimizer = optim.SGD(model.parameters(), lr=lr,
	                      momentum=momentum, weight_decay=5e-4)
	scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=200)
This section sets the device to perform training on (GPU in this case) and fixes some training-specific parameters.
It then creates the initial model object and the PyTorch objects used to optimize the model parameters during the
training process.

.. code-block::
	:linenos:

	for epoch in range(num_epochs):
	    print('\nEpoch: %d' % (epoch + 1))

	    model.train()
	    train_loss = 0
	    correct = 0
	    total = 0

	    for batch_idx, (inputs, targets) in enumerate(trainloader):
	        inputs, targets = inputs.to(device), targets.to(device)

	        optimizer.zero_grad()
	        outputs = model(inputs)
	        loss = criterion(outputs, targets)

	        loss.backward()
	        optimizer.step()

	        _, predicted = outputs.max(1)
	        total += targets.size(0)
	        correct += predicted.eq(targets).sum().item()

	        sys.stdout.write('\r'+f"\rEpoch Accuracy: {(100*correct/total):.2f}%")
	    print('\n')

	    if ((epoch + 0) % 5 == 0):
	        model.eval()
	        test_loss = 0
	        correct = 0
	        total = 0

	        with torch.no_grad():
	            for batch_idx, (inputs, targets) in enumerate(testloader):
	                inputs, targets = inputs.to(device), targets.to(device)
	                outputs = model(inputs)
	                loss = criterion(outputs, targets)

	                test_loss += loss.item()
	                _, predicted = outputs.max(1)
	                total += targets.size(0)
	                correct += predicted.eq(targets).sum().item()

	        acc = 100.*correct/total
	        print(f"Accuracy on val set: {acc}%")

Finally, this section handles the actual training of the model.  Training on the train dataset occurs every epoch,
and validation set accuracy is computed every five epochs.

In summary, this code trains the VGG-16 model on the CIFAR-10 dataset for 200 epochs.

Integration with BasicClient
----------------------------

In STADLE, the purpose of a client is to act as an interface between the model training being done locally
and the FL process managed by STADLE's other components.  ``BasicClient`` is an implementation of the STADLE
client, intended for cases where maximal control of the FL process or minimal integration are desired.

The process of integrating with STADLE using the BasicClient can be broken down into four steps:

1. Create and properly configure the BasicClient object
2. Connect the BasicClient to STADLE (via an aggregator)
3. Modify the training loop to send a model to STADLE after some period of local training and to wait to
   receive the aggregated model as a checkpoint to resume local training.
4. Disconnect from STADLE when training is complete

The CIFAR-10 example will be used to show how these steps can be implemented.

Step 1: Create/Configure BasicClient
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First, BasicClient has to be imported from the ``stadle`` library; this is done with

.. code-block::
	:linenos:

	from stadle import BasicClient

The BasicClient object can then be created. The configuration information of the BasicClient can be set by passing a config file path through the constructor. Refer to :ref:`Config File Documentation` for details
on the config file parameters.

.. code-block::
	:linenos:

	client_config_path = r"/path/to/config/file.json"
	stadle_client = BasicClient(config_file=client_config_path)

Alternatively, specific config parameter values can be set directly with the
BasicClient constructor.  Information on the config file and these parameters,
as well as all subsequent function calls, can be found at :ref:`Client API Documentation`.

Step 2: Connect BasicClient to STADLE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The connection between the BasicClient and the aggregator it is configured to
connect to can then be opened with

.. code-block::
	:linenos:

	stadle_client.connect(model)

Note that we pass the recently-intialized model (in this case, the VGG model) to
the client for use as a container for the aggregated parameters received each round.

Step 3: Modify Training Loop
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The local training code previously shown trains the VGG model for 200 epochs.  In
order to apply federated learning to this training process, these 200 epochs must
be broken into numerous short local training periods.  For this example, these local
training periods will be two epochs long; thus, 100 aggregation rounds of two epochs
each will be run.

After one such training period, all of the CIFAR-10 "agents" connected to an aggregator
send their locally-trained models to the aggregator, waiting to receive the aggregated
model before starting the next training period with the received model.  The following
shows an example of how this can be done within the main training loop of the local
training code:

.. code-block::
    :linenos:
    
    for epoch in range(num_epochs):
        print('\nEpoch: %d' % (epoch + 1))
        
        """
        Addition for STADLE integration
        """
        if (epoch % 2 == 0):
            # Don't send model at beginning of training
        
        if (epoch != 0):
            stadle_client.send_trained_model(agent.target_net)

        sg_model_dict = stadle_client.wait_for_sg_model()

        model.load_state_dict(sg_model_dict)

        model.train()
        train_loss = 0
        correct = 0
        total = 0

        for batch_idx, (inputs, targets) in enumerate(trainloader):
            inputs, targets = inputs.to(device), targets.to(device)

            optimizer.zero_grad()
            outputs = model(inputs)
            loss = criterion(outputs, targets)

            loss.backward()
            optimizer.step()

            _, predicted = outputs.max(1)
            total += targets.size(0)
            correct += predicted.eq(targets).sum().item()

            sys.stdout.write('\r'+f"\rEpoch Accuracy: {(100*correct/total):.2f}%")
        print('\n')

Step 4: Disconnect from STADLE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Finally, the BasicClient can be disconnected with

.. code-block::
	:linenos:

	stadle_client.disconnect()

once all training rounds have completed or some other condition has been met.


Running Client-Side STADLE Components
*************************************

After starting the required server-side STADLE components (i.e., the persistence server and one or more aggregators), the final step to initialize the FL process is to upload the base model to the STADLE persistence server. This base model is used internally to convert between specific ML frameworks and STADLE's framework-agnostic model representation.

Uploading the Base Model via CLI
--------------------------------

To upload the base model, use the `stadle upload-model` command along with a configuration file that includes the base model specification.

.. code-block:: bash

    stadle upload-model --config_path <path_to_config_file>

The configuration file must include a `base_model` section structured as follows:

.. code-block:: json

    "base_model": {
        "model_name": "CIFAR-10 VGG Model",
        "model_fn_src": "vgg",
        "model_fn": "VGG",
        "model_fn_args": {
            "vgg_name": "VGG16"
        },
        "model_format": "PyTorch"
    }

Here is what each field represents:

- `"model_name"`: Name to be associated with the base model object.
- `"model_fn_src"`: Module containing the function that returns the model object. This can refer to a local file or an installed module.
- `"model_fn"`: Name of the model class or function to instantiate
- `"model_fn_args"`: Arguments passed as kwargs to model_fn (optional)
- `"model_format"`: Type of model loaded by the `model_fn` function.

Example
~~~~~~~

Assuming the file `vgg.py` contains a model class `VGG`, and the configuration file includes the above `base_model` block, you would upload the model like so:

.. code-block:: bash

    stadle upload-model --config_path ./config/vgg_config.json

Once the upload is successful, you can verify the base model on your STADLE project dashboard.

Running Agents
--------------

After uploading the base model, you can begin running the agent process from the previous section to connect to the aggregator and participate in federated learning.

Each agent will communicate with the aggregator, send local model updates, and receive aggregated models in return.

Execution Order Summary
------------------------

1. Start the **persistence server**
2. Start one or more **aggregators**
3. Run `stadle upload-model` with a configuration file containing the `base_model` block
4. Run **agent(s)** to participate in federated training