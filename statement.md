# Smitsimax

*This is a work in progress. I will add more information and phopefully some runnable code*

Recently I have been working on a bot for Coders Strike Back (CSB), a bot-arena on codingame. I reached top 10 in about 20 hours of coding with a bot that still has various problems, such as: timeouts, missing features and a suboptimal language choice (C#). The only thing this bot really has going for it is the search. Most searches that are being used in CSB are a variant of Genetic Algorithms or Minimax. I am not completely sure if my search doesn't already exist in some form, but if it doesn't, I have coined the name "Smitsimax", more as joke than anything else. If anyone recognizes it and knows its true name, do let me know.

CSB is a two player zero sum game. This makes something like minimax a natural choice as a search algorithm. The main problem is the simultaneous nature of the game. Both players decide what to do simultaneously and after this, the turn is calculated. Minimax requires players to move one after the other, knowing what the other player did the turn (ply) before. To solve the problem of simultaneous gameplay, one can make the choice to use the "paranoid" option. This means you assume the other player is going to know what you did and does the best possible counter. This is an assumption that can be good or bad, depending on the state of the game and the possible strategies you can use. Another possible option is to calculate all possible combinations of moves. If one player has 6 possible moves and the other as well, then there are a total of 36 possible combinations. You can then average the result, minimize the worst result or maximize the best result for each of your moves. 




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
