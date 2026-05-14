+++
title = 'Building a Grid Inventory System in Godot'
date = 2026-05-13T20:29:02-04:00
draft = true
+++
# What's huh?
A couple months ago, a game jam that I did not complete sent me on an adventure. I chose to make a game with a grid inventory system, and since I knew it would be atleast a little difficult, I wanted to make it the main interface for interaction. After struggling to figure out something that worked during the jam time, I ended up just wanting to solve this problem for myself, so I gave up on the jam to do just that, and in the end I came up with a solution I really like!

Just for reference, I am going for a "Resident Evil style" inventory system, so organizing objects with different sizes and shapes, and being able to rotate them to fit the together.

> NOTE! This isn't going to be an in-depth tutorial, mostly a discussion of some cool tricks to make this system work nicely. If you want to try this out for yourself though, the full source is availble on my github in my [godot tools collection](https://github.com/KevinStirling/godot-tools/tree/main/components/grid_inventory).


# Some basics
So obviously the first thing to tackle is clicking and dragging if we are going to make an interface for a player to arrange objects on a grid. This is the easy part, I ended up using a button attached to my Item node that matches the size of the sprite using a tool scirpt, but theres a couple ways to approach this one. I particularly like this way of leveraging a control node for the click interaction, because I can just use the built in `button_up` / `button_down` signals. Here's what that looks like.

_item.gd_
```gdscript
@tool
class_name Item
extends Node2D

@export var handle: Button
@export var item_sprite: Texture2D:
	set(value):
		item_sprite = value
		if %Sprite:
			%Sprite.texture = value
@export var item_sprite_size: Vector2 = Vector2(128,128)

var dragging: bool = false
var offset: Vector2 = Vector2.ZERO

func _ready() -> void:
	handle.size = item_sprite_size
	handle.position = -item_sprite_size * .5
	
func _process(delta):
	if dragging:
		position = get_global_mouse_position() - offset

func _on_handle_button_down() -> void:
	dragging = true
	top_level = true
	offset = get_global_mouse_position() - global_position

func _on_handle_button_up() -> void:
	dragging = false
	top_level = false

```
I'm glossing over some details here (and will continue to do so throughout this post!), but I also exported the sprite so I can just use this Item node as a instanced scene later, and just toss whatever sprite on the instance. Later I could even dump all that into a resource, but let's not get ahead of ourselves.

