# Smitsimax

*This is a work in progress. I will add more information and hopefully some runnable code*

Recently I have been working on a bot for Coders Strike Back (CSB), a bot-arena on codingame. I reached top 10 in about 20 hours of coding with a bot that still has various problems, such as: timeouts, missing features and a suboptimal language choice (C#). The only thing this bot really has going for it is the search. Most searches that are being used in CSB are a variant of Genetic Algorithms or Minimax. I am not completely sure if my search doesn't already exist in some form, but if it doesn't, I have coined the name "Smitsimax", more as joke than anything else. If anyone recognizes it and knows its true name, do let me know.

CSB is a two player zero sum game. This makes something like minimax a natural choice as a search algorithm. The main problem is the simultaneous nature of the game. Both players decide what to do simultaneously and after this, the turn is calculated. The basic version of minimax requires players to move one after the other, knowing what the other player did the turn (ply) before. To solve the problem of simultaneous gameplay, one can make the choice to use the "paranoid" option. This means you assume the other player is going to know what you did and makes the best possible counter. This is an assumption that can be good or bad, depending on the state of the game and the possible strategies you can use. Another possible option is to calculate all possible combinations of moves. If one player has 6 possible moves and the other does as well, then there are a total of 36 possible combinations. You can then average the result, minimize the worst result or maximize the best result for each of your moves. All of these choices may have good or bad results depending on many factors.

Smitsimax has some things in common with minimax, but is also different. Both search algorithms have a tree of possible moves. Each move has a node and each node has children that correspond with the moves on the next turn. In minimax there is only one tree. The tree has moves by both players in it and each node corresponds to a single absolute gamestate. When you get to this node, you know exactly what the game looks like. Smitsimax has a separate tree for each player (or each pod, as in CSB, meaning two trees per player) and each node on the tree does not directly correspond to a gamestate. At first glance the trees are completely separate. During the search, moves are selected on each tree, randomly at first and by Upper Bound Confidence formula later. Each turn is simulated with the selected moves. When sufficient depth is reached, an evaluation is done for each player (pod) in the game and the result is backpropagated along the respective trees. The first time this is done, every node does correspond to a single gamestate. However, the next time the same node is selected by a player, the other player(s) may select different nodes, leading to a different gamestate. The differences may be larger for greater depth levels. If the search is able to converge, this is not a problem. The (best) branches to which each tree converges should together, correspond to a single gamestate per node. 

Let's look at this step by step in pseudo code, written for CSB:


At the start of a turn you get all the information you need as input. You have your Pod class with information such as:

class Pod
{
    position
    velocity
    shield 
    ...
}

You need two instances of each pod. One is the base instance you get as you start the turn. You need to keep this information so that you can reset to it after every search-run. The other is the evolving pod that changes during the search.

There are various ways to set up your simulation and Smitsimax-search. I currently use a static Sim class with 4 variables and 4 methods that looks somewhat like this:

    static class Sim
    {
        Node[4] current 
        lowestscore[4]
        highestscore[4]
        scaleparameter[4]
        
        void Reset()   //To reset the sim after search
        
        void Search() //The important bit, the actual search
        
        void Play() //a CSB specific algorithm that handles movement and collisions (see Magus CSB postmortem)
        
        void BackPropagate() // each tree has the score result backpropagated along the branch of the tree.
        
    }
    
    




```C# runnable
// { autofold
using System;

class Hello 
{
    static void Main() 
    {
// }

Console.WriteLine("Hello World!");

// { autofold
    }
}
// }
```

# Advanced usage

If you want a more complex example (external libraries, viewers...), use the [Advanced C# template](https://tech.io/select-repo/386)
