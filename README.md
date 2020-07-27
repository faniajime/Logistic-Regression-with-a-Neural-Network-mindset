# Logistic-Regression-with-a-Neural-Network-mindset
This is for the course of Deep Learning by Andrew Ng

{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Logistic Regression with a Neural Network mindset\n",
    "\n",
    "Welcome to your first (required) programming assignment! You will build a logistic regression classifier to recognize  cats. This assignment will step you through how to do this with a Neural Network mindset, and so will also hone your intuitions about deep learning.\n",
    "\n",
    "**Instructions:**\n",
    "- Do not use loops (for/while) in your code, unless the instructions explicitly ask you to do so.\n",
    "\n",
    "**You will learn to:**\n",
    "- Build the general architecture of a learning algorithm, including:\n",
    "    - Initializing parameters\n",
    "    - Calculating the cost function and its gradient\n",
    "    - Using an optimization algorithm (gradient descent) \n",
    "- Gather all three functions above into a main model function, in the right order."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## <font color='darkblue'>Updates</font>\n",
    "This notebook has been updated over the past few months.  The prior version was named \"v5\", and the current versionis now named '6a'\n",
    "\n",
    "#### If you were working on a previous version:\n",
    "* You can find your prior work by looking in the file directory for the older files (named by version name).\n",
    "* To view the file directory, click on the \"Coursera\" icon in the top left corner of this notebook.\n",
    "* Please copy your work from the older versions to the new version, in order to submit your work for grading.\n",
    "\n",
    "#### List of Updates\n",
    "* Forward propagation formula, indexing now starts at 1 instead of 0.\n",
    "* Optimization function comment now says \"print cost every 100 training iterations\" instead of \"examples\".\n",
    "* Fixed grammar in the comments.\n",
    "* Y_prediction_test variable name is used consistently.\n",
    "* Plot's axis label now says \"iterations (hundred)\" instead of \"iterations\".\n",
    "* When testing the model, the test image is normalized by dividing by 255."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## 1 - Packages ##\n",
    "\n",
    "First, let's run the cell below to import all the packages that you will need during this assignment. \n",
    "- [numpy](www.numpy.org) is the fundamental package for scientific computing with Python.\n",
    "- [h5py](http://www.h5py.org) is a common package to interact with a dataset that is stored on an H5 file.\n",
    "- [matplotlib](http://matplotlib.org) is a famous library to plot graphs in Python.\n",
    "- [PIL](http://www.pythonware.com/products/pil/) and [scipy](https://www.scipy.org/) are used here to test your model with your own picture at the end."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "import numpy as np\n",
    "import matplotlib.pyplot as plt\n",
    "import h5py\n",
    "import scipy\n",
    "from PIL import Image\n",
    "from scipy import ndimage\n",
    "from lr_utils import load_dataset\n",
    "\n",
    "%matplotlib inline"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## 2 - Overview of the Problem set ##\n",
    "\n",
    "**Problem Statement**: You are given a dataset (\"data.h5\") containing:\n",
    "    - a training set of m_train images labeled as cat (y=1) or non-cat (y=0)\n",
    "    - a test set of m_test images labeled as cat or non-cat\n",
    "    - each image is of shape (num_px, num_px, 3) where 3 is for the 3 channels (RGB). Thus, each image is square (height = num_px) and (width = num_px).\n",
    "\n",
    "You will build a simple image-recognition algorithm that can correctly classify pictures as cat or non-cat.\n",
    "\n",
    "Let's get more familiar with the dataset. Load the data by running the following code."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "# Loading the data (cat/non-cat)\n",
    "train_set_x_orig, train_set_y, test_set_x_orig, test_set_y, classes = load_dataset()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "We added \"_orig\" at the end of image datasets (train and test) because we are going to preprocess them. After preprocessing, we will end up with train_set_x and test_set_x (the labels train_set_y and test_set_y don't need any preprocessing).\n",
    "\n",
    "Each line of your train_set_x_orig and test_set_x_orig is an array representing an image. You can visualize an example by running the following code. Feel free also to change the `index` value and re-run to see other images. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "y = [1], it's a 'cat' picture.\n"
     ]
    },
    {
     "data": {
      "image/png": "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\nK+0AAAAASUVORK5CYII=\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7fcd92e68400>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Example of a picture\n",
    "index = 25\n",
    "plt.imshow(train_set_x_orig[index])\n",
    "print (\"y = \" + str(train_set_y[:, index]) + \", it's a '\" + classes[np.squeeze(train_set_y[:, index])].decode(\"utf-8\") +  \"' picture.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Many software bugs in deep learning come from having matrix/vector dimensions that don't fit. If you can keep your matrix/vector dimensions straight you will go a long way toward eliminating many bugs. \n",
    "\n",
    "**Exercise:** Find the values for:\n",
    "    - m_train (number of training examples)\n",
    "    - m_test (number of test examples)\n",
    "    - num_px (= height = width of a training image)\n",
    "Remember that `train_set_x_orig` is a numpy-array of shape (m_train, num_px, num_px, 3). For instance, you can access `m_train` by writing `train_set_x_orig.shape[0]`."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Number of training examples: m_train = 209\n",
      "Number of testing examples: m_test = 50\n",
      "Height/Width of each image: num_px = 64\n",
      "Each image is of size: (64, 64, 3)\n",
      "train_set_x shape: (209, 64, 64, 3)\n",
      "train_set_y shape: (1, 209)\n",
      "test_set_x shape: (50, 64, 64, 3)\n",
      "test_set_y shape: (1, 50)\n"
     ]
    }
   ],
   "source": [
    "### START CODE HERE ### (≈ 3 lines of code)\n",
    "m_train = train_set_x_orig.shape[0]\n",
    "m_test = test_set_x_orig.shape[0]\n",
    "num_px = train_set_x_orig.shape[1]\n",
    "### END CODE HERE ###\n",
    "\n",
    "print (\"Number of training examples: m_train = \" + str(m_train))\n",
    "print (\"Number of testing examples: m_test = \" + str(m_test))\n",
    "print (\"Height/Width of each image: num_px = \" + str(num_px))\n",
    "print (\"Each image is of size: (\" + str(num_px) + \", \" + str(num_px) + \", 3)\")\n",
    "print (\"train_set_x shape: \" + str(train_set_x_orig.shape))\n",
    "print (\"train_set_y shape: \" + str(train_set_y.shape))\n",
    "print (\"test_set_x shape: \" + str(test_set_x_orig.shape))\n",
    "print (\"test_set_y shape: \" + str(test_set_y.shape))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Expected Output for m_train, m_test and num_px**: \n",
    "<table style=\"width:15%\">\n",
    "  <tr>\n",
    "    <td>**m_train**</td>\n",
    "    <td> 209 </td> \n",
    "  </tr>\n",
    "  \n",
    "  <tr>\n",
    "    <td>**m_test**</td>\n",
    "    <td> 50 </td> \n",
    "  </tr>\n",
    "  \n",
    "  <tr>\n",
    "    <td>**num_px**</td>\n",
    "    <td> 64 </td> \n",
    "  </tr>\n",
    "  \n",
    "</table>\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "For convenience, you should now reshape images of shape (num_px, num_px, 3) in a numpy-array of shape (num_px $*$ num_px $*$ 3, 1). After this, our training (and test) dataset is a numpy-array where each column represents a flattened image. There should be m_train (respectively m_test) columns.\n",
    "\n",
    "**Exercise:** Reshape the training and test data sets so that images of size (num_px, num_px, 3) are flattened into single vectors of shape (num\\_px $*$ num\\_px $*$ 3, 1).\n",
    "\n",
    "A trick when you want to flatten a matrix X of shape (a,b,c,d) to a matrix X_flatten of shape (b$*$c$*$d, a) is to use: \n",
    "```python\n",
    "X_flatten = X.reshape(X.shape[0], -1).T      # X.T is the transpose of X\n",
    "```"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "train_set_x_flatten shape: (12288, 209)\n",
      "train_set_y shape: (1, 209)\n",
      "test_set_x_flatten shape: (12288, 50)\n",
      "test_set_y shape: (1, 50)\n",
      "sanity check after reshaping: [17 31 56 22 33]\n"
     ]
    }
   ],
   "source": [
    "# Reshape the training and test examples\n",
    "\n",
    "### START CODE HERE ### (≈ 2 lines of code)\n",
    "train_set_x_flatten = train_set_x_orig.reshape(m_train, -1).T\n",
    "test_set_x_flatten = test_set_x_orig.reshape(m_test,-1).T\n",
    "### END CODE HERE ###\n",
    "\n",
    "print (\"train_set_x_flatten shape: \" + str(train_set_x_flatten.shape))\n",
    "print (\"train_set_y shape: \" + str(train_set_y.shape))\n",
    "print (\"test_set_x_flatten shape: \" + str(test_set_x_flatten.shape))\n",
    "print (\"test_set_y shape: \" + str(test_set_y.shape))\n",
    "print (\"sanity check after reshaping: \" + str(train_set_x_flatten[0:5,0]))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Expected Output**: \n",
    "\n",
    "<table style=\"width:35%\">\n",
    "  <tr>\n",
    "    <td>**train_set_x_flatten shape**</td>\n",
    "    <td> (12288, 209)</td> \n",
    "  </tr>\n",
    "  <tr>\n",
    "    <td>**train_set_y shape**</td>\n",
    "    <td>(1, 209)</td> \n",
    "  </tr>\n",
    "  <tr>\n",
    "    <td>**test_set_x_flatten shape**</td>\n",
    "    <td>(12288, 50)</td> \n",
    "  </tr>\n",
    "  <tr>\n",
    "    <td>**test_set_y shape**</td>\n",
    "    <td>(1, 50)</td> \n",
    "  </tr>\n",
    "  <tr>\n",
    "  <td>**sanity check after reshaping**</td>\n",
    "  <td>[17 31 56 22 33]</td> \n",
    "  </tr>\n",
    "</table>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "To represent color images, the red, green and blue channels (RGB) must be specified for each pixel, and so the pixel value is actually a vector of three numbers ranging from 0 to 255.\n",
    "\n",
    "One common preprocessing step in machine learning is to center and standardize your dataset, meaning that you substract the mean of the whole numpy array from each example, and then divide each example by the standard deviation of the whole numpy array. But for picture datasets, it is simpler and more convenient and works almost as well to just divide every row of the dataset by 255 (the maximum value of a pixel channel).\n",
    "\n",
    "<!-- During the training of your model, you're going to multiply weights and add biases to some initial inputs in order to observe neuron activations. Then you backpropogate with the gradients to train the model. But, it is extremely important for each feature to have a similar range such that our gradients don't explode. You will see that more in detail later in the lectures. !--> \n",
    "\n",
    "Let's standardize our dataset."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 11,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "train_set_x = train_set_x_flatten/255.\n",
    "test_set_x = test_set_x_flatten/255."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<font color='blue'>\n",
    "**What you need to remember:**\n",
    "\n",
    "Common steps for pre-processing a new dataset are:\n",
    "- Figure out the dimensions and shapes of the problem (m_train, m_test, num_px, ...)\n",
    "- Reshape the datasets such that each example is now a vector of size (num_px \\* num_px \\* 3, 1)\n",
    "- \"Standardize\" the data"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## 3 - General Architecture of the learning algorithm ##\n",
    "\n",
    "It's time to design a simple algorithm to distinguish cat images from non-cat images.\n",
    "\n",
    "You will build a Logistic Regression, using a Neural Network mindset. The following Figure explains why **Logistic Regression is actually a very simple Neural Network!**\n",
    "\n",
    "<img src=\"images/LogReg_kiank.png\" style=\"width:650px;height:400px;\">\n",
    "\n",
    "**Mathematical expression of the algorithm**:\n",
    "\n",
    "For one example $x^{(i)}$:\n",
    "$$z^{(i)} = w^T x^{(i)} + b \\tag{1}$$\n",
    "$$\\hat{y}^{(i)} = a^{(i)} = sigmoid(z^{(i)})\\tag{2}$$ \n",
    "$$ \\mathcal{L}(a^{(i)}, y^{(i)}) =  - y^{(i)}  \\log(a^{(i)}) - (1-y^{(i)} )  \\log(1-a^{(i)})\\tag{3}$$\n",
    "\n",
    "The cost is then computed by summing over all training examples:\n",
    "$$ J = \\frac{1}{m} \\sum_{i=1}^m \\mathcal{L}(a^{(i)}, y^{(i)})\\tag{6}$$\n",
    "\n",
    "**Key steps**:\n",
    "In this exercise, you will carry out the following steps: \n",
    "    - Initialize the parameters of the model\n",
    "    - Learn the parameters for the model by minimizing the cost  \n",
    "    - Use the learned parameters to make predictions (on the test set)\n",
    "    - Analyse the results and conclude"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## 4 - Building the parts of our algorithm ## \n",
    "\n",
    "The main steps for building a Neural Network are:\n",
    "1. Define the model structure (such as number of input features) \n",
    "2. Initialize the model's parameters\n",
    "3. Loop:\n",
    "    - Calculate current loss (forward propagation)\n",
    "    - Calculate current gradient (backward propagation)\n",
    "    - Update parameters (gradient descent)\n",
    "\n",
    "You often build 1-3 separately and integrate them into one function we call `model()`.\n",
    "\n",
    "### 4.1 - Helper functions\n",
    "\n",
    "**Exercise**: Using your code from \"Python Basics\", implement `sigmoid()`. As you've seen in the figure above, you need to compute $sigmoid( w^T x + b) = \\frac{1}{1 + e^{-(w^T x + b)}}$ to make predictions. Use np.exp()."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 12,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "# GRADED FUNCTION: sigmoid\n",
    "\n",
    "def sigmoid(z):\n",
    "    \"\"\"\n",
    "    Compute the sigmoid of z\n",
    "\n",
    "    Arguments:\n",
    "    z -- A scalar or numpy array of any size.\n",
    "\n",
    "    Return:\n",
    "    s -- sigmoid(z)\n",
    "    \"\"\"\n",
    "\n",
    "    ### START CODE HERE ### (≈ 1 line of code)\n",
    "    s = 1/(1+np.exp(-z))\n",
    "    ### END CODE HERE ###\n",
    "    \n",
    "    return s"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 13,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "sigmoid([0, 2]) = [ 0.5         0.88079708]\n"
     ]
    }
   ],
   "source": [
    "print (\"sigmoid([0, 2]) = \" + str(sigmoid(np.array([0,2]))))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Expected Output**: \n",
    "\n",
    "<table>\n",
    "  <tr>\n",
    "    <td>**sigmoid([0, 2])**</td>\n",
    "    <td> [ 0.5         0.88079708]</td> \n",
    "  </tr>\n",
    "</table>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### 4.2 - Initializing parameters\n",
    "\n",
    "**Exercise:** Implement parameter initialization in the cell below. You have to initialize w as a vector of zeros. If you don't know what numpy function to use, look up np.zeros() in the Numpy library's documentation."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 14,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "# GRADED FUNCTION: initialize_with_zeros\n",
    "\n",
    "def initialize_with_zeros(dim):\n",
    "    \"\"\"\n",
    "    This function creates a vector of zeros of shape (dim, 1) for w and initializes b to 0.\n",
    "    \n",
    "    Argument:\n",
    "    dim -- size of the w vector we want (or number of parameters in this case)\n",
    "    \n",
    "    Returns:\n",
    "    w -- initialized vector of shape (dim, 1)\n",
    "    b -- initialized scalar (corresponds to the bias)\n",
    "    \"\"\"\n",
    "    \n",
    "    ### START CODE HERE ### (≈ 1 line of code)\n",
    "    w = np.zeros((dim,1))\n",
    "    b = 0\n",
    "    ### END CODE HERE ###\n",
    "\n",
    "    assert(w.shape == (dim, 1))\n",
    "    assert(isinstance(b, float) or isinstance(b, int))\n",
    "    \n",
    "    return w, b"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 15,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "w = [[ 0.]\n",
      " [ 0.]]\n",
      "b = 0\n"
     ]
    }
   ],
   "source": [
    "dim = 2\n",
    "w, b = initialize_with_zeros(dim)\n",
    "print (\"w = \" + str(w))\n",
    "print (\"b = \" + str(b))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Expected Output**: \n",
    "\n",
    "\n",
    "<table style=\"width:15%\">\n",
    "    <tr>\n",
    "        <td>  ** w **  </td>\n",
    "        <td> [[ 0.]\n",
    " [ 0.]] </td>\n",
    "    </tr>\n",
    "    <tr>\n",
    "        <td>  ** b **  </td>\n",
    "        <td> 0 </td>\n",
    "    </tr>\n",
    "</table>\n",
    "\n",
    "For image inputs, w will be of shape (num_px $\\times$ num_px $\\times$ 3, 1)."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### 4.3 - Forward and Backward propagation\n",
    "\n",
    "Now that your parameters are initialized, you can do the \"forward\" and \"backward\" propagation steps for learning the parameters.\n",
    "\n",
    "**Exercise:** Implement a function `propagate()` that computes the cost function and its gradient.\n",
    "\n",
    "**Hints**:\n",
    "\n",
    "Forward Propagation:\n",
    "- You get X\n",
    "- You compute $A = \\sigma(w^T X + b) = (a^{(1)}, a^{(2)}, ..., a^{(m-1)}, a^{(m)})$\n",
    "- You calculate the cost function: $J = -\\frac{1}{m}\\sum_{i=1}^{m}y^{(i)}\\log(a^{(i)})+(1-y^{(i)})\\log(1-a^{(i)})$\n",
    "\n",
    "Here are the two formulas you will be using: \n",
    "\n",
    "$$ \\frac{\\partial J}{\\partial w} = \\frac{1}{m}X(A-Y)^T\\tag{7}$$\n",
    "$$ \\frac{\\partial J}{\\partial b} = \\frac{1}{m} \\sum_{i=1}^m (a^{(i)}-y^{(i)})\\tag{8}$$"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 16,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "# GRADED FUNCTION: propagate\n",
    "\n",
    "def propagate(w, b, X, Y):\n",
    "    \"\"\"\n",
    "    Implement the cost function and its gradient for the propagation explained above\n",
    "\n",
    "    Arguments:\n",
    "    w -- weights, a numpy array of size (num_px * num_px * 3, 1)\n",
    "    b -- bias, a scalar\n",
    "    X -- data of size (num_px * num_px * 3, number of examples)\n",
    "    Y -- true \"label\" vector (containing 0 if non-cat, 1 if cat) of size (1, number of examples)\n",
    "\n",
    "    Return:\n",
    "    cost -- negative log-likelihood cost for logistic regression\n",
    "    dw -- gradient of the loss with respect to w, thus same shape as w\n",
    "    db -- gradient of the loss with respect to b, thus same shape as b\n",
    "    \n",
    "    Tips:\n",
    "    - Write your code step by step for the propagation. np.log(), np.dot()\n",
    "    \"\"\"\n",
    "    \n",
    "    m = X.shape[1]\n",
    "    \n",
    "    # FORWARD PROPAGATION (FROM X TO COST)\n",
    "    ### START CODE HERE ### (≈ 2 lines of code)\n",
    "    A = sigmoid(np.dot(w.T,X)+b)                                    # compute activation\n",
    "    cost = (-1/m)*np.sum((Y*np.log(A))+(1-Y)*np.log(1-A))                                # compute cost\n",
    "    ### END CODE HERE ###\n",
    "    \n",
    "    # BACKWARD PROPAGATION (TO FIND GRAD)\n",
    "    ### START CODE HERE ### (≈ 2 lines of code)\n",
    "    dw = (1/m)*np.dot(X,(A-Y).T)\n",
    "    db = (1/m)*np.sum(A-Y)\n",
    "    ### END CODE HERE ###\n",
    "\n",
    "    assert(dw.shape == w.shape)\n",
    "    assert(db.dtype == float)\n",
    "    cost = np.squeeze(cost)\n",
    "    assert(cost.shape == ())\n",
    "    \n",
    "    grads = {\"dw\": dw,\n",
    "             \"db\": db}\n",
    "    \n",
    "    return grads, cost"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 34,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "dw = [[ 0.99845601]\n",
      " [ 2.39507239]]\n",
      "db = 0.00145557813678\n",
      "cost = 5.80154531939\n"
     ]
    }
   ],
   "source": [
    "w, b, X, Y = np.array([[1.],[2.]]), 2., np.array([[1.,2.,-1.],[3.,4.,-3.2]]), np.array([[1,0,1]])\n",
    "grads, cost = propagate(w, b, X, Y)\n",
    "print (\"dw = \" + str(grads[\"dw\"]))\n",
    "print (\"db = \" + str(grads[\"db\"]))\n",
    "print (\"cost = \" + str(cost))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Expected Output**:\n",
    "\n",
    "<table style=\"width:50%\">\n",
    "    <tr>\n",
    "        <td>  ** dw **  </td>\n",
    "      <td> [[ 0.99845601]\n",
    "     [ 2.39507239]]</td>\n",
    "    </tr>\n",
    "    <tr>\n",
    "        <td>  ** db **  </td>\n",
    "        <td> 0.00145557813678 </td>\n",
    "    </tr>\n",
    "    <tr>\n",
    "        <td>  ** cost **  </td>\n",
    "        <td> 5.801545319394553 </td>\n",
    "    </tr>\n",
    "\n",
    "</table>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### 4.4 - Optimization\n",
    "- You have initialized your parameters.\n",
    "- You are also able to compute a cost function and its gradient.\n",
    "- Now, you want to update the parameters using gradient descent.\n",
    "\n",
    "**Exercise:** Write down the optimization function. The goal is to learn $w$ and $b$ by minimizing the cost function $J$. For a parameter $\\theta$, the update rule is $ \\theta = \\theta - \\alpha \\text{ } d\\theta$, where $\\alpha$ is the learning rate."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 17,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "# GRADED FUNCTION: optimize\n",
    "\n",
    "def optimize(w, b, X, Y, num_iterations, learning_rate, print_cost = False):\n",
    "    \"\"\"\n",
    "    This function optimizes w and b by running a gradient descent algorithm\n",
    "    \n",
    "    Arguments:\n",
    "    w -- weights, a numpy array of size (num_px * num_px * 3, 1)\n",
    "    b -- bias, a scalar\n",
    "    X -- data of shape (num_px * num_px * 3, number of examples)\n",
    "    Y -- true \"label\" vector (containing 0 if non-cat, 1 if cat), of shape (1, number of examples)\n",
    "    num_iterations -- number of iterations of the optimization loop\n",
    "    learning_rate -- learning rate of the gradient descent update rule\n",
    "    print_cost -- True to print the loss every 100 steps\n",
    "    \n",
    "    Returns:\n",
    "    params -- dictionary containing the weights w and bias b\n",
    "    grads -- dictionary containing the gradients of the weights and bias with respect to the cost function\n",
    "    costs -- list of all the costs computed during the optimization, this will be used to plot the learning curve.\n",
    "    \n",
    "    Tips:\n",
    "    You basically need to write down two steps and iterate through them:\n",
    "        1) Calculate the cost and the gradient for the current parameters. Use propagate().\n",
    "        2) Update the parameters using gradient descent rule for w and b.\n",
    "    \"\"\"\n",
    "    \n",
    "    costs = []\n",
    "    \n",
    "    for i in range(num_iterations):\n",
    "        \n",
    "        \n",
    "        # Cost and gradient calculation (≈ 1-4 lines of code)\n",
    "        ### START CODE HERE ### \n",
    "        grads, cost = propagate(w,b,X,Y)\n",
    "        ### END CODE HERE ###\n",
    "        \n",
    "        # Retrieve derivatives from grads\n",
    "        dw = grads[\"dw\"]\n",
    "        db = grads[\"db\"]\n",
    "        \n",
    "        # update rule (≈ 2 lines of code)\n",
    "        ### START CODE HERE ###\n",
    "        w = w - learning_rate*dw\n",
    "        b = b - learning_rate*db\n",
    "        ### END CODE HERE ###\n",
    "        \n",
    "        # Record the costs\n",
    "        if i % 100 == 0:\n",
    "            costs.append(cost)\n",
    "        \n",
    "        # Print the cost every 100 training iterations\n",
    "        if print_cost and i % 100 == 0:\n",
    "            print (\"Cost after iteration %i: %f\" %(i, cost))\n",
    "    \n",
    "    params = {\"w\": w,\n",
    "              \"b\": b}\n",
    "    \n",
    "    grads = {\"dw\": dw,\n",
    "             \"db\": db}\n",
    "    \n",
    "    return params, grads, costs"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 36,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "w = [[ 0.19033591]\n",
      " [ 0.12259159]]\n",
      "b = 1.92535983008\n",
      "dw = [[ 0.67752042]\n",
      " [ 1.41625495]]\n",
      "db = 0.219194504541\n"
     ]
    }
   ],
   "source": [
    "params, grads, costs = optimize(w, b, X, Y, num_iterations= 100, learning_rate = 0.009, print_cost = False)\n",
    "\n",
    "print (\"w = \" + str(params[\"w\"]))\n",
    "print (\"b = \" + str(params[\"b\"]))\n",
    "print (\"dw = \" + str(grads[\"dw\"]))\n",
    "print (\"db = \" + str(grads[\"db\"]))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Expected Output**: \n",
    "\n",
    "<table style=\"width:40%\">\n",
    "    <tr>\n",
    "       <td> **w** </td>\n",
    "       <td>[[ 0.19033591]\n",
    " [ 0.12259159]] </td>\n",
    "    </tr>\n",
    "    \n",
    "    <tr>\n",
    "       <td> **b** </td>\n",
    "       <td> 1.92535983008 </td>\n",
    "    </tr>\n",
    "    <tr>\n",
    "       <td> **dw** </td>\n",
    "       <td> [[ 0.67752042]\n",
    " [ 1.41625495]] </td>\n",
    "    </tr>\n",
    "    <tr>\n",
    "       <td> **db** </td>\n",
    "       <td> 0.219194504541 </td>\n",
    "    </tr>\n",
    "\n",
    "</table>"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Exercise:** The previous function will output the learned w and b. We are able to use w and b to predict the labels for a dataset X. Implement the `predict()` function. There are two steps to computing predictions:\n",
    "\n",
    "1. Calculate $\\hat{Y} = A = \\sigma(w^T X + b)$\n",
    "\n",
    "2. Convert the entries of a into 0 (if activation <= 0.5) or 1 (if activation > 0.5), stores the predictions in a vector `Y_prediction`. If you wish, you can use an `if`/`else` statement in a `for` loop (though there is also a way to vectorize this). "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 18,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "# GRADED FUNCTION: predict\n",
    "\n",
    "def predict(w, b, X):\n",
    "    '''\n",
    "    Predict whether the label is 0 or 1 using learned logistic regression parameters (w, b)\n",
    "    \n",
    "    Arguments:\n",
    "    w -- weights, a numpy array of size (num_px * num_px * 3, 1)\n",
    "    b -- bias, a scalar\n",
    "    X -- data of size (num_px * num_px * 3, number of examples)\n",
    "    \n",
    "    Returns:\n",
    "    Y_prediction -- a numpy array (vector) containing all predictions (0/1) for the examples in X\n",
    "    '''\n",
    "    \n",
    "    m = X.shape[1]\n",
    "    Y_prediction = np.zeros((1,m))\n",
    "    w = w.reshape(X.shape[0], 1)\n",
    "    \n",
    "    # Compute vector \"A\" predicting the probabilities of a cat being present in the picture\n",
    "    ### START CODE HERE ### (≈ 1 line of code)\n",
    "    A = sigmoid(np.dot(w.T,X)+b)\n",
    "    ### END CODE HERE ###\n",
    "    \n",
    "    for i in range(A.shape[1]):\n",
    "        \n",
    "        # Convert probabilities A[0,i] to actual predictions p[0,i]\n",
    "        ### START CODE HERE ### (≈ 4 lines of code)\n",
    "        if A[0,i] > 0.5:\n",
    "            Y_prediction[0,i]=1\n",
    "        else:\n",
    "            Y_prediction[0,i]=0\n",
    "        ### END CODE HERE ###\n",
    "    \n",
    "    assert(Y_prediction.shape == (1, m))\n",
    "    \n",
    "    return Y_prediction"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 19,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "predictions = [[ 1.  1.  0.]]\n"
     ]
    }
   ],
   "source": [
    "w = np.array([[0.1124579],[0.23106775]])\n",
    "b = -0.3\n",
    "X = np.array([[1.,-1.1,-3.2],[1.2,2.,0.1]])\n",
    "print (\"predictions = \" + str(predict(w, b, X)))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Expected Output**: \n",
    "\n",
    "<table style=\"width:30%\">\n",
    "    <tr>\n",
    "         <td>\n",
    "             **predictions**\n",
    "         </td>\n",
    "          <td>\n",
    "            [[ 1.  1.  0.]]\n",
    "         </td>  \n",
    "   </tr>\n",
    "\n",
    "</table>\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<font color='blue'>\n",
    "**What to remember:**\n",
    "You've implemented several functions that:\n",
    "- Initialize (w,b)\n",
    "- Optimize the loss iteratively to learn parameters (w,b):\n",
    "    - computing the cost and its gradient \n",
    "    - updating the parameters using gradient descent\n",
    "- Use the learned (w,b) to predict the labels for a given set of examples"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## 5 - Merge all functions into a model ##\n",
    "\n",
    "You will now see how the overall model is structured by putting together all the building blocks (functions implemented in the previous parts) together, in the right order.\n",
    "\n",
    "**Exercise:** Implement the model function. Use the following notation:\n",
    "    - Y_prediction_test for your predictions on the test set\n",
    "    - Y_prediction_train for your predictions on the train set\n",
    "    - w, costs, grads for the outputs of optimize()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 24,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "# GRADED FUNCTION: model\n",
    "\n",
    "def model(X_train, Y_train, X_test, Y_test, num_iterations = 2000, learning_rate = 0.5, print_cost = False):\n",
    "    \"\"\"\n",
    "    Builds the logistic regression model by calling the function you've implemented previously\n",
    "    \n",
    "    Arguments:\n",
    "    X_train -- training set represented by a numpy array of shape (num_px * num_px * 3, m_train)\n",
    "    Y_train -- training labels represented by a numpy array (vector) of shape (1, m_train)\n",
    "    X_test -- test set represented by a numpy array of shape (num_px * num_px * 3, m_test)\n",
    "    Y_test -- test labels represented by a numpy array (vector) of shape (1, m_test)\n",
    "    num_iterations -- hyperparameter representing the number of iterations to optimize the parameters\n",
    "    learning_rate -- hyperparameter representing the learning rate used in the update rule of optimize()\n",
    "    print_cost -- Set to true to print the cost every 100 iterations\n",
    "    \n",
    "    Returns:\n",
    "    d -- dictionary containing information about the model.\n",
    "    \"\"\"\n",
    "    \n",
    "    ### START CODE HERE ###\n",
    "    \n",
    "    # initialize parameters with zeros (≈ 1 line of code)\n",
    "    w, b = initialize_with_zeros(X_train.shape[0])\n",
    "    \n",
    "    # Gradient descent (≈ 1 line of code)\n",
    "    parameters, grads, costs = optimize(w, b, X_train, Y_train, num_iterations, learning_rate, print_cost)\n",
    "    \n",
    "    # Retrieve parameters w and b from dictionary \"parameters\"\n",
    "    w = parameters[\"w\"]\n",
    "    b = parameters[\"b\"]\n",
    "    \n",
    "    # Predict test/train set examples (≈ 2 lines of code)\n",
    "    Y_prediction_test = predict(w,b, X_test)\n",
    "    Y_prediction_train = predict(w,b, X_train)\n",
    "    \n",
    "    ### END CODE HERE ###\n",
    "\n",
    "    # Print train/test Errors\n",
    "    print(\"train accuracy: {} %\".format(100 - np.mean(np.abs(Y_prediction_train - Y_train)) * 100))\n",
    "    print(\"test accuracy: {} %\".format(100 - np.mean(np.abs(Y_prediction_test - Y_test)) * 100))\n",
    "\n",
    "    \n",
    "    d = {\"costs\": costs,\n",
    "         \"Y_prediction_test\": Y_prediction_test, \n",
    "         \"Y_prediction_train\" : Y_prediction_train, \n",
    "         \"w\" : w, \n",
    "         \"b\" : b,\n",
    "         \"learning_rate\" : learning_rate,\n",
    "         \"num_iterations\": num_iterations}\n",
    "    \n",
    "    return d"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Run the following cell to train your model."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 25,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Cost after iteration 0: 0.693147\n",
      "Cost after iteration 100: 0.584508\n",
      "Cost after iteration 200: 0.466949\n",
      "Cost after iteration 300: 0.376007\n",
      "Cost after iteration 400: 0.331463\n",
      "Cost after iteration 500: 0.303273\n",
      "Cost after iteration 600: 0.279880\n",
      "Cost after iteration 700: 0.260042\n",
      "Cost after iteration 800: 0.242941\n",
      "Cost after iteration 900: 0.228004\n",
      "Cost after iteration 1000: 0.214820\n",
      "Cost after iteration 1100: 0.203078\n",
      "Cost after iteration 1200: 0.192544\n",
      "Cost after iteration 1300: 0.183033\n",
      "Cost after iteration 1400: 0.174399\n",
      "Cost after iteration 1500: 0.166521\n",
      "Cost after iteration 1600: 0.159305\n",
      "Cost after iteration 1700: 0.152667\n",
      "Cost after iteration 1800: 0.146542\n",
      "Cost after iteration 1900: 0.140872\n",
      "train accuracy: 99.04306220095694 %\n",
      "test accuracy: 70.0 %\n"
     ]
    }
   ],
   "source": [
    "d = model(train_set_x, train_set_y, test_set_x, test_set_y, num_iterations = 2000, learning_rate = 0.005, print_cost = True)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Expected Output**: \n",
    "\n",
    "<table style=\"width:40%\"> \n",
    "\n",
    "    <tr>\n",
    "        <td> **Cost after iteration 0 **  </td> \n",
    "        <td> 0.693147 </td>\n",
    "    </tr>\n",
    "      <tr>\n",
    "        <td> <center> $\\vdots$ </center> </td> \n",
    "        <td> <center> $\\vdots$ </center> </td> \n",
    "    </tr>  \n",
    "    <tr>\n",
    "        <td> **Train Accuracy**  </td> \n",
    "        <td> 99.04306220095694 % </td>\n",
    "    </tr>\n",
    "\n",
    "    <tr>\n",
    "        <td>**Test Accuracy** </td> \n",
    "        <td> 70.0 % </td>\n",
    "    </tr>\n",
    "</table> \n",
    "\n",
    "\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Comment**: Training accuracy is close to 100%. This is a good sanity check: your model is working and has high enough capacity to fit the training data. Test accuracy is 68%. It is actually not bad for this simple model, given the small dataset we used and that logistic regression is a linear classifier. But no worries, you'll build an even better classifier next week!\n",
    "\n",
    "Also, you see that the model is clearly overfitting the training data. Later in this specialization you will learn how to reduce overfitting, for example by using regularization. Using the code below (and changing the `index` variable) you can look at predictions on pictures of the test set."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 44,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "y = 1, you predicted that it is a \"cat\" picture.\n"
     ]
    },
    {
     "data": {
      "image/png": "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\nSUVORK5CYII=\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f6efbce1c50>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Example of a picture that was wrongly classified.\n",
    "index = 1\n",
    "plt.imshow(test_set_x[:,index].reshape((num_px, num_px, 3)))\n",
    "print (\"y = \" + str(test_set_y[0,index]) + \", you predicted that it is a \\\"\" + classes[d[\"Y_prediction_test\"][0,index]].decode(\"utf-8\") +  \"\\\" picture.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Let's also plot the cost function and the gradients."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 45,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f6efd666d30>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Plot learning curve (with costs)\n",
    "costs = np.squeeze(d['costs'])\n",
    "plt.plot(costs)\n",
    "plt.ylabel('cost')\n",
    "plt.xlabel('iterations (per hundreds)')\n",
    "plt.title(\"Learning rate =\" + str(d[\"learning_rate\"]))\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Interpretation**:\n",
    "You can see the cost decreasing. It shows that the parameters are being learned. However, you see that you could train the model even more on the training set. Try to increase the number of iterations in the cell above and rerun the cells. You might see that the training set accuracy goes up, but the test set accuracy goes down. This is called overfitting. "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## 6 - Further analysis (optional/ungraded exercise) ##\n",
    "\n",
    "Congratulations on building your first image classification model. Let's analyze it further, and examine possible choices for the learning rate $\\alpha$. "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "#### Choice of learning rate ####\n",
    "\n",
    "**Reminder**:\n",
    "In order for Gradient Descent to work you must choose the learning rate wisely. The learning rate $\\alpha$  determines how rapidly we update the parameters. If the learning rate is too large we may \"overshoot\" the optimal value. Similarly, if it is too small we will need too many iterations to converge to the best values. That's why it is crucial to use a well-tuned learning rate.\n",
    "\n",
    "Let's compare the learning curve of our model with several choices of learning rates. Run the cell below. This should take about 1 minute. Feel free also to try different values than the three we have initialized the `learning_rates` variable to contain, and see what happens. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 46,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "learning rate is: 0.01\n",
      "train accuracy: 99.52153110047847 %\n",
      "test accuracy: 68.0 %\n",
      "\n",
      "-------------------------------------------------------\n",
      "\n",
      "learning rate is: 0.001\n",
      "train accuracy: 88.99521531100478 %\n",
      "test accuracy: 64.0 %\n",
      "\n",
      "-------------------------------------------------------\n",
      "\n",
      "learning rate is: 0.0001\n",
      "train accuracy: 68.42105263157895 %\n",
      "test accuracy: 36.0 %\n",
      "\n",
      "-------------------------------------------------------\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f6efd59ae48>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "learning_rates = [0.01, 0.001, 0.0001]\n",
    "models = {}\n",
    "for i in learning_rates:\n",
    "    print (\"learning rate is: \" + str(i))\n",
    "    models[str(i)] = model(train_set_x, train_set_y, test_set_x, test_set_y, num_iterations = 1500, learning_rate = i, print_cost = False)\n",
    "    print ('\\n' + \"-------------------------------------------------------\" + '\\n')\n",
    "\n",
    "for i in learning_rates:\n",
    "    plt.plot(np.squeeze(models[str(i)][\"costs\"]), label= str(models[str(i)][\"learning_rate\"]))\n",
    "\n",
    "plt.ylabel('cost')\n",
    "plt.xlabel('iterations (hundreds)')\n",
    "\n",
    "legend = plt.legend(loc='upper center', shadow=True)\n",
    "frame = legend.get_frame()\n",
    "frame.set_facecolor('0.90')\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "**Interpretation**: \n",
    "- Different learning rates give different costs and thus different predictions results.\n",
    "- If the learning rate is too large (0.01), the cost may oscillate up and down. It may even diverge (though in this example, using 0.01 still eventually ends up at a good value for the cost). \n",
    "- A lower cost doesn't mean a better model. You have to check if there is possibly overfitting. It happens when the training accuracy is a lot higher than the test accuracy.\n",
    "- In deep learning, we usually recommend that you: \n",
    "    - Choose the learning rate that better minimizes the cost function.\n",
    "    - If your model overfits, use other techniques to reduce overfitting. (We'll talk about this in later videos.) \n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## 7 - Test with your own image (optional/ungraded exercise) ##\n",
    "\n",
    "Congratulations on finishing this assignment. You can use your own image and see the output of your model. To do that:\n",
    "    1. Click on \"File\" in the upper bar of this notebook, then click \"Open\" to go on your Coursera Hub.\n",
    "    2. Add your image to this Jupyter Notebook's directory, in the \"images\" folder\n",
    "    3. Change your image's name in the following code\n",
    "    4. Run the code and check if the algorithm is right (1 = cat, 0 = non-cat)!"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 51,
   "metadata": {
    "scrolled": false
   },
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "y = 1.0, your algorithm predicts a \"cat\" picture.\n"
     ]
    },
    {
     "data": {
      "image/png": "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\ndQAAAABJRU5ErkJggg==\n",
      "text/plain": [
       "<matplotlib.figure.Figure at 0x7f6efbd08438>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "## START CODE HERE ## (PUT YOUR IMAGE NAME) \n",
    "my_image = \"kitty.jpeg\"   # change this to the name of your image file \n",
    "## END CODE HERE ##\n",
    "\n",
    "# We preprocess the image to fit your algorithm.\n",
    "fname = \"images/\" + my_image\n",
    "image = np.array(ndimage.imread(fname, flatten=False))\n",
    "image = image/255.\n",
    "my_image = scipy.misc.imresize(image, size=(num_px,num_px)).reshape((1, num_px*num_px*3)).T\n",
    "my_predicted_image = predict(d[\"w\"], d[\"b\"], my_image)\n",
    "\n",
    "plt.imshow(image)\n",
    "print(\"y = \" + str(np.squeeze(my_predicted_image)) + \", your algorithm predicts a \\\"\" + classes[int(np.squeeze(my_predicted_image)),].decode(\"utf-8\") +  \"\\\" picture.\")"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "<font color='blue'>\n",
    "**What to remember from this assignment:**\n",
    "1. Preprocessing the dataset is important.\n",
    "2. You implemented each function separately: initialize(), propagate(), optimize(). Then you built a model().\n",
    "3. Tuning the learning rate (which is an example of a \"hyperparameter\") can make a big difference to the algorithm. You will see more examples of this later in this course!"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Finally, if you'd like, we invite you to try different things on this Notebook. Make sure you submit before trying anything. Once you submit, things you can play with include:\n",
    "    - Play with the learning rate and the number of iterations\n",
    "    - Try different initialization methods and compare the results\n",
    "    - Test other preprocessings (center the data, or divide each row by its standard deviation)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Bibliography:\n",
    "- http://www.wildml.com/2015/09/implementing-a-neural-network-from-scratch/\n",
    "- https://stats.stackexchange.com/questions/211436/why-do-we-normalize-images-by-subtracting-the-datasets-image-mean-and-not-the-c"
   ]
  }
 ],
 "metadata": {
  "coursera": {
   "course_slug": "neural-networks-deep-learning",
   "graded_item_id": "XaIWT",
   "launcher_item_id": "zAgPl"
  },
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.6.0"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}
