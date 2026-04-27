+++
title = 'Godot Grid Inventory'
date = 2026-04-26T20:01:54-04:00
draft = true
+++

For a recent gamejam I chose to make a game that had most of the gameplay focused on a grid based inventory system. I had never made one before and thought it might be a fun way to learn how. In the end, it sent me down a rabbit hole of trying various approaches (and never finished the jam whoopsie) and eventually landed on something that felt simple and flexible to work with.

# Forget about the grid
My first instinct when I started tinkering was to utilize the GridContainer UI node. It organizes a bunch of nodes into a grid, and in theory you can treat all the interations with the items as a UI action.

No, this is bad, don't do it (I mean you can, it just gets way too complex in my opinion). If you are making an inventory where each item only takes up on grid cell, this approach is actually probably fine, but if you want items at take up various dimensions of cells, this gets messy fast.

After some research I learned of a Vector2 method I have somehow avoided knowing for years: `Vector2.snapped()`. Takes a Vector2, and returns a Vector2 rounded to the nearest multiple of the `step` value. If you use this to update the position of a node thats being dragged by a mouse, this effectively makes the whole screen a grid.

I would have used this along with the 2d array concept from the first attempt, but after some tinkering, it was just too much to constantly check the array to see if the grid position the dragged item is hovering over is a valid drop position. (TODO: Update this with an explaination of the goal for hwo this should be handed). That's when I realized... im just checking for collision.

# Forget about the grid (no, seriously just do it)
All we have to do is add collision shapes to the items that are dragged around, and track how many collision objects are colliding with it. (when a collision is detected, incredment count, when the shape is left, decrement the count). Any position that is on the grid while collision count value is 0, is a valid posiiton! No checking if a 2d array contains something at the grid position, no math (yet) to determine what cells are needed to check based on the size of the item, just simple collision shapes.

After that, pretty much all thats left to check is if the item is within the grid bounds itself. One way to do this would be to maybe set a grid position as an export variable so I can update it's position on the screen, and only have the `snapped()` movement and collision checks happen when the items is within that grid position + grid size. However, I wanted to make this a little easier to work with and be able to dynamically move the grids position around the screen if needed. Which leads me to SubViewports...

# The game within the game
- (brief) discuss decision to utilize subviewports, shout out bibbinbits godot con talk about using it to show the grid in dome keeper
- (tie it together) explain how using subviewports + the snapped() vector2 method + collision shapes allows you to keep it simple, avoid using a UI node to handle the grid + snapping + posisition tracking.
