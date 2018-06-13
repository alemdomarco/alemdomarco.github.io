---
layout: post
title:  Project Eva
buttons: <a class="github-button" href="https://github.com/decorators-squad/eva" data-icon="octicon-star" data-count-href="/amihaiemil/eva/stargazers" data-count-api="/repos/decorators-squad/eva#stargazers_count" data-count-aria-label="# stargazers on GitHub" aria-label="Star decorators-squad/eva on GitHub">Star</a> <a class="github-button" href="https://github.com/amihaiemil/eva/fork" data-icon="octicon-repo-forked" data-count-href="/decorators-squad/eva/network" data-count-api="/repos/decorators-squad/eva#forks_count" data-count-aria-label="# forks on GitHub" aria-label="Fork decorators-squad/eva on GitHub">Fork</a>
date:   2016-04-20
tags: projects github ai
author: amihaiemil
comments: true
shareable: true
preview: A simple AI framework which you can use to solve complicated problems.
---

### Intro

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;According to Wikipedia, "a heuristic is any approach to problem solving, learning, or discovery that employs a practical method not guaranteed to be optimal or perfect, but sufficient for the immediate goals". As a developer, you may use a *heuristic* when there's no other way to solve the problem - when you cannot find a propper **algorithm** to solve it.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Take the backpack's problem for instance: you have a backpack which can hold 10 units of weight and 5 objects of weights: 2, 2, 3, 4 and 5. How do you fill the backpack best? A method is to order the objects decreasingly and keep adding them until you cannot add another one. You then have 5, 4, 3, 2 and 2. Take the first two and you filled the backpack 9/10, but really you could have filled it 10/10 by taking 5, 3 and 2. Of course, the problem becomes much more complicated when you have thousands of objects.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Eva is an simple framework which you can use to get a satisfying solution for this kind of problems. It's intended to be usable by everyone, unlike other frameworks which require solid knowledge in the field of [genetic algorithms](https://en.wikipedia.org/wiki/Genetic_algorithm).

See more details about how it works on the project's [site](http://eva.amihaiemil.com).

### Example

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Assume you have a backpack which can hold **100 units** and **10 objects** with the following weights:

12 31 16 42 41 27 63 76 67 46

In the backpack's problem the solution is an array or 0's and 1's corresponding to each object: 1 - taken, 0 - left out

One solution for this input would be

0 1 0 1 0 1 0 0 0 0

So we take the objects with weights: 31, 42 and 27 to fill the backpack.

### The code
{% highlight java %}
//long, because the api requires so.
long[] weights = new long[] {12, 31, 16, 42, 41, 27, 63, 76, 67, 46}

//Generator of random solutions
BinaryArraySolutionsGenerator generator = new BinaryArraySolutionsGenerator(weights.length) ;
//Fitness evaluator (how is a solution evaluated? How do you calculate how good ("fit") it is?
FitnessForBackpackEvaluator evaluator = new FitnessForBackpackEvaluator(weights);
//optional stopping conditions
Condition solutionIsAcceptable = new Condition() {
    public boolean passed(Solution solution) {
        return solution.isAcceptable();
    }
};

Eva eva = new SimpleEvolutionaryAlgorithm()
                .with(generator)
                .with(evaluator)
                .with(solutionIsAcceptable);
Solution solution = eva.calculate();
NumericalRepresentation rep = NumericalRepresentation.class.cast(solution.getRepresentation());
for(int i=0;i<rep.getSize();i++) {
    System.out.print(rep.get(i) + " ");
}

{% endhighlight %}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Note that it is not mandatory to add any conditions, but it's recommended to specify at least the above condition **if you don't want a long execution time**. It is not guaranteed that the found solution will meet the specified Conditon (although ideally it should).

The idea of the Condition is to make the algorithm faster by stopping it when it finds a satisfactory solution.

The maximum number of iterations performed is number of generations times population size and a Condition cannot add iterations (i.e. iterate until a good solution is found) because there are big chances of creating an infinite or really really long run.

In this backpack example, ``Solution.isAcceptable()`` only checks if the total weight is in the following interval: **(0, backpackWeight]**. You could add more conditions to always get a solution with total weight, say, over 50%.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Tip** Use ``com.amihaiemil.eva.Conditions`` to specify more conditions, like this:

{% highlight java %}
...
//Like I said above, this one should always be present
Condition solutionIsAcceptable = new Condition() {
    public boolean passed(Solution solution) {
        return solution.isAcceptable();
    }
};

Condition conditions = new Conditions(solutionIsAcceptable)
                        .and(new Condition() {...});
			.or(new Condition() {...});
Eva eva = new SimpleEvolutionaryAlgorithm()
                .with(generator)
                .with(evaluator)
                .with(conditions);

...
{% endhighlight %}

If you specify more conditions you minimize the chances of the algorithm skipping
a really good solution.
