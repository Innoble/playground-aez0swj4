# Smitsimax

*This is a work in progress. I will add more information and hopefully some runnable code*

Recently I have been working on a bot for Coders Strike Back (CSB), a bot-arena on codingame. I reached top 10 in about 20 hours of coding
with a bot that still has various problems, such as: timeouts, missing features and a suboptimal language choice (C#). The only thing this
bot really has going for it is the search. Most searches that are being used in CSB are a variant of Genetic Algorithms or Minimax. This
article is mostly meant to answer the many questions I've been asked about this surprisingly succesful search.

I first thought of this search during the Code of Kutulu contest, where I placed 4th. Kutulu is a 4-player simultaneous game in which it
is very hard to write a good search, because of the exploding searchspace. Smitsimax helped me solve this problem. I was inspired by my
work to code an ultimate tic tac toe (UTTT) bot using MCTS. This is where my use of the UCB formula comes from. *Disclaimer: I am not
completely sure if my search doesn't already exist in some form, but if it doesn't, I have coined the name "Smitsimax" , more as joke than
anything else. If anyone is more versed in search algorithms and knows its true name, do let me know.*

CSB is a two player zero sum game. This makes something like minimax a natural choice as a search algorithm. The main problem is the
simultaneous nature of the game. Both players decide what to do simultaneously and after this, the turn is calculated. The basic version of
minimax requires players to move one after the other, knowing what the other player did the turn (ply) before. To solve the problem of
simultaneous gameplay, you can make the choice to use the "paranoid" option. This means you assume the other player is going to know what
you did and chooses the best possible counter. This is an assumption that can be good or bad, depending on the state of the game and the
possible strategies you can use. Another possible option is to calculate all possible combinations of moves. If one player has 6 possible
moves and the other does as well, then there are a total of 36 possible combinations. You can then average the result, minimize the worst
result or maximize the best result for each of your moves. All of these choices may have good or bad results depending on many factors.

Smitsimax has a few things in common with minimax, but is also different. Both search algorithms have a tree of possible moves. Each move
has a node and each node has children that correspond with the moves on the next turn. In minimax there is only one tree. The tree has
moves by both players in it and each node corresponds to a single absolute gamestate. When you get to this specific node, you know exactly
what the game looks like. Smitsimax has a separate tree for each player (or each pod, as in CSB, meaning two trees per player) and each
node on tree does not directly correspond to a gamestate. At first glance the trees are completely separate. During the search, moves are
selected on each tree, randomly at first and by Upper Bound Confidence formula later. Each turn is simulated with the selected moves. When
sufficient depth is reached, an evaluation is done for each player (pod) in the game and the result is backpropagated along the
respective trees. The first time this is done, every node does correspond to a single gamestate. However, the next time the same node is
selected by a player, the other pods may select different nodes, leading to a different gamestate. The differences in possible gamestates
will be larger for greater depth levels. If the search is able to converge, this is not a problem. The (best) branches to which each tree
converges should, together, correspond to a single gamestate per node. 

Let's look at this step by step in pseudo code, written for CSB:

At the start of a turn you get all the information you need as input. You have your Pod class with information such as:

```C#
class Pod
{
    position
    velocity
    shield 
    ...
}
```

You need two instances of each pod. One is the base instance you get as you update using the turn-input. You need to keep this information
so that you can reset to it after every search-run. The other is the evolving pod that changes during the search. 

There are various ways to set up your simulation and Smitsimax-search. I currently use a static Sim class with 4 variables and 4 methods
that looks somewhat like this:

```C#
static class Sim
{
    Pods[4]  // all 4 pods that will be simulated
    Node[4] current  // 4 current nodes, one for each pod. Current nodes start as a reference to the root nodes of each tree
    float[4] lowestscore // the lowest score earned by the tree, one for each pod.
    float[4] highestscore // the highest score earned by the tree, one for each pod.
    float[4] scaleparameter  // the scaleparameter calculated by subtracting the lowest score from the highest, needed for the UCB formula.
    
    void Reset() // To reset the sim after search
    
    void Search() // The important bit, the actual search
    
    void Play()  // a CSB specific algorithm that handles movement and collisions (see Magus CSB postmortem)
    
    void BackPropagate()  // each tree has the score result backpropagated along the branch of the tree.
}
```
    
The tree consists of nodes and each node in the tree needs the following things:
    
    class Node 
    {
        Node parent // each node has a parent except the root node
        float score // the total score obtained by this node. To get the average, you can divide by the number of visits
        int firstChildIndex // I use a preallocated array that I use this index for
        int childCount // the number of children
        int visits // the number of times this node has been visited
        int angle // this is part of the move, a CSB bot needs an angle
        int thrust // a CSB bot needs a thrust
        ....
    }
 

The Search method looks like this:
```C#
void Search()
{
    CreateRootNodes() // create 4 nodes, one for each tree. These are also the "current" nodes of the sim.

    while (we still have calculation time left)
    {
        Reset() // reset the sim using the pod instances you obtained during the update
        int depth = 0;

        while (depth < SIMULATION_DEPTH)
        {
            for (each pod, 4 times total)
            {
                Node node = current[i];

                if (node.visits == 1)
                    node.MakeChildren() // give the node children if it doesnt have them yet, each with a possible move

                Node child = node.Select() // select a node that you want to use for the sim. 
                //At first it is best to select randomly a few times. 
                //I currently random 10 times. After this I use the UCB formula to select a child. 

                child.visits++; // increment the child visitcount

                current[i] = child; // the child becomes the current node

                pod.ApplyMove // do the stuff the child node tells you to do (rotate, accelerate, shield etc.)
            }

            Play(); // Simulate what actually happens, including movement and collisions
            depth++;
        }
        
        float[4] score = GetScore(); // get a score for each pod when the simulation depth is reached. The way the score is calculated is
        not that different from what it would be in minimax or GA or any other search method. There are many ways to do this.

        for (each pod, 4 times total)
        {
            update the lowest score, highest score and scaleparameter for this pod. 
        }

        Backpropagate() // We go up the tree back to the root node, adding the score to each node. We do this for each pod (4x)
    }
}
```
    
The UCB formula that is used on child selection looks like this:

```C#
float ucb = (score / (visits_of_child * scaleParameter)) + Sqrt(Log(visits of parent)) * (1/Sqrt(visits_of_child);
```

Score divided by visits is the average score of the node. The scale parameter normalizes this score in a range between 0 and 1. The
other term in the formula is identical to UCB as it would be used in MCTS (for example).

## Why does this work at all?

The first few search iterations every pod will select nodes randomly, or just very badly. That means all evaluations are relatively bad and
untrustworthy. At some point, pods will start "discovering" better moves. This will lead to several things happening. 

1) Your better moves will be evaluated as better moves, meaning they are more likely to be picked next time (by UCB formula)
2) The parts of the evaluation that contain opponent state information (travelled distance and such) will be scored more accurately
3) Because of better opponent prediction, your own moves are scored more accurately. 

