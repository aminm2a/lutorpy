# Lutorpy

Lutorpy is a libray built for deep learning with torch in python,  by a two-way bridge between Python/Numpy and Lua/Torch, you can use any Torch modules(nn, rnn etc.) in python, and easily convert variables(array and tensor) between torch and numpy.

# Features

* import any lua/torch module to python and use it like python moduels
* use lua objects directly in python, conversion are done automatically
* create torch tensor from numpy array with `torch.fromNumpyArray(arr)`
* use `tensor.asNumpyarray()` to convert a torch tensor to a numpy array with memory sharing
* support zero-base indexing (lua uses 1-based indexing)
* automatic prepending self to function by `"._"` syntax, easily convert `":"` operator in lua to python

#### * Interested in Lutorpy project? Please let us know by giving a star.

# Convert from Lua to Python/Lutorpy
```lua
-- lua code                             # python code (with lutorpy)
--                                      import lutorpy as lua
require "nn"                    ===>    require("nn")
model = nn.Sequential()         ===>    model = nn.Sequential()
-- use ":" to access add        ===>    # use "._" to access add
model:add(nn.Linear(10, 3))     ===>    model._add(nn.Linear(10, 3))
--                                      import numpy as np
x = torch.Tensor(10):zero()     ===>    arr = np.zeros(10)
-- torch style(painful?)        ===>    # numpy style(elegent?) 
x:narrow(1, 2, 6):fill(1)       ===>    arr[1:7] = 1
--                                      # convert numpy array to a torch tensor
--                                      x = torch.fromNumpyArray(arr)
--                                      # or you can still use torch style
x:narrow(1, 7, 2):fill(2)       ===>    x._narrow(1, 7, 2)._fill(2)
-- 1-based index                ===>    # 0-based index
x[10] = 3                       ===>    x[9] = 3
y = model:forward(x)            ===>    y = model._forward(x)
--                                      # you can convert y to a numpy array
--                                      yArr = y.asNumpyArray()
```

# Quick Start

## basic usage
``` python
import lutorpy as lua
import numpy as np

## use require("MODULE") to import lua modules
require("nn")

## run lua code in python with minimal modification:  replace ":" to "._"
t = torch.DoubleTensor(10,3)
print(t._size()) # the corresponding lua version is t:size()

## or, you can use numpy array
xn = np.random.randn(100)
## convert the numpy array into torch tensor
xt = torch.fromNumpyArray(xn)

## convert torch tensor to numpy array
### Note: the underlying object are sharing the same memory, so the conversion is instant
arr = xt.asNumpyArray()
print(arr.shape)
```
## example 1: multi-layer perception
``` python
## minimal example of multi-layer perception(without training code)
mlp = nn.Sequential()
mlp._add(nn.Linear(100, 30))
mlp._add(nn.Tanh())
mlp._add(nn.Linear(30, 10))

## generate a numpy array and convert it to torch tensor
xn = np.random.randn(100)
xt = torch.fromNumpyArray(xn)
## process with the neural network
yt = mlp._forward(xt)
print(yt)

## or for example, you can plot your result with matplotlib
yn = yt.asNumpyArray()
import matplotlib.pyplot as plt
plt.plot(yn)
```
## example 2: load pre-trained model with torch and apply it
```python
import numpy as np
import lutorpy as lua

# load your torch file(for example xx.t7 containing the model and weights)
model = torch.load('PATH TO YOUR MODEL FILE')

# generate your input data with numpy
arr = np.random.randn(100)

# convert your numpy array into torch tensor
x = torch.fromNumpyArray(arr)

# apply model forward method with "._" syntax(which is equivalent to ":" in lua)
yt = model._forward(x)

# convert to numpy array
yn = yt.asNumpyArray()
```

You can also have a look at the step-by-step tutorial and more complete example.

# Installation
You need to install torch before you start (only LuaJIT engine is supported for now)
``` bash
# in a terminal, run the commands WITHOUT sudo
git clone https://github.com/torch/distro.git ~/torch --recursive
cd ~/torch
./clean.sh
bash install-deps
./install.sh
```
Then, you can use luarocks to install torch/lua modules
``` bash
luarocks install nn
```
If you don't have numpy installed, install it by pip
``` bash
pip install numpy
```
Now, we are ready to install lutorpy, just use pip to install the released version:
``` bash
pip install lutorpy
```

Or, install from git repository:
``` bash
git clone https://github.com/imodpasteur/lutorpy.git
cd lutorpy
sudo python setup.py install
```
#### note that for now, lutorpy is only tested on ubuntu, please report issue if you encountered error.


# Step-by-step tutorial

## import lutorpy

``` python
import lutorpy as lua
# setup runtime and use zero-based index(optional, enabled by default)
lua.LuaRuntime(zero_based_index=True)

### note: zero-based index will only work getter operator such as "t[0]", for torch function like narrow, you still need 1-based indexing.
```
## hello world

``` python
lua.execute(' greeting = "hello world" ')
print(greeting)
```

###  Alternatively you could also switch back to one-based indexing

Note that if you do this, all the following code should change acorrdingly.

```
lua.LuaRuntime(zero_based_index=False)
```

## execute lua code

``` python
a = lua.eval(' {11, 22} ') # define a lua table with two elements
print(a[0])

```

## use torch
``` python
require("torch")
z = torch.Tensor(4,5,6,2)
print(torch.isTensor(z))
s = torch.LongStorage(6)
print(torch.type(s))
```

## convert torch tensor to numpy array


