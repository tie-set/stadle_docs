Usage
=============

This section covers the requisite steps for integrating STADLE with a basic deep learning training process.
Please refer to :ref:`Installation` to set up the virtual environment used to run STADLE.

Running Server-Side STADLE Components
**************************************

There are two main STADLE-side components that must be running to manage a federated learning process.

The first is the **persistence server** - this component is in charge of managing the model parameters and other
FL-specific data generated while using STADLE.  With the STADLE virtual environment active, the persistence server
can be started with the following command:

.. code-block::
    :linenos:
	stadle persistence-server

The persistence server can be configured with a config file by including the path to the file as an argument:


.. code-block::
    :linenos:
	stadle persistence-server --config_file /path/to/config/file.json

Specific parameters can also be set using command line arguments - refer to :ref:`Config File Documentation` for details
on the config file parameters, and run ``stadle persistence-server --help`` to see the accepted command line arguments for the persistence server.


The second STADLE-side component is the **aggregator** - as the name suggests, this component is in charge of collecting
and aggreagting the models it receives from the clients performing local training to produce cluster models.  It then
communicates with the persistence server to aggregate a sample of cluster models and produce a semi-global model, which is
sent back to the clients for the next round of local training.  Similarly, the aggregator can be started with the following
command:

.. code-block::
    :linenos:
	stadle aggregator --config_file /path/to/config/file.json
Specific parameters can be set using command line arguments - refer to :ref:`Config File Documentation` for details
on the config file parameters, and run ``stadle aggregator --help`` to see the accepted command line arguments for the aggregator.

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

The BasicClient object can then be created.  The configuration information of the
BasicClient can be set by passing a config file path through the constructor

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


Integration with IntegratedClient
---------------------------------

Using the ``IntegratedClient`` allows for the management of the local training process to be passed to STADLE,
as opposed to the more hands-off approach taken by the BasicClient.  As a result, the integration process to
be able to use the 	IntegratedClient is slightly more in-depth.

This process can be broken down into x steps:

1. Create and properly configure the IntegratedClient object
2. Construct a training, cross-validation, and test function (segmentation of the local training process)
   and pass the functions to the IntegratedClient
3. Construct a termination function to determine when to stop the FL process
4. Connect the IntegratedClient to STADLE and start the entire FL process

Similarly to the BasicClient, the CIFAR-10 example will be used to show how these steps can be implemented.

Step 1: Create/Configure IntegratedClient
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

IntegratedClient is imported from the ``stadle`` library; this is done with

.. code-block::
	:linenos:

	from stadle import IntegratedClient

The BasicClient object can then be created and configured like the BasicClient:

.. code-block::
	:linenos:

	client_config_path = r"/path/to/config/file.json"
	stadle_client = IntegratedClient(config_file=client_config_path)

Alternatively, specific config parameter values can be set directly with the
IntegratedClient constructor.  Information on the config file and these parameters,
as well as all subsequent function calls, can be found at :ref:`Client API Documentation`.

Step 2: Construct Local Training Functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When STADLE manages the local training part of the FL process, it works with abstracted versions of the training,
cross-validation, and test functions.  As a result, any specific implementations of these functions must match
these abstractions in format.  The following are template implementations of the functions in question:

Train Function:

.. code-block::
	:linenos:

	def train(model, data, **kwargs):
		# Use data to locally train model
		# kwargs used to pass general parameters to function

	    return locally_trained_model, average_training_loss

Cross-Validation Function:

.. code-block::
	:linenos:

	def cross_validate(model, data, **kwargs):
		# Use data to compute accuracy or other performance metric (validation set)
		# kwargs used to pass general parameters to function

	    return acc, ave_loss

Test Function:

.. code-block::
	:linenos:

	def test(model, data, **kwargs):
		# Use data to compute accuracy or other performance metric (test set)
		# kwargs used to pass general parameters to function

	    return acc, ave_loss


The IntegratedClient will go through the following steps to fulfill the agent-side role in FL:

1. Check termination function output, continue if false
2. Receive previous round aggregated model from aggregator
3. Run cross_validate function on aggregated model
4. Run train function to train model locally
5. Run cross_validate function on locally-trained model
6. Send locally-trained model to aggregator

The CIFAR-10 local training example code can then be segmented into these functions in the following way:

Train Function (CIFAR-10):

