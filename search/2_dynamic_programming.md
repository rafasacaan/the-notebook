# Dynammic Programming

The objective is to find the future cost of state $s$.

<img width="417" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/4715744a-2f7f-4741-b139-0a0b75eec0f9">

So, we need to calculate:

$FutureCost(s) = min_{a}\lbrack{{cost(s,a) + FutureCost(s')}}\rbrack$

Meaning that the future cost of state $s$ is the cost of best possible action in $s$ and the future cost of that new state $s'$.

## Example: Route finding
Find the minimum cost path from city 1 to city $n$, only moving forward. It costs $c_{ij}$ to go from $i$ to $j$.
The tree for this problem is:

<img width="563" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/d0fa6034-3909-4bda-89ae-5ef7f43bce95">

Observation: in this case, future costs only depend on current city (not on past sequence of actions). We can see that, for example, the future value when we are in city number 5, is always the same. We can store this value and call it in the future. Given this observation, we can plot the tree as a graph:

<img width="455" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/e5e56e98-4049-4330-ae1d-ee65dfdf425b">

**Key idea**
> A **state** is a summary of all the past actions sufficient to choose future actions **optimally**.

**Assumption**
> The state graph is defined by $Actions(s)$ and $Succ(s,a)$ is **acyclic**.
As we are computing future costs, we need to impose structure so we have an order of how to proceed and calculate future costs.

Let´s implement DP.
```python
def dynammicProgramming(problem):
  cache = {} # state -> futureCost(state)

  def futureCost(state):
    # Base case
    if problem.isEnd(state):
      return 0
    if state in cache:
      return cache[state]

    result = min(cost + futureCost(newState) \
        for action, newState, cost in problem.succAndCost(state))
    cache[state] = result

    return result
  return (futureCost(problem.startState()), [])
```

We can check we obtain the same solution as with backsearch search.

### Handling constraints
Lets add the following constraint:
> Can´t visit three odd cities in a row

We can redefine the states as:

$S(\text{if prev. city was odd}, \text{current city})$

or better

$S(\text{min(num. of odd cities visited, 3)}, \text{current city})$

so we can shorten the number of states from $N^2/2$ to $3N$. Then, the state graph would look like this:

<img width="625" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/d915a8c3-b6b7-4c68-a8c0-9a29a26d1a22">

## Conclusions
- **State** is going to be a summary of past actions sufficient to choose future actions optimnally
- DP = backtracking search with memoization


# Uniform Cost Search (UCS)
It can handle cyclic grpahs! (as opposed to DP). In DP, we imposed the acyclic assumption so that $FutureCost(s)$ is computed before $FutureCost(s')$.
- UCS enumerates states in order of increasing costs.
- **Assumption** - all actions costs are non-negative: $Cost(s,a) >= 0$

We need to keep track of three sets:

<img width="567" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/58dbcf55-a227-4fc9-ab65-9efca7b7ca1d">

<img width="654" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/2e81ba15-a281-4ab6-874f-80118e2259cb">




