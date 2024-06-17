# Tree search
taken from Stanford, CS221.

## Example 1
Let´s take the Farmer, Cabbage, Goat, Wolf problem.

Building a search tree is like asking "what if" questions to identify possible actions.

<img width="1404" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/b8069819-6cfb-464c-b9f2-727874f8746e">

## Example 2

**Transportation** - A street with blocks numbered 1 to $n$. Walking from $s$ to $s+1$ takes 1 minute. Taking a magic tram from $s$ to $2s$ takes 2 minutes. How to travel from 1 to $n$ in the least time?

Imagine each path as a tree:
<img width="703" alt="image" src="https://github.com/rafasacaan/the-notebook/assets/10575866/79ba4225-1272-4a55-9cd8-c7ce12d05d75">


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
| Backtracking search  | Searches a tree which goes through each branch, and allows **any** cost assigned at each branch | Any | $O({branching}^{depth})$ | $O(depth)$ |
| Depth-first search (DFS)  | Finds a solution and does not need to find more solutions if you already reached a good one | $c=0$ | $O({branching}^{depth})$ | $O(depth)$ |
| Breadth-first search (BFS)  | Assume equal costs per actions and explores tree per horizontal layers until a solution is found. If we continue searching, we will only add more $c$ to the final solution (i.e. worse) | $c>=0$ | $O({branching}^{small_depth})$ | $O(small_depth)$ |
| DFS - ID (iterative deepening)  | Run DFS for different levels of depth | $c>=0$ | $O({branching}^{small_depth})$ | $O(small_depth)$ |


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

### 2.2. First improvement: depth-first search (DFS)
Same as backtracking, but once its find a solution, then it is done (does not explore anymore). We can translate this to adding an constraint to the search, equivalent as making the branch costs equal to zero.

> Assume action costs $Cost(s,a) = 0$

### 2.3. Breadth-first search (BFS)
Now, search the tree thorugh its horizontal layers, from top to bottom. Assume all actions have the same cost $c$. Once we find a solution, there is no need to keep searching as we will only get solutions that taje more $c$ to arrive. One problem is that you need memory: as we go into the first node from left to right, i need to store all of its history, as I dont know if the un-explored solutions to the right will provide an end state.

### 2.4. DFS-ID
Modify DFS to stop at a maximum depth. Call DFS for maximum depths: 1,2,... . On depth $d$ ask if there is a solution of $d$ actions (such as what BFS looks for). If it is the case, stop. Else, keep searching for larger depths.


 
