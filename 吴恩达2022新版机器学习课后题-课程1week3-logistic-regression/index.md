# 吴恩达2022新版机器学习课后题-课程1week3-Logistic Regression

<script type="text/javascript" src="/js/src/bai.js"></script>

## 不带正则的
### 问题描述
Suppose that you are the administrator of a university department and you want to determine each applicant’s chance of admission based on their results on two exams.

You have historical data from previous applicants that you can use as a training set for logistic regression.
For each training example, you have the applicant’s scores on two exams and the admissions decision.
Your task is to build a classification model that estimates an applicant’s probability of admission based on the scores from those two exams.

### sigmoid函数
the model:
$$ f_{\mathbf{w},b}(x) = g(\mathbf{w}\cdot \mathbf{x} + b)$$
where function $g$ is the sigmoid function. The sigmoid function is defined as:

$$g(z) = \frac{1}{1+e^{-z}}$$

```python
# UNQ_C1
# GRADED FUNCTION: sigmoid

def sigmoid(z):
    """
    Compute the sigmoid of z

    Args:
        z (ndarray): A scalar, numpy array of any size.

    Returns:
        g (ndarray): sigmoid(z), with the same shape as z
         
    """
          
    ### START CODE HERE ### 
    g = 1/(1 + np.exp(-z))

    
    ### END SOLUTION ###  
    
    return g

print ("sigmoid(0) = " + str(sigmoid(0)))
```


### cost function
cost function的式子：
$$ J(\mathbf{w},b) = \frac{1}{m}\sum_{i=0}^{m-1} \left[ loss(f_{\mathbf{w},b}(\mathbf{x}^{(i)}), y^{(i)}) \right] \tag{1}$$


其中单个数据集上的Loss为：


