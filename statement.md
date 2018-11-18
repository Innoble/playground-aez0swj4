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
        Pods[4]  
        // all 4 pods that will be simulated
        Node[4] current  
        // 4 current nodes, one for each pod. You start with root nodes.
        float[4] lowestscore 
        // the lowest score earned by the tree, one for each pod.
        float[4] highestscore 
        // the highest score earned by the tree, one for each pod.
        float[4] scaleparameter  
        // the scaleparameter calculated by subtracting the lowest score from the highest. This is needed for the UCB formula.
        
        void Reset()   
        // To reset the sim after search
        
        void Search() 
        // The important bit, the actual search
        
        void Play() 
        // a CSB specific algorithm that handles movement and collisions (see Magus CSB postmortem)
        
        void BackPropagate() 
        // each tree has the score result backpropagated along the branch of the tree.
    }

    
The tree consists of nodes and each node in the tree needs the following things:
    
    class Node 
    {
        Node parent // each node has a parent except the root node
        float score // the total score obtained by this node. To get the average, you can divide by the number of visits
        int firstChildIndex // I use a preallocated array where the first child of the node has an index for this array.
        int childCount // the number of children
        int visits // the number of times this node has been visited
        int angle // this is part of the move, a CSB bot needs an angle
        int thrust // a CSB bot needs a thrust
        bool shield // whether or not the shield is activated. This could be part of the thrust variable (set to -1 or something)
    
    }
 
    The Search method looks like this:
    
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
                            node.MakeChildren() // give the node children if it doesnt have them, all possible moves you want to allow

                        Node child = node.Select() // select a node that you want to use for the sim. At first it is best to random a few times. I currently random 10 times. After this I use the UCB formula to select a child. 

                        child.visits++;

                        current[i] = child; // the child becomes the current node

                        pod.ApplyMove // do the stuff the nodes tell you to do (rotate, accelerate, shield etc.)
                    }

                    Play(); // Simulate what actually happens, including movement and collisions
                    depth++;
                }
                
                float[4] score = GetScore(); // get a score for each pod when the simulation depth is reach

                for (each pod, 4 times total)
                {
                    update the lowest score, highest score and scaleparameter for this pod. 
                }

                Backpropagate() // We go up the tree back to the root node, adding the score to each node. We do this for each pod (4x)
            }

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
