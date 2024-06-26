# Autodiff + neural network training in Haskell

This code shows how to implement and train a very simple feedforward neural net in Haskell.
The focus is on building everything from scratch, including autodiff and gradient descent / backprop training code. There are zero frameworks/libs used.

There aren't many examples online on coding neural nets in Haskell, and the ones I found either use [full-fledged frameworks](https://hackage.haskell.org/package/neural) or are too basic and don't apply autodiff + backprop. I wanted a simple yet complete implementation so here's this.

Everyone's seen something similar in python/numpy but trust me autodiff is way more elegant in Haskell.

<img src="./images/haskell.png" width="500px"></img>

I named this `microhaskell` after karpathy's [micrograd](https://github.com/karpathy/micrograd).

## Overview

Modules:

- **Types**: The core data types (`Matrix`, `Vector`, and `NeuralNet`).
- **AutoDiff**: Automatic differentiation. 
- **Initialization**: Initializing vectors/matrices with random values. Used in layer initialization.
- **LinearAlgebra**: Matrix/vector ops.
- **ActivationFunctions**: Sigmoid activation.
- **ForwardProp**: Handles the forward propagation logic through the net.
- **Backprop**: Backpropagation and param updates.
- **LossFunctions**: MSE loss and gradient.
- **Training**: Training loop and epoch update functions.
- **Main**: Initializes the network and trains it on the XOR dataset.

The neural net is the most basic FFN with just 2 layers (bare minimum to learn XOR).


## Automatic differentiation approach - Dual numbers

`Dual` numbers [simplify the gradient computation process](https://www.danielbrice.net/blog/automatic-differentiation-is-trivial-in-haskell/) since they store not only the value of a variable but also its derivative. More formally a `Dual` number is:

> a pair `(x, x')`, where `x` is the value of the function at some point and `x'` is the derivative of the function at that point.

meaning gradients can be computed automatically as arithmetic operations are performed.

At the moment in this implementation the actual derivatives in backprop are formulated explicitly (see [TODO](#todo))

### `Dual` type

```haskell
data Dual a = Dual a a
  deriving (Eq, Read, Show)
```

The `Dual` type holds two values of type `a`: The function's value and its derivative.

### `Num` typeclass

To enable arithmetic ops on dual numbers we create a typeclass instance of the `Num` typeclass for `Dual`.

```haskell
instance Num a => Num (Dual a) where
  (Dual u u') + (Dual v v') = Dual (u + v) (u' + v')
  (Dual u u') * (Dual v v') = Dual (u * v) (u' * v + u * v')
  (Dual u u') - (Dual v v') = Dual (u - v) (u' - v')
  abs (Dual u u') = Dual (abs u) (u' * signum u)
  signum (Dual u u') = Dual (signum u) 0
  fromInteger n = Dual (fromInteger n) 0
```

### Differentiation

The `differentiate` function computes the derivative of a single-variable function.

```haskell
differentiate :: Num a => (Dual a -> Dual c) -> a -> c
differentiate f x = getDualDerivative . f $ Dual x 1
```

Consider $f(x) = x^2 + 3x + 2$. To find its derivative at $x = 5$:

```haskell
f :: Num a => Dual a -> Dual a
f x = x ^ 2 + 3 * x + 2

let result = differentiate f 5
-- | result is 13
```


## How to run

You'll need to have Haskell and `cabal` installed:

### 1. Clone the repo
```
git clone https://github.com/tensor-fusion/microhaskell.git
cd microhaskell
```

### 2. Build the project
```
cabal build
```

### 1. Run the executable
```
cabal run
```

This will initialize the neural net, train it on the XOR dataset, and print the epochs and final predictions.

```
$ cabal run
Epoch: 1000     Loss: 0.24617433
Epoch: 2000     Loss: 0.01372039
Epoch: 3000     Loss: 0.00385795
Epoch: 4000     Loss: 0.00217716
Epoch: 5000     Loss: 0.00150440
Epoch: 6000     Loss: 0.00114530
Epoch: 7000     Loss: 0.00092287
Epoch: 8000     Loss: 0.00077192
Epoch: 9000     Loss: 0.00066290
Epoch: 10000    Loss: 0.00058057

Real            Predicted
0               0.03088127
1               0.99898587
1               0.99898348
0               0.03694156

Training complete
```

#### Disclaimer
The code could use some love. I'm no Haskell expert.
There might be bugs!


### Contributing

You are free to do whatever you want with this. If you spot bugs, want to experiment and/or change stuff, PRs are welcome! (although not guaranteed to be reviewed/accepted immediately). You can fork the repo or just copy the code and change it on your own as you please.

## TODO
- [ ] Leverage `Dual` diff for derivatives instead of explicitly formulating them.
- [ ] More complex nets/datasets (e.g. MNIST).
- [ ] Viz

### License

MIT