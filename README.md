# easy-tensorflow

**easy-tensorflow** provides an easy way to train and evaluate TensorFlow 
models. The goal of this project is not to build an off-the-shelf tool for 
industrial or commercial purposes but to *simplify* programming with TensorFlow
API. With a standardized pipeline, users do not have to worry about extra 
book-keeping steps but can focus entirely on input pre-processing and model 
engineering. 

We make use of [TF-Slim](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/contrib/slim) to make the code concise and flexible.  

##0. Cifar-10 demo

~~~~
$ cd cifar10
~~~~

Download Cifar-10 Python version:

~~~~
$ wget https://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz
~~~~

Extract downloaded file:

~~~~
$ tar -zxvf cifar-10-python.tar.gz
~~~~

Convert raw data to tf.Example protos:

~~~~
$ python convert_to_tf_example_cifar10.py --input_data_dir=cifar-10-batches-py --output_data_dir=data
~~~~

Run training:

~~~~
$ python train_cifar10.py --logdir=/tmp/cifar10
~~~~

Run evaluation (can be run simultaneously with training):

~~~~
$ python eval_cifar10.py --logdir=/tmp/cifar10
~~~~

Monitor stats on TensorBoard:

~~~~
$ tensorboard --logdir=/tmp/cifar10
~~~~

Then go to any browser (e.g. Chrome)'s address bar and type "0.0.0.0:6006" to 
visualize TensorBoard. This is TensorBoard after 30k training iterations.

![](http://khanhxnguyen.com/wp-content/uploads/2016/09/Selection_105.png)

##1. Code structure (Cifar-10 example)

          TrainEvalBase                 ModelBase             InputReaderBase
             /     \                        |                       |
            /       \                       |                       |
           /         \                      |                       |
       Trainer     Evaluator            ModelCifar10         InputReaderCifar10


+ **TrainEvalBase**: base (abstract) class for training and evaluating. 

+ **Trainer**: a subclass of `TrainEvalBase`, trains a model with provided data 
and loss function. 

+ **Evaluator**: a subclass of `TrainEvalBase`, computes evaluating metrics on a
trained model.

+ **ModelBase**: base class for specifying a model architecture. Two methods are 
required to implemented by any subclass: `arg_scope`, configurations of the 
model's layers, and `compute`, computing outputs of the model from a batch of 
input examples.

+ **ModelCifar10**: a subclass of `ModelBase`, implements `arg_scope` and 
`compute`.

+ **InputReaderBase**: base (abstract) class for reading input, requires the 
method `read_input` to be implemented by any subclass. 

+ **InputReaderCifar10**: a subclass of `InputReaderBase`, implements 
`read_input` and an input preprocessing method. 

##2. Define a new model (Cifar-10 example):

To define a new model, we need to create 5 core files (see the `cifar10` 
directory):

+ **convert_to_tf_example_cifar10.py**: convert Cifar-10 data in Python version 
to tf.Example protos. 

+ **input_reader_cifar10.py**: reads examples from files containing tf.Example 
protos (records) and makes a batch of examples. 

+ **model_cifar10.py**: specifies the model architecture. It implements 
`arg_scope` to configure model's layers (e.g. weight decay, regularize 
techniques, activation functions. etc.) and `compute` to arrange model's layers
(e.g. which layers follow which layers) in order to return a batch of outputs 
from a batch of inputs. 

+ **train_cifar10.py**: runs training. It creates a `Trainer` object, specifying 
a training model object, loss function, computation graph, input reader object. 
Then it invokes the `run` method of the `Trainer` object to start training. 

+ **eval_cifar10.py**: runs evaluating. It creates an `Evaluator` object, 
specifying an evaluating model object, loss function, computation graph, input 
reader object. Similarly to training, it invokes the `run` method of the 
`Evaluator` object to start evaluating. **NOTE**: an evaluating object is 
created by setting the `is_training` parameter of `ModelBase` to False. 

*Although there seem to be a lot of files, the amount of code in each file is 
minimal. For most files, users simply have to copy them to a new directory and 
make tiny changes. The majority of modifications goes into 
`convert_to_tf_example_cifar10.py`, `model_cifar10.py` and 
`input_reader_cifar10.py`.*

##3. Common TensorFlow concepts:

**tf.Example proto**: a feature vector and can be considered as a Python dict. Each 
element is a pair of (key, value). The key is the feature name. The value is 
either a list of type bytes (string), int64, or float. See:
 https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/example/example.proto
for more details. 

For example, if data are labeled, then each proto has two features: one for the 
observation (e.g. image) and one for the label. 




