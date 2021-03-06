# AlphaGo Zero

# Main Points
Play many games of self-play. During a self-play game, moves are made by running Monte-carlo tree search at each step guided by the action probabilities and value estimates of the neural network. The neural network's parameters are optimize so that it's outputed action probabilities match the probabilities obtained from the tree searches and it's value estimates match the final winner of the game.



* Monte-Carlo Tree Search
    * **Select**: A node is a board state. Select node on frontier to expand by traversing tree until leaf node $s_L$ is reached.
        * UCT (Upper Confidence Bound (UCB) for Trees)
            * Select action $a$ where $a = \mathrm{argmax}_a Q_{(s_t, a)} + U_{(s_t, a)}$
                * $U_{(s, a)} = c_{puct} P_{(s, a)} \frac{\sqrt{\sum_b N_{(s, b)}} }{1 + N_{(s, a)}}$
        * Virtual loss to discourage redundant exploration
            * During the traversal, the node statistics are temporarily updated with a large negative value N, W as if this path leads to large number of lost games. This *virtual loss* is removed in the backup.
    * **Neural network evaluation**: Run neural network on $s_L$ to get $\mathbf{p}$ and $v$. (**No rollout!**)
        * **Expand**: Each edge of a state is a (state, action) pair. Initialize edges of $s_L$ with the necessary stats.
            * For all actions $a$ possible, initialize at edge ($s_L, a$) the values, N = 0, W, Q, and P 
                * N=0: number of times this edge has been followed
                * W=0: total state-action value in sub-tree
                * Q=0: mean state-action value in sub-tree
                * P=$p_a$ + [$\mathrm{Dir}$ if $s_L$ is root]: stored *prior probability* of taking action $a$ at state $s_L$
                    * add Dirichlet loss to root node to encourage exploration
        * **Backup**: Propagate $v$ back up the tree.
            * For each edge ($s_t$, $a_t$) along the path from the roto to $s_L$:
                * $W_{(s_t, a_t)} = W_{(s_t, a_t)} + v$
                * $N_{(s_t, a_t)} = N_{(s_t, a_t)} + 1$
                * $Q_{(s_t, a_t)} = \frac{W_{(s_t, a_t})}{N_({s_t, a_t})}$

* Play move: After MCTS, play action $a$ of root node $s_0$ with some probability proportional to the *exponentiated traversal count* of ($s_0$, $a$)(i.e. the $N$ recorded on ($s_0$, $a$)).
    * $\pi(a | s_0 ) = \frac{ {N_{( s_0 , a )}}^{\frac{1}{\tau}} }{ \sum_{b \in A_{s_0}}\  {{N_{ (s_0 , b)} }}^{\frac{1}{\tau}}  }$ 
    * Save subtree of $s_0$ corresponding to taken action for further tree searchs.

* Neural Network training:
    * Save samples $(s_t, \pi_t, z_t)$ from past self-play games where 
        * $s_t$: the state at time $t$
        * $\pi_t$: the action probabilities derived from MCTS at time $t$
        * $z_t$: +1 if the player at time $t$ ended up winning, else -1
    * Train on saved samples: Make neural net's value estimates closer to actual winners and action probabilities closer to MCTS search probabilities
        * Loss function: Equally weighted sum of mean-squared error and cross-entropy loss
        * For single example from time-step $t$, $(s_t , \pi_t , z_t )$, $\mathcal{L} = (z_t-v)^2 + (-\mathbf{\pi_t}^{\intercal}\mathrm{log}\mathbf{p}) + c\|\theta\|^2$ where $(\mathbf{p}, v) = f_\theta(s_t)$ and $c$ is a L2 regularization parameter

## Policy Iteration
> The AlphaGo Zero self-play algorithm can similarly be understood as an approximate policy iteration scheme in which MCTS is used for both policy improvement and policy evaluation. Policy improvement starts with a neural network policy, executes an MCTS based on that policy’s recommendations, and then projects the (much stronger) search policy back into the function space of the neural network. Policy evaluation is applied to the (much stronger) search  policy: the outcomes of self-play games are also projected back into the function space of the neural network. These projection steps are achieved by training the neural network parameters to match the search probabilities and self-play game outcome respectively.

## References
* [DeepMind AlphaGo Zero 網誌](https://deepmind.com/blog/alphago-zero-learning-scratch/)
"AlphaGo Zero: Learning from scratch"
DeepMind Blog Post

* [AlphaGo Zero paper](https://www.nature.com/articles/nature24270.epdf?author_access_token=VJXbVjaSHxFoctQQ4p2k4tRgN0jAjWel9jnR3ZoTv0PVW4gB86EEpGqTRDtpIz-2rmo8-KG06gqVobU5NSCFeHILHcVFUeMsbvwS-lxjqQGg98faovwjxeTUgZAUMnRQ) 
Silver, D. et al. *Mastering the game of Go without
human knowledge* Nature. (2017)
* [原AlphaGo paper](https://storage.googleapis.com/deepmind-media/alphago/AlphaGoNaturePaper.pdf)
Silver, D. et al. *Mastering the game of Go with deep neural networks and tree
search* Nature. (2016)
* [Paper discussing using MCTS in Go](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Publications_files/grand-challenge.pdf)
Gelly, S. et al. *The Grand Challenge of Computer Go:
Monte Carlo Tree Search and Extensions* Communications of the ACM. (2012)
* [MCTS survey](https://www.researchgate.net/publication/235985858_A_Survey_of_Monte_Carlo_Tree_Search_Methods)
Browne, C. et al. *A Survey of Monte Carlo Tree Search Methods* IEEE Transactions on Computational Intelligence and AI in Games. (2012)