.. code-block::
	:linenos:

	def train(model, data, **kwargs):
		lr = float(kwargs.get("lr")) if kwargs.get("lr") else 0.001
	    momentum = float(kwargs.get("momentum")) if kwargs.get("momentum") else 0.9
	    epochs = int(kwargs.get("epochs")) if kwargs.get("epochs") else 2
	    device = kwargs.get("device") if kwargs.get("device") else 'cpu'

	    model = model.to(device)

	    criterion = nn.CrossEntropyLoss()
	    optimizer = optim.SGD(model.parameters(), lr=lr,
	                          momentum=momentum, weight_decay=5e-4)
	    scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=200)

	    ave_loss = []

	    for epoch in range(epochs):  # loop over the dataset multiple times

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

	            train_loss += loss.item()
	            ave_loss.append(train_loss)
	            _, predicted = outputs.max(1)
	            total += targets.size(0)
	            correct += predicted.eq(targets).sum().item()

	    ave_loss = sum(ave_loss) / len(ave_loss)

	    model = model.to('cpu')

	    return model, ave_loss


Cross-Validation Function (CIFAR-10):

.. code-block::
	:linenos:

	def cross_validate(test_model, data, **kwargs):
	    device = kwargs.get("device") if kwargs.get("device") else 'cpu'

	    test_model = test_model.to(device)

	    correct = 0
	    total = 0
	    overall_accuracy = 0

	    with torch.no_grad():
	        for (inputs, targets) in data:
	            inputs, targets = inputs.to(device), targets.to(device)
	            # calculate outputs by running images through the network
	            outputs = test_model(inputs)
	            # the class with the highest energy is what we choose as prediction
	            _, predicted = torch.max(outputs.data, 1)
	            total += targets.size(0)
	            correct += (predicted == targets).sum().item()

	    overall_accuracy = 100 * correct / total
	    print('Accuracy of the network on the 10000 test images: %d %%' % (overall_accuracy))

	    # prepare to count predictions for each class
	    correct_pred = {classname: 0 for classname in classes}
	    total_pred = {classname: 0 for classname in classes}

	    with torch.no_grad():
	        for (inputs, targets) in data:
	            inputs, targets = inputs.to(device), targets.to(device)
	            outputs = test_model(inputs)
	            _, predictions = torch.max(outputs, 1)
	            # collect the correct predictions for each class
	            for target, prediction in zip(targets, predictions):
	                if prediction == target:
	                    correct_pred[classes[target]] += 1
	                total_pred[classes[target]] += 1

	    # print accuracy for each class
	    # Capture average accuracy across all classes
	    for classname, correct_count in correct_pred.items():
	        accuracy = 100 * float(correct_count) / total_pred[classname]
	        print("Accuracy for class {:5s} is: {:.1f} %".format(classname,
	                                                             accuracy))
	    return overall_accuracy, 0

We can use the same implementation for the test function in this case, simply changing the dataset passed to
the function.

Test Function (CIFAR-10):

.. code-block::
	:linenos:

	def test(test_model, data, **kwargs):
	    device = kwargs.get("device") if kwargs.get("device") else 'cpu'

	    test_model = test_model.to(device)

	    correct = 0
	    total = 0
	    overall_accuracy = 0

	    with torch.no_grad():
	        for (inputs, targets) in data:
	            inputs, targets = inputs.to(device), targets.to(device)
	            # calculate outputs by running images through the network
	            outputs = test_model(inputs)
	            # the class with the highest energy is what we choose as prediction
	            _, predicted = torch.max(outputs.data, 1)
	            total += targets.size(0)
	            correct += (predicted == targets).sum().item()

	    overall_accuracy = 100 * correct / total
	    print('Accuracy of the network on the 10000 test images: %d %%' % (overall_accuracy))

	    # prepare to count predictions for each class
	    correct_pred = {classname: 0 for classname in classes}
	    total_pred = {classname: 0 for classname in classes}

	    with torch.no_grad():
	        for (inputs, targets) in data:
	            inputs, targets = inputs.to(device), targets.to(device)
	            outputs = test_model(inputs)
	            _, predictions = torch.max(outputs, 1)
	            # collect the correct predictions for each class
	            for target, prediction in zip(targets, predictions):
	                if prediction == target:
	                    correct_pred[classes[target]] += 1
	                total_pred[classes[target]] += 1

	    # print accuracy for each class
	    # Capture average accuracy across all classes
	    for classname, correct_count in correct_pred.items():
	        accuracy = 100 * float(correct_count) / total_pred[classname]
	        print("Accuracy for class {:5s} is: {:.1f} %".format(classname,
	                                                             accuracy))
	    return overall_accuracy, 0


