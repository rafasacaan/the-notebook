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

Now, let´s talk about algorithms to solve this problem.

| Algorithm  | Description | Cost | Time | Space |
| :------------- | :------------- | :------------- | :------------- | :------------- |
| Backtracking search  | Searches a tree which goes through each branch, and allows **any** cost assigned at each branch | $O(branches^depth) | $O(depth)$ |

