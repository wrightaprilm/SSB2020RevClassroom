# Introduction to Graphical Models

Jeremy Brown

## Graphical Model Node Types

### Constant Nodes

The first, and simplest, type of node in a graphical model is a constant node. These behave essentially just like standard variables in any programming language, and take fixed values. This is also the only type of node that does not depend on the value of any other node in the graph (it is independent of all other nodes).

```
# x is a constant node with a fixed value
x <- 2.3
```

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/ConstantNode.jpg"/>
</p>

### Deterministic Nodes

The next type of node is a deterministic node. The values of deterministic nodes do depend on the values of other nodes, but in a completely deterministic fashion. For instance, a simple deterministic node, _y_, could always have a value that is twice that of _x_.

```
# Printing the starting value of x
print("x = " + x)

# Defining y using the syntax for a deterministic node
y := 2 * x
print("y = " + y)

# Updating the value of x, but printing the new value of y
# Notice that y itself was never directly changed!
x <- 4.7
print("y = " + y)
```

Notice how the value of _y_ changes as _x_ changes, without having to make a new assignment to _y_.

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/DeterministicNode.jpg"/>
</p>

More complicated deterministic relationships could depend on the values of two or more other nodes.

```
# A more complicated deterministic relationship
k <- 3
m <- 2
n := k^m
print("n = " + n)
```

---

### _ASIDE: Bernoulli Distribution_

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/penny.png"/>
</p>

Perhaps the simplest of all probability distributions is the Bernoulli. The Bernoulli distribution only has two discrete outcomes, 0 and 1. The probability of drawing a 1 (often called a "success") is _p_, and the probability of drawing a 0 is therefore 1-_p_. If we define a Bernoulli-distributed random variable, _X_, we would say that _P_(_X_=1)=_p_ and _P_(_X_=0)=1-_p_.

---

### Stochastic Nodes

The third, and final, type of node in a graphical model is a stochastic node. These nodes represent random variables, whose distribution must be specified when the node is created. The parameters of the stochastic node's distribution can either be fixed or can be determined by other variables (nodes) in the model.

```
# Specifying the probability of success for a Bernoulli with a constant node
p <- 0.5

# Creating a new stochastic node to represent the outcome of a Bernoulli trial
z ~ dnBernoulli(p)
print("z = " + z)
```

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/StochasticNode.jpg"/>
</p>

If you forget what parameters (arguments) are required by any RevBayes distribution, you can call the built-in help by preceding the distribution name with _?_. For instance, to call the help for the Bernoulli distribution, we would use

```
? dnBernoulli
```

Note that each time you assign a distribution to a stochastic node, it draws a new value of the random variable.

```
for (i in 1:10){
    z ~ dnBernoulli(p)
    print("z = " + z)
}
```

---

### _ASIDE: Binomial Distribution_

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/penny.png"/>
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/penny.png"/>
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/penny.png"/>
</p>

The Binomial distribution is a simple extension of the Bernoulli. However, instead of thinking about a single trial with probability of success _p_, we will now think about _n_ trials and we will keep track of the number of successes, _k_. Therefore, while the Bernoulli distribution only had one parameter, _p_, the Binomial distribution has two: _n_ and _p_.

---

> _Practice Exercise_
>
> Try to construct a stochastic node whose values are binomially distributed. Remember that the binomial distribution requires two parameters, so start by creating those as constant nodes. If you need some guidance, you can consult the built-in RevBayes help by typing ? and then the name of the function you're interested in.
> 
> `# Write Rev code to create a random variable called R, assign it a binomial distribution, and draw 10 values from it.`

### Clamped Stochastic Nodes

In order to be able to learn about the unknown values of parameters in our models, we must have a way to include observed data. In the context of graphical models, this is known as clamping. More specifically, we can clamp observations to stochastic nodes (think of the data as the observed values of a random variable or set of random variables).

```
# Here's a simple example of clamping an observation (# of successes) to a Binomial r.v.
# This should work if you've defined R properly in the exercise above.
R.clamp(4)
print("Clamped value of R is " + R + ".")
```

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/ClampedStochasticNode.jpg"/>
</p>


## Building Graphical Models

Now that we've got our four basic building blocks in place (constant, deterministic, stochastic, and clamped stochastic nodes), we can begin building more interesting models.

If we've clamped data to a stochastic node with a Binomial distribution, we can learn something about the probability of success, _p_, that produced those outcomes. However, in this case, we don't want to specify the exact value of _p_ as a constant node - if we did, there would be nothing to learn! Instead, we'll assign a prior probability distribution to _p_, which in this case will be a Beta(1,1).

```
clear()
alpha <- 1
beta <- 1
p ~ dnBeta(alpha,beta)
n <- 10
R ~ dnBinomial(n,p)
R.clamp(4)
```

Also note that you can print out the likelihood (i.e., the probability density of the current parameter value) of a clamped stochastic variable.

```
R.probability()
R.lnProbability()
```

### Plates

DESCRIPTION OF PLATES HERE

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/plates.jpg"/>
</p>

> _Practice Exercise_
>
> Use the skills we practiced above to construct a model of Normal distribution, which has two parameters: mean and sd (standard deviation).
> 
> (1) Start by drawing n values from a Normal distribution with a particular mean and sd using: rnorm(n,mean,sd) and saving them in a variable called data.
>
> (2) Now, create a set of Normal distributions with shared, but unknown, mean and sd. For now, use a uniform distribution - dnUnif(min,max) - to specify your prior distributions for the mean and standard deviation. Be sure to create each Normal distribution and clamp a value using a for loop (plate).
>
> (3) Print out the current values for mean and sd. What are they? Now, print out their probability densities for your Normal distributions. Do these make sense given the data?

## Performing Bayesian Inference

Once we've specified our model and the dependencies among all its parameters, we need to set up the machinery to perform Markov chain Monte Carlo (MCMC) sampling from the posterior distribution. The first thing we do is to create a single model object that contains our entire graphical model. The one argument we pass to the model constructor can be any of the variables we've included in our model. Notice that we use the `=` assignment operator for the model, because it is a workspace variable and not a part of the graphical model itself (i.e., it is not any of the four types of nodes we discussed previously).

```
# For this example, we'll use the Binomial model we just set up
# Since alpha is one node in our graph, we can pass it as an argument to the model constructor
myModel = model(alpha)
```

Now that we've set up our model, we need to assign Metropolis-Hastings moves (i.e., proposal distributions) to some of the variables so that we can sample from the posterior distribution. For the sake of convenience, we often create a vector called moves to store a list of all the moves applied to various parameters of our model.

```
moves[1] = mvSlide(p,delta=0.1,weight=1)

# If there were other parameters to infer, we could add additional moves here.
```

We need to also set up monitors to allow us to keep track of the progress of the MCMC. We can print out our progress to the screen at an interval of our choosing using printgen, and we can also have parameter values, posteriors, likelihoods, and priors written to a file.

```
# This monitor prints to the screen
monitors[1] = mnScreen(printgen=100,p)

# We can also add a monitor that prints to file
# monitors[2] = mnModel(filename="binomialMCMC.log",printgen=1000)
```

Now we can take our model object, our moves, and our monitors and combine them into an MCMC object.

`myMCMC = mcmc(myModel, moves, monitors)`

Now that we've put all these pieces together, we can simply tell RevBayes to run our MCMC analysis and for how long! It takes care of all the propose/accept/reject cycles for us.

`myMCMC.run(10000)`
