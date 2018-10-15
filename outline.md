
# Outline

Thank you for being here. This is a very exciting talk to give. ArviZ is an open source library for visualizing and criticising Bayesian anaylsis in Python. We have been working hard on it for about 8 months, and I am really excited to start sharing it with the larger PyData community.

-------------

I have a lot of goals for this talk. One is a light introduction to Bayesian statistics. I urge everyone to go see Sean Talt's master class coming up after this, or Michael Betancourt's tomorrow if you want more details. Another is to encourage everyone to go use ArviZ and then let me know you are using it. Secretly, though, this whole talk is about how choosing the right data structure - xarray - for our problem allowed us to quickly build a powerful library. There is a whole other talk to be had about how this choice also helped us move quickly as a distributed, open source, volunteer team, but we are already being pretty ambitious here.

-------------

  A reasonable definition of Bayesian statistics is that we consider the parameters of a model to be uncertain and the data fixed, as compared to frequentist statistics, where the data is uncertain and parameters fixed. This is Bayes' rule, and it becomes Bayesian inference when we interpret `X` as our data, and `θ` as our parameters.

-------------

Here is Bayes rule written down for this linear regression model. This is a posterior, likelihood, and prior. After observing some data `x` and `y`, we might ask what the model says about our slope and intercept. This is different from an introduction to machine learning in that I am not asking first about applying our model to new data! If you are doing Bayesian inference, you probably care about the uncertainty in your parameters.

-------------

Looking even more closely at linear regression, suppose we have 20 `x` and `y` points. Using `scikit-learn`, we can fit a line to these points. The true slope and intercept are -1 and 1, and `scikit-learn` does a pretty good job recovering those. Let me also step back for a second here and say that machine learning is great, and there are lots of great reasons to use `scikit-learn`.  Fitting these points took 0 seconds on my laptop, and making 1000 new predictions would take an extra 0 seconds.

-------------

What if we care about uncertainty in our parameters? `scikit-learn` has given us *point estimates* for the slope and the intercept. Using a library like PyMC3, we get distributions over the slope and intercept. Suppose for whatever reason that you get paid $100 if the slope is at least 1, and $10 if it is less than 1. You might appreciate having access to this posterior distribution.

-------------

Now if we go back to plotting the line, everything turns into a distribution. Instead of a single line of best fit, we have a distribution of lines. We might pause for a moment and wonder how we store this data - this is 500 `x, y` tuples, and there is secretly a little more structure that makes it 4 dimensional.

 We can also use the model to generate new datasets that we might see in the future, or to decide how surprising our model thinks each data point was. This is how we might do prediction with a Bayesian model.

-------------

Another example we might care about is classification. This is a synthetic dataset of five cluster that overlap. Fitting this with an algorithm like K-Means gives a single classification for each point. We can write down a generative model and fit that to instead have a distribution of labels for each point.

-------------

Now what is ArviZ?

-------------

The easiest way to show it off is as a visualization library. We can start off by generating random numbers in numpy and putting them into a dictionary. So `x` and `y` are independent uniform random numbers, and `z` should be correlated to both of them.

-------------

Here are the individual densities. This is called a *kernel density estimate*, and smooths out a histogram. You get a quick idea of the shape, but notice that it is not necessarily what you would get with more data. This is the same synthetic data with 2 million points instead of 200.

-------------

One of my favorite plots is `plot_pair`. It shows the relationship between two distributions, and we can see pretty clearly the structure we put into our data set. It has a few options that you can switch around depending on what sort of data you are looking at. The 2d kde is the most visually pleasing, but might show you some artifacts.

-------------

A feature of Bayesian inference is that many parameters are high dimensional. We saw an example with the mixture model earlier. This analysis is one from the PyMC3 docs, and estimates some latent measure of offense for European rugby teams. Again, we have a few styles available. This forest plot gives us a glimpse at how Bayesian inference works. We are just getting a numpy array of samples for each parameter. For technical reasons, we repeat this multiple times, and will analyse these chains to make sure our algorithm actually worked. Stepping back and counting, we have 500 draws in 4 chains of 6 teams, making this data 3 dimensional.

 The forest plot is a way to visually check that each chain looks similar. This plot also exposes the effective sample size and R-hat statistic, which are also used to assess convergence. R-hat should be very close to 1, or else you have had some trouble.

-------------

In case you have only two variables, looking at a joint plot will include the marginal distributions. This is a fun example - I spent some time building a library on top of PyMC3 to do something like this, and ArviZ does it for free.

-------------

