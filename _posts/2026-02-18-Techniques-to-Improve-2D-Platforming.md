---
layout: single
title:  "Techniques to Improve 2D Platforming"
date:   2026-02-18 02:00:50 +0000
---
You can greatly improve the game feel of 2D platforming with some simple key mechanics. If the majority of your gameplay is platforming, it better feel good.

In this post I will share the techniques I include in my game to subtly improve the game feel of platforming. These techniques are what I consider the minimum for a platforming game where platforming is perhaps secondary to adventure or puzzling. In the end, I will briefly introduce some extra techniques you can use for more platforming-centric games.

The following implementations are extensions of the [Godot CharacterController2D Basic Movement Template](https://github.com/godotengine/godot/blob/master/modules/gdscript/editor/script_templates/CharacterBody2D/basic_movement.gd).
# Jump Input Buffering
Jump input buffering allows the player to press their jump action slightly before landing and the jump will still execute. This avoids the frustration of a jump the player thought should work not executing or having to perfectly press jump right when you are grounded in scenarios where you want to jump again as soon as you land, which is extremely common for efficiently traversing platforming scenes.

It works by changing the jump input from the jump action to a new jump buffering actions. The jump buffering action sets a boolean to true and back to false after it times out. During the time it is true, if the player touches the floor the jump action is executed.

Here is my Godot implementation:
```gdscript
@onready var jump_buffer_timer: Timer = %JumpBufferTimer
var is_jump_buffered: bool = false

func _physics_process(delta: float) -> void:
	# Add the gravity.
	if not is_on_floor():
		velocity += get_gravity() * delta

	# Buffer jump.
	if Input.is_action_just_pressed("jump"):
		is_jump_buffered = true
		jump_buffer_timer.start()

	# Handle jump.
	if is_jump_buffered &&  is_on_floor():
		is_jump_buffered = false
		velocity.y = JUMP_VELOCITY

	move_and_slide()

func _on_jump_buffer_timer_timeout() -> void:
	is_jump_buffered = false
```
# Coyote Time
Coyote time, named after Wile E. Coyote, allows the player to still jump for a short time after running off of a ledge. This helps a player make a long jump because they will want to time it to the last possible moment to press jump, which often would result in them unceremoniously falling off of the edge. Coyote time instead extends a grace period during this fall where if the jump action is pressed within it, the jump still goes ahead.

By keeping a boolean value from the last frame of whether you were grounded, and comparing it to the current frame, you can determine if the player just left the ground. If you just left the ground, and you didn't jump, it starts the coyote timer which sets a coyote boolean to true. Now in the jump action, you check if the player is on the floor *or* if coyote is true.

In Godot, I implemented it as such:
```gdscript
@onready var coyote_timer: Timer = %CoyoteTimer
var is_coyote: bool = false
var was_grounded: bool = false

func _physics_process(delta: float) -> void:
	...
	# Handle jump.
	if is_jump_buffered && (is_on_floor() or is_coyote):
		is_coyote = false
		is_jump_buffered = false
		velocity.y = JUMP_VELOCITY

	move_and_slide()

	# Handle coyote timer
	if was_grounded && !is_on_floor() && !Input.is_action_just_pressed("jump"):
		is_coyote = true
		coyote_timer.start()

	if is_on_floor():
		is_coyote = false

	was_grounded = is_on_floor()

```

{: .notice--info}
Note: is_on_floor() is only updated after move_and_slide(), so we need to do the coyote check after move_and_slide() to have an up to date is_on_floor() and before updating was_grounded to know if the character left the floor in the previous frame.
# Variable Jump Height
Variable jump height allows for the player to control jump height based on how long they hold the jump button. A short tap results in a small hop, and a full hold results in a max height leap. This is desirable because it gives players control and expression over how they move throughout a scene and let's them optimise away air time.

Most implementations you will see of this reduce gravity while jump is held. This adds height to the standard jump. I prefer to treat it as a reductive process, and add gravity when jump is released. This allows the developer to implement this without having to reconfigure their standard jump height.

I implement it in Godot like this:
```gdscript
func _physics_process(delta: float) -> void:
	...
	# Halve velocity when jump is released while ascending
	if Input.is_action_just_released("jump") && velocity.y < 0:
		velocity.y = velocity.y / 2
	...
```

{: .notice--info}
Instead of adding gravity I simply halve the vertical velocity which creates a smooth apex in one operation.
# Other Techniques
The techniques above can be incorporated into almost any platformer and feel right. The following techniques are a bit more specialised and may not be suitable for your game. I will introduce the concept and explain why it could be good, but I won't provide details as to how they work.

**Apex hang time** - By reducing gravity at the apex of a jump you can create a short hang time which creates a floaty feeling that feels heroic.

**Horizontal acceleration** - Instead of instantly being at max speed when moving, horizontal velocity ramps up and down following acceleration and deceleration curves. It is recommended to decelerate faster than the acceleration. This gives the movement a feeling of weight while letting it still feel responsive.

**Reduced air control** - By allowing the player to influence their horizontal velocity less in the air, you make the player be more deliberate about their jumps because they cannot make corrections as easily. This can be taken the other way and allow for more air control to achieve an opposing effect.

**Turn acceleration** - Detect when the player changes direction and increase acceleration. This creates a snappier horizontal movement.

**Ledge snapping** - Snap the character's position to a ledge if they would barely miss. This softens the difficulty a bit and rewards intent rather than execution. This ties into player-centred design which favours the player's intent over the game's physics.
# Juice
Mechanics is only half the puzzle of game feel. To make a mechanic really feel good, you need 'Juice' - or visual and audio feedback.

The principles of animation such as squash and stretch and anticipation will make all of the movements feel more real and impressive by adding a perception of weight and intention to the character.

Visual effects such as dust particles when running, landing or turning quickly can help make motion feel real, readable, and identifiable.

Good sound design goes a long way into making a movement system feel good. A soft jump sound and strong landing sound can help to give a sense of impact and subtly reinforce timing to the player. A subtle footstep loop quietly makes the movements feel real and immersive.

A heavy landing could reverb and cause dust to fall in a cave scene or create a large water spray in a pond. How you juice the mechanics is largely dependent on what you are trying to achieve with your game design.

---
That's all! Thank you for reading. Follow me on the platforms in the footer so I look cool online and subscribe via RSS if you want to catch the next one.
