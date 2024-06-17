# Tree search

## Example 1
Let´s take the Farmer, Cabbage, Goat, Wolf problem.

Building a search tree is like asking "what if" questions to identify possible actions.

<img width="1404" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/b8069819-6cfb-464c-b9f2-727874f8746e">

## Example 2

**Transportation** - A street with blocks numbered 1 to $n$. Walking from $s$ to $s+1$ takes 1 minute. Taking a magic tram from $s$ to $2s$ takes 2 minutes. How to travel from 1 to $n$ in the least time?

Let´s define the problem.

```python
class TransportationProblem(object):
  def __init__(self, N):
    self.N = N   # N = number of blocks

  def startState(self):
    return 1

  def isEnd(self, state):
    return state == self.N

  def succAndCost(self, state):
    # return list of possible (action, newState, cost) triples
    result = []

    if state + 1 <= self.N:
      result.append(('walk', state+1, 1))

    if state*2 <= self.N:
      result.append(('tram',state*2, 2))
      
    return result
```

```python
def printSolution(solution):
  totalCost, history = solution
  print('totalCost: {}'.format(totalCost))
  for item in history:
    print(item)
```

Now, let´s talk about algorithms to solve this problem.

| Algorithm  | Description | Cost | Time | Space |
| :------------- | :------------- | :------------- | :------------- | :------------- |
| Backtracking search  | Searches a tree which goes through each branch, and allows **any** cost assigned at each branch | Any | $O({branching}^{actions})$ | $O(actions)$ |


### 2.1 Backtracking search

```python
def backtrackingSearch(problem):
  # best solution found
  best = {
    'cost': float('+inf'),
    'history': None
  }

  # recursive function that explores a branch of the tree
  # and its child branches
  def recurse(state, history, totalCost):
    # update
    if problem.isEnd(state):
      if totalCost < best['cost']:
        best['cost'] = totalCost
        best['history'] = history
      return
    
    # explore
    for action, newState, cost in problem.succAndCost(state):
      recurse(newState, history + [(action, newState, cost)], totalCost + cost)
  
  # run
  recurse(problem.startState(), history=[], totalCost=0)
  return (best['cost'], best['history'])
```
And let´s try it out
```python
prob = TransportationProblem(N=20)
print(printSolution(backtrackingSearch(prob)))
#totalCost: 8
#('walk', 2, 1)
#('walk', 3, 1)
#('walk', 4, 1)
#('walk', 5, 1)
#('tram', 10, 2)
#('tram', 20, 2)
#None
```