$$loss(f_{\mathbf{w},b}(\mathbf{x}^{(i)}), y^{(i)}) = (-y^{(i)} \log\left(f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right) - \left( 1 - y^{(i)}\right) \log \left( 1 - f_{\mathbf{w},b}\left( \mathbf{x}^{(i)} \right) \right) \tag{2}$$




计算代价函数：
```python
# UNQ_C2
# GRADED FUNCTION: compute_cost
def compute_cost(X, y, w, b, lambda_= 1):
    """
    Computes the cost over all examples
    Args:
      X : (ndarray Shape (m,n)) data, m examples by n features
      y : (array_like Shape (m,)) target value 
      w : (array_like Shape (n,)) Values of parameters of the model      
      b : scalar Values of bias parameter of the model
      lambda_: unused placeholder
    Returns:
      total_cost: (scalar)         cost 
    """

    m, n = X.shape
    
    ### START CODE HERE ###
    loss_sum = 0 

     # Loop over each training example
    for i in range(m): 
         # First calculate z_wb = w[0]*X[i][0]+...+w[n-1]*X[i][n-1]+b
        z_wb = 0 
         # Loop over each feature
        for j in range(n): 
             # Add the corresponding term to z_wb
            z_wb_ij = w[j]*X[i][j]# Your code here to calculate w[j] * X[i][j]
            z_wb += z_wb_ij # equivalent to z_wb = z_wb + z_wb_ij
         # Add the bias term to z_wb
        z_wb += b # equivalent to z_wb = z_wb + b

        f_wb = sigmoid(z_wb)# Your code here to calculate prediction f_wb for a training example
        # ?
        loss =   -y[i] * np.log(f_wb) - (1 - y[i]) * np.log(1 - f_wb)# Your code here to calculate loss for a training example

        loss_sum += loss # equivalent to loss_sum = loss_sum + loss

    total_cost = (1 / m) * loss_sum  
    ### END CODE HERE ### 

    return total_cost
```





### 梯度下降

![20220711145459](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220711145459.png)

```python
def compute_gradient(X, y, w, b, lambda_=None):
    m, n = X.shape
    dj_dw = np.zeros(w.shape)
    dj_db = 0.

    ### START CODE HERE ###
    err = 0.
    for i in range(m):
        # Calculate f_wb (exactly as you did in the compute_cost function above)
        z_wb = 0
        # Loop over each feature
        for j in range(n):
            # Add the corresponding term to z_wb
            z_wb_ij = X[i, j] * w[j]
            z_wb += z_wb_ij

        # Add bias term
        z_wb += b

        # Calculate the prediction from the model
        f_wb = sigmoid(z_wb)

        # Calculate the  gradient for b from this example
        dj_db_i =  f_wb - y[i]# Your code here to calculate the error

        # add that to dj_db
        dj_db += dj_db_i

        # get dj_dw for each attribute
        for j in range(n):
            # You code here to calculate the gradient from the i-th example for j-th attribute
            dj_dw_ij =(f_wb - y[i])* X[i][j]
            dj_dw[j] += dj_dw_ij

    # divide dj_db and dj_dw by total number of examples
    dj_dw = dj_dw / m
    dj_db = dj_db / m
    ### END CODE HERE ###

    return dj_db, dj_dw

```

梯度下降：
```python
def gradient_descent(X, y, w_in, b_in, cost_function, gradient_function, alpha, num_iters, lambda_): 
    """
    Performs batch gradient descent to learn theta. Updates theta by taking 
    num_iters gradient steps with learning rate alpha
    
    Args:
      X :    (array_like Shape (m, n)
      y :    (array_like Shape (m,))
      w_in : (array_like Shape (n,))  Initial values of parameters of the model
      b_in : (scalar)                 Initial value of parameter of the model
      cost_function:                  function to compute cost
      alpha : (float)                 Learning rate
      num_iters : (int)               number of iterations to run gradient descent
      lambda_ (scalar, float)         regularization constant
      
    Returns:
      w : (array_like Shape (n,)) Updated values of parameters of the model after
          running gradient descent
      b : (scalar)                Updated value of parameter of the model after
          running gradient descent
    """
    
    # number of training examples
    m = len(X)
    
    # An array to store cost J and w's at each iteration primarily for graphing later
    J_history = []
    w_history = []
    
    for i in range(num_iters):

        # Calculate the gradient and update the parameters
        dj_db, dj_dw = gradient_function(X, y, w_in, b_in, lambda_)   

        # Update Parameters using w, b, alpha and gradient
        w_in = w_in - alpha * dj_dw               
        b_in = b_in - alpha * dj_db              
       
        # Save cost J at each iteration
        if i<100000:      # prevent resource exhaustion 
            cost =  cost_function(X, y, w_in, b_in, lambda_)
            J_history.append(cost)

        # Print cost every at intervals 10 times or as many iterations if < 10
        if i% math.ceil(num_iters/10) == 0 or i == (num_iters-1):
            w_history.append(w_in)
            print(f"Iteration {i:4}: Cost {float(J_history[-1]):8.2f}   ")
        
    return w_in, b_in, J_history, w_history #return w and J,w history for graphing

```


### 预测
```python
# UNQ_C4
# GRADED FUNCTION: predict

def predict(X, w, b): 
    """
    Predict whether the label is 0 or 1 using learned logistic
    regression parameters w
    
    Args:
    X : (ndarray Shape (m, n))
    w : (array_like Shape (n,))      Parameters of the model
    b : (scalar, float)              Parameter of the model

    Returns:
    p: (ndarray (m,1))
        The predictions for X using a threshold at 0.5
    """
    # number of training examples
    m, n = X.shape   
    p = np.zeros(m)
   
    ### START CODE HERE ### 
    # Loop over each example
    for i in range(m):   
        z_wb = 0
        # Loop over each feature
        for j in range(n): 
            # Add the corresponding term to z_wb
            z_wb += X[i, j] * w[j]
        
        # Add bias term 
        z_wb += b
        
        # Calculate the prediction for this example
        f_wb = sigmoid(z_wb)


        # Apply the threshold
        p[i] = f_wb >= 0.5
        
    ### END CODE HERE ### 
    return p

#Compute accuracy on our training set
p = predict(X_train, w,b)
print('Train Accuracy: %f'%(np.mean(p == y_train) * 100))
```


### 代码
```python
"""


"""

import numpy as np
import matplotlib.pyplot as plt
from utils import *
import copy
import math


def cat_data():
    """
    查看训练数据
    :return:
    """
    print("First five elements in X_train are:\n", X_train[:5])
    print("Type of X_train:", type(X_train))
    print("First five elements in y_train are:\n", y_train[:5])
    print("Type of y_train:", type(y_train))

    print('The shape of X_train is: ' + str(X_train.shape))
    print('The shape of x_dim is: ' + str(X_train.ndim))
    print('The shape of y_train is: ' + str(y_train.shape))
    print('We have m = %d training examples' % (len(y_train)))

    # Plot examples
    plot_data(X_train, y_train[:], pos_label="Admitted", neg_label="Not admitted")
    # Set the y-axis label
    plt.ylabel('Exam 2 score')
    # Set the x-axis label
    plt.xlabel('Exam 1 score')
    plt.legend(loc="upper right")
    plt.show()


# UNQ_C1
# GRADED FUNCTION: sigmoid

def sigmoid(z):
    """
    Compute the sigmoid of z

    Args:
        z (ndarray): A scalar, numpy array of any size.

    Returns:
        g (ndarray): sigmoid(z), with the same shape as z

    """
    g = 1 / (1 + np.exp(-z))

    return g


# UNQ_C2
# GRADED FUNCTION: compute_cost
def compute_cost(X, y, w, b, lambda_=1):
    """
    Computes the cost over all examples
    Args:
      X : (ndarray Shape (m,n)) data, m examples by n features
      y : (array_like Shape (m,)) target value
      w : (array_like Shape (n,)) Values of parameters of the model
      b : scalar Values of bias parameter of the model
      lambda_: unused placeholder
    Returns:
      total_cost: (scalar)         cost
    """

    m, n = X.shape

    ### START CODE HERE ###
    loss_sum = 0

    # Loop over each training example
    for i in range(m):
        # First calculate z_wb = w[0]*X[i][0]+...+w[n-1]*X[i][n-1]+b
        z_wb = 0
        # Loop over each feature
        for j in range(n):
            # Add the corresponding term to z_wb
            z_wb_ij = w[j] * X[i][j]  # Your code here to calculate w[j] * X[i][j]
            z_wb += z_wb_ij  # equivalent to z_wb = z_wb + z_wb_ij
        # Add the bias term to z_wb
        z_wb += b  # equivalent to z_wb = z_wb + b

        f_wb = sigmoid(z_wb)  # Your code here to calculate prediction f_wb for a training example
        # 单个loss
        loss = -y[i] * np.log(f_wb) - (1 - y[i]) * np.log(
            1 - f_wb)  # Your code here to calculate loss for a training example

        loss_sum += loss  # equivalent to loss_sum = loss_sum + loss

    total_cost = (1 / m) * loss_sum
    ### END CODE HERE ###

    return total_cost


def compute_gradient(X, y, w, b, lambda_=None):
    m, n = X.shape
    dj_dw = np.zeros(w.shape)
    dj_db = 0.

    ### START CODE HERE ###
    err = 0.
    for i in range(m):
        # Calculate f_wb (exactly as you did in the compute_cost function above)
        z_wb = 0
        # Loop over each feature
        for j in range(n):
            # Add the corresponding term to z_wb
            z_wb_ij = X[i, j] * w[j]
            z_wb += z_wb_ij

        # Add bias term
        z_wb += b

        # Calculate the prediction from the model
        f_wb = sigmoid(z_wb)

        # Calculate the  gradient for b from this example
        dj_db_i = f_wb - y[i]  # Your code here to calculate the error

        # add that to dj_db
        dj_db += dj_db_i

        # get dj_dw for each attribute
        for j in range(n):
            # You code here to calculate the gradient from the i-th example for j-th attribute
            dj_dw_ij =(f_wb - y[i])* X[i][j]
            dj_dw[j] += dj_dw_ij

    # divide dj_db and dj_dw by total number of examples
    dj_dw = dj_dw / m
    dj_db = dj_db / m
    ### END CODE HERE ###

    return dj_db, dj_dw


def gradient_descent(X, y, w_in, b_in, cost_function, gradient_function, alpha, num_iters, lambda_):
    """
    Performs batch gradient descent to learn theta. Updates theta by taking
    num_iters gradient steps with learning rate alpha

    Args:
      X :    (array_like Shape (m, n)
      y :    (array_like Shape (m,))
      w_in : (array_like Shape (n,))  Initial values of parameters of the model
      b_in : (scalar)                 Initial value of parameter of the model
      cost_function:                  function to compute cost
      alpha : (float)                 Learning rate
      num_iters : (int)               number of iterations to run gradient descent
      lambda_ (scalar, float)         regularization constant

    Returns:
      w : (array_like Shape (n,)) Updated values of parameters of the model after
          running gradient descent
      b : (scalar)                Updated value of parameter of the model after
          running gradient descent
    """

    # number of training examples
    m = len(X)

    # An array to store cost J and w's at each iteration primarily for graphing later
    J_history = []
    w_history = []

    for i in range(num_iters):

        # Calculate the gradient and update the parameters
        dj_db, dj_dw = gradient_function(X, y, w_in, b_in, lambda_)

        # Update Parameters using w, b, alpha and gradient
        w_in = w_in - alpha * dj_dw
        b_in = b_in - alpha * dj_db

        # Save cost J at each iteration
        if i < 100000:  # prevent resource exhaustion
            cost = cost_function(X, y, w_in, b_in, lambda_)
            J_history.append(cost)

        # Print cost every at intervals 10 times or as many iterations if < 10
        if i % math.ceil(num_iters / 10) == 0 or i == (num_iters - 1):
            w_history.append(w_in)
            print(f"Iteration {i:4}: Cost {float(J_history[-1]):8.2f}   ")

    return w_in, b_in, J_history, w_history  # return w and J,w history for graphing


# UNQ_C4
# GRADED FUNCTION: predict

def predict(X, w, b):
    """
    Predict whether the label is 0 or 1 using learned logistic
    regression parameters w

    Args:
    X : (ndarray Shape (m, n))
    w : (array_like Shape (n,))      Parameters of the model
    b : (scalar, float)              Parameter of the model

    Returns:
    p: (ndarray (m,1))
        The predictions for X using a threshold at 0.5
    """
    # number of training examples
    m, n = X.shape
    p = np.zeros(m)

    ### START CODE HERE ###
    # Loop over each example
    for i in range(m):
        z_wb = 0
        # Loop over each feature
        for j in range(n):
            # Add the corresponding term to z_wb
            z_wb += X[i, j] * w[j]

        # Add bias term
        z_wb += b

        # Calculate the prediction for this example
        f_wb = sigmoid(z_wb)

        # Apply the threshold
        p[i] = f_wb >= 0.5

    ### END CODE HERE ###
    return p


if __name__ == '__main__':
    # load dataset
    # X_train包含两次考试成绩，表示该生是否被录取
    X_train, y_train = load_data("data/ex2data1.txt")

    # cat_data()
    # m, n = X_train.shape
    #
    # # Compute and display cost with w initialized to zeroes
    # initial_w = np.zeros(n)
    # initial_b = 0.
    # cost = compute_cost(X_train, y_train, initial_w, initial_b)
    # print('Cost at initial w (zeros): {:.3f}'.format(cost))

    # # Compute and display gradient with w initialized to zeroes
    # initial_w = np.zeros(n)
    # initial_b = 0.
    #
    # dj_db, dj_dw = compute_gradient(X_train, y_train, initial_w, initial_b)
    # print(f'dj_db at initial w (zeros):{dj_db}' )
    # print(f'dj_dw at initial w (zeros):{dj_dw.tolist()}' )

    np.random.seed(1)
    initial_w = 0.01 * (np.random.rand(2).reshape(-1,1) - 0.5)
    initial_b = -8


    # Some gradient descent settings
    iterations = 10000
    alpha = 0.001

    w,b, J_history,_ = gradient_descent(X_train ,y_train, initial_w, initial_b,
                                       compute_cost, compute_gradient, alpha, iterations, 0)

    # Compute accuracy on our training set
    p = predict(X_train, w, b)
    print('Train Accuracy: %f' % (np.mean(p == y_train) * 100))


```


## 带正则化的
问题描述：
Suppose you are the product manager of the factory and you have the test results for some microchips on two different tests.

From these two tests, you would like to determine whether the microchips should be accepted or rejected.
To help you make the decision, you have a dataset of test results on past microchips, from which you can build a logistic regression model.



![20220711151224](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220711151224.png)
![20220711151329](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220711151329.png)
![20220711151529](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/20220711151529.png)
```python
"""
Suppose you are the product manager of the factory and you have the test results for some microchips on two different tests.

From these two tests, you would like to determine whether the microchips should be accepted or rejected.
To help you make the decision, you have a dataset of test results on past microchips, from which you can build a logistic regression model.

"""

import numpy as np
import matplotlib.pyplot as plt
from utils import *
import copy
import math



def cat_data():
    # Plot examples
    plot_data(X_train, y_train[:], pos_label="Accepted", neg_label="Rejected")

    # Set the y-axis label
    plt.ylabel('Microchip Test 2')
    # Set the x-axis label
    plt.xlabel('Microchip Test 1')
    plt.legend(loc="upper right")
    plt.show()


def map_feature(X1, X2):
    """
    Feature mapping function to polynomial features
    we will map the features into all polynomial terms of  𝑥1  and  𝑥2  up to the sixth power.
    """
    X1 = np.atleast_1d(X1)
    X2 = np.atleast_1d(X2)
    degree = 6
    out = []
    for i in range(1, degree+1):
        for j in range(i + 1):
            out.append((X1**(i-j) * (X2**j)))
    return np.stack(out, axis=1)


# UNQ_C1
# GRADED FUNCTION: sigmoid

def sigmoid(z):
    """
    Compute the sigmoid of z

    Args:
        z (ndarray): A scalar, numpy array of any size.

    Returns:
        g (ndarray): sigmoid(z), with the same shape as z

    """

    ### START CODE HERE ###
    g = 1 / (1 + np.exp(-z))

    ### END SOLUTION ###

    return g

# UNQ_C2
# GRADED FUNCTION: compute_cost
def compute_cost(X, y, w, b, lambda_=1):
    """
    Computes the cost over all examples
    Args:
      X : (ndarray Shape (m,n)) data, m examples by n features
      y : (array_like Shape (m,)) target value
      w : (array_like Shape (n,)) Values of parameters of the model
      b : scalar Values of bias parameter of the model
      lambda_: unused placeholder
    Returns:
      total_cost: (scalar)         cost
    """

    m, n = X.shape

    ### START CODE HERE ###
    loss_sum = 0

    # Loop over each training example
    for i in range(m):
        # First calculate z_wb = w[0]*X[i][0]+...+w[n-1]*X[i][n-1]+b
        z_wb = 0
        # Loop over each feature
        for j in range(n):
            # Add the corresponding term to z_wb
            z_wb_ij = w[j] * X[i][j]  # Your code here to calculate w[j] * X[i][j]
            z_wb += z_wb_ij  # equivalent to z_wb = z_wb + z_wb_ij
        # Add the bias term to z_wb
        z_wb += b  # equivalent to z_wb = z_wb + b

        f_wb = sigmoid(z_wb)  # Your code here to calculate prediction f_wb for a training example
        # ?
        loss = -y[i] * np.log(f_wb) - (1 - y[i]) * np.log(
            1 - f_wb)  # Your code here to calculate loss for a training example

        loss_sum += loss  # equivalent to loss_sum = loss_sum + loss

    total_cost = (1 / m) * loss_sum
    ### END CODE HERE ###

    return total_cost


# UNQ_C5
def compute_cost_reg(X, y, w, b, lambda_=1):
    """
    计算正则项
    Computes the cost over all examples
    Args:
      X : (array_like Shape (m,n)) data, m examples by n features
      y : (array_like Shape (m,)) target value
      w : (array_like Shape (n,)) Values of parameters of the model
      b : (array_like Shape (n,)) Values of bias parameter of the model
      lambda_ : (scalar, float)    Controls amount of regularization
    Returns:
      total_cost: (scalar)         cost
    """

    m, n = X.shape

    # Calls the compute_cost function that you implemented above
    cost_without_reg = compute_cost(X, y, w, b)

    # You need to calculate this value
    reg_cost = 0.

    ### START CODE HERE ###
    for j in range(n):
        reg_cost_j = w[j] ** 2  # Your code here to calculate the cost from w[j]
        reg_cost = reg_cost + reg_cost_j

    ### END CODE HERE ###

    # Add the regularization cost to get the total cost
    total_cost = cost_without_reg + (lambda_ / (2 * m)) * reg_cost

    return total_cost




def compute_gradient(X, y, w, b, lambda_=None):
    m, n = X.shape
    dj_dw = np.zeros(w.shape)
    dj_db = 0.

    ### START CODE HERE ###
    err = 0.
    for i in range(m):
        # Calculate f_wb (exactly as you did in the compute_cost function above)
        z_wb = 0
        # Loop over each feature
        for j in range(n):
            # Add the corresponding term to z_wb
            z_wb_ij = X[i, j] * w[j]
            z_wb += z_wb_ij

        # Add bias term
        z_wb += b

        # Calculate the prediction from the model
        f_wb = sigmoid(z_wb)

        # Calculate the  gradient for b from this example
        dj_db_i = f_wb - y[i]  # Your code here to calculate the error

        # add that to dj_db
        dj_db += dj_db_i

        # get dj_dw for each attribute
        for j in range(n):
            # You code here to calculate the gradient from the i-th example for j-th attribute
            dj_dw_ij =(f_wb - y[i])* X[i][j]
            dj_dw[j] += dj_dw_ij

    # divide dj_db and dj_dw by total number of examples
    dj_dw = dj_dw / m
    dj_db = dj_db / m
    ### END CODE HERE ###

    return dj_db, dj_dw


# UNQ_C6
def compute_gradient_reg(X, y, w, b, lambda_=1):
    """
    Computes the gradient for linear regression

    Args:
      X : (ndarray Shape (m,n))   variable such as house size
      y : (ndarray Shape (m,))    actual value
      w : (ndarray Shape (n,))    values of parameters of the model
      b : (scalar)                value of parameter of the model
      lambda_ : (scalar,float)    regularization constant
    Returns
      dj_db: (scalar)             The gradient of the cost w.r.t. the parameter b.
      dj_dw: (ndarray Shape (n,)) The gradient of the cost w.r.t. the parameters w.

    """
    m, n = X.shape

    dj_db, dj_dw = compute_gradient(X, y, w, b)

    ### START CODE HERE ###
    for j in range(n):
        dj_dw_j_reg = (lambda_ / m) * w[j]
        dj_dw[j] = dj_dw[j] + dj_dw_j_reg

    ### END CODE HERE ###

    return dj_db, dj_dw


def gradient_descent(X, y, w_in, b_in, cost_function, gradient_function, alpha, num_iters, lambda_):
    """
    Performs batch gradient descent to learn theta. Updates theta by taking
    num_iters gradient steps with learning rate alpha

    Args:
      X :    (array_like Shape (m, n)
      y :    (array_like Shape (m,))
      w_in : (array_like Shape (n,))  Initial values of parameters of the model
      b_in : (scalar)                 Initial value of parameter of the model
      cost_function:                  function to compute cost
      alpha : (float)                 Learning rate
      num_iters : (int)               number of iterations to run gradient descent
      lambda_ (scalar, float)         regularization constant

    Returns:
      w : (array_like Shape (n,)) Updated values of parameters of the model after
          running gradient descent
      b : (scalar)                Updated value of parameter of the model after
          running gradient descent
    """

    # number of training examples
    m = len(X)

    # An array to store cost J and w's at each iteration primarily for graphing later
    J_history = []
    w_history = []

    for i in range(num_iters):

        # Calculate the gradient and update the parameters
        dj_db, dj_dw = gradient_function(X, y, w_in, b_in, lambda_)

        # Update Parameters using w, b, alpha and gradient
        w_in = w_in - alpha * dj_dw
        b_in = b_in - alpha * dj_db

        # Save cost J at each iteration
        if i < 100000:  # prevent resource exhaustion
            cost = cost_function(X, y, w_in, b_in, lambda_)
            J_history.append(cost)

        # Print cost every at intervals 10 times or as many iterations if < 10
        if i % math.ceil(num_iters / 10) == 0 or i == (num_iters - 1):
            w_history.append(w_in)
            print(f"Iteration {i:4}: Cost {float(J_history[-1]):8.2f}   ")

    return w_in, b_in, J_history, w_history  # return w and J,w history for graphing


def predict(X, w, b):
    """
    Predict whether the label is 0 or 1 using learned logistic
    regression parameters w

    Args:
    X : (ndarray Shape (m, n))
    w : (array_like Shape (n,))      Parameters of the model
    b : (scalar, float)              Parameter of the model

    Returns:
    p: (ndarray (m,1))
        The predictions for X using a threshold at 0.5
    """
    # number of training examples
    m, n = X.shape
    p = np.zeros(m)

    ### START CODE HERE ###
    # Loop over each example
    for i in range(m):
        z_wb = 0
        # Loop over each feature
        for j in range(n):
            # Add the corresponding term to z_wb
            z_wb += X[i, j] * w[j]

        # Add bias term
        z_wb += b

        # Calculate the prediction for this example
        f_wb = sigmoid(z_wb)

        # Apply the threshold
        p[i] = f_wb >= 0.5

    ### END CODE HERE ###
    return p


def plot_decision_boundary(w, b, X, y):
    # Credit to dibgerge on Github for this plotting code

    plot_data(X[:, 0:2], y)

    if X.shape[1] <= 2:
        plot_x = np.array([min(X[:, 0]), max(X[:, 0])])
        plot_y = (-1. / w[1]) * (w[0] * plot_x + b)

        plt.plot(plot_x, plot_y, c="b")

    else:
        u = np.linspace(-1, 1.5, 50)
        v = np.linspace(-1, 1.5, 50)

        z = np.zeros((len(u), len(v)))

        # Evaluate z = theta*x over the grid
        for i in range(len(u)):
            for j in range(len(v)):
                z[i, j] = sig(np.dot(map_feature(u[i], v[j]), w) + b)

        # important to transpose z before calling contour
        z = z.T

        # Plot z = 0
        plt.contour(u, v, z, levels=[0.5], colors="g")


if __name__ == '__main__':
    # X_train contains the test results for the microchips from two tests
    # y_train contains the results of the QA
    X_train, y_train = load_data("data/ex2data2.txt")

    # cat_data()

    print("Original shape of data:", X_train.shape)

    mapped_X = map_feature(X_train[:, 0], X_train[:, 1])
    print("Shape after feature mapping:", mapped_X.shape)

    # print("X_train[0]:", X_train[0])
    # print("mapped X_train[0]:", mapped_X[0])
    X_mapped = map_feature(X_train[:, 0], X_train[:, 1])
    np.random.seed(1)
    initial_w = np.random.rand(X_mapped.shape[1]) - 0.5
    initial_b = 0.5
    lambda_ = 0.5
    cost = compute_cost_reg(X_mapped, y_train, initial_w, initial_b, lambda_)

    print("Regularized cost :", cost)

    X_mapped = map_feature(X_train[:, 0], X_train[:, 1])
    np.random.seed(1)
    initial_w = np.random.rand(X_mapped.shape[1]) - 0.5
    initial_b = 0.5

    lambda_ = 0.5
    dj_db, dj_dw = compute_gradient_reg(X_mapped, y_train, initial_w, initial_b, lambda_)

    print(f"dj_db: {dj_db}", )
    print(f"First few elements of regularized dj_dw:\n {dj_dw[:4].tolist()}", )

    # Initialize fitting parameters
    np.random.seed(1)
    initial_w = np.random.rand(X_mapped.shape[1]) - 0.5
    initial_b = 1.

    # Set regularization parameter lambda_ to 1 (you can try varying this)
    lambda_ = 0.01
    # Some gradient descent settings
    iterations = 10000
    alpha = 0.01

    w, b, J_history, _ = gradient_descent(X_mapped, y_train, initial_w, initial_b,
                                          compute_cost_reg, compute_gradient_reg,
                                          alpha, iterations, lambda_)


    #Compute accuracy on the training set
    p = predict(X_mapped, w, b)

    print('Train Accuracy: %f'%(np.mean(p == y_train) * 100))





```

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/%E5%90%B4%E6%81%A9%E8%BE%BE2022%E6%96%B0%E7%89%88%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E8%AF%BE%E5%90%8E%E9%A2%98-%E8%AF%BE%E7%A8%8B1week3-logistic-regression/  

