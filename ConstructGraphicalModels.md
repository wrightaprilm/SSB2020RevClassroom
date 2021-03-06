# Constructing Graphical Models

Jeremy Brown

## Graphical Model Fundamentals

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

### _PROBABILITY: Bernoulli Distribution_

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
Z ~ dnBernoulli(p)
print("Z = " + Z)
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
    Z ~ dnBernoulli(p)
    print("Z = " + Z)
}
```

---

### _PROBABILITY: Binomial Distribution_

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/penny.png"/>
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/penny.png"/>
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/penny.png"/>
</p>

The Binomial distribution is a simple extension of the Bernoulli. However, instead of thinking about a single trial with probability of success _p_, we will now think about _n_ trials and we will keep track of the number of successes, _k_. Therefore, while the Bernoulli distribution only had one parameter, _p_, the Binomial distribution has two: _n_ and _p_. And while the outcome of the Bernoulli was a single 1 (success) or 0 (failure), a Binomial random variable can take any integer value between 0 and _n_.

---

> _Practice Exercise_
>
> Try to construct a stochastic node whose values are Binomially distributed with 6 trials (_n_=6). Remember that the Binomial distribution requires two parameters, so start by creating those as constant nodes. If you need some guidance, you can consult the built-in RevBayes help by typing ? and then the name of the function you're interested in.
> 
> (1) Write Rev code to create a random variable called R and assign it a Binomial distribution with a specific p and n.
> 
> (2) Print out the value of R. Now, reassign the Binomial several times to R and print out the value each time. Does it stay the same?
>
> (3) Now change the value of p and reassign the Binomial several times again, printing out the value each time. How different are these values than with the original value of p?

### Clamped Stochastic Nodes

In order to be able to learn about the unknown values of parameters in our models, we must have a way to include observed data. In the context of graphical models, this is known as clamping. More specifically, we clamp data to stochastic nodes (think of the data as the observed values of a random variable or set of random variables represented by those nodes). For instance, let's say we've flipped a coin 6 times and observed 4 heads. If we define a Binomial random variable with _n_=6, we can then clamp our observed number of "successes" (_k_=4).

```
# Here's a simple example of clamping an observation (# of successes) to a Binomial r.v.
# This should work if you've defined R properly in the exercise above.
R.clamp(4)
print("Clamped value of R is " + R + ".")
```

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/ClampedStochasticNode.jpg"/>
</p>

### Indicating Dependencies

We've already implicitly relied on dependencies when constructing all of our node types, other than constant nodes. In each of the other cases, the value of the node we were creating depended on some other value. To indicate these dependencies in a graphical model, we use arrows. For instance, let's say that we wanted to create a deterministic node whose value depended on a random variable (stochastic node). We would draw an arrow from the stochastic node to the value of the deterministic node. The direction of the arrow indicates the nature of the dependency.

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/Dependency.jpg"/>
</p>

In this case, we actually have two dependencies. Our Bernoulli distribution depends on the fixed probability of success that we indicate with the constant node, _p_, and the deterministic node, _S_, is always 3 times larger than the stochastic value of _Z_. All the nodes in a particular graphical model must be connected by dependencies. 

## Building Graphical Models

Now that we've got our basic building blocks in place (constant, deterministic, stochastic, and clamped stochastic nodes, and dependencies between them), we can begin building interesting models that allow us to learn from data.

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

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/BinomialModel.jpg"/>
</p>


Also note that you can print out the likelihood (i.e., the probability of the data given the current model parameter values) of a clamped stochastic variable.

```
R.probability()
R.lnProbability()
```

While we've now constructed our first complete Bayesian graphical model, we've only been able to clamp a single number to a single node to indicate our observed data. However, in many situations, we want to include a series of observations. To do this, we need one more graphical model tool known as a plate.

### Plates

Plates are constructs that are used to represent repetition in a graphical model. Graphically, they are represented by dashed boxes around nodes (usually clamped stochastic nodes). In the Rev language, plate repetition is usually accomplished with a _for_ loop. Within such _for_ loops, we typically need to first specify the type of stochastic node (e.g., Binomial) and then clamp an observation to each node individually. For instance, instead of a single Binomial outcome (say, flipping one coin 6 times), let's say we flipped 5 coins each 6 times and we think they have the same probability of success. We can set up 5 Binomial random variables and clamp the observed number of successes to each of them.

<p align="center">
  <img src="https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/images/plates.jpg"/>
</p>

As above, you can look at the likelihoods for each of these clamped nodes

```
B[1].probability()
B[2].probability()
B[3].probability()
B[4].probability()
B[5].probability()
```

> _Practice Exercise_
>
> Use the skills we practiced above to construct a model of Normal distribution, which has two parameters: mean and sd (standard deviation). You can use the image below for guidance.
> 
> (1) Start by drawing n values from a Normal distribution with a known mean and sd using: rnorm(n,mean,sd) and saving them in a variable called `data`. These are your simulated "observations".
>
> (2) Now, create a set of Normally distributed random variables with shared, but unknown, mean and sd. Use the name `mu` for the node specifying the mean and `sig` for the node specifying the standard deviation. For now, use a uniform distribution - dnUnif(min,max) - to specify your prior distributions for the mean and standard deviation. Be sure to create each Normal distribution and clamp a value using a for loop (plate).
>
> (3) Print out the current values for mean and sd. What are they? Now, print out the likelihoods for each of the n Normal distributions. Do these likelihoods make sense given the clamped values, and the current mean and sd? (HINT: values further from the mean should have lower likelihood, especially if the standard deviation is small.)

<p align="center">
  <img src="https://github.com/wrightaprilm/SSB2020RevClassroom/blob/master/images/NormalModel.jpg"/>
</p>

## Performing Bayesian Inference

Once we've specified a model and the dependencies among all its parameters, we need to set up the machinery to perform Markov chain Monte Carlo (MCMC) sampling from the posterior distribution. The first thing we do is to create a single model object that contains our entire graphical model. The one argument we pass to the model constructor can be any of the variables we've included in our model. Notice that we use the `=` assignment operator for the model, because it is a workspace variable and not a part of the graphical model itself (i.e., it is not any of the four types of nodes we discussed previously).

```
# For this example, we'll use the Normal model you just set up in the last practice exercise
# Since mu is one node in our graph, we can pass it as an argument to the model constructor
myModel = model(mu)
```

Now that we've set up our model, we need to assign Metropolis-Hastings moves (i.e., proposal distributions) to the variables whose values we want to infer so that we can sample from the posterior distribution. For the sake of convenience, we often create a vector called moves to store a list of all the moves applied to various parameters of our model.

```
# Setting up MCMC moves
moves = VectorMoves()
moves.append( mvSlide(mu,delta=0.1,weight=1) )
moves.append( mvSlide(sig,delta=0.1,weight=1) )

