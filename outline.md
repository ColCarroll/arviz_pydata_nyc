
# Outline

Thank you for being here. This is a very exciting talk to give - ArviZ is a project that four of us have been coding hard on for about six months, with helpful input and pull requests from other members of the statistical computing community. ArviZ is a library for visualizing and criticising Bayesian anaylsis in Python. I hope today to do three things.


In order for you to be properly impressed with the technical brilliance of ArviZ, I will give a brief overview of Bayesian statistics. Then I will show a little of what you can do with ArviZ. This is secretly also an overview of Bayesian statistics, in the spirit of form fitting function: I hope that those of you less familiar with these techniques will learn about the sorts of questions you can or should be asking about your data. We will conclude with a discussion of how we used xarray to interoperate between many different Bayesian libraries, and how useful the right datastructure for the job can be.


As with last year, there are a number of very good Bayesian talks, including a master class in Bayesian statsitics directly after this session. We will consider this a light warmup of looking at Bayes' theorem. A reasonable definition of Bayesian statistics is that we consider the parameters of a model to be uncertain, as compared to frequentist statistics, where the data is uncertain. This is an example of a graphical model for linear regression. The box is called a plate, and the "20" indicates that `y` is actually 20 dimensional. Once we observe some data, we fill in that node, and use Bayes theorem to update our beliefs about the slope and the intercept.


Getting rid of the theta and capital X, both the left and the right hand side here are functions of our parameters, the slope and intercept, while the data is fixed.


Looking even more closely at linear regression, we might have some collection of `x` and `y` points. Using `scikit-learn`, we can fit a line to this example. The true slope and intercept are -1 and 1, and `scikit-learn` does a pretty good job recovering those. Let me also step back for a second here and say that machine learning is great, and there are lots of great reasons to use `scikit-learn`. Bayesian modeling should be considered another tool you can use.


Looking at our parameters, `scikit-learn` has given us *point estimates* for the slope and the intercept. If we write down and fit the graphical model I showed earlier, we get distributions over the slope and intercept. Suppose for whatever reason that you get paid $100 if the slope is at least 1, and $10 if it is less than 1. You might appreciate having access to this posterior distribution.


Now if we go back to plotting the line, everything turns into a distribution. Instead of a single line of best fit, we have a distribution of lines. We can also use the model to generate new datasets that we might see in the future, or to decide how surprising our model thinks each data point was. These are visualizations of the posterior and posterior predictive distributions, respectively.


Another example we might care about is classification. This is a synthetic dataset of five cluster that overlap. Fitting this with an algorithm like K-Means gives a single classification for each point. We can write down a generative model and fit that to instead have a distribution of labels for each point.


Now what is ArviZ?


The easiest way to show it off is as a visualization library. We can start off by generating random numbers in numpy and putting them into a dictionary. These are uniformly distributed random numbers, but I put them together to have some correlation between `z` and `x` or `y`, but `x` and `y` are independent.


One of my favorite plots is `plot_pair`. It shows right away the relationship between any two distributions. It has a few options that you can switch around depending on what sort of data you are looking at. The 2d kde is the most visually pleasing, but might show you some artifacts.


A feature of Bayesian inference is that many parameters are high dimensional. This analysis is one from the PyMC3 docs, that estimates some latent measure of offense for European rugby teams. Again, we have a few styles available. This forest plot gives us a glimpse at how Bayesian inference works. We are just getting a numpy array of samples for each parameter. For technical reasons, we repeat this multiple times, and will analyse these chains to make sure our algorithm actually worked. The forest plot is a way to visually check that each chain looks similar. This plot also exposes the effective sample size and R-hat statistic, which are also used to assess convergence. R-hat should be very close to 1, or else you have had some trouble.


In case you have only two variables, looking at a joint plot will include the marginal distributions. This is a fun example, since I used a custom PyMC3 library to sample from the PyMC3 and Stan logos. ArviZ works really well with PyMC3, so I can just plot this without too much trouble, other than adjusting the aspect ratio.


