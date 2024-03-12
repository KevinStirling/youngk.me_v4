---
title: "Giving snake an apple"
date: 2022-11-05T10:57:44-04:00
description: 'Feeding the snake for growth, some stupid simple state management, and some code simplification'
image: images/posts/micro_snek.png
tags: ["godot4"]
draft: false 
---
Before tackling any new features, I wanted to make sure I atleast had a super bare minimum state management system in place. And a new beta for Godot 4 is out and I get to use a new (very minor) enhancement :D 

## Managing state

This is pretty much as simple as you can get, which I think fits for the tiny scope of this project. For now this is what that looks like

```
# GameManager.gd

...
var current_state := GlobalVars.State.PAUSE

func set_state(new_state: GlobalVars.State):
	current_state = new_state
	match current_state:
		GlobalVars.State.PLAY:
			timer.start()
			timer.paused = false
		GlobalVars.State.PAUSE:
			timer.paused = true
		GlobalVars.State.DEAD:
			timer.stop()
...
```

As you can see, it really only needs to manage the timer for now.

`GlobalVars.gd` is an autoloaded file that simply has an enum called State.

```
# GlobalVars.gd

extends Node

enum State {
	PAUSE,
	PLAY,
	DEAD
}
```

Now, any child nodes of the GameManager.tscn can set the state of the game by creating a reference to the GameManager node, and calling the `set_state` func.

```
# Snake.gd

...
@onready var manager = get_parent()
...

...
manager.set_state(GlobalVars.State.PLAY)
...
```

## Time to eat
Setting up the apple scene is simple. A `StaticBody2d` for the root node, attatch a `Sprite` node which for now I will lazily drop the snake tail's sprite into as a placehodler, and a `CollisionShape2d`.  For the snake to be able to eat this thing, we'll need a `Area2d` with a `CollisionShape2d` as its child node, which I have named `PickUpBoundary`.

My inital plan was to use the `Area2d`'s `body_entered` signal to trigger the function to make the snake grow. However for whatever reason, that signal is not working when the `Snake` body enters it. Everything is on the same collision layer and mask layer too, so it should be detectable. Might investigate this later and see if its a Godot 4 beta bug. 

So for a backup plan, I gave the `snake.tscn` an `Area2d` node called `PickUpArea`, and I instead connected the `area_entered` signal. 

```
# Apple.gd

extends Node2D

@onready var pickUpArea = $PickUpBoundary

signal spawn

func _ready():
	pickUpArea.area_entered.connect(_area_pickup)

func _area_pickup(area):
	if area.get_parent().has_method("grow"):
		area.get_parent().grow()
		emit_signal("spawn")

```

The callback function simply checks if the parent of the `Area2d` that entered the `PickUpBoundary` has a `grow` method, and calls it if it does.  Which looks like this

```
# Snake.gd

...

func grow():
	var new_seg = grid_coords[grid_coords.size() - 1]
	grid_coords.append(new_seg)

...
```

The reason I am able to simply set the  `Vector2` position for the new segment, rather than doing some vector math to calculate the next position, is thanks to my logic for moving the snake in my `_timer_timeout` func in the `GameManager.gd`.

```
#GameManager.gd

...
func _timer_timeout():
	var segment_next
	var segment_previous
...
	for s in range(snake.grid_coords.size()):
...
		else:
			grid.erase_cell(1, snake.grid_coords[s])
			segment_previous = snake.grid_coords[s]
			snake.grid_coords[s] = segment_next
			snake.body_segments[s].position = grid.map_to_local(snake.grid_coords[s])
			segment_next = segment_previous
	if current_state == GlobalVars.State.PLAY : 
		timer.start()
```

Because I'm just scooting the `Vector2`'s by keeping the previous segment in memory, it just kinda works. It might _feel_ a little lazy to have the last 2 `Vector2`'s in the array be the same value for a moment, but in reality this means that when that last line `segment_next = segment_previous` is executed for the new snake tail segment, it's making the previous snake's last tail segment the new one, which is exactly what we want. It ends up having effect of "growing forward" as the snake moves, since the end of the tail does not move for one of the timer timeouts.

![snek_apple](/images/posts/snek_apple.gif "snek")

## More apple
As you can see in the gif above, we get a new apple when we eat one! I created a `spawn` signal in `Apple.gd` that is emitted when the apple's `area_entered` callback is triggered by the snake eating the apple. 

In the `GameManager.gd`, I set up a `_new_apple()` func that will spawn the new instance in a random location in the playarea.

```
# GameManager.gd

...

func _new_apple():
	if get_node_or_null("Apple") == null :
		apple = apple.instantiate()
		add_child(apple)
		apple.spawn.connect(_new_apple)
	var temp_coords = bg_layer_coords 
	var new_apple_pos : Vector2 = temp_coords[randi() % temp_coords.size()]
	while snake.grid_coords.has(new_apple_pos):
		new_apple_pos = temp_coords.pick_random()
	apple.position = grid.map_to_local(new_apple_pos)
	
...
```

Here, we check to see if there is an Apple node in the scene tree. If there isn't we create an instance of one, and connect the `spawn` signal, the same signal that it emits on pickup. This is to make sure there is only one `Apple.tscn` instance, but this could change in the future.jh}

Oh, and thanks to the newest Godot 4 beta, which is Beta 4, I get to use a new built in array method, `pick_random`! 

Previously to get a random array element you would have to do something like this 
```
new_apple_pos = temp_coords[randi() % temp_coords.size()]
```

Not the most riveting new feature to test but hey I'm down. After that I have a while loop that will garuntee that the random value for the new Apple position is not occupied by the snake body.

## Other improvements
I moved the code that is used to update the snake when there are new segments available at the moment of a timer timeout to an `update_snake()` function. This way it can be used by the `ready()`, `_timer_timeout()`, and whatever other function may need it.

```
# GameManager.gd

...
func update_snake(grid_coords_pos = snake.grid_coords.size() - 1): 
	var current_segment = snake_segment.instantiate()
	add_child(current_segment)
	current_segment.position = grid.map_to_local(snake.grid_coords[grid_coords_pos])
	snake.body_segments.append(current_segment)
...
```

I also had to define a default value for the `grid_coords_pos` argument, since the `_timer_timeout()` func specifically needs the last segment of the snake to be updated, but the `ready()` func actually needs to call this for every snake body segment after the head, so the `ready()` func now looks like this 

```
# GameManager.gd

...
func _ready():
	bg_layer_coords = grid.get_used_cells(0)
	timer.timeout.connect(_timer_timeout)
	snake = snake_head.instantiate()
	for s in range(snake.grid_coords.size()):
		if s == 0:
			add_child(snake)
			snake.position = grid.map_to_local(snake.grid_coords[s])
			snake.body_segments.push_front(snake)
		else:
			update_snake(s)
	_new_apple()
...
```

Next update will most likely be triggering the death state, and some bug fixes