# If there were other parameters to infer, we could add additional moves here.
```

We need to also set up monitors to allow us to keep track of the progress of the MCMC. We can print out our progress to the screen at an interval of our choosing using _printgen_, and we can also have parameter values, posteriors, likelihoods, and priors written to a file.

```
# Setting up MCMC monitors
monitors = VectorMonitors()
monitors.append( mnScreen(printgen=100,mu,sig) ) # This monitor prints to the screen
monitors.append( mnModel(filename="NormalModel_MCMC.log",printgen=100,stochasticOnly=TRUE) ) # To file
```

Now we can take our model object, our moves, and our monitors and combine them into an MCMC object.

`myMCMC = mcmc(myModel, moves, monitors)`

Now that we've put all these pieces together, we can simply tell RevBayes to run our MCMC analysis and for how long! It takes care of all the propose/accept/reject cycles for us.

```
ngens <- 50000
myMCMC.run(ngens)
```

> _Practice Exercise_
>
> (1) Set up an MCMC analysis for your Normal model. Remember, you'll need to (1) simulate your data, (2) set up the graphical model (as covered in the previous practice exercise), then (3) set up your MCMC analysis. We recommend putting all these commands together in a text file that you name with a `.rev` extension.
>
> (2) You can then call all the commands in this file from within RevBayes using the `source()` command [e.g., `source("./myNormalAnalysis.rev")`]. Go ahead and run your MCMC analysis.
> 
> (3) Examine the MCMC output in the log file (NormalModel_MCMC.log), either by eye or using your favorite program (e.g., Tracer). Paying specific attention to the values near the end of the analysis, does the chain sample values that are similar to the mean and sd you used in your simulation? How big is your dataset?
>
> If you have trouble setting up your analysis, you can compare to the analysis file [we created by clicking here](https://raw.githubusercontent.com/wrightaprilm/SSB2020RevClassroom/master/NormalModel_MCMC.rev).