Step 3: Construct Termination Function
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The termination function is a user-defined function that controls when an agent exits a FL process.
The function is run by the agent at the beginning of each round, and the agent exits if the function
retuns True.

One simple termination function is to return True after a certain number of rounds has passed; the following
is an implementation of such a function:

.. code-block::
	:linenos:

	def judge_termination(**kwargs) -> bool:
	    """
	    Decide if it finishes training process and exits from FL platform
	    :param training_count: int - the number of training done
	    :param sg_arrival_count: int - the number of times it received SG models
	    :return: bool - True if it continues the training loop; False if it stops
	    """

	    keep_running = True
	    client = kwargs.get('client')
	    current_fl_round = client.federated_training_round

	    if current_fl_round >= int(kwargs.get("round_to_exit")):
	        keep_running = False
	        client.stop_model_exchange_routine()
	    return keep_running


Step 4: Setup, Connect IntegratedClient to STADLE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following is example code to set up the IntegratedClient with the previously defined functions and
start the FL process:

.. code-block::
	:linenos:
	parser = argparse.ArgumentParser(description='STADLE CIFAR10 Training')
    parser.add_argument('--lr', default=0.1, type=float, help='learning rate')
    parser.add_argument('--lt_epochs', default=3)
	args = parser.parse_args()
    device = 'cuda'
	model = VGG('VGG16')

Read in learning rate and number of local training epochs from command line arguments, set training device
and define model to be trained.

.. code-block::
	:linenos:
	trainset = torchvision.datasets.CIFAR10(
		root='data', train=True, download=True, transform=transform_train)
	trainloader = torch.utils.data.DataLoader(
		trainset, batch_size=64, shuffle=True, num_workers=2)
	testset = torchvision.datasets.CIFAR10(
		root='data', train=False, download=True, transform=transform_test)
	testloader = torch.utils.data.DataLoader(
		testset, batch_size=64, shuffle=False, num_workers=2)

Use the same CIFAR-10 datasets as the local training example

.. code-block::
	:linenos:

	stadle_client.set_termination_function(judge_termination, round_to_exit=20, client=stadle_client)
    stadle_client.set_training_function(train, trainloader, lr=args.lr, epochs=args.lt_epochs, device=device, agent_name=args.agent_name)
    stadle_client.set_cross_validation_function(cross_validate, testloader, device=device)
    stadle_client.set_testing_function(test, testloader)

Pass functions to IntegratedClient for use in internal training loop

.. code-block::
	:linenos:
    stadle_client.set_bm_obj(model)
    stadle_client.start()

Set the container model for the client, then start the agent FL process


Running Client-Side STADLE Components
**************************************

After starting the requisite server-side STADLE components, there is one final step that must be run to fully
initialize an FL process with STADLE and prepare for agent connections.  The component responsible for this
is called the *admin agent* - its role in this case is to send the model structure and information to the
persistence server for use in converting between specific model frameworks and the framework-agnostic model
representation used by STADLE.  The following is example admin agent code for the CIFAR-10 example:

.. code-block::
	:linenos:

	from stadle import AdminAgent
	from stadle import BaseModelConvFormat
	from stadle.lib.entity.model import BaseModel
	from stadle.lib.util import admin_arg_parser

	from vgg import VGG

This section imports the required objects from STADLE, as well as a function for reading command line arguments
and the VGG model.  The BaseModel object acts as a container for information on the model being trained with
STADLE, and is passed to the AdminAgent to be sent to the persistence server.

.. code-block::
	:linenos:

	base_model = BaseModel("PyTorch-CIFAR10-Model", VGG('VGG16'), BaseModelConvFormat.pytorch_format)

The specific BaseModel object is then created with the VGG16 model structure and information.

.. code-block::
	:linenos:

	args = admin_arg_parser()
    admin_agent = AdminAgent(config_file=args.config_path, simulation_flag=args.simulation,
                             aggregator_ip_address=args.ip_address, reg_socket=args.reg_port,
                             exch_socket=args.exch_port, model_path=args.model_path, base_model=base_model,
                             agent_running=args.agent_running)
    admin_agent.preload()
    admin_agent.initialize()

The command line arguments are parsed and used to create the AdminAgent object, along with the base model.
The preload function prepares the base model to be sent (converting to agnostic representation internally)
and the initialize function sends the base model information, preparing all of the aggregators to connect
to agents by extension.

After the admin agent is run, the main agent client-side code can freely be run.  In summary, the order to run
components is as follows:

1. Start persistence server
2. Start aggregator(s)
3. Run admin agent (only once)
4. Run agent(s) - client-side code
