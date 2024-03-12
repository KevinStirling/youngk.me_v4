---
title: "A new space for snake"
date: 2022-11-06T14:01:46-05:00
description: 'Bug fixes, new playarea, and new death triggers!'
image: images/posts/new_playarea.png
tags: ["godot4", "devlog"]
draft: false 
---

Oh yeah. We're rollin now. Bug fixes and new state trigger logic!

## Diagonal movement bug
Had to change the movement vector code to fix a bug that would allow the player to move in diagonals... which kind of breaks the whole rule set of Snake now doesn't it. This is because the `Input.get_vector()` func will create a vector based on all inputs it recieves. Good for some games like top down 2d or 3d games, not good for snake.

```
# Snake.gd

...
func _process(_delta):
	if Input.get_vector("left", "right", "up", "down") != Vector2.ZERO:
		if manager.current_state == GlobalVars.State.PAUSE :
			manager.set_state(GlobalVars.State.PLAY)
	if Input.is_action_just_pressed("left"):
		input_buffer = Vector2(-1, 0)
	elif Input.is_action_just_pressed("right"):
		input_buffer = Vector2(1, 0)
	elif Input.is_action_just_pressed("up"):
		input_buffer = Vector2(0, -1)
	elif Input.is_action_just_pressed("down"):
		input_buffer = Vector2(0, 1)
...
```

So I unfortunately had to break up my nice neatly coded movement code into a sequence of if statements, while still keeping the `get_vector` for detecting an input to start the game. I'm sure theres a hip way to consolidate this but for now this'll do.

## Dead snake
To detect if the snake's head has made contact with the snake's tail, I just added a `CollisionArea2d` to the `SnakeTail.tscn`, and add the following code to connect the `body_entered` signal.

```
# SnakeTail.gd

extends StaticBody2D

@onready var col_area = $CollisionArea
@onready var manager = get_parent()

func _ready():
	col_area.body_entered.connect(_on_area_2d_body_entered)

func _on_area_2d_body_entered(body):
	if body.is_in_group("SnakeHead"):
		manager.set_state(GlobalVars.State.DEAD)
```

I added the SnakeHead node to a group "SnakeHead" as a lazy check, I'll admit. This may change later if I find it stupid but it works well enough for now.


### Play area changes
As you can see from the header image, its pretty minimal. I added a new `TileMap` layer for the "frame", so the playarea does not end up being massive and take forever to traverse, and I can also decorate it later if I want / when I choose a final visual style. Overall a simple but pleasing enhancement from the janky floating window with dimensions that were slightly off from the `TileMap` :D

### Detecting wall collision
Ideally, in Godot 4 I could simply add a physics collision layer to the `TileMap`. However, because I've chosen not to use any physics based movement, I don't think this is possible.

The good news is, since I'm already storing the coordinates for all the cells that make up the playarea, all I need to do is check to see if the cell that the snake's head is about to move into is going to have an x or y value greater or less than the maximum x or y values used for the playarea.

```
# GameManager.gd

...
func _ready():
	bg_layer_coords = grid.get_used_cells(1)
	min_nav_x_y = bg_layer_coords[0]
	max_nav_x_y = bg_layer_coords[bg_layer_coords.size() - 1]
	
...

func is_in_boundaries(grid_coord):
	if (min_nav_x_y.x < grid_coord.x) and (max_nav_x_y.x > grid_coord.x):
		if (min_nav_x_y.y < grid_coord.y) and (max_nav_x_y.y > grid_coord.y):
			return true
	return false
...
```

This function gets called in the `_timer_timeout()` func.

```
# GameManager.gd

...
func _timer_timeout():
...
	if snake.grid_coords.size() > snake.body_segments.size() :
		update_snake()
	for s in range(snake.grid_coords.size()):
		if s == 0:
			if !is_in_boundaries(snake.grid_coords[s]):
				set_state(GlobalVars.State.DEAD)
...
```

_So close I can almost smell the polishing phase._