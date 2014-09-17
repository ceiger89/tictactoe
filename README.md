Reinforcement Learning With TicTacToe
=====================================

Making computers learn how to play tic-tac-toe.

I started messing around with reinforcement learning when I heard about a [Flappy Bird RL project](http://sarvagyavaish.github.io/FlappyBirdRL) on Github. Out of the box, the algorithm took about 7 or 8 hours to train. I figured it could learn faster if multiple instances of the same algorithm spread out over the internet could all work to update the same matrix. So after forking the repo and creating a [distributed learning version](https://github.com/rolyatmax/FlappyBirdRL), I was able to get it to train in about 30 minutes with 8 browser tabs. I found myself wanting to explore reinforcement learning a bit more, and so this little project was born.

So, a general overview. This learning algorithm doesn't know how to play tic-tac-toe. It doesn't know the rules; it doesn't learn the rules; it doesn't even know it's playing against an opponent. All it knows is what the board looks like and what its options are. When it has made a move, it is rewarded or punished based on that move for that particular position. After a few thousand games, it effectively "learns" how to play. The algorithm's "knowledge" is represented in a matrix (called a "policy") where values are assigned to every possible move for each board state the algorithm has encountered.

The first two options presented, "Train Locally" and "Train with Distributed Learning" simply refer to where the q-matrix or "policy" is stored. "Train Locally" keeps the all the training confined to the browser tab. You can watch it train from nothing (it might take 2 or 3 minutes). "Distributed Learning" trains locally as well, but it also persists with the server. This policy tends to be trained fairly well already and rarely loses.

For the "distributed learning" mode, The learner pushes all policy updates onto a stack and periodically sends the batch of updated values to the server. Each update consists of a state (the board), an action (a move), and a value for that state-action pair (higher values = better moves). Currently, the server goes through the updates, writing over the state-action pairs in its own canonical policy, and then sends back an entire copy of the new-and-improved policy with which the client replaces its own.

As you might have guessed, there is some clobbering that takes place on the canonical policy. However, one might be able to avoid clobbering completely by only passing to the server a state, an action, and a reward. This lets the server run its own evaluation function instead of relying on clients' states. This would also require the constants in the evaluation function to match on the client and the server.

The algorithm trains by playing against a version of itself. The "Smart" version (listed under the "Scores" section) always selects a move its trained policy recommends. The "Kinda Smart" version (a bit of a misnomer) will occasionally select random moves to see the outcome. It is this "exploration" that actually allows the algorithm to learn. Because you can play a tic-tac-toe game perfectly and not win, the best measure of a policy's efficacy is how many wins it has given up to its opponent. Because the "Kinda Smart" version makes random moves for some percentage of its total plays, it happens to give up quite a few wins to its opponent.

You can pause the training and play against the "Smart" version at any time. After the algorithm moves, it displays its options in the bottom corner of the screen. You can mouse over these options to see which ones were most favored.

There were some interesting optimizations I made which seemed to have helped speed up learning quite a bit. The most important optimization was probably the work I did normalizing equivalent board states. For (almost) every possible tic-tac-toe board, there are at least a few other tic-tac-toe boards that are essentially equivalent. For example, you can take a given board and rotate three times or flip it along the vertical and/or horizontal axes. Making sure these board states were considered equivalent cut down on the amount of memory required to store the policy and, consequently, the amount of time to learn the policy.

To accomplish the normalization, the choosing function turns the board state into a string:

```
 X | O |                        |   | O                      O |   |
-----------                  -----------                    -----------
   | X |     is the same as   O | X |      is the same as      | X | O
-----------                  -----------                    -----------
   |   | O                    X |   |                          |   | X
```

would become `AB--A---B` if you are Xs and `BA--B---A` if you are Os. It then checks the matrix for any equivalent permutations by rotating and flipping the board. If it finds one, it remembers how many rotations and flips it used so that it can apply the same transformations to the move it selects - "translating", so to speak, the moves between the actual board and the permutation.

See it in action at [tbaldw.in/tictactoe](https://tbaldw.in/tictactoe). Check out the code at [github.com/rolyatmax/tictactoe](https://github.com/rolyatmax/tictactoe). It's a bit messy in parts (as it has changed tremendously over time), but the meat of the learning algorithm is in `public/js/Q.js`. For more info about how Q-Learning works, check out the [Wikipedia article](http://en.wikipedia.org/wiki/Q-learning).


To run on your own:
------------------

`npm install`

`cd public && bower install`

`cd .. && node app`

Point your browser to `localhost:8080`