# The secret sauce
Now, we are going to need this item to snap to the grid. Somehow I had gone this long (I don't know, like 5 years of Godot tinkering?) without realizing that Vector2 has a method called [snapped( )](https://docs.godotengine.org/en/stable/classes/class_vector2.html#class-vector2-method-snapped). You can read the docs there, but basically it takes the Vector2 you call it on, and snaps the x and y to the closest multiple of the Vector2 you pass in the `step` parameter. 

You can totally add this `snapped()` movement logic to the `item.gd` code I wrote above and have the item snap to different grid cells as you drag it around, but I wanted something a little different. In my case, I wanted the item node the player is dragging around to match the mouse movements exactly. Underneath this node, I wanted to show a preview of where the item would be placed as well, and THAT node's position should be snapped to the grid.

So, I made a seperate script for an ItemPreview node

_item_preview.gd_
```gdscript
class_name ItemPreview 
extends Node2D

@export var snap: int = 128

var dragging: bool = false
var parent_item: Item

## position of the SubViewportContainer in main scene space.
var container_position: Vector2

func _ready() -> void:
	InventoryEvents.drag_started.connect(show_preview)
	InventoryEvents.drag_stopped.connect(hide_preview)
	container_position = get_viewport().get_parent().global_position

func _process(delta):
	if dragging:
		var grid_origin = %Inventory.global_position
		var pos = scene_to_viewport(parent_item.get_origin_offset())
		var local_pos = pos - grid_origin
		var snapped_local = Vector2(snapped(local_pos.x, snap), snapped(local_pos.y, snap))
		global_position = grid_origin + snapped_local + (parent_item.get_item_sprite_size() / 2)

```

As you may have noticed, this code is referencing a `SubViewportContainer`... and that is the second part of this secret sauce of mine. 

## SubViewport magic
In my previous iterations, I was finding it annoying when I made a grid, and I wanted to reposition it on the screen, I would have to do all this extra math to figure out where the Item is supposed to be dropped in global space. Then I remembered a [ fantasitc talk ](https://www.youtube.com/watch?v=cwZGq1qJYoQ) from Godotcon a few years back, done by Raffaele Picca, about how amazing SubViewports can be for so many use cases. 

What I ended up doing was putting my `Inventory` node inside of a `SubViewport`, and simply supplying the method to get the grid coordinate of the local scene inside the `SubViewport`, from a global position outside of the `SubViewport`.

_inventory.gd_
```gdscript
class_name Inventory
extends Node2D

@export var grid_size: Vector2 = Vector2(5,5)
@export var cell_size: Vector2 = Vector2(128,128)

var inventory_list: Array

func _draw() -> void:
	# draw the grid based on grid_size & cell_size
	for row in range(grid_size.y + 1):
		draw_line(Vector2(0, cell_size.y * row), Vector2(cell_size.x * grid_size.x, cell_size.x * row),Color.ALICE_BLUE, 2.0, false)
	for col in range(grid_size.x + 1):
		draw_line(Vector2(cell_size.x * col, 0), Vector2(cell_size.x * col, cell_size.y * grid_size.y), Color.ALICE_BLUE, 2.0, false)

## Returns true if a position is in bounds of the grid, and accounts for the area of the item at that position
func in_bounds(position: Vector2, area: Vector2) -> bool:
	var margin = cell_size / 2
	var grid_rect = Rect2(global_position - margin, grid_size * cell_size + margin * 2)
	var item_rect = Rect2(position, area)
	return grid_rect.encloses(item_rect)

## Get grid coord from global position
func global_to_grid(position: Vector2) -> Vector2i:
	return Vector2i((position - global_position) / cell_size)

func add_to_inventory(item) -> void:
	inventory_list.append(item)

func remove_from_inventory(item) -> void:
	inventory_list.erase(item)
```

The benfits of this are actually two fold! I now have no need to worry about making sure my `ItemPreview` node is only visible when the node is overlapping the grid. I simply set the visibility of the `ItemPreview` node to true when the player is dragging the item. Since I can make the SubViewport's size to be equal to the grid's size, the preview effect will never be visible when the node is not overlapping with the grid.

We also need to give some help to the `ItemPreview`, so it can figure out where it is.
_item_preview.gd_
```gdscript
...
## position of the SubViewportContainer in main scene space.
var container_position: Vector2

func _ready() -> void:
	InventoryEvents.drag_started.connect(show_preview)
	InventoryEvents.drag_stopped.connect(hide_preview)
	InventoryEvents.rotated.connect(rotate_preview)
	container_position = get_viewport().get_parent().global_position

...
## convert main scene position to SubViewport position.
func scene_to_viewport(pos: Vector2) -> Vector2:
	return pos - container_position

## convert SubViewport position to main scene position.
func viewport_to_scene(pos: Vector2) -> Vector2:
	return pos + container_position

```

So at this point my Scenetree looks like this.
```
> Node2D
	> SubViewportContainer
		> SubViewport
			> Inventory
			> InventoryPreview
	> Item
	> Item 2
	...
```


Okay! I glossed over a lot there, so... as for how the `ItemPreview` knows what to do, I'm using a global event bus to listen for the player dragging an `Item`, dropping and `Item`, and rotating and `Item`... yeah I didn't go over rotation either but that's not very interesting. I'll tell ya what is interesting...

# Where we droppin' though
Figuring out how to manage where you are allowed to drop an `Item`, and how to keep track of it was the real part that drove me insane in the early iterations. I was trying a data-first approach, storing the node refs in a multi-dimensional array that represented the grid. Then I was using that array to check if there were collisions with other nodes on the desired drop position for the `Item` and.... yeah don't do that. You know what you should do?

Just use `Area2D` and `CollisionShape` _facepalm_

Each item gets an `Area2D`, and tracks how many collsions are happening at a given time. Then you just check if the collision count is not zero. And yes, you'll want to use a count of collisions specifically though, not just a boolean that is flipped when the `area_entered` and `area_exited` signals are fired. Since you can have multiple collisions at one time, things get weird without a collision count.

So... we add these to
_item.gd_
```gdscript
...

@export var collision_shape: Shape2D:
	set(value):
		collision_shape = value
		var col_node = %CollisionShape2D
		if value:
			if value is ConcavePolygonShape2D:
				col_node.position = -item_sprite_size / 2
			col_node.position = Vector2.ZERO
			col_node.shape = value
		else:
			col_node.shape = null
var _overlapping_count: int = 0
var colliding: bool:
	get:
		return _overlapping_count > 0

...


func _on_area_2d_area_exited(area: Area2D) -> void:
	_overlapping_count -= 1

func _on_area_2d_area_entered(area: Area2D) -> void:
	_overlapping_count += 1

```

... and check it on _item_preview.gd_
```gdscript
... 

## shows a snapped copy of the item being dragged.
func show_preview(item: Node, texture: Texture2D) -> void:
	parent_item = item
	%Preview.texture = parent_item.item_sprite
	%Preview.rotation_degrees = parent_item.get_item_rotation()
	dragging = true
	visible = true

## hides the preview of the dragged item. used when dragging has stopped.
func hide_preview(position: Vector2) -> void:
	if in_grid_bounds():
		if !parent_item.colliding:
			parent_item.global_position = viewport_to_scene(global_position)
	parent_item = null
	dragging = false
	visible = false

```
To give the player a little bit of wiggle room to be less precise with their placement, you can also shrink the collision shapes a bit (origin at the center of the shape) to give it some margins.

And just like that, not tracking of node refs in any arrays, just pure collision logic. Then all you need to track is one array that has all items that were dropped in valid locations as signals are fired off.

Oh right, and to answer the age old of question of where we're droppin' (boys)... well we've already figured out the position of the snapped grid cell for the `ItemPreview`, so just set the item's global position to that of the `ItemPreviews`!
