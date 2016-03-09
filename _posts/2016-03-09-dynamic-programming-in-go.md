---
layout: post
title: Dynamic Programming in Go
---
>Consider the following solitaire game: you are given a list of moves,$$ x_1, x_2, \dots, x_m $$, and their costs, $$y_1, y_2, \ldots, y_m$$.
>Moves and their costs both take the form of positive integers.
>Such a list of moves and their costs is called an instance of the game.
>To play the game, you start with a positive integer $$n$$.
>On each turn of the game you have two choices: subtract 1 from $$n$$ incurring a cost of 1 or pick some move $$x_i$$ such that $$n$$ is divisible by $$x_i$$ and divide $$n$$ by $$x_i$$ incurring a cost of $$y_i$$.
>The goal is to get to 0 while incurring the minimum possible cost.

At first glance, a possible greedy solution would be to divide $$n$$ by move $$x$$ with penalty $$y$$ such that $$x/y$$ is maximized.

We first need to create a way to store all our possible moves in way that can be sorted.

{%highlight go %}
//Create type Move
type Move struct {
  mv   int
  cost int
}
//SortMove[] will hold moves and implements sort.Interface
type SortMove []Move

func (s SortMove) Len() int {
  return len(s)
}
func (s SortMove) Swap(i, j int) {
  s[i], s[j] = s[j], s[i]
}
func (s SortMove) Less(i, j int) bool {

  return float64(s[i].mv)/float64(s[i].cost) > float64(s[j].mv)/float64(s[j].cost)
}
{% endhighlight %}

We can then take in a list of moves and costs, add them to our `Move` array.

{%highlight go %}
var moves = []int{...}
var costs = []int{...
var movelist = []Move{}

for i := 0; i < len(moves); i++ {
  newMove := Move{mv: moves[i], cost: costs[i]}
  movelist = append(movelist, newMove)
}
{% endhighlight %}

Our greedy algorithm will then sort the array of Moves the ratio of move value to cost, and then take the move best move available.
If there are no moves available then we subtract 1 from x and add 1 to the cost.

{%highlight go %}
func greedy(x int, movelist []Move) int {
  sort.Sort(SortMove(movelist))
  var cost = 0
  OUTER:
  for x > 0 {
    for index, move := range movelist {
      if math.Mod(float64(x), float64(move.mv)) == 0 {
        x = x / move.mv
        cost = cost + move.cost
        continue OUTER
      }
    }
    x = x - 1
    cost = cost + 1
  }
  return cost
}
{% endhighlight %}

However, this solution does not always give us the correct answer.
For example, for moves $$3,1$$ and penalties $$2,1$$, the greedy solution will return 11, while an optimal solution would return 10.

To find the optimal solution for all cases, we turn to the idea of dynamic programming.
Dynamic programming involves breaking the problem into smaller subproblems, and using the solutions to the subproblems to craft a solution to the whole problem.

Notice for each move $$x_1 \dots x_m$$ we can take from $$n$$, we get an intermediate value $$j_i = n_i/x_i$$.
If we could get the optimal cost $$opt(j_i)$$ for any $$j_i$$, then the optimal cost for $$x$$ would be:

$$min\{ y_i + opt(j_i): j_i=n_i/x_i\}$$

In other words, the optimal cost for $$n$$ is the minimum of the optimal costs of all the intermediate values plus the cost to get to the intermediate values.
To keep track of the intermediate solutions we can create a dictionary that maps from intermediate values of $$n$$ to their optimal solutions, starting at 0 = 0

{%highlight go %}


func optimal(x int, movelist []Move) int {
  var m = make(map[int]int) //Our map of n to its optimal solution
  m[0] = 0                  //Our base case

  for i := 1; i < x; i++ {
  var min = Math.MaxInt32
    for _, move := range movelist {
      if math.Mod(float64(x), float64(move.mv)) == 0 {
        imm = x / move.mv
        if m[imm]+move.cost < min {
          min = m[imm] + move.cost
        }
      m[x] = min
      }
      return m[x]
    }
  }
}
{% endhighlight %}
