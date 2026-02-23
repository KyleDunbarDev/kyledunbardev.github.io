---
layout: single
title:  "Box Colliders Are Better Than Capsules"
date:   2026-02-20 21:52:50 +0000
gallery:
  - url: /assets/images/colliders1.png
    image_path: /assets/images/colliders1.png
    alt: "A box collider and a capsule collider on a level platform, standing at the same height"
    title: "Both colliders stand at the same height"
  - url: /assets/images/colliders2.png
    image_path: /assets/images/colliders2.png
    alt: "A box collider and a capsule collider standing over the corners or a platform, at different heights."
    title: "On the corners, capsules have an unwanted height."
---
Capsule colliders offer smooth sliding against stairs and bumpy walls as the curved faces can roll along them - as opposed to a box collider which would snag on bumpy geometry. Capsules also give the ability for the character to roll up the corner of a ledge which adds a bit of tolerance to difficult platforming.

However they are not without their cons and through experience I posit that a box or cylinder collider, in 2D and 3D respectively, with additional systems is better for serious platforming.


# The Problem
Capsule colliders have curves on the feet and head. You can stand on the edge of a ledge and be lower than normal, upsetting any tight movement controls you might have calibrated. If your movement is physics based you will slip off the edge, and jumping into an edge will bounce you in an awkward direction.

{% include gallery caption="Unlike the box, the capsule's height differs when standing on a corner." %}

Additionally, with a cylinder the character will semi-smoothly glide up each stair, rather than abruptly stepping up which would give the feeling of climbing a staircase. Smoothly stepping may be desirable behaviour but should actually be achieved by modelling the collision for the stairs as a slope, as the capsule will still have some bounce.

# The Solution
A box or cylinder collider on the other hand has flat faces on the feet and head. This allows movement to be rigid and exact, but needs extra mechanics implemented to not feel frustrating. It removes the problem of variable heights when dealing with edges in exchange for requiring extra coded systems for **step handling**, **ledge snapping** (also called corner correction), and more complex **grounded checks** when dealing with slopes.

#### Step Handling
Step handling systems behave by detecting a step in front of the player, and if it is shorter than a variable step height limit, moving the player onto the step - often with a small interpolation so its not *too* jolting, or game-breakingly quick to ascend a flight of stairs. This allows the level designer to choose whether a staircase should feel like a staircase or like a ramp depending on it's collision model.

#### Ledge Snapping
Interlinked with step handling is ledge snapping. This allows a player to move or jump into a ledge's corner and snap on top of it, giving a variably small tolerance to landing on a ledge. It is similar to step handling but occurs in the air and zeroes the player's Y velocity.

This also introduces a micro optimisation technique into the game which may be desirable; by jumping precisely at a ledge you can snap to it and reset your Y velocity, reducing airtime which could lead to a small time save. This only really concerns speedrunning but could be desirable in a precision platformer or any platforming game that wants to make speedrunning more interesting.

#### Sloped Grounded Checks
When standing on a slope with a flat footed collider, the centre of the collider will not be touching the ground. If you were to cast a ray to the ground for a grounded check, it may be too far away and appear as though you should be falling. It may be tempting to address this with collision based grounded checks, but this doesn't work because as you traverse a slope, with the collider's sharp corners you will flicker between touching and not touching the slope, making it so that your jump could not trigger or the wrong animation plays.

Instead, have three raycasts. One for the left side, the centre and the right side. If any of them are close enough to the ground, grounded is true. You can have a more robust set of rays if needed, but these 3 should be sufficient in most cases. You may also want to include a max slope angle using the normal of the raycast collision that returns false on a grounded check if the slope is too steep. This turns slopes that are 2 steep into walls instead of ground. If you still experience flickering because of context-dependent variables, like scale and angles, you can add extra gravity when grounded to push the player into the ground.

{: .notice--info}
I recommend a cylinder collider for 3D so that the character can have the aforementioned benefits of flat feet and head, while being able to smoothly roll over bumpy walls. It also avoids an issue that a box collider has of non-uniform distance to walls at different angles.

# Conclusion
The additional systems may seem painful to implement when a capsule collider can cover the scenarios pretty well but the advantage overall of using flat faced colliders in this way is a much tighter control of movement technology and less random-feeling behaviour. For games where tight movement isn't an important element you can use a capsule collider just fine. But if you are serious about movement, use a box.

---

This post is a follow-up of the previous post you can check out here: [Techniques to Improve 2D platforming](https://www.kyledunbar.dev/2026/02/18/Techniques-to-Improve-2D-Platforming.html)

Thanks for reading! I will be posting once a week for the near future so come back soon or subscribe via RSS in the footer.
