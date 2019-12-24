# Introduction to Graphical Models

The first, and simplest, type of node in a graphical model is a constant node. These behave essentially just like standard variables in any programming language, and take fixed values. This is also the only type of node that does not depend on the value of any other node in the graph (it is independent of all other nodes).

```
# x is a constant node with a fixed value
x <- 2.3
```

The next type of node is a deterministic node. The values of deterministic nodes do depend on the values of other nodes, but in a completely deterministic fashion. For instance, a simple deterministic node, y, could always have a value that is twice that of x.

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

Notice how the value of y changes as x changes, without having to make a new assignment to y. More complicated deterministic relationships could depend on the values of two or more other nodes.

```
# A more complicated deterministic relationship
k <- 3
m <- 2
n := k^m
print("n = " + n)
```

The third, and final, type of node in a graphical model is a stochastic node. These nodes represent random variables, whose distribution must be specified when the node is created. The parameters of the stochastic node's distribution can either be fixed or can be determined by other variables (nodes) in the model.

```
# Specifying the probability of success for a Bernoulli with a constant node
p <- 0.5

# Creating a new stochastic node to represent the outcome of a Bernoulli trial
z ~ dnBernoulli(p)
print("z = " + z)
```

Note that each time you assign a distribution to a stochastic node, it draws a new value of the random variable.

```
for (i in 1:10){
    z ~ dnBernoulli(p)
    print("z = " + z)
}
```

> _Practice Exercise_
>
> In the code block below, repeat what we did above for the Bernoulli distribution, but this time construct a stochastic node whose values are binomially distributed. Remember that the binomial distribution requires two parameters. If you need some help, you can consult the list of RevBayes commands and distributions.
> 
> `# Write Rev code to define a binomial distribution and draw 10 values from it.`
>
> `# YOUR CODE HERE`

In order to be able to learn about the unknown values of parameters in our models, we must have a way to include observed data. In the context of graphical models, this is known as clamping. More specifically, we can clamp observations to stochastic nodes (think of the data as the observed values of a set of random variables).

```
# Here's a simple example of clamping an observation (success) to a Bernoulli r.v.
z.clamp(1)
print("Clamped value of z is " + z + ".")
```

Now that we've got our four basic building blocks in place (constant, deterministic, stochastic, and clamped stochastic nodes), we can begin building more interesting models.

If we want to actually learn something about the probability of success, we need to record more than one observation. But since an individual Bernoulli r.v. has only a single outcome, we need to think about recording the outcomes of a series of Bernoulli r.v.s with a shared _p_. We also can't specify the exact value of _p_ - otherwise there would be nothing to learn, because we would already know the value with certainty! Instead, we'll assign a prior probability distribution to _p_, which in this case will be a Beta(1,1).

```
alpha <- 1
beta <- 1
p ~ dnBeta(alpha,beta)
obs <- [1,0,0,1,0,1]
for (i in 1:6){
    obsNodes[i] ~ dnBernoulli(p)
    obsNodes[i].clamp(obs[i])
}
```

We can also write out this same model more simply as a single binomial distribution.

```
clear()
alpha <- 1
beta <- 1
p ~ dnBeta(alpha,beta)
n <- 6
k ~ dnBinomial(n,p)
k.clamp(3)
```

Also note that you can print out the likelihood (i.e., the probability density of the current parameter value) for a given value of a model parameter, conditioned on the current states of other model parameters.

```
p.probability()
p.lnProbability()
```

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
