# 1 Delegates

Delegates are an Object Type that allows Type-Safe handling of Method Pointers. In other words: They allow you to store Methods in Variables, pass them somewhere else and then execute the Methods in that other place.

They are often used for:
- callbacks, when you want to be informed as soon as something has happened which will take a certain amount of time
  - e.g. when the Game has Started or Ended
  - when a new Monster has Spawned
  - when a player has joined
- injections, if you want a caller to be able to inject code into your existing method
  - e.g. a Filter-Method when searching for Players

## 1.1 Quick Example:

```cs
public class Spawner {
    public delegate void SpawnDelegate(GameObject gameObject, Vector3 position);

    public void SpawnPlayer(string name, SpawnDelegate onSpawn);
}

public class GameStartButton(){
    void Start(){
        spawner.SpawnPlayer("Marc", OnSpawn);
    }

    void OnSpawn(GameObject gameObject, Vector3 position){
        player = gameObject;
        leaderboard.Add(gameObject);
        playerList.Add(gameObject);
        player.GetComponent<Player>().SetUpLooks(settings);
    }
}
```

## 1.2 Defining Delegates

Delegates define the Signature of Methods. They define the
- return type
- the list of parameters, including types

```cs
public delegate void GameStartDelegate();

// matches:
void GameStart(){
    SpawnPowerUps();
}
```


```cs
public delegate bool PlayerFilter(Player player);

// matches:
bool GetActivePlayer(Player player){
    return player.IsActive;
}
```

```cs
public delegate void SpawnDelegate(GameObject gameObject, Vector3 position);

// matches:
void SetNewTarget(GameObject gameObject, Vector3 position){
    path = FindPathTo(position);
    target = gameObject;
}
```

## 1.3 Using Delegates

You can use Delegate Types for Members, Parameters, Generic Parameters and Local Variables, just like any other type:

```cs
public class GameManager {
    public GameStartDelegate OnGameStart;

    public IEnumerable<Player> FindPlayers(PlayerFilter filter){
        foreach(var player in allPlayers)
            if(filter(player))
                yield return player;
    }

    public void Execute(Action action){
        action();
    }
}
```

## 1.4 Invoking Delegates

Just receiving a Method Pointer is boring. What makes them so interesting is being able to actually call the Method which was passed:

```cs
public delegate void SpawnDelegate(GameObject gameObject, Vector3 position);

void SpawnPlayer(SpawnDelegate onSpawn) {
    Transform spawnSpot = SelectRandomSpawnSpot();
    GameObject player = Instantiate(playerPrefab, spawnSpot.position, spawnSpot.rotation);
    onSpawn(player, spawnSpot.position);
}
```

This is the convenient short-hand writing for invoking a Delegate:

```cs
onSpawn(player, spawnSpot.position);
```

This is the more explicit (old-fashioned way):

```cs
onSpawn.Invoke(player, spawnSpot.position);
```

## 1.5 Reference Type

Delegates are reference types and can therefore be null:

```cs
public GameStartDelegate OnGameStart;


void StartGame() {
    // Inform about Game Start:
    OnGameStart(); // Possible NullReferenceException!
}
```

### 1.5.1 Null-Check:

The long way:

```cs
if(OnGameStart != null)
    OnGameStart();
```

### 1.5.2 Null-Propagating Operator:

My Favorite solution:

```cs
OnGameStart?.Invoke();
```

### 1.5.3 Just Assign a Non-Null Value:

This solution uses lambda expression, more on that later.\
My old Lead Programmer preferred this solution, because it removes many null-checks from your code. But the Null-Propagating Operator did not exist, yet, at that point.

```cs
public GameStartDelegate OnGameStart = () => {};
```

### 1.5.4 No Null-Check

Sometimes, you don't want to check for null, because you don't allow for a function argument to be null, for example. In this case, don't just Null-Propagate the issue that someone e.g. called your Filter-Method without passing a valid Delegate.

## 1.6 Chaining

A pretty cool Function: You can actually combine Delegates and pass a chain of them on:

```cs
public GameStartDelegate OnGameStart;
```

```cs
public class Player {
    void Start() {
        gameManager.OnGameStart += InitializePlayer;
    }

    void InitializePlayer(){/*...*/}
}
```

This way, if there is multiple players, they will all just add themselves to a List of Delegates and be invoked one after another:

```cs
delegate void SimpleDelegate();

static void Main(){
    SimpleDelegate methods = MethodA;
    methods += MethodB;
    methods += MethodB;
    methods();
}

static void MethodA => Console.Write("A");
static void MethodB => Console.Write("B");
```

Output:
```
ABB
```

### 1.6.1 Removing from the Chain

In the Same way, you can remove a Delegate from the Chain:

```cs
public class Player {
    void OnDestroy() {
        gameManager.OnGameStart -= InitializePlayer;
    }

    void InitializePlayer(){/*...*/}
}
```

