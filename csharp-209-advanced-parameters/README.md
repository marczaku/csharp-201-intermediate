# 1 Params

```cs
public Matrix4x4 Max(params int[] numbers){
  int max = numbers[0];
  for(int i = 1, i < numbers.Length; i++)
    max = Max(max, numbers[i]);
  return max;
}
```

The `params` Keyword is used in combination with an Array-Type Parameter.
- It can only be used for the last Parameter

```cs
Console.WriteLine(Max(3, 5, 7, 12));
```

It allows callers to call this method passing multiple single arguments.


```cs
Console.WriteLine(new int[]{3, 5, 7, 12});
```

It has the same effect as manually creating an Array on the fly:
- The new Array is allocated on the Heap
- This will produce garbage, if the array is not used or referenced any further

Therefore: Use with Caution!

# 2 In

```cs
struct Matrix4x4;
```


```cs
public Matrix4x4 Multiply(in Matrix4x4 a, in Matrix4x4 b){

}
```

The `in` Keyword is used in combination with Value-Type Parameters.
- It makes the arguments readonly within the Method.
- Which leads to Compile Errors, if you try to change any argument' value

```cs
MatrixMaths.Multiply(in player.matrix, in enemy.matrix);
```

This allows the Compiler
- To safely pass in the Value-Type as a Reference
- Without creating a Copy
- This can greatly boost your performance!

# 3 Ref

The Ref keywords lets you pass arguments as Reference rather than value.\
This can be used in two ways:

## 3.1 Ref (Value-Type)

```cs
struct Vector3;
```

```cs
public void Invert(ref Vector3 vector){
  vector.x = -vector.x;
  vector.y = -vector.y;
  vector.z = -vector.z;
}
```

```cs
Vector3 vector = new Vector(3f, 1f, 0f);
VectorMaths.Invert(ref vector);
Console.WriteLine(vector); // Output: [-3, -1, 0]
```

The `ref` Keyword here will make your `struct` behave like a `class`:
- The value will not be copied
- Changes to the value will affect the value outside the Method as well

## 3.2 Ref (Reference-Type)

```cs
abstract class Enemy {public int level;}
class SimpleEnemy : Enemy {}
class SuperBoss : Enemy {}
```

```cs
public void LevelUp(ref Enemy enemy){
  if(enemy.level < 9){
    enemy.level++;
  } else {
    enemy = new SuperBoss();
  }
}
```

```cs
Enemy target = new SimpleEnemy();
target. level = 8;
GameManager.LevelUp(ref target);
Console.WriteLine(enemy.GetType()); // Output: SimpleEnemy
GameManager.LevelUp(ref target);
Console.WriteLine(enemy.GetType()); // Output: SuperBoss
```

The `ref` Keyword here will actually allow the Method to change the reference of your class Variable!
In other words: it can
- not only change the values within the object instance that you passed (it always can with reference types)
- but it can also change what object instance your field or variable points to

# 4 Out

```cs
public bool TryGetEnemy(out Enemy enemy){
  if(enemyQueue.IsEmpty){
    enemy = null;
    return false;
  }
  enemy = enemyQueue.Dequeue();
  return true;
}
```

The `out`-Keywords allows your Method to return additional values in your Methods.
- The `out`-Parameter MUST be assigned before the Method ends / returns

```cs
Enemy enemy;
if(TryGetEnemy(out enemy)){
  Attack(enemy);
}
```

As you can see, the `enemy`-Variable does not need to be initialized.


```cs
if(TryGetEnemy(out Enemy enemy)){
  Attack(enemy);
}
```

This pattern is so popular, that C# even has a convenience syntax to pass `out`-Param-References into Methods.

# Summary

There is some Similarities, but also big differences between these keywords. Here's an overview to compare:

| | `in` | `ref` | `out`|
|-|:----:|:-----:|:----:|
| Is Passed as reference |✅|✅|✅|
| Avoids Copying / Cloning |✅|✅|✅|
| Needs to be initialized before calling |✅|✅|❌|
| Needs to be assigned within the method |❌|❌|✅|
| Can be assigned to within the method |❌|✅|✅|
| Caller needs to explicitly acknowledge* |❌|✅|✅|

*meaning, that the caller needs to say e.g. `CallMethod(ref argument)`