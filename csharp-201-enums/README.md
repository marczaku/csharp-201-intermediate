Enums are an Excellent Tool in C# to represent Collections of Identifiers in a Type-Safe manner. Think of different kind of Elements, or Chess Piece Types, or different Phases of a Card Game. We will take a look at the original problem, and how different approaches, that are possible without the use of Enums, are not satisfying enough.

# 1 Using Multiple Bools to Represent Multiple Values

- Sometimes, we want to track a variable, that can have a fixed set of different values
- **For example:** GameState, CharacterType, Element, AttackType, Difficulty, Rarity, …
- We could try using `bool`, but then our Code would end up with a lot of bools very quickly:

```cs
public class PlayerChoice {
   public bool isRock;
   public bool isPaper;
   public bool isScissors;
}
```

Also, it would allow for unintended double-values:

```cs
PlayerChoice choice = new PlayerChoice();
choice.isRock = true;
choice.isPaper = true; // Error, it is supposed to only have one Value..
```


Okay, this is annoying. What else can we do?

# 2 Using an int to Represent Multiple Values

- We could use a number to represent that value
- One integer can store a lot of different values
- It can always only have one value at a time!
- Easy to change, good performance!

```cs
public class Game {
  // 0 = rock, 1 = paper, 2 = scissors
  public int playerChoice;
  // 0 = rock, 1 = paper, 2 = scissors
  public int aiChoice;
  
  public void ChooseWinner() {
    if(aiChoice == 0 { // ai chose rock
      if (playerChoice == 0) { // player chose rock
        Console.WriteLine("Draw!");
      } else if (playerChoice == 1) { // player chose paper
        Console.WriteLine("Win!");
      } else if (playerChoice == 2) {
        Console.WriteLine("Lose!");
      }
    }
  }
 }
```

BUT: It is difficult to keep track of the face that state 2 is equivalent to "scissors"

```cs
public void ChooseScissors() {
  this.playerChoice = 2;
}
```
  
ALSO: It is not safe: any programmer can assign any value:

```cs
  public void GetAiChoice() {
    this.aiChoice = -1337;
  }
}
```

---

# 3 Using a string to Represent the Value Human-Readable

- We could use a string to represent that value
- Easy to change, Readable

```cs
public class Game {
  // "rock", "paper", "scissors"
  public string playerChoice;
  // "rock", "paper", "scissors"
  public string aiChoice;
  
  public void ChooseWinner() {
    if(aiChoice == "rock" {
      if (playerChoice == "rock") {
        Console.WriteLine("Draw!");
      } else if (playerChoice == "paper") {
        Console.WriteLine("Win!");
      } else if (playerChoice == "scissors") {
        Console.WriteLine("Lose!");
      }
    }
  }
 }
```

BUT: We can easily have Spelling Errors:

```cs
public void ChooseScissors() {
  this.playerChoice = "Scisssors";
}
```

ALSO: It allows for invalid value, still:

```cs
  public void GetAiChoice() {
    this.aiChoice = "pizza";
  }
}
```

ALSO: The Performance of comparing strings is generally not so good.

---

# 4 Using constant values as a Look-Up

- How about pre-defining numbers?
- We could have a class that contains static fields that we then use

```cs
public class Sign {
  public const int Rock = 0;
  public const int Paper = 1;
  public const int Scissors = 2;
}
```

- Easy to change, good performance, readable

```cs
public class Game {
  // Sign
  public int playerChoice;
  // Sign
  public int aiChoice;
  
  public void ChooseWinner() {
    if(aiChoice == Sign.Rock {
      if (playerChoice == Sign.Rock) {
        Console.WriteLine("Draw!");
      } else if (playerChoice == Sign.Paper) {
        Console.WriteLine("Win!");
      } else if (playerChoice == Sign.Scissors) {
        Console.WriteLine("Lose!");
      }
    }
  }
 }
```

COOL: Spelling Errors will now give a Compile Error and become obvious right away:

```cs
public void ChooseScissors() {
  this.playerChoice = Sign.Scisssors; // ERROR: Symbol does not exist!
}
```

BUT: I can still assign Invalid Values:

```cs
  public void GetAiChoice() {
    this.aiChoice = -1337;
  }
}
```

ALSO: I can by mistake assign numbers that are not meant to be assigned here:

```cs
  public void GetAiChoice() {
    this.aiChoice = ChessPiece.Rook;
  }
}
```

---

# 5 Using Enums

```cs
public enum Sign {
  Rock,
  Paper,
  Scissors
}
```

- You can use Enums like Types:
- And access Enum Values like Static Fields:
```cs
public class Game {
  // You can use Enums like Types:
  public Sign playerChoice;
  public Sign aiChoice;
  
  public void ChooseWinner() {
    if(aiChoice == Sign.Rock {
      if (playerChoice == Sign.Rock) {
        Console.WriteLine("Draw!");
      } else if (playerChoice == Sign.Paper) {
        Console.WriteLine("Win!");
      } else if (playerChoice == Sign.Scissors) {
        Console.WriteLine("Lose!");
      }
    }
  }
 }
```
      
It detects spelling errors:
```cs
playerChoice = Sign.Scisssors; // ERROR: Symbol does not exist!
```

It does not allow assigning invalid values:
```cs
playerChoice = -1337; // ERROR: Cannot cast `int` to `Sign`
playerChoice = ChessPiece.Rook; // ERROR: Cannot cast `ChessPiece` to `Sign`
```


---

# 6 Casting
You can explicitly cast enum values from numbers:

```cs
int userChoice = 2;
Sign sign = (Sign)userChoice;
```

Or from strings:

```cs
string userChoice = "Rock";
Sign sign = Enum.Parse<Sign>(userChoice);
```

## 7. Enum Values
Enums are internally stores as numbers:
```cs
public enum GameState {
  MainMenu, // = 0
  Pause, // = 1
  Playing // = 2
}
```

You can assign enum values manually:
```cs
public enum GameState {
  MainMenu = 0,
  Pause = 1,
  Playing = 2
}
```

It's useful in case that you delete some values and in your SaveGame you stored the Numeric Valaue of this Enum.
```cs
public enum GameState {
  Mainmenu = 0,
  Playing = 2
}
```
