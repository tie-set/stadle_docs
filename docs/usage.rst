*****
Usage
*****

This section covers the requisite steps for integrating STADLE with a basic deep learning training process.
Please refer to :ref:`Installation` to set up the virtual environment used to run STADLE.

Running STADLE Components
=========================

There are two main STADLE-side components that must be running to manage a federated learning process.

The first is the **persistence server** - this component is in charge of managing the model parameters and other
FL-specific data generated while using STADLE.  With the STADLE virtual environment active, the persistence server
can be started with the following command:

::

	stadle persistence-server
The persistence server can be configured with a config file by including the path to the file as an argument:

::

	stadle persistence-server --config_file /path/to/config/file.json
Specific parameters can also be set using command line arguments - refer to :ref:`Documentation` for details
on the config file parameters and the respective accepted command line arguments for the persistence server.


The second STADLE-side component is the **aggregator** - as the name suggests, this component is in charge of collecting
and aggreagting the models it receives from the clients performing local training to produce cluster models.  It then
communicates with the persistence server to aggregate a sample of cluster models and produce a semi-global model, which is
sent back to the clients for the next round of local training.  Similarly, the aggregator can be started with the following
command:

::

	stadle aggregator --config_file /path/to/config/file.json
Specific parameters can be set using command line arguments - refer to :ref:`Documentation` for details
on the config file parameters and the respective accepted command line arguments for the aggregator.

Client-side STADLE Integration
==============================

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

The BasicClient object can then be created.  The configuration information of the
BasicClient can be set by passing a config file path through the constructor

.. code-block::
	:linenos:

	bc_config_path = r"/path/to/config/file.json"
	stadle_client = BasicClient(config_file=bc_config_path)

Alternatively, specific config parameter values can be set directly with the
BasicClient constructor.  Information on the config file and these parameters,
as well as all subsequent function calls, can be found at :ref:`Documentation`.

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
