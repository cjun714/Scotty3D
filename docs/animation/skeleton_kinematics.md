---
layout: default
title: "Skeleton Kinematics"
permalink: /animation/skeleton_kinematics
---

# Skeleton Kinematics

A `Skeleton`(defined in `scene/skeleton.h`) is what we use to drive our animation. You can think of them like the set of bones we have in our own bodies and joints that connect these bones. For convenience, we have merged the bones and joints into the `Joint` class which holds the orientation of the joint relative to its parent as euler angle in its `pose`, and `extent` representing the direction and length of the bone with respect to its parent `Joint`. Each `Mesh` has an associated `Skeleton` class which holds a rooted tree of `Joint`s, where each `Joint` can have an arbitrary number of children.

All of our joints are ball `Joint`s which have a set of 3 rotations around the [[task2_media/0027.png|height=9px]], [[task2_media/0028.png|height=12px]], and [[task2_media/0029.png|height=9px]] axes, called _Euler angles_. Whenever you deal with angles in this way, a fixed order of operations must be enforced, otherwise the same set of angles will not represent the same rotation. In order to get the full rotational transformation matrix, [[task2_media/0030.png|height=14px]], we can create individual rotation matrices around the [[task2_media/0031.png|height=14px]], [[task2_media/0032.png|height=14px]], and [[task2_media/0033.png|height=14px]] axes, which we call [[task2_media/0034.png|height=16px]], [[task2_media/0035.png|height=18px]], and [[task2_media/0036.png|height=16px]] respectively. The particular order of operations that we adopted for this assignment is that [[task2_media/0037.png|height=18px]].

### Forward Kinematics

_Note: These diagrams are in 2D for visual clarity, but we will work with a 3D kinematic skeleton._

When a joint's parent is rotated, that transformation should be propagated down to all of its children. In the diagram below, [[task2_media/0038.png|height=12px]] is the parent of [[task2_media/0039.png|height=12px]] and [[task2_media/0040.png|height=12px]] is the parent of [[task2_media/0041.png|height=12px]]. When a translation of [[task2_media/0042.png|height=12px]] and rotation of [[task2_media/0043.png|height=16px]] is applied to [[task2_media/0044.png|height=12px]], all of the descendants are affected by this transformation as well. Then, [[task2_media/0045.png|height=12px]] is rotated by [[task2_media/0046.png|height=16px]] which affects itself and [[task2_media/0047.png|height=12px]]. Finally, when rotation of [[task2_media/0048.png|height=16px]] is applied to [[task2_media/0049.png|height=12px]], it only affects itself because it has no children.

[[media/forward_kinematic_diagram.jpg]]

You need to implement these routines in `student/skeleton.cpp` for forward kinematics.

*   `Joint::joint_to_bind` 
    Rreturn a matrix transforming points in the space of this joint
    to points in mesh space in bind position  up to the base of this joint (end of its parent joint). You should traverse upwards from this joint's parent all the way up to the root joint and accumulate their transformations. 
*   `Joint::joint_to_posed`
    Return a matrix transforming points in the space of this joint to points in mesh space, taking into account joint poses. Again, you should traverse upwards from this joint's parent to the root joint.
*   `Skeleton::end_of`
    Returns the end position of the joint in world coordinate frame, and you should take into account the base position of the skeleton (`Skeleton::base_pos`).
*   `Skeleton::posed_end_of`
    Returns the end position of the joint in world coordinate frame with poses, and you should take into account `Skeleton::base_pos`.
*   `Skeleton::joint_to_bind` 
    Rreturn a matrix transforming points in the space of this joint
    to points in mesh space in bind position but with the base position of the skeleton taken in to account. Hint: use some function that you have implemented wisely!
*   `Skeleton::joint_to_posed`
    Return a matrix transforming points in the space of this joint to points in mesh space, taking into account joint poses but with the base position of the skeleton taken in to account. Hint: use some function that you have implemented wisely!

Once you have implemented these basic kinematics, you should be able to define skeletons, set their positions at a collection of keyframes, and watch the skeleton smoothly interpolate the motion (see the [user guide](../guide/animate.md) for an explanation of the interface). The gif below shows a very hasty demo defining a few joints and interpolating their motion.

![gif1](task2_media/gif1.gif)

![gif1](task2_media/gif2.gif)

Note that the skeleton does not yet influence the geometry of the cube in this scene -- that will come in Task 3!