This has great potential for convergence to an optimal series of moves for all pods. If player 1's runner pod has found a good path, player
2's blocker pod will converge to moves that best counter this path, which will lead to player 1's runner adjusting its path, which means
the blocker will adjust again... etcetera. 

Since we use exploration and expansion by UCB, all moves are evaluated statistically. Your opponent is more likely to pick its best moves
and you are more likely to pick yours. However, if some unlikely move is a great counter to the current best opponent path, the exploration
part of your search will soon pick it up and multiple succesful iterations will cause the opponent to "rethink" its best path. As with
every search, the more calculation time you have, the better this search will converge. With Smitsimax, you can freely select your
simulation depth. If you set the depth too high, your search can never converge. You will end up using random moves, or too few moves in
the last few depth levels. However, because of UCB, you will not need to explore the entire searchspace, like you would with (non-pruned)
minimax. The exploration and expansion will focus on interesting parts of the tree and you can get much deeper than you would expect. My 
CSB bot currently uses roughly 60k simulations per turn where my rivals in the top 10 use over a million for the same depth of search. I
can only  imagine how well this bot will perform once it is properly optimized. 


## What are the advantages and limitations of this approach?

### Advantages

-Maximum quality opponent prediction. The opponent prediction has the same quality as your own search, since everything is symmetric
between you and your opponent. Genetic algorithms don't share this feature and minimax might be less effective at this.

-Quick convergence: Relatively few sims needed for a reliable result.

-Potential for emergent behavior. An example of this is seen in my CSB blocker-pod. I currently have it set to try to reduce the opponent
travelled distance, but it also gets more score if my runner pod travels farther. This sometimes leads to my blocker using its shield to
boost my runner ahead. My runner and blocker each have their own tree, but they share goals in the evaluation, which tends to make them
cooperate. This is a strong feature of genetic algorithms that is also present in Smitsimax. 

-Few heuristics needed. In the evaluation you merely have to give "score" for what you want to achieve and you dont have to specify how to
achieve it. The possible moves are decided when creating children on the nodes and if done right, will be selected to achieve the highest
score. If you have some experience using this search, you can quickly code a usable bot.

### Limitations

-Because gamestates that correspond to a node are not uniquely determined (the opponent may do different things on each iteration) this
search might not be usable for many types of games. For example, if a game allows 4 types of moves: A, B, C and D and if in some
situations caused by the opponent,  move C is illegal, this search will not work. The allowed (legal) moves have to be independent of
opponents choices. This is the case in CSB, as you can always thrust, steer and shield, no matter what the opponent does. The same is true
for Code of Kutulu. In that game you always know which moves are legal, no matter what the opponent does (excluding the "yell" ability). 

-It is hard to use heuristics as part of the search, because you really dont know what is going to happen until after you finish your sim
and even then, things will be different on the next iteration. You lean heavily on the evaluation score telling your pods what is good and
what is bad. For example, you can't have a node select its children to "steer away to the right if it sees a blocker on the left". In one
particular iteration of the search, there could be a blocker on the left, but next time when you get to this node, there might not be. Only
statistics and evaluation will help you here.