```cs
delegate void SimpleDelegate();

static void Main(){
    SimpleDelegate methods = MethodA;
    methods += MethodB;
    methods += MethodB;
    methods -= MethodB;
    methods();
}

static void MethodA => Console.Write("A");
static void MethodB => Console.Write("B");
```

Output:
```
AB
```

### 1.6.2 Overriding the Chain

If you just assign a value to a Delegate Variable, the Chain gets overridden:

```cs
delegate void SimpleDelegate();

static void Main(){
    SimpleDelegate methods = MethodA;
    methods += MethodB;
    methods += MethodB;
    methods -= MethodB;
    methods = MethodC;
    methods();
}

static void MethodA => Console.Write("A");
static void MethodB => Console.Write("B");
static void MethodC => Console.Write("C");
```

Output:
```
C
```

- Removing a Delegate won't throw an error if it's not in the chain.
- If a Delegate is in the chain twice, it will be removed just once.
- You should not rely on the order of delegates in the chain, but you can.
- Internally, it creates a new Object instance containing an Array every time, so be aware of the Heap Allocations

# 2 Events

```cs
public class GameManager{
    public delegate void GameStartDelegate(int numberOfPlayers, DateTime startTime, Team a, Team b);

    public GameStartDelegate OnGameStart;
}
```

```cs
public class Player{
    void Start(){
        DisablePlayer();
        gameManager.OnGameStart += InitializePlayer;
    }

    void InitializePlayer(int numberOfPlayers, DateTime startTime, Team a, team b){
        this.color = a.Contains(this) ? Color.Red : Color.Blue;
        StartCountdown(startTime);
        ShowPlayerList(numberOfPlayers);
        EnablePlayer();
    }
}
```

Delegates are very commonly used for Events / Event Listeners, Like in the example above:
- Classes can subscribe for updates on `OnGameStart`-Events.
- The GameManager will invoke all subscribers when the Game Start happens.

The Problems:

## 2.1 Anyone can Invoke

```cs
    public GameStartDelegate OnGameStart;
```

Anybody that can see the Delegate, can Invoke it:

```cs
void Start(){
    gameManager.OnGameStart.Invoke(default, default, default, default);
}
```

## 2.2 Anyone can override all other subscribers

```cs
void Start(){
    DisablePlayer();
    // Assigns only the own Delegate, removing all others from the Chain:
    gameManager.OnGameStart = InitializePlayer;
}
```

## 2.3 Events

```cs
    public event GameStartDelegate OnGameStart;
```

By using the `event` Keyword, you allow ONLY the class owning the Field:
- to Invoke it.
- to assign using `=` instead of `+=` or `-=`

All other classes can just use `+=` and `-=` in order to subscribe and unsubscribe to the event.

# 3 Generic Delegates