Besides visualizing your distributions, ArviZ includes diagnostics: ways to make sure the algorithm you were using worked. There are two ways for a model to fail: the model can fail to represent the data, and the algorithm can fail to fit the model.

 The eight schools model is a famous one in Bayesian statistics: the idea is that the same SAT prep course was taught at eight schools, and a mean improvement and standard deviation was reported from these eight schools. There is the idea that we might be able to pool the data from the schools to get a better estimate of individual school performance.

This is an example of a hierarchical model, and it can actually be difficult for algorithms to sample from. Specifically, this centered model will do badly, and the non-centered model does well. Both these models ship with ArviZ: you can `pip install arviz` and recreate all these plots.

I will talk about how to use ArviZ to see that you have a problem, but I encourage you to attend some of those Bayesian master classes to learn why these are problems, and how to fix them.

-------------

First the trace plot, which shows up in many tutorials for PyMC3. This is a little hard to spot, but one of the four chains got stuck. I can single it out to see better. This is a bad sign! You should inspect your model to fix it! The non-centered trace does not have any such obvious artifacts.

-------------

This is a plot of the distribution of energy, which is a quantity computed in modern, gradient based MCMC samplers. The two distributions should match up. These do not, which also indicates a problem. Again, the non-centered model looks better.

-------------

One reason we love gradient based MCMC samplers is that when they have trouble, it is obvious. These divergences are recorded by both PyMC3 and PyStan, and arviz will annotate your pair plot with the divergences.

This problem is very related to the trace plot problem we saw earlier, but it is easier to see here that the problems in the centered model show up when `tau` is small. The non-centered parameterization I am working with also encountered some divergences! I should worry about this as well, but there is no similar pattern as with the non-centered version. Both Pystan and PyMC3 will actually give you automated advice that will fix those divergences.

-------------

Finally, this parallel plot is a quick way to check whether there is some coherence in where divergences happen. Again we see that a small value of `tau` seems to be the problem, but there is no similar pattern in the non-centered model.

-------------

There are also a number of statistical computations built into Arviz! These are less flashy, but arguably more useful. Again, if we were comparing our two models, we might notice that the effective sample size of the non-centered model was much higher than for the centered model.

-------------

To conclude, I want to talk about how ArviZ does what it does. This is a simplified version of a Bayesian workflow. Building a model and inferring parameters is hard. You should use a specialized library like Stan or PyMC3 for those. ArviZ lives at this third node. Our goal with ArviZ is to provide visualization and diagnostic functions for the libraries that did the inference.

-------------

What does that look like now? Here are four popular Bayesian inference libraries. The objects used for these libraries is usually designed with that inference in mind. There is usally a hodgepodge of methods attached to it, and it is very hard to persist that object. If you want to inspect the model your coworker fit, that is very hard, and probably insecure if you use pickle.

ArviZ wanted to work with all these libraries, and we decided that the right way to do that was to use a common data structure, and to implement our functions on that datastructure.

-------------

So pandas is great, right? This was the very first version of ArviZ, in fact. The problem is that a parameter might have 500 chains of 4 draws each, and be 2 dimensional, meaning we need a 4d data structure to store it.

-------------

While using pandas, we were constantly using awkward constructs to get around the tabular representation of data. Really, we needed high dimensional pandas.

-------------

Thank goodness there is a library built for storing high dimensional, labelled data. Xarray is a joy to work with, it cares about high dimensional labelled data, it is performant, and it naturally serializes as netcdf files.

-------------

This is what ArviZ uses as a data structure. We have a very small `InferenceData` class that holds multiple xarray datasets. It is about 20 lines of code. This class emulates the groups that are available in the netcdf format but in memory. Every InferenceData object can have all or some of these groups. The `posterior` is perhaps the most used one, but the more groups you have, the more interesting analyses you can do. `sample_statistics`, for example, stores the data about divergences and energy.

-------------

In code, this is what the inference data object looks like.

-------------

Attribute access gives you the xarray dataset. Notice that we can also attach JSON metadata to the dataset that travels with it. It is on the roadmap to make these attributes more informative.

-------------

This approach has allowed us to implement a number of conversion functions. Any of these libraries are natively supported in ArviZ. In fact, if you call a function in ArviZ with an object from one of these libraries, ArviZ calls these conversion functions internally and typically "does the right thing". Other libraries can put objects into a dictionary of numpy arrays and get these visualizations and diagnostics for free.

-------------

The other side effect of using xarray and netcdf is that we can ship models easily. This is the part of the ArviZ roadmap I am most excited about - building a model zoo so ArviZ users can download and analyze models that other people fit. So this is my call to action for all of you: if you have an interesting model that you have fit, and want to add it to this "model zoo", please get in touch or open an issue!

Thanks so much.
