# 初学决策树





## 问题陈述
问题陈述：
Suppose you are starting a company that grows and sells wild mushrooms.

Since not all mushrooms are edible, you'd like to be able to tell whether a given mushroom is edible or poisonous based on it's physical attributes
You have some existing data that you can use for this task.
Can you use the data to help you identify which mushrooms can be sold safely?

Note: The dataset used is for illustrative purposes only. It is not meant to be a guide on identifying edible mushrooms.

You have 10 examples of mushrooms. For each example, you have
- Three features
    - Cap Color (Brown or Red),
    - Stalk Shape (Tapering or Enlarging), and
    - Solitary (Yes or No)
- Label
    - Edible (1 indicating yes or 0 indicating poisonous)


**回顾建立决策树的步骤：**
1.Start with all examples at the root node
2.Calculate information gain for splitting on all possible features, and pick the one with the highest information gain
3.Split dataset according to the selected feature, and create left and right branches of the tree
4.Keep repeating splitting process until stopping criteria is met




## 建立决策树
### 示例数据
```python
X_train = np.array(
    [[1, 1, 1], [1, 0, 1], [1, 0, 0], [1, 0, 0], [1, 1, 1], [0, 1, 1], [0, 0, 0], [1, 0, 1], [0, 1, 0], [1, 0, 0]])
y_train = np.array([1, 1, 0, 0, 1, 0, 0, 1, 1, 0])
# cat_data()


def cat_data():
    print("First few elements of X_train:\n", X_train[:3])
    print("Type of X_train:", type(X_train))

    print("First few elements of y_train:", y_train[:3])
    print("Type of y_train:", type(y_train))

    print('The shape of X_train is:', X_train.shape)
    print('The dim of X_train is:', X_train.ndim)
    print('The shape of y_train is: ', y_train.shape)
    print('The dim of y_train is: ', y_train.ndim)
    print('Number of training examples (m):', len(X_train))


```



### 计算熵

