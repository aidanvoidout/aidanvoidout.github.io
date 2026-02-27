---
layout: post
title:  "genetic algorithms"
date:   2025-08-29 09:29:20 +0700
categories: jekyll update
usemathjax: true
---

genetic algorithms are a subset of artificial intelligence inspired by the process of natural selection, offering a way to solve optimisation problems by simulating evolution.

### the process of genetic algorithms

genetic algorithms operate on a <b>population</b> of candidate solutions, which allows the algorithm to explore multiple solutions in parallel.

as per evolution, genetic algorithms need a way to separate the strong from the weak. this is done through a <i>fitness function</i>, which serves as a way to evaluate the quality of this solution via a set of criteria. for example, if the problem was to find the most optimised schedule, solutions might be evaluated on how many tasks were fit within the schedule's time period and penalised based on the amount of idle time left over.

in nature, fitter individuals are more likely to survive due to their traits, allowing them to reproduce and their attributes on. genetic algorithms combine the traits of two parents in a process called <i>crossover</i>. in problem-solving terms, this is done so because two fit solutions/two strong parents will be able to produce fitter solutions/stronger descendants.

moreover, genetic algorithms also have to introduce the process of mutation to ensure diversity in solutions and prevent overfitting. suppose a problem had one variable and the genetic algorithm converged until all members of the population perfectly met the fitness function's requirements. if this variable were to change, none of the population would be able to adapt to the new conditions, as all of their traits had already converged to perfectly fit the old condition, and no amount of crossover will be able to produce a solution. in biological terms, mutation is the element of randomness that introduces unique traits in the hope that the trait will come in useful in the event of a great filter.

once a new generation of individuals is created through selection, crossover, and mutation, the old population is replaced. the cycle repeats: fitness is evaluated again, parents are chosen, and new children are created. over successive generations, the population tends to improve,  as species in nature adapt over many lifetimes.

this video by Computerphile explores the application of genetic algorithms with the well-known knapsack problem (commonly solved in competitive programming using dynamic programming, but oh well)

<iframe width="560" height="315" src="https://www.youtube.com/embed/MacVqujSXWE?si=WB98LLiBERNSQOEb" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### simulation!
i made a simulation to illustrate the principles of genetic algorithms, optimising the problem of camouflage (matching colours).

oops, currently being fixed
<div class="game-container">
  <iframe src="{{ '/assets/demos/camouflage/genetic algorithms!.html' | relative_url }}" frameborder="0" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe>
</div>


in this simulation, a population of 100 matches their colour to a target colour. parents are chosen randomly, but are weighted by their fitness scores.

population is randomly generated at the start:

```swift
face.modulate = Color(randf(), randf(), randf())
```

fitness function evaluates the closeness of each population's color to the target color, by taking the difference of each color and taking the magnitude:
```swift
func fitness(face: Node2D) -> float:
    var c = face.modulate
    var dr = c.r - target_color.r
    var dg = c.g - target_color.g
    var db = c.b - target_color.b
    var d = sqrt(dr * dr + dg * dg + db * db)
    return 1.0 - (d / sqrt(3.0))
```

to breed the next generation of solutions, we select fit parents using roulette wheel selection:
```swift
func _select_parent() -> Color:
    var total_fitness = 0.0
    for face in population:
        total_fitness += fitness(face)
    ...
    var pick = randf() * total_fitness
    var running_sum = 0.0
    for face in population:
        running_sum += fitness(face)
        if running_sum >= pick:
            return face.modulate
```

each RGB component of the child is randomly inherited from one of the parents. in a former patch, the child inherited the average of the parents, but this caused the simulation to tend towards a dull grey with varying tones of other colours.

```swift
func _crossover(c1: Color, c2: Color) -> Color:
    var red = c1.r if randf() < 0.5 else c2.r
    var green = c1.g if randf() < 0.5 else c2.g
    var blue = c1.b if randf() < 0.5 else c2.b
    return Color(red, green, blue)
```

to keep the simulation diverse and prevent premature convergence:
```swift
func _mutate(c: Color, rate: float = 0.05) -> Color:
    if randf() < rate:
        return Color(
            clamp(c.r + randf_range(-0.1, 0.1), 0, 1),
            clamp(c.g + randf_range(-0.1, 0.1), 0, 1),
            clamp(c.b + randf_range(-0.1, 0.1), 0, 1)
        )
    return c
```

as an interesting result of this, when the target colour has rgb components at the limits (0.0 or 1.0), the children tend to evolve colours closer to the target colour. this could be because the bounds of the child's colour is limited, causing larger proportions of the children to develop such traits.

---

this small project demonstrates the power and beauty of genetic algorithms. while my example evolves colors toward a target, the same principles can scale to far more complex challenges. genetic algorithms don’t guarantee perfection, but they provide a robust way to discover good solutions in large search spaces.