``` python
require('torch')
t = torch.randn(10,10)
print(t)
arr = t.asNumpyArray()
print(arr)

```
                                
## convert numpy array to torch tensor

Note: both torch tensor and cuda tensor are supported.

``` python
arr = np.random.randn(100)
print(arr)
t = torch.fromNumpyArray(arr)
print(t)

```

## convert to/from cudaTensor
``` python
require('cutorch')
cudat = torch.CudaTensor(10,10)
#convert cudaTensor to numpy array
arr = cudat.asNumpyArray()
print(arr)

arr = np.random.randn(100)
print(arr)
t = torch.fromNumpyArray(arr)
cudat = t._cuda()
print(cudat)
```

## load image and use nn module
``` python
require("image")
img_rgb = image.lena()
print(img_rgb.size(img_rgb))
img = image.rgb2y(img_rgb)
print(img.size(img))

# use SpatialConvolution from nn to process the image
require("nn")
n = nn.SpatialConvolution(1,16,12,12)
res = n.forward(n, img)
print(res.size(res))

```

## build a simple model

``` python
mlp = nn.Sequential()
module = nn.Linear(10, 5)
mlp.add(mlp, module)
print(module.weight)
print(module.bias)
print(module.gradWeight)
print(module.gradBias)
x = torch.Tensor(10) #10 inputs
# pass self to the function
y = mlp.forward(mlp, x)
print(y)
```

## prepending 'self' as the first argument automatically
In lua, we use syntax like 'mlp:add(module)' to use a function without pass self to the function. But in python, it's done by default, there are two ways to prepend 'self' to a lua function in lutorpy.

The first way is inline prepending by add '_' to before any function name, then it will try to return a prepended version of the function:
``` python
mlp = nn.Sequential()
module = nn.Linear(10, 5)

# lua style
mlp.add(mlp, module)

# inline prepending
mlp._add(module) # equaliant to mlp:add(module) in lua
```

## build another model and training it

Train a model to perform XOR operation (see [this torch tutorial](https://github.com/torch/nn/blob/master/doc/training.md)).

``` python
require("nn")
mlp = nn.Sequential()
mlp._add(nn.Linear(2, 20)) # 2 input nodes, 20 hidden nodes
mlp._add(nn.Tanh())
mlp._add(nn.Linear(20, 1)) # 1 output nodes
criterion = nn.MSECriterion() 
for i in range(2500):
    # random sample
    input= torch.randn(2)    # normally distributed example in 2d
    output= torch.Tensor(1)
    if input[0]*input[1] > 0:  # calculate label for XOR function
        output[0] = -1 # output[0] = -1
    else:
        output[0] = 1 # output[0] = 1
    # feed it to the neural network and the criterion
    criterion._forward(mlp._forward(input), output)
    # train over this example in 3 steps
    # (1) zero the accumulation of the gradients
    mlp._zeroGradParameters()
    # (2) accumulate gradients
    mlp._backward(input, criterion.backward(criterion, mlp.output, output))
    # (3) update parameters with a 0.01 learning rate
    mlp._updateParameters(0.01)

```

Train a model with nn trainer.
```python
require("nn")
mlp = nn.Sequential()
mlp._add(nn.Linear(2, 20)) # 2 input nodes, 20 hidden nodes
mlp._add(nn.Tanh())
mlp._add(nn.Linear(20, 1)) # 1 output nodes

class DataSet():
    def __init__(self):
        self.data = []
        for i in range(2500):
            # random sample
            input= torch.randn(2)    # normally distributed example in 2d
            output= torch.Tensor(1)
            if input[0]*input[1] > 0:  # calculate label for XOR function
                output[0] = -1 # output[0] = -1
            else:
                output[0] = 1 # output[0] = 1
            # here we need to add one empty column because lua index will start at 1
            self.data.append((i, input,output))
    def __getitem__(self, key):
        if key == 'size':
            return lambda x: len(self.data)
        return self.data[key-1]

dataset = DataSet()
criterion = nn.MSECriterion()  
trainer = nn.StochasticGradient(mlp, criterion)
trainer.learningRate = 0.01
trainer._train(dataset)
```
## test the model

``` python

x = torch.Tensor(2)
x[0] =  0.5; x[1] =  0.5; print(mlp._forward(x))
x[0] =  0.5; x[1] = -0.5; print(mlp._forward(x))
x[0] = -0.5; x[1] =  0.5; print(mlp._forward(x))
x[0] = -0.5; x[1] = -0.5; print(mlp._forward(x))

```

# Details of implementation

 * For applying tensor.asNumpyArray() method to a torch tensor, if the tensor is contiguous, the memory will be shared between numpy array and torch tensor, if the tensor is not contiguous, a contiguous clone of the tensor will be used, so the created numpy array won't share memory with the old tensor.
 
 * For torch.fromNumpyArray(), there will be no memory sharing between the numpy array and the tenosr created.


# More details about using lua in python
Lutorpy is built upon [lupa](https://github.com/scoder/lupa), there are more features provided by lupa could be also useful, please check it out.

# Notes
* python2 and python3 are both supported
* unfortunately, OSX is not supported so far

# Bug tracker

Have a bug? Please create an issue here on GitHub at https://github.com/imodpasteur/lutorpy/issues.

# Support lutorpy project
Like lutorpy project? It solved your problem? Let us know by giving lutorpy project a star, so we know how many people are interested, thank you.

# Acknowledge

This is a project inspired by [lunatic-python](https://github.com/bastibe/lunatic-python) and [lupa](https://github.com/scoder/lupa), and it's based on lupa.