![20220712120924](http://image.xpshuai.cn/20220712120924.png)


```python

def compute_entropy(y):
    """
    Computes the entropy for

    Args:
       y (ndarray): Numpy array indicating whether each example at a node is
           edible (`1`) or poisonous (`0`)

    Returns:
        entropy (float): Entropy at that node

    """
    # You need to return the following variables correctly
    entropy = 0.

    ### START CODE HERE ###
    if len(y) != 0:
        # Your code here to calculate the fraction of edible examples (i.e with value = 1 in y)
        p1 = len(y[y == 1]) / len(y)

        # For p1 = 0 and 1, set the entropy to 0 (to handle 0log0)
        if p1 != 0 and p1 != 1:
            # Your code here to calculate the entropy using the formula provided above
            entropy = -p1 * np.log2(p1) - (1 - p1) * np.log2(1 - p1)
        else:
            entropy = 0.

    ### END CODE HERE ###

    return entropy

print("Entropy at root node: ", compute_entropy(y_train)) 
```



### Split the dataset

![20220712121431](http://image.xpshuai.cn/20220712121431.png)

```python


def split_dataset(X, node_indices, feature):
    """
    Splits the data at the given node into
    left and right branches

    Args:
        X (ndarray):             Data matrix of shape(n_samples, n_features)
        node_indices (ndarray):  List containing the active indices. I.e, the samples being considered at this step.
        feature (int):           Index of feature to split on

    Returns:
        left_indices (ndarray): Indices with feature value == 1
        right_indices (ndarray): Indices with feature value == 0
    """

    # You need to return the following variables correctly
    left_indices = []
    right_indices = []

    ### START CODE HERE ###
    for i in node_indices:
        if X[i][feature] == 1:  # Your code here to check if the value of X at that index for the feature is 1
            left_indices.append(i)
        else:
            right_indices.append(i)

    ### END CODE HERE ###

    return left_indices, right_indices


root_indices = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Feel free to play around with these variables
# The dataset only has three features, so this value can be 0 (Brown Cap), 1 (Tapering Stalk Shape) or 2 (Solitary)
feature = 0

left_indices, right_indices = split_dataset(X_train, root_indices, feature)

print("Left indices: ", left_indices)
print("Right indices: ", right_indices)
```



### 计算信息增益
![20220712124434](http://image.xpshuai.cn/20220712124434.png)


```python

# UNQ_C3
# GRADED FUNCTION: compute_information_gain

def compute_information_gain(X, y, node_indices, feature):
    
    """
    Compute the information of splitting the node on a given feature
    
    Args:
        X (ndarray):            Data matrix of shape(n_samples, n_features)
        y (array like):         list or ndarray with n_samples containing the target variable
        node_indices (ndarray): List containing the active indices. I.e, the samples being considered in this step.
   
    Returns:
        cost (float):        Cost computed
    
    """    
    # Split dataset
    left_indices, right_indices = split_dataset(X, node_indices, feature)
    
    # Some useful variables
    X_node, y_node = X[node_indices], y[node_indices]
    X_left, y_left = X[left_indices], y[left_indices]
    X_right, y_right = X[right_indices], y[right_indices]
    
    # You need to return the following variables correctly
    information_gain = 0
    ### START CODE HERE ###
    # Your code here to compute the entropy at the node using compute_entropy()
    node_entropy = compute_entropy(y_node)
    left_entropy = compute_entropy(y_left)
    right_entropy = compute_entropy(y_right

    # Your code here to compute the proportion of examples at the left branch
    w_left = len(X_left) / len(X_node)
    w_right = len(X_right) / len(X_node)
 
    # Your code here to compute weighted entropy from the split using 
    # w_left, w_right, left_entropy and right_entropy
    weighted_entropy = w_left * left_entropy + w_right * right_entropy
 
    # Your code here to compute the information gain as the entropy at the node
    # minus the weighted entropy
    information_gain = node_entropy - weighted_entropy
    ### END CODE HERE ###
                                    
                                                     
        
    return information_gain

info_gain0 = compute_information_gain(X_train, y_train, root_indices, feature=0)
print("Information Gain from splitting the root on brown cap: ", info_gain0)
    
info_gain1 = compute_information_gain(X_train, y_train, root_indices, feature=1)
print("Information Gain from splitting the root on tapering stalk shape: ", info_gain1)

info_gain2 = compute_information_gain(X_train, y_train, root_indices, feature=2)
print("Information Gain from splitting the root on solitary: ", info_gain2)


```


### Get best split


```python
# UNQ_C4
# GRADED FUNCTION: get_best_split

def get_best_split(X, y, node_indices):   
    """
    Returns the optimal feature and threshold value
    to split the node data 
    
    Args:
        X (ndarray):            Data matrix of shape(n_samples, n_features)
        y (array like):         list or ndarray with n_samples containing the target variable
        node_indices (ndarray): List containing the active indices. I.e, the samples being considered in this step.

    Returns:
        best_feature (int):     The index of the best feature to split
    """    
    
    # Some useful variables
    num_features = X.shape[1]
    
    # You need to return the following variables correctly
    best_feature = -1
    
    ### START CODE HERE ###
    max_info_gain = 0

    # Iterate through all features
    for feature in range(num_features): 

        # Your code here to compute the information gain from splitting on this feature
        info_gain = compute_information_gain(X, y, node_indices, feature)

        # If the information gain is larger than the max seen so far
        if info_gain > max_info_gain:  
            # Your code here to set the max_info_gain and best_feature
            max_info_gain = info_gain
            best_feature = feature
       
    ### END CODE HERE ##    
   
    return best_feature

best_feature = get_best_split(X_train, y_train, root_indices)
print("Best feature to split on: %d" % best_feature)
```


### Building the tree
```python
# Not graded
tree = []

def build_tree_recursive(X, y, node_indices, branch_name, max_depth, current_depth):
    """
    Build a tree using the recursive algorithm that split the dataset into 2 subgroups at each node.
    This function just prints the tree.
    
    
    Args:
        X (ndarray):            Data matrix of shape(n_samples, n_features)
        y (array like):         list or ndarray with n_samples containing the target variable
        node_indices (ndarray): List containing the active indices. I.e, the samples being considered in this step.
        branch_name (string):   Name of the branch. ['Root', 'Left', 'Right']
        max_depth (int):        Max depth of the resulting tree. 
        current_depth (int):    Current depth. Parameter used during recursive call.
   
    """ 

    # Maximum depth reached - stop splitting
    if current_depth == max_depth:
        formatting = " "*current_depth + "-"*current_depth
        print(formatting, "%s leaf node with indices" % branch_name, node_indices)
        return
   
    # Otherwise, get best split and split the data
    # Get the best feature and threshold at this node
    best_feature = get_best_split(X, y, node_indices) 
    tree.append((current_depth, branch_name, best_feature, node_indices))
    
    formatting = "-"*current_depth
    print("%s Depth %d, %s: Split on feature: %d" % (formatting, current_depth, branch_name, best_feature))
    
    # Split the dataset at the best feature
    left_indices, right_indices = split_dataset(X, node_indices, best_feature)
    
    # continue splitting the left and the right child. Increment current depth
    build_tree_recursive(X, y, left_indices, "Left", max_depth, current_depth+1)
    build_tree_recursive(X, y, right_indices, "Right", max_depth, current_depth+1)


build_tree_recursive(X_train, y_train, root_indices, "Root", max_depth=2, current_depth=0)
```


## code
```python

"""
问题陈述：
Suppose you are starting a company that grows and sells wild mushrooms.

Since not all mushrooms are edible, you'd like to be able to tell whether a given mushroom is edible or poisonous based on it's physical attributes
You have some existing data that you can use for this task.
Can you use the data to help you identify which mushrooms can be sold safely?

Note: The dataset used is for illustrative purposes only. It is not meant to be a guide on identifying edible mushrooms.

You have 10 examples of mushrooms. For each example, you have
- Three features
    - Cap Color (Brown or Red),
    - Stalk Shape (Tapering or Enlarging), and
    - Solitary (Yes or No)
- Label
    - Edible (1 indicating yes or 0 indicating poisonous)


回顾建立决策树的步骤：
1.Start with all examples at the root node
2.Calculate information gain for splitting on all possible features, and pick the one with the highest information gain
3.Split dataset according to the selected feature, and create left and right branches of the tree
4.Keep repeating splitting process until stopping criteria is met

"""

import numpy as np
import matplotlib.pyplot as plt


def compute_entropy_test(target):
    y = np.array([1] * 10)
    result = target(y)

    assert result == 0, "Entropy must be 0 with array of ones"

    y = np.array([0] * 10)
    result = target(y)

    assert result == 0, "Entropy must be 0 with array of zeros"

    y = np.array([0] * 12 + [1] * 12)
    result = target(y)

    assert result == 1, "Entropy must be 1 with same ammount of ones and zeros"

    y = np.array([1, 0, 1, 0, 1, 1, 1, 0, 1])
    assert np.isclose(target(y), 0.918295, atol=1e-6), "Wrong value. Something between 0 and 1"
    assert np.isclose(target(-y + 1), target(y), atol=1e-6), "Wrong value"

    print("\033[92m All tests passed.")


def split_dataset_test(target):
    X = np.array([[1, 0],
                  [1, 0],
                  [1, 1],
                  [0, 0],
                  [0, 1]])
    X_t = np.array([[0, 1, 0, 1, 0]])
    X = np.concatenate((X, X_t.T), axis=1)

    left, right = target(X, list(range(5)), 2)
    expected = {'left': np.array([1, 3]),
                'right': np.array([0, 2, 4])}

    assert type(left) == list, f"Wrong type for left. Expected: list got: {type(left)}"
    assert type(right) == list, f"Wrong type for right. Expected: list got: {type(right)}"

    assert type(left[0]) == int, f"Wrong type for elements in the left list. Expected: int got: {type(left[0])}"
    assert type(right[0]) == int, f"Wrong type for elements in the right list. Expected: number got: {type(right[0])}"

    assert len(left) == 2, f"left must have 2 elements but got: {len(left)}"
    assert len(right) == 3, f"right must have 3 elements but got: {len(right)}"

    assert np.allclose(right, expected['right']), f"Wrong value for right. Expected: {expected['right']} \ngot: {right}"
    assert np.allclose(left, expected['left']), f"Wrong value for left. Expected: {expected['left']} \ngot: {left}"

    X = np.array([[0, 1],
                  [1, 1],
                  [1, 1],
                  [0, 0],
                  [1, 0]])
    X_t = np.array([[0, 1, 0, 1, 0]])
    X = np.concatenate((X_t.T, X), axis=1)

    left, right = target(X, list(range(5)), 0)
    expected = {'left': np.array([1, 3]),
                'right': np.array([0, 2, 4])}

    assert np.allclose(right, expected['right']) and np.allclose(left, expected[
        'left']), f"Wrong value when target is at index 0."

    X = (np.random.rand(11, 3) > 0.5) * 1  # Just random binary numbers
    X_t = np.array([[0, 1, 0, 1, 0, 1, 1, 0, 0, 0, 0]])
    X = np.concatenate((X, X_t.T), axis=1)

    left, right = target(X, [1, 2, 3, 6, 7, 9, 10], 3)
    expected = {'left': np.array([1, 3, 6]),
                'right': np.array([2, 7, 9, 10])}

    assert np.allclose(right, expected['right']) and np.allclose(left, expected[
        'left']), f"Wrong value when target is at index 0. \nExpected: {expected} \ngot: \{left:{left}, 'right': {right}\}"

    print("\033[92m All tests passed.")


def compute_information_gain_test(target):
    X = np.array([[1, 0],
                  [1, 0],
                  [1, 0],
                  [0, 0],
                  [0, 1]])

    y = np.array([[0, 0, 0, 0, 0]]).T
    node_indexes = list(range(5))

    result1 = target(X, y, node_indexes, 0)
    result2 = target(X, y, node_indexes, 0)

    assert result1 == 0 and result2 == 0, f"Information gain must be 0 when target variable is pure. Got {result1} and {result2}"

    y = np.array([[0, 1, 0, 1, 0]]).T
    node_indexes = list(range(5))

    result = target(X, y, node_indexes, 0)
    assert np.isclose(result, 0.019973, atol=1e-6), f"Wrong information gain. Expected {0.019973} got: {result}"

    result = target(X, y, node_indexes, 1)
    assert np.isclose(result, 0.170951, atol=1e-6), f"Wrong information gain. Expected {0.170951} got: {result}"

    node_indexes = list(range(4))
    result = target(X, y, node_indexes, 0)
    assert np.isclose(result, 0.311278, atol=1e-6), f"Wrong information gain. Expected {0.311278} got: {result}"

    result = target(X, y, node_indexes, 1)
    assert np.isclose(result, 0, atol=1e-6), f"Wrong information gain. Expected {0.0} got: {result}"

    print("\033[92m All tests passed.")


def get_best_split_test(target):
    X = np.array([[1, 0],
                  [1, 0],
                  [1, 0],
                  [0, 0],
                  [0, 1]])

    y = np.array([[0, 0, 0, 0, 0]]).T
    node_indexes = list(range(5))

    result = target(X, y, node_indexes)

    assert result == -1, f"When the target variable is pure, there is no best split to do. Expected -1, got {result}"

    y = X[:, 0]
    result = target(X, y, node_indexes)
    assert result == 0, f"If the target is fully correlated with other feature, that feature must be the best split. Expected 0, got {result}"
    y = X[:, 1]
    result = target(X, y, node_indexes)
    assert result == 1, f"If the target is fully correlated with other feature, that feature must be the best split. Expected 1, got {result}"

    y = 1 - X[:, 0]
    result = target(X, y, node_indexes)
    assert result == 0, f"If the target is fully correlated with other feature, that feature must be the best split. Expected 0, got {result}"

    y = np.array([[0, 1, 0, 1, 0]]).T
    result = target(X, y, node_indexes)
    assert result == 1, f"Wrong result. Expected 1, got {result}"

    y = np.array([[0, 1, 0, 1, 0]]).T
    node_indexes = [2, 3, 4]
    result = target(X, y, node_indexes)
    assert result == 0, f"Wrong result. Expected 0, got {result}"

    n_samples = 100
    X0 = np.array([[1] * n_samples])
    X1 = np.array([[0] * n_samples])
    X2 = (np.random.rand(1, 100) > 0.5) * 1
    X3 = np.array([[1] * int(n_samples / 2) + [0] * int(n_samples / 2)])

    y = X2.T
    node_indexes = list(range(20, 80))
    X = np.array([X0, X1, X2, X3]).T.reshape(n_samples, 4)
    result = target(X, y, node_indexes)

    assert result == 2, f"Wrong result. Expected 2, got {result}"

    y = X0.T
    result = target(X, y, node_indexes)
    assert result == -1, f"When the target variable is pure, there is no best split to do. Expected -1, got {result}"
    print("\033[92m All tests passed.")


def cat_data():
    print("First few elements of X_train:\n", X_train[:3])
    print("Type of X_train:", type(X_train))

    print("First few elements of y_train:", y_train[:3])
    print("Type of y_train:", type(y_train))

    print('The shape of X_train is:', X_train.shape)
    print('The dim of X_train is:', X_train.ndim)
    print('The shape of y_train is: ', y_train.shape)
    print('The dim of y_train is: ', y_train.ndim)
    print('Number of training examples (m):', len(X_train))


# Calculate the entropy at a node
# UNQ_C1
# GRADED FUNCTION: compute_entropy

def compute_entropy(y):
    """
    Computes the entropy for

    Args:
       y (ndarray): Numpy array indicating whether each example at a node is
           edible (`1`) or poisonous (`0`)

    Returns:
        entropy (float): Entropy at that node

    """
    # You need to return the following variables correctly
    entropy = 0.

    ### START CODE HERE ###
    if len(y) != 0:
        # Your code here to calculate the fraction of edible examples (i.e with value = 1 in y)
        p1 = len(y[y == 1]) / len(y)

        # For p1 = 0 and 1, set the entropy to 0 (to handle 0log0)
        if p1 != 0 and p1 != 1:
            # Your code here to calculate the entropy using the formula provided above
            entropy = -p1 * np.log2(p1) - (1 - p1) * np.log2(1 - p1)
        else:
            entropy = 0.

    ### END CODE HERE ###

    return entropy

# Split the dataset at a node into left and right branches based on a given feature
# UNQ_C2
# GRADED FUNCTION: split_dataset

def split_dataset(X, node_indices, feature):
    """
    Splits the data at the given node into
    left and right branches

    Args:
        X (ndarray):             Data matrix of shape(n_samples, n_features)
        node_indices (ndarray):  List containing the active indices. I.e, the samples being considered at this step.
        feature (int):           Index of feature to split on

    Returns:
        left_indices (ndarray): Indices with feature value == 1
        right_indices (ndarray): Indices with feature value == 0
    """

    # You need to return the following variables correctly
    left_indices = []
    right_indices = []

    ### START CODE HERE ###
    for i in node_indices:
        if X[i][feature] == 1:  # Your code here to check if the value of X at that index for the feature is 1
            left_indices.append(i)
        else:
            right_indices.append(i)

    ### END CODE HERE ###

    return left_indices, right_indices

# Calculate the information gain from splitting on a given feature
# UNQ_C3
# GRADED FUNCTION: compute_information_gain

def compute_information_gain(X, y, node_indices, feature):
    """
    Compute the information of splitting the node on a given feature

    Args:
        X (ndarray):            Data matrix of shape(n_samples, n_features)
        y (array like):         list or ndarray with n_samples containing the target variable
        node_indices (ndarray): List containing the active indices. I.e, the samples being considered in this step.

    Returns:
        cost (float):        Cost computed

    """
    # Split dataset
    left_indices, right_indices = split_dataset(X, node_indices, feature)

    # Some useful variables
    X_node, y_node = X[node_indices], y[node_indices]
    X_left, y_left = X[left_indices], y[left_indices]
    X_right, y_right = X[right_indices], y[right_indices]

    # You need to return the following variables correctly
    information_gain = 0
    ### START CODE HERE ###
    # Your code here to compute the entropy at the node using compute_entropy()
    node_entropy = compute_entropy(y_node)
    left_entropy = compute_entropy(y_left)
    right_entropy = compute_entropy(y_right)

    # Your code here to compute the proportion of examples at the left branch
    w_left = len(X_left) / len(X_node)
    w_right = len(X_right) / len(X_node)

    # Your code here to compute weighted entropy from the split using
    # w_left, w_right, left_entropy and right_entropy
    weighted_entropy = w_left * left_entropy + w_right * right_entropy

    # Your code here to compute the information gain as the entropy at the node
    # minus the weighted entropy
    information_gain = node_entropy - weighted_entropy
    ### END CODE HERE ###

    return information_gain

# Choose the feature that maximizes information gain
# UNQ_C4
# GRADED FUNCTION: get_best_split

def get_best_split(X, y, node_indices):
    """
    Returns the optimal feature and threshold value
    to split the node data

    Args:
        X (ndarray):            Data matrix of shape(n_samples, n_features)
        y (array like):         list or ndarray with n_samples containing the target variable
        node_indices (ndarray): List containing the active indices. I.e, the samples being considered in this step.

    Returns:
        best_feature (int):     The index of the best feature to split
    """

    # Some useful variables
    num_features = X.shape[1]

    # You need to return the following variables correctly
    best_feature = -1

    ### START CODE HERE ###
    max_info_gain = 0

    # Iterate through all features
    for feature in range(num_features):

        # Your code here to compute the information gain from splitting on this feature
        info_gain = compute_information_gain(X, y, node_indices, feature)

        # If the information gain is larger than the max seen so far
        if info_gain > max_info_gain:
            # Your code here to set the max_info_gain and best_feature
            max_info_gain = info_gain
            best_feature = feature

    ### END CODE HERE ##

    return best_feature


# 创建树
# Not graded
tree = []


def build_tree_recursive(X, y, node_indices, branch_name, max_depth, current_depth):
    """
    Build a tree using the recursive algorithm that split the dataset into 2 subgroups at each node.
    This function just prints the tree.


    Args:
        X (ndarray):            Data matrix of shape(n_samples, n_features)
        y (array like):         list or ndarray with n_samples containing the target variable
        node_indices (ndarray): List containing the active indices. I.e, the samples being considered in this step.
        branch_name (string):   Name of the branch. ['Root', 'Left', 'Right']
        max_depth (int):        Max depth of the resulting tree.
        current_depth (int):    Current depth. Parameter used during recursive call.

    """

    # Maximum depth reached - stop splitting
    if current_depth == max_depth:
        formatting = " " * current_depth + "-" * current_depth
        print(formatting, "%s leaf node with indices" % branch_name, node_indices)
        return

    # Otherwise, get best split and split the data
    # Get the best feature and threshold at this node
    best_feature = get_best_split(X, y, node_indices)
    tree.append((current_depth, branch_name, best_feature, node_indices))

    formatting = "-" * current_depth
    print("%s Depth %d, %s: Split on feature: %d" % (formatting, current_depth, branch_name, best_feature))

    # Split the dataset at the best feature
    left_indices, right_indices = split_dataset(X, node_indices, best_feature)

    # continue splitting the left and the right child. Increment current depth
    build_tree_recursive(X, y, left_indices, "Left", max_depth, current_depth + 1)
    build_tree_recursive(X, y, right_indices, "Right", max_depth, current_depth + 1)


if __name__ == '__main__':
    X_train = np.array(
        [[1, 1, 1], [1, 0, 1], [1, 0, 0], [1, 0, 0], [1, 1, 1], [0, 1, 1], [0, 0, 0], [1, 0, 1], [0, 1, 0], [1, 0, 0]])
    y_train = np.array([1, 1, 0, 0, 1, 0, 0, 1, 1, 0])
    # cat_data()
    # print("Entropy at root node: ", compute_entropy(y_train))
    root_indices = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

    # Feel free to play around with these variables
    # The dataset only has three features, so this value can be 0 (Brown Cap), 1 (Tapering Stalk Shape) or 2 (Solitary)
    feature = 0

    # left_indices, right_indices = split_dataset(X_train, root_indices, feature)
    # 
    # print("Left indices: ", left_indices)
    # print("Right indices: ", right_indices)
    # 
    build_tree_recursive(X_train, y_train, root_indices, "Root", max_depth=2, current_depth=0)

```

---

> 作者: [剑胆琴心](http://geoer.cn)  
> URL: https://geoer.cn/%E5%88%9D%E5%AD%A6%E5%86%B3%E7%AD%96%E6%A0%91/  