Since defining a Delegate Type and then using them can lead to a lot of boilerplate code (unnecessary overhead that's just needed due to make thing work), you can also use C#'s Generic Delegate Types:

```cs
public delegate void SpawnDelegate(GameObject gameObject, Vector3 position);
public event SpawnDelegate OnSpawn;
```

Then becomes:

```cs
public Action<GameObject, Vector3> OnSpawn;
```

## 3.1 System.Action

System.Action is the Base-Type that you can use for Delegates with `void` Return Type:

### 3.1 No Parameter

```cs
public Action OnGameStart;

OnGameStart += SpawnPlayer;

void SpawnPlayer(){/*...*/}
```

### 3.2 One Parameter

```cs
public Action<bool> OnGamePaused;

OnGamePaused += PauseMusic;

void PauseMusic(bool isPaused){/*...*/}
```

### 3.2 More Parameters

```cs
public Action<List<Player>, Map> OnGameStarted;

OnGameStarted += InitializeHUD;

void InitializeHUD(List<Player> players, Map map){/*...*/}
```

## 3.1 System.Func

System.Func is the Base-Type that you can use for Delegates with Return Types which are not `void`:

### 3.1 No Parameter

```cs
public Func<int> HealthChanged;

HealthChanged += UpdateUI;

void UpdateUI(int newHealth) => healthLabel.text = newHealth;
```

### 3.2 One Parameter

```cs
public IEnumerable<Player> Filter(Func<Player, bool> filterFunc){
    foreach(var player in allPlayers){
        if(filterFunc(player))
            yield return player;
    }
}

void Start(){
    var activePlayers = Filter(IsPlayerConnected);
    /*...*/
}

bool IsPlayerConnected(Player player){
    return player.Disconnected;
}
```

### 3.2 More Parameters

```cs
public IEnumerable<Player> Sort(Func<Player, Player, int> sortFunc){
    // implement sorting.
    // Instead of if a > b, use:
    // if(sortFunc(playerA, playerB) > 0)
}

void Start(){
    Sort(ComparePlayerStrength)
}

int ComparePlayerStrength(Player playerA, Player playerB){
    return playerA.Strength.CompareTo(playerB.Strength);
}
```

# 4 Lambda Functions

- What Generic Delegates are for Defining Delegates...
- ...Lambda Functions are for Passing Delegates

```cs
void OnEnable(){
    gameManager.OnGameStart += SpawnPowerUps;
}

void SpawnPowerUps(){
    for(int i = 0; i < powerUpCount; i++){
        SpawnPowerUp();
    }
}
```

Would translate to:

```cs
void OnEnable(){
    gameManager.OnGameStart += () => {
        for(int i = 0; i < powerUpCount; i++){
            SpawnPowerUp();
        }
    };
}
```

You see, we can make up an anonymous function on the spot.

This is exceptionally useful for Callbacks and LINQ:

## 4.1 Callback Example

```cs
public class Server {
    public void ConnectToServer(string userName, string password, Action<bool> callback){
        /*...*/
        // This process takes some time. Maybe 0.5 seconds, maybe 10.
        // We use the callback so we can continue when the login has completed.
        // Instead of freezing the app until it happens
    } 
}
```

```cs
void OnLoginButtonClick(){
    server.ConnectToServer(userName, password, (success)=>{
        if(success)
            SceneManager.Load("MainMenu");
        else
            ShowErrorPopup();
    });
}
```

## 4.2 LINQ Example

```cs
var foodBowls = animals
    .Where(animal => animal.IsHungry)
    .OfType<Cat>()
    .Where(cat => cat.FavoriteFood == "Tuna")
    .Select(cat => cat.FoodBowl);

assistant.OrderToFill("Tuna", foodBowls);
```

## 4.3 Syntax

### 4.3.1 Statement Lambda - Long, Explicit Version

This one basically looks exactly like a Method with the Exception of the `=>` Arrow-Operator.

```cs
(int a, int b) => {
    return a.CompareTo(b);
}
```

### 4.3.2 Statement Lambda - Implicit Parameter Types

Since the Parameter Types are usually very clear due to the Lambda Function's Context, they are usually skipped:

```cs
(a, b) => {
    return a.CompareTo(b);
}
```

### 4.3.3 Expression Lambda

If your Function only consists of one line of code, you can use the Expression Body Syntax:

```cs
(a, b) => a.CompareTo(b);
```

### 4.3.4 No Parentheses for Simple Parameters

If there is only one argument, you also do not need to use `()`:

```cs
player => player.IsDisconnected;
```

The last example is the same as:

```cs
(Player player) => {
    return player.IsDisconnected;
}
```

Powerful!

## 4.4 Capturing Variables

You can access variables from the outside within your Lambda Functions:

```cs
void Start(){
    ui.AskForUserName((name)=>{
        ui.AskForPassword((password) => {
            server.Login(name, password, success => {
                if(!success)
                    ui.ShowErrorPopup($"User {name} failed to login.");
                else
                    LoadGame();
            });
        });
    });
}
```

These variables are "captured" and will not be cleaned up by the Garbage Collector until the Lambda Delegate is not referenced anymore.

## 4.4.1 Unexpected Side Effects

```cs
void Start(){
    for(int i = 0; i < 5; i++){
        SpawnPowerUp((success) => {
            if(!success){
                Debug.Log($"Power Up Spawn #{i} Failed.");
            }
        })
    }
}

public void SpawnPowerUp(Action<bool> callback){
    StartCoroutine(Co_SpawnPowerUp(callback));
} 

IEnumerator Co_SpawnPowerUp(Action<bool> callback){
    yield return new WaitForSeconds(0.5f);
    callback(false);
}
```

Output:
```
Power Up Spawn #5 Failed.
Power Up Spawn #5 Failed.
Power Up Spawn #5 Failed.
Power Up Spawn #5 Failed.
Power Up Spawn #5 Failed.
```

To Fix this: You need to capture the variable that's modified in the outer scope as a local variable within the inner scope:

```cs
void Start(){
    for(int i = 0; i < 5; i++){
        int current = i;
        SpawnPowerUp((success) => {
            if(!success){
                Debug.Log($"Power Up Spawn #{current} Failed.");
            }
        })
    }
}
```

Output:
```
Power Up Spawn #1 Failed.
Power Up Spawn #2 Failed.
Power Up Spawn #3 Failed.
Power Up Spawn #4 Failed.
Power Up Spawn #5 Failed.
```

Now, the Compiler knows, that `current` will become invalid after each Iteration and therefore, it needs to be captured by itself and cannot be shared.

## 5 Summary

As you can see, Delegate and Lambda Functions are an extremely powerful, but also difficult tool.

Often, you can use `interfaces` instead of `delegates`. Delegates are basically interfaces with only one Function.

Delegates and Lambdas are a bit more flexible and light-weight, but they can become insane complex in certain edge cases.

You should get comfortable using Delegates for:
- callbacks
- event listeners

You should get comfortable using Lambdas for:
- quick use of callbacks
- LINQ
- writing extensible methods