---
layout: post
title: "Pytorch - Simplest Working Example"
date: 2025-03-22 00:00:00 -0700
tags: python
---

This is a simple example of using Pytorch to build a single node neural network.

{% highlight python %}
import torch
import torch.nn as nn
import torch.optim as optim

# define a simple linear model
class LinearRegression(nn.Module):
    
    def __init__(self, input_size, output_size):
        super(LinearRegression, self).__init__()
        self.linear = nn.Linear(input_size, output_size)

    def forward(self, x):
        return self.linear(x)

# generate some dummy data
X = torch.tensor([[1.0], [2.0], [3.0], [4.0]], dtype=torch.float32)
y = torch.tensor([[2.0], [4.0], [6.0], [8.0]], dtype=torch.float32) # y = 2x

# instantiate the model, loss function, and optimizer
input_size = 1
output_size = 1
model = LinearRegression(input_size, output_size)

# mean squared error MSE loss
# the loss function tells the network how bad its predictions are
criterion = nn.MSELoss()

# stochastic gradient descent
# the optimizer is how the network learns to improve
optimizer = optim.SGD(model.parameters(), lr=0.01) 

# train the model
num_epochs = 100
for epoch in range(num_epochs):
    
    # forward pass
    outputs = model(X)
    loss = criterion(outputs, y)

    # backward and optimize
    optimizer.zero_grad() # clear gradients from previous iteration
    loss.backward() # compute gradients
    optimizer.step() # update weights

    if (epoch + 1) % 10 == 0:
        print(f'Epoch [{epoch+1}/{num_epochs}], Loss: {loss.item():.4f}')

# make a prediction
predicted = model(torch.tensor([[5.0]], dtype=torch.float32))
print(f'Prediction after training: {predicted.item():.4f}')
{% endhighlight %}

Sample output:

{% highlight console %}
Epoch [10/100], Loss: 0.3927
Epoch [20/100], Loss: 0.0174
Epoch [30/100], Loss: 0.0073
Epoch [40/100], Loss: 0.0066
Epoch [50/100], Loss: 0.0062
Epoch [60/100], Loss: 0.0059
Epoch [70/100], Loss: 0.0055
Epoch [80/100], Loss: 0.0052
Epoch [90/100], Loss: 0.0049
Epoch [100/100], Loss: 0.0046
Prediction after training: 9.8840
{% endhighlight %}