Besides visualizing your distributions, ArviZ includes diagnostics: ways to make sure the algorithm you were using worked. There are two ways for a model to fail: the model can fail to represent the data, and the algorithm can fail to fit the model. The eight schools model is a famous one in Bayesian statistics: the idea is that the same SAT prep course was taught at eight schools, and a mean improvement and standard deviation was reported from these eight schools. There is the idea that we might be able to pool the data from the schools to get a better estimate of individual school performance.

This is an example of a hierarchical model, and it can actually be difficult for algorithms to sample from. Specifically, this centered model will do badly, and the non-centered model does well. I encourage you to attend Sean Talts course today, and Michael Betancourt's tomorrow if you want to know more about why. I want to show you how you can use ArviZ to decide that the non-centered algorithm captured the posterior better.


First the trace plot, which shows up in many tutorials for PyMC3 at least. This is a little hard to spot, but you can see one of the four chains got stuck. I can single it out to see better. This is a bad sign, and you should consult your local statistician for how to fix it. The non-centered trace does not have any obvious such artifacts.


This is a plot of the distribution of energy, which is a quantity computed in modern, gradient based MCMC samplers. The two distributions should match up. These do not, which also indicates a problem. Again, the non-centered model looks better.


One reason we love gradient based MCMC samplers is that when they have trouble, it is obvious. These divergences are recorded by both PyMC3 and PyStan, and arviz will annotate your pair plot with the divergences. We get more information here, that the problems in the centered model have something to do with `tau` being small. The non-centered parameterization I am working with also encountered some divergences! I am worried about this, and might do some further checking, but there is no good pattern I see in the pair plot.


Finally, this parallel plot is a quick way to check whether there is some coherence in where divergences happen. Again we see that a small value of `tau` seems to be the problem, but there is no similar pattern in the non-centered model.


There are also a number of statistical computations built into Arviz! These are less flashy, but arguably more useful. Again, if we were comparing our two models, we might notice that the effective sample size of the non-centered model was much higher than for the centered model.


To conclude, I want to talk about how ArviZ does what it does. This is a simplified version of a Bayesian workflow. Building a model and inferring parameters is hard. You should use a specialized library like Stan or PyMC3 for those. ArviZ lives at this third node. We provide visualization and diagnostic functions for the libraries that did the inference.


When you do Bayesian inference with these libraries, the object that returned is usually designed with that inference in mind. There is usally a hodgepodge of methods attached to it, and it is very hard to persist that object. If you want to inspect the model your coworker fit, that is very hard, and probably insecure if you use pickle.

ArviZ wanted to work with all these libraries, and we decided that the right way to do that was to use a common data structure, and implement our functions on that datastructure.


So pandas is great, right? This was the very first version of ArviZ, in fact.


The problem is that a parameter might have 500 chains of 4 draws each, and be 2 dimensional, meaning we need a 4d data structure to store it.


Thank goodness there is a library built for storing high dimensional, labelled data. Xarray is a joy to work with, it cares about high dimensional labelled data, it is performant, and it naturally serializes as netcdf files.


This is what ArviZ uses as a data structure. We have a very small `InferenceData` class that holds multiple xarray datasets. This class emulates the groups that are available in the netcdf format but in memory. Every InferenceData object can have all or some of these groups. The `posterior` is perhaps the most used one, but the more groups you have, the more interesting analyses you can do.


In code, this is what it looks like.


Attribute access gives you the xarray dataset. Notice that we can also attach JSON metadata to the dataset that travels with it.


This approach has allowed us to implement a number of conversion functions. Any of these libraries are natively supported in ArviZ. In fact, if you call a function in ArviZ with an object from one of these libraries, ArviZ calls these conversion functions internally and typically "does the right thing". Other libraries can put objects into a dictionary of numpy arrays and get these visualizations and diagnostics for free.


The other side effect of using xarray and netcdf is that we can ship models easily. This is the part of the ArviZ roadmap I am most excited about - building a model zoo so ArviZ users can download and analyze models that other people fit. So this is my call to action for all of you: if you have an interesting model that you have fit, and want to add it to this "model zoo", please get in touch or open an issue!

Thanks so much.