<!---
### Task 2b - Inverse Kinematics

#### Single Target IK

Now that we have a logical way to move joints around, we can implement Inverse Kinematics, which will move the joints around in order to reach a target point. There are a few different ways we can do this, but for this assignment we'll implement an iterative method called gradient descent in order to find the minimum of a function. For a function [[task2_media/0050.png|height=16px]], we'll have the update scheme:

[[task2_media/0051.png|height=18px]]

Where [[task2_media/0052.png|height=9px]] is a small timestep. For this task, we'll be using gradient descent to find the minimum of the cost function:

[[task2_media/0053.png|height=36px]]

Where [[task2_media/0054.png|height=19px]] is the position in world space of the target joint, and [[task2_media/0055.png|height=12px]] is the position in world space of the target point.

More specifically, we'll be using a technique called Jacobian Transpose, which relies on the assumption that:

[[task2_media/0056.png|height=21px]]

Where:

*   [[task2_media/0057.png|height=14px]] (n x 1) is the function [[task2_media/0058.png|height=19px]], where [[task2_media/0059.png|height=19px]] is the angle of joint [[task2_media/0060.png|height=14px]] around the axis of rotation
*   [[task2_media/0061.png|height=9px]] is a constant
*   [[task2_media/0062.png|height=16px]] (3 x n) is the Jacobian of [[task2_media/0063.png|height=14px]]

Note that here [[task2_media/0064.png|height=9px]] refers to the number of joints in the skeleton. Although in reality this can be reduced to just the number of joints between the target joint and the root, inclusive, because all joints not on that path should stay where they are, so their columns in [[task2_media/0065.png|height=16px]] will be 0\. So [[task2_media/0066.png|height=9px]] can just be the number of joints between the target and the root, inclusive. Additionally note that since this will get multiplied by [[task2_media/0067.png|height=9px]] anyways, you can ignore the value of [[task2_media/0068.png|height=9px]], and just consider the timestep as [[task2_media/0069.png|height=18px]].

Now we just need a way to calcluate the Jacobian of [[task2_media/0070.png|height=14px]]. For this, we can use the fact that:

[[task2_media/0071.png|height=19px]]

Where:

*   [[task2_media/0072.png|height=16px]] is the [[task2_media/0073.png|height=16px]] column of [[task2_media/0074.png|height=19px]]
*   [[task2_media/0075.png|height=14px]] is the axis of rotation
*   [[task2_media/0076.png|height=16px]] is the vector from the base of joint [[task2_media/0077.png|height=14px]] to the end point of the target joint

For a more in-depth derivation of Jacobian transpose (and a look into other inverse kinematics algorithms), please check out [this presentation](https://web.archive.org/web/20190501035728/https://autorob.org/lectures/autorob_11_ik_jacobian.pdf). (Pages 45-56 in particular)

Now, all of this will work for updating the angle along a single axis, but we have 3 axes to deal with. Luckily, extending it to 3 dimensions isn't very difficult, we just need to update the angle along each axis independently.

#### Multi-Target

We'll extend this so we can have multiple targets, which will then use the function to minimize:

[[task2_media/0078.png|height=40px]]

which is a simple extension actually. Since each term is independent and added together, we can get the gradient of this new cost function just by summing the gradients of each of the constituent cost functions!

You should implement multi-target IK, which will take a `std::map` of `Joint`s and target points for that joint. Each joint can only have 1 target point.

In order to implement this, you should update `Joint::calculateAngleGradient` and `Skeleton::reachForTarget`. `Joint::calculateAngleGradient` should calculate the gradient of [[task2_media/0079.png|height=14px]] in the x,y, and z directions, and add them to `Joint::ikAngleGradient` for all relevant joints. `Skeleton::reachForTarget` should actually do the gradient descent calculations and update the angles of each joint, saving them with `Joint::setAngle`. In this function, you should probably use a very small timestep, but do several iterations (say, 10s to 100s) of gradient descent in order to speed things up. For even faster and better results, you can also implement a variable timestep instead of just using a fixed one. Note also that the root joint should never be updated.

A key thing for this part is to _remember what coordinate frame you're in_, because if you calculate the gradients in the wrong coordinate frame or use the axis of rotation in the wrong coordinate frame your answers will come out very wrong!

#### Using your IK!
Once you have IK implemented, you should be able to create a series of joints, and get a particular joint to move to the desired final position you have selected.

[[media/ik.gif]]
-->