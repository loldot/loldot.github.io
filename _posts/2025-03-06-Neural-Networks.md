
## Linear regression

An linear function is a function with a single variable whose graph looks like a straight line.
We can generalize the function as having a weight $w$ and a bias $b$. Changing the weight will
change the slope of the line, and changing the bias will change where the line intersects the 
y-axis. You might remember this equation from high school:

$f(x) = wx + b$

This simple equation can have predictive power if the data fits a line. For example assume we 
own a small ice cream shop and start measuring the temperature at midday and count our number 
of ice creams sold. After a few days of measurements, we have the following data:

![A scatter plot of the ice cream sales data](/assets/imgs/icecreamchart.png)

| Temperature (°C) |  20 |  19 |  25 |  15 |  11 |  24 |  30 |  25 |  16 |  17 |
| ---------------- | --: | --: | --: | --: | --: | --: | --: | --: | --: | --: |
| Ice creams sold  | 100 | 102 | 123 |  23 |   9 | 160 | 251 | 130 |  77 |	11 |

Our mission is to create a formula that takes the temperature and predicts the number of ice 
creams sold that day. To approximate this manually, we can choose to points that seem to go
somewhere in the middle of the data. Let's say we choose the data points (15°C, 23) and (30°C, 251). We can then calculate the weight: $w = \frac{251 - 23}{30 - 15} = 15.2$ This means that if the temperature increases by one degree, we can expect to sell about 15 more ice creams. We 
can now update our function to $f(x) = 15.2x + b$. To find a value for $b$ we can plot the values from our first data point. 

$
f(15) = 15.2 * 15 + b = 23 => b = 23 - 15.2 * 15 = -205
$

Our prediction function is now: $f(x)= 15.2x - 205$. Now if the weather forecast for tomorrow shows 27°C we can predict that we will sell $15.2*27 - 205 = 204$ ice creams. 

In our manual approximation, we just chose two "good" data points to find our line, and as it turns out, that was pretty hard to do even for a human. There is a better method to find our values for $w$ and $b$, called linear regression. Linear regression will find values that minimize the difference between our actual measurements and the line drawn with our weight and bias. 

I added a linear regression line to the previous Excel chart, and it appears we were slightly too optimistic with our weight; we can only expect our sales to increase by 12 ice creams when the temperature increases by one - not 15 as we predicted.

![A scatter plot of the ice cream sales data, including a trend line with the equation f(x)=12.026x-144.32](/assets/imgs/icecreamcharttrend.png)

Next we will learn a better method to find this line programmatically.

### Cost function

Suppose we just guess our initial values for $w$ and $b$. Then what we need is a function that can help us determine how well our line fits the data and a function to tell us how we can change $w$ and $b$ to improve the fit. A function that measures the difference between our measurements and our prediction is often referred to as the cost function of our model

One way such function is the mean squared error function. 

$$
\frac{1}{m} \sum_{i=1}^m{(y_i-\hat{y}_i)^2}
$$

- $m$ is the total number of observations 
- $y_i$ is the observed number of ice creams sold for data point i 
- $\hat{y}_i$ is the predicted number of ice creams sold for data point i.

```C
// The function we use to predict our number of ice creams sold given a temperature x
// It will become clear later why we call this function "forward"
float forward(float x)
{
    return w * x + b;
}


float cost(const size_t m, const float[m] x, const float[m] y)
{
    float cost = 0.0f;

    for (size_t i = 0; i < m; i++)
    {
        float output = forward(x[i]);
        float error = y[i] - output;

        cost += error * error;
    }

    return cost / m;
}

```

### Gradient Descent

## Neural networks

### Feeding Forward