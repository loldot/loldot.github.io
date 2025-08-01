---
layout: post
title: Writing a neural network from scratch in C (part 2)
author: Lorentz Vedeler
date: 2025-07-30
excerpt_separator: <!--end_excerpt-->
tags:   
    - ML
---

This post is part 2 in a series.

- [Part 1: Linear regression](/2025/07/28/neural-net-linear/)
- Part 2: Neural networks

In this part we will take a look at how perceptrons and neural networks imitate the human brain.
<!--end_excerpt-->

### Perceptrons

The perceptron is an artificial neuron and is the most basic building block in neural networks. Similar to how a neuron receives signals and "fires" when triggered, a perceptron receives inputs and may or may not trigger an output signal.

![A diagram of a perceptron](/assets/imgs/perceptron.svg)

The perceptron consists of a vector $$\vec{w}$$ of weights, a scalar $$b$$ for bias and receives inputs in a vector $$\vec{x}$$. It also requires an activation function that will determine if the inputs should trigger a signal or not.
The most important property of an activation function is that it should not be ambiguous whether it has fired or not. There are many such functions that are commonly used. [This table](https://en.wikipedia.org/wiki/Activation_function#Table_of_activation_functions) on wikipedia contains common functions and uesful properties. In our neural net, we will use the hyperbolic tangent function or tanh as our activation function, we will also need to know its derivative, $$1-tanh^2(x)$$, which we can find in the table.

```c
const double activation(double x)
{
    return tanh(x);
}

// derivative of tanh = (1-tanh²(x))
const double activation_prime(double tanh)
{
    return (1 - tanh) * (1 + tanh);
}
```

The formula to compute the input for the activation function in a perceptron is:

$$
\vec{w}\cdot\vec{x} + b
$$

This should look familiar if you read the previous post in this series; it looks almost exactly like the linear function we found the weight and bias for with our linear regression model. Perhaps we can use gradient descent to find the correct weights and biases?

### Neural nets

In our brain and nervous system, neurons are interconnected and can even form new connections. In our neural network, we will simulate this by using an input layer, two hidden layers, and finally an output layer. When we train our neural network by optimizing the weights and biases, we simulate the formation of new connections between the perceptrons.

In the image below, you can see the outline of a neural net with an input layer, a hidden layer and an output layer. Every perceptron in the hidden layer is connected to every input. Every perceptron signals its output to each of the perceptrons in the output layer. Propagating signals through layers in this way is called feeding forward.

![A diagram depicting the basic structure of a neural net](/assets/imgs/neuralnet.svg)

```c
// Input layer
float weights_0[input_size][hidden_size];
float bias_0[hidden_size];

// Hidden layer 1
float weights_1[hidden_size][hidden_size];
float bias_1[hidden_size];

// Hidden layer 2
float weights_2[hidden_size][output_size];
float bias_2[output_size];

// Feed forward
void forward(
    const int m,
    const float input[m],
    const int n,
    float output[n],
    const float weights[m][n],
    const float bias[n])
{
    for (int i = 0; i < n; i++)
    {
        float sum = .0f;
        for (int j = 0; j < m; j++)
        {
            sum += input[j] * weights[j][i];
        }
        output[i] = activation(sum + bias[i]);
    }
}
```

### Softmax

Our neural network will attempt to classify digits in the MNIST handwritten digits dataset. Our output layer will have 10 neurons, each representing a digit from 0-9.
We chose this representation and not just a single neuron which outputs the integer value of the digit for a reason. Some digits can be very similar when written by hand, for example it can be hard to tell 1 apart from 7, 4 from 9 etc. To accomodate this, we want the neural net to indicate how certain it is that an image contains a given digit. We want to output a probability distribution; if an image contains a perfectly written 8, the neuron representing 8 should be close to 1 or 100%, while a poorly writting 7, might be more close to 50-50 between 1 and 7. We can use a function called softmax to convert any range of numbers into a probability distribution that sums up to one. The softmax function is a bit different than other activation functions, in that it requires the output of all neurons in the layer to be calculated, not just a single neuron. The formula for softmax is:

$$
\text{softmax}(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{K} e^{z_j}}
$$

where $$z_i$$ is the input to the $$i$$-th neuron and $$K$$ is the total number of neurons in the output layer.

```c
void softmax(const int m, float input[m])
{
    float sum = 0.0f;
    for (int i = 0; i < m; i++)
    {
        sum += exp(input[i]);
    }

    for (int i = 0; i < m; i++)
    {
        input[i] = exp(input[i]) / sum;
    }
}
```
