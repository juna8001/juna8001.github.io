---
layout: post
title: "When bool is not enough"
image: ../assets/posts/banners/abstract_dots.png
excerpt: In this post I want to share with you my approach for multi-layered boolean state control.
tags: [tip, code]
---

# When a bool is not enough

In games, it is often the case that we want a single thing to be blocked from multiple sources. Situations like that can be solved by a number of booleans, but it can get messy really easily. In this post I want to share with you my approach for that, which I find very useful.

## Problem

Let's say we are building a platformer. We have a `PlayerMovement` class that handles the player character moving around. A simple version could look like this:

```c#
public class PlayerMovement : MonoBehaviour
{
    public float moveSpeed = 5f;
    public float jumpForce = 5f;
    public Rigidbody2D rb;

    void Update()
    {
        float move = Input.GetAxisRaw("Horizontal");
        bool jump = Input.GetButtonDown("Jump");

        // Horizontal movement
        rb.velocity = new Vector2(move * moveSpeed, rb.velocity.y);

        if (jump)
        {
            // Jump movement
            rb.AddForce(Vector2.up * jumpForce, ForceMode2D.Impulse);
        }
    }
}
```

It has a clear purpose: it reads the input and turns it into player character movement.

Now, let's say we want to disable player input when we open the pause menu. The first thing that comes to mind is using a simple boolean:

```c#
public bool isMovementBlocked;
```

and then modifying input reading:

```c#
float move;
bool jump;

if (isMovementBlocked)
{
    move = 0;
    jump = false;
}
else
{
    move = Input.GetAxisRaw("Horizontal");
    jump = Input.GetButtonDown("Jump");
}
```

This would allow us to set the value of `isMovementBlocked` to `true` when we open a pause menu, then to `false` when we close it.

This works, but it quickly becomes fragile. Let's say that now we also want to block player movement when we are in a cutscene. We could try to use that same field, and it should mostly work, but let's see what happens when those two states (pause menu and a cutscene) overlap:

```
1. cutscene starts -> we block movement
2. player opens pause menu during cutscene -> we block movement that was already blocked
3. player closes pause menu -> we unlock movement, but we are still in a cutscene
4. cutscene finishes -> movement is unlocked
```

When we get to 3, there is a problem: the pause menu unlocks movement, but we are still in a cutscene.

A common solution is to use multiple boolean values. We could have:

```c#
bool isPauseMenuOpened;
bool isCutscenePlaying;

// Using property to calculate the final value
bool IsMovementBlocked => isPauseMenuOpened || isCutscenePlaying;
```

That's a solid solution — we have two independent values that affect the final result, and there is no problem when they overlap. But each time we have another reason to block movement, we need to add another boolean in `PlayerMovement` — a script that should ideally have no information about pause menus or cutscenes.

So, what would be a more elegant solution?

## Simple Solution — Blockable

My solution was to create a `Blockable` class (still looking for a better name). It looks like this:

```c#
public class Blockable
{
    public bool IsBlocked => _counter > 0;
    public event Action<bool> OnBlockChanged; 
    private int _counter;

    public void AddBlock()
    {
        _counter++;
    }

    public void RemoveBlock()
    {
        _counter--;
    }
}
```

It uses a hidden counter to track the number of blocks applied to a `Blockable`.

With this, we can create a field in our `PlayerMovement` class:

```c#
public Blockable blockable;
```

This allows us to lock and unlock from multiple places without worrying about overlaps. We can call `AddBlock()` when we want to block something, and then `RemoveBlock()` when we want to unblock.

This is great for when we are sure we will only call those methods in pairs — preferably with a `try/finally` block to ensure no exception breaks the sequence. If we, for example, call `AddBlock()` once and then `RemoveBlock()` twice, the counter would become `-1`, breaking the `Blockable`.

The simple implementation above works great when we have a linear flow of things happening. It works especially well with async methods, for example with the UniTask package:

```c#
blockable.AddBlock();
try
{
    // Do something...
}
catch (Exception e)
{
    // Handle exception
}
finally
{
    blockable.RemoveBlock();
}
```

But simple flows like that are rare in games. Very often we rely on events that may trigger multiple times. So, to make using `Blockable` safer and simpler, I created…

## Blockers — a Game Changer

If `Blockable` is something we can block, the `Blocker` class represents a single source of block. It looks like this:

```c#
public class Blocker
{
    public readonly Blockable Blockable;
    public bool IsBlocking { get; private set; }

    public Blocker(Blockable blockable)
    {
        Blockable = blockable;
    }

    public void Block()
    {
        if (!IsBlocking)
        {
            Blockable.AddBlock();
            IsBlocking = true;
        }
    }

    public void Unblock()
    {
        if (IsBlocking)
        {
            Blockable.RemoveBlock();
            IsBlocking = false;
        }
    }
}
```

`Blocker` allows me to call `Block()` and `Unblock()` multiple times in a row without breaking the `Blockable`. It’s essentially going back to the multiple-bools solution, but with the ability to create blockers dynamically at runtime.

I also like to add a shortcut method to `Blockable`:

```c#
public Blocker CreateBlocker()
{
    return new Blocker(this);
}
```

Blockers are great because you don’t have to think too much — you create a separate instance of `Blocker` for each block reason and just call `Block()` and `Unblock()` when that reason changes state. It might look like this:

```c#
public class SampleClass : MonoBehaviour
{
    public PlayerMovement movement;

    private Blocker movementBlocker;

    void Start()
    {
        movementBlocker = movement.blockable.CreateBlocker();
    }

    void SomethingHappensToBlock()
    {
        movementBlocker.Block();
    }

    void SomethingHappensToUnblock()
    {
        movementBlocker.Unblock();
    }
}
```

So, with the player movement example, we can create a blocker for a pause menu, a blocker for a cutscene, and a blocker for any other reason — none of which require any changes to the `PlayerMovement` class.

## Bonus — Final Improvements

One more change I like to include in my `Blockable` class is the `OnBlockChanged` callback — it allows me to react to block changes immediately instead of polling the value each frame.

The final version of a minimal `Blockable` for me looks like this:

```c#
public class Blockable
{
    public bool IsBlocked { get; private set; }
    public event Action<bool> OnBlockChanged; 
    private int _counter;

    public void AddBlock()
    {
        _counter++;
        AfterCounterChanged();
    }

    public void RemoveBlock()
    {
        _counter--;
        AfterCounterChanged();
    }

    public Blocker CreateBlocker()
    {
        return new Blocker(this);
    }

    private void AfterCounterChanged()
    {
        var blocked = _counter > 0;
        if (blocked != IsBlocked)
        {
            IsBlocked = blocked;
            OnBlockChanged?.Invoke(blocked);
        }
    }
}
```

Depending on project needs, this class is easy to customize — you can, for example, use a different type of callback or require each blocker to have a name and store them in the `Blockable`.

I hope you’ll find this useful. It’s a simple piece of code, but it has made my game code much cleaner and more bug-free.
