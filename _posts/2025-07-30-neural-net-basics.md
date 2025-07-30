---
layout: post
title: Writing a neural network from scratch in C (part 2)
author: Lorentz Vedeler
date: 2025-07-30
tags:   
    - ML
---

This post is part 2 in a series.

- [Part 1: Linear regression](https://lorentzvedeler.com/2025/07/28/neural-net-linear/)
- Part 2: Neural networks

## Perceptrons

The perceptron is an artificial neuron and is the most basic building block in neural networks. Similar to how a neuron receives signals and "fires" when triggered, a perceptron receives inputs and may or may not trigger an output signal.

![A diagram of a perceptron](/assets/imgs/perceptron.svg)

The perceptron consists of a vector $$\vec{w}$$ of weights, a scalar $$b$$ for bias and receives inputs in a vector $$\vec{x}$$. It also requires an activation function that will determine if the inputs should trigger a signal or not.
The most important property of an activation function is that it should not be ambiguous whether it has fired or not. There are many such functions that are commonly used. [This table](https://en.wikipedia.org/wiki/Activation_function#Table_of_activation_functions) on wikipedia contains common functions and uesful properties. In our neural net, we will use the hyperbolic tangent function or tanh as our activation function, we will also need to remember its derivative, $$1-(tanh(x))^2$$,which we can find in the table.

The formula to compute the input for the activation function in a perceptron is:

$$
\vec{w}\cdot\vec{x} + b
$$

This should look familiar if you read the previous post in this series; it looks almost exactly like the linear function we found the weight and bias for with our linear regression model.

In our brain and nervous system, neurons are interconnected and can even form new connections. In our neural network, we will simulate this by using an input layer, two hidden layers, and finally an output layer. When we train our neural network by optimizing the weights and biases, we simulate the formation of new connections between the perceptrons.

In the image below, you can see the outline of a neural net with an input layer, a hidden layer and an output layer. Every perceptron in the hidden layer is connected to every input. Every perceptron signals its output to each of the perceptrons in the output layer. Propagating signals through layers in this way is called feeding forward.

![A diagram depicting the basic structure of a neural net](/assets/imgs/neuralnet.svg)

```c

// Input layer
double weights_0[input_size][hidden_size];
double bias_0[hidden_size];

// Hidden layer 1
double weights_1[hidden_size][hidden_size];
double bias_1[hidden_size];

// Hidden layer 2
double weights_2[hidden_size][output_size];
double bias_2[output_size];

const double activation(double x)
{
    return tanh(x);
}

// derivative of tanh = (1-tanh²(x))
const double activation_prime(double tanh)
{
    return (1 - tanh) * (1 + tanh);
}

// Feed forward
void forward(
    const int m,
    const double input[m],
    const int n,
    double output[n],
    const double weights[m][n],
    const double bias[n])
{
    for (int i = 0; i < n; i++)
    {
        double sum = .0f;
        for (int j = 0; j < m; j++)
        {
            sum += input[j] * weights[j][i];
        }
        output[i] = activation(sum + bias[i]);
    }
}

void forward_softmax(
    const int m,
    const double input[m],
    const int n,
    double output[n],
    const double weights[m][n],
    const double bias[n])
{
    double total = 0.0f;
    for (int i = 0; i < n; i++)
    {
        double sum = .0f;

        for (int j = 0; j < m; j++)
        {
            sum += input[j] * weights[j][i];
        }
        output[i] = exp(sum + bias[i]);
        total += output[i];
    }

    for (int i = 0; i < n; i++)
    {
        output[i] /= total;
    }
}

void predict(
    double input[input_size],
    double hidden[hidden_size],
    double hidden2[hidden_size],
    double output[output_size])
{
    forward(
        input_size, input,
        hidden_size, hidden,
        weights_0, bias_0);
    forward(
        hidden_size, hidden,
        hidden_size, hidden2,
        weights_1, bias_1);
    forward_softmax(
        hidden_size, hidden2,
        output_size, output,
        weights_2, bias_2);
}
```
