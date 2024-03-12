---
title: "Learning Godot 4 Tilemaps via Snake"
date: 2022-10-30T17:29:03-04:00
description: "Look at that bad boy shmove"
tags: ["godot4"]
draft: false
---

With Godot 4 now in beta, I've been starting to mess around with the new kit. And although there's a lot of really exciting 3d features coming to Godot 4, the 2d space is a bit more in my wheelhouse for now, as I continue to learn the engine.

One new feature I am looking forward to using in Godot 4 2d is the new `TileMap` node updates. To get myself farmiliar with the new `TileMap` api, I wanted a project that would let me dip my toes into this node, but I also wanted a project that had a small scope and ideally I wouldn't have to think up rules or functionality from scratch. So, I chose to remake Snake, using a `TileMap` as my main playarea!

## Concept

While this isn't really going to be using many of the new features of `TileMap`, most of it is new to me anyway as a Godot noob. The idea is to use the `TileMap` as a grid for the play space. The snake the player controls is made up of a series of grid squares, and every `x` seconds, the snake moves one grid square in the direction the snake is pointing, which is determined by input that is buffered between the movement timer's timeout duration.

## On the grid

The main scene contains a `TileMap` and a `Timer` node, underneath the parent `Node2d`, which has a script attached called `GameManager.gd`. The `TileMap` being the play area, and the `Timer` being the amount of time between each movement of the snake, which I currently have set to .5 seconds.

Using the `TileMap` node's Layer system, I am able to split the background layer from the snake. This way, when I'm modifying grid squares of the `TileMap` to show the snake move, I don't need to worry about redrawing the background.

![tilemap_layer](/images/posts/tilemap_layer.png "tilemap")

I created a `Snake` scene and added a `Snake.gd` script to the root `StaticBody2d` node, as well as adding a `Sprite2d` and `CollisionShape2d` of course.

Here's what that script looks like at the moment for just covering the movement.

```
# Snake.gd

extends StaticBody2D

@onready var manager = get_parent()

var input_buffer := Vector2.ZERO

var grid_coords := [Vector2(21, 22), Vector2(22,22), Vector2(23,22), Vector2(24,22)]
var body_segments: Array

func _process(_delta):
	if Input.get_vector("left", "right", "up", "down") != Vector2.ZERO:
		input_buffer = Input.get_vector("left", "right", "up", "down")

```

It's a little different from a basic 2d game set up, since I'm not moving the player kinematically, hence the `StaticBody2d` node. I'm just storing the vector from `Input.get_vector`, and using that to influce the direction the snake will move on the timer's timeout duration. The `grid_coords` is an array that will be storing the position of each snake segment position on the `TileMap`. Which leads me to the `SnakeTail` scene.... which is actually pretty much identical to the `Snake` scene at the moment, just without a script and a different sprite. For now it is just serving the purpose of being an instanced scene.

Now to tie it all together in the `GameManager.gd` script.

```
# GameManager.gd

extends Node2D

@onready var grid = $TileMap;
@onready var timer = $Timer;
@onready var snake_head = preload("res://Snake/Snake.tscn");
@onready var snake_segment = preload("res://Snake/SnakeTail.tscn");

var snake

func _ready():
	timer.timeout.connect(_timer_timeout)
	snake = snake_head.instantiate()
	for s in range(snake.grid_coords.size()):
		if s == 0:
			add_child(snake)
			snake.position = grid.map_to_local(snake.grid_coords[s])
			snake.body_segments.push_front(snake)
		else:
			var current_segment = snake_segment.instantiate()
			add_child(current_segment)
			current_segment.position = grid.map_to_local(snake.grid_coords[s])
			snake.body_segments.append(current_segment)
	timer.start()

func _timer_timeout():
	var segment_next
	var segment_previous
	for s in range(snake.grid_coords.size()):
		if s == 0:
			segment_next = snake.grid_coords[s]
			snake.grid_coords[s] = snake.grid_coords[s] + snake.input_buffer
			snake.position = grid.map_to_local(snake.grid_coords[s])
		else:
			grid.erase_cell(1, snake.grid_coords[s])
			segment_previous = snake.grid_coords[s]
			snake.grid_coords[s] = segment_next
			snake.body_segments[s].position = grid.map_to_local(snake.grid_coords[s])
			segment_next = segment_previous
		timer.start()
```

Basically, in the `_ready` func we set up the snake by instancing all of the body parts from the `Vector2`'s that are stored in the `Snake` scene's `grid_coords` array. To make things easy, the snake's head is always position `0` of the array, and the rest of the snake tail segments are added in sequential order. For both the snake head and body segments, instances of each scene are created for each of the `Vector2`'s in `grid_coords`, and pushed to the `Snake` scene's `body_segments` array. Since they are in sequential order, we know that `grid_coords[2]` is the position of the instanced scene stored in `body_segments[2]`. Ain't that fun.

Here you can see some of the `TileMap` api coming into play, utilizing the `map_to_local(Vector2i)` func, which takes a `Vector2` and returns the cenetered position of the cell in the `TileMap`'s local coordinate space. We do this to prvent the snake's sprites being off center on the grid, which would make things confusing when we start moving things around and adding pick up items to the `TileMap`.

Then, to actually move this bad boy, we connect the timer's timeout signal. When the `Timer` times out, the signal will call the `_timer_timeout()` func, and start to shift the position of the snake's head stored in `grid_coord` by adding the `input_vector` to it's current position `Vector2`. Then we just use the previous position of the snake's head as the position of the next object in `grid_coords`, and do the same for each body segment.

![snek_move](/images/posts/snek.gif "snek")

And that's it for basic movement! Next up is adding some basic state management, and adding the apple pickup item so you can grow up to be a nice big snake.
