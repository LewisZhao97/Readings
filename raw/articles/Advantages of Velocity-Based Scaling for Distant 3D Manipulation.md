---
title: Advantages of Velocity-Based Scaling for Distant 3D Manipulation
author:
  - Curtis Wilkes
  - Doug A. Bowman
source: https://dl.acm.org/doi/10.1145/1450579.1450585
created: 2026-05-08
tags:
  - xr
  - vr
  - 3d-interaction
  - object-manipulation
  - distant-manipulation
  - homer
  - scaled-homer
  - velocity-based-scaling
  - control-display-gain
  - prism
  - vrst-2008
status: true
ingested: 2026-05-08
---
# Abstract

Immersive virtual environments (VEs) have the potential to offer rich three-dimensional interaction to users. In many instances, however, 3D interaction tasks are difficult due to both the imprecision of tracking devices and the inability of users to achieve and maintain precise hand positions in 3D space. One way to improve upon existing interaction techniques is to dynamically change the sensitivity of the interaction technique based on user input. Previous research has applied this principle to virtual hand-based manipulation techniques; when the user slows down the movement of her physical hand, the virtual hand slows down even more to allow precise manipulation. In this study we extend the prior research by applying the velocity-based scaling principle to HOMER, an existing at-a-distance manipulation technique based on ray-casting. The scaled HOMER technique offers the user the freedom to accomplish both long- and shortdistance manipulation tasks with higher levels of precision without compromising speed. We present results from a user study that shows that the addition of scaling to HOMER significantly improves user performance on 3D manipulation tasks.

Categories and Subject Descriptors

I.3.7 [Three-Dimensional Graphics and Realism]: Virtual Reality

# Keywords

Usability, 3D Interaction, User Studies

# 1. Introduction

Manipulation is one of the most common user interaction tasks in immersive virtual environments (VEs) (Bowman et al. 2004). For example, in an interior design application (Lindsey and McLain-Kark 1998), users need to position and orient virtual objects (such as furniture) in a three-dimensional model of a room. For highlyinteractive VE applications, the usability of the manipulation interface greatly affects the overall usability of the system. Thus, the design of 3D manipulation techniques is critical for such applications.

Many applications have additional requirements with respect to 3D manipulation. First, high levels of precision may be required.

Permission to make digital or hard copies of all or part of this work for personal or classroom use is granted without fee provided that copies are not made or distributed for profit or commercial advantage, and that copies bear this notice and the full citation on the first page. To copy otherwise, to republish, to post on servers or to redistribute to lists, requires prior specific permission and/or a fee. VRST’08, October 27-29, 2008, Bordeaux, France Copyright 2008 ACM ISBN 978-1-59593-951-7/08/10 ...$5.00

For example, users of molecular docking applications (Brooks et al. 1990) need to precisely place molecules relative to one another. Second, it is often useful to allow users to manipulate objects at a distance, rather than requiring them to move within arm’s reach. In a structural design application (Bowman et al. 2003), for example, the user may wish to see the entire structure at a glance while manipulating its components.

Although there has been a great deal of research on the design of 3D manipulation techniques, their performance in real-world applications often falls below expectations. Most manipulation techniques depend on accurate hand tracking. While tracking hardware has improved, many systems still exhibit significant jitter and latency. Even in the presence of perfect tracking, however, precise manipulation would still be problematic, due to human limitations. It is inherently difficult for humans to position their hands precisely in 3D space, and to maintain that position, due to hand jitter. When manipulation is done at a distance, any error may be magnified because distant manipulation techniques often scale up the input.

One way to improve the precision of 3D manipulation is to scale down, or dampen, the movements of the user’s hand. In the PRISM technique (Frees et al. 2007) , the motion of a selected object is scaled down when the user’s hand velocity drops below a certain threshold. This work established that scaling of this sort is possible and does improve performance. However, this work was limited to virtual-hand-based techniques that operated only in the user’s local area.

In our research, we seek to apply the same principle to the HOMER technique (Bowman et al. 1999). HOMER combines ray-casting selection and hand-centered manipulation to allow intuitive manipulation at a distance. For this reason, HOMER is more practical and expressive than simple virtual hand techniques for most VE applications. However, it is not trivial to apply scaling to object motion in distant techniques like HOMER.

In this paper, we present the design of the scaled HOMER technique, which retains the feel and expressiveness of HOMER, while allowing precise manipulation at both near and far distances. We describe an experiment that shows the performance benefits of scaled HOMER in a range of manipulation task scenarios.

# 2. Related Work

Researchers have designed a wide variety of 3D manipulation techniques. Simple virtual hand techniques (Bowman et al. 1999), where the user directly selects objects with the hand, although natural and intuitive, limit users to manipulating objects that are within arm’s reach.

To address this limitation, arm-extension techniques such as Go-Go (Poupyrev et al. 1996) allow the user to manipulate objects that are located at a distance by applying a non-linear scaling factor to the position of the virtual hand. Although this gives the user great freedom to manipulate objects within the world, the

scaling up of user hand motion can reduce precision of manipulation.

Another method to allow at-a-distance manipulation is ray-casting (Bowman and Hodges 1997), in which the user does not manipulate the object directly with the virtual hand, but instead uses a ray extending into the environment for selection and manipulation. Ray-casting techniques not only suffer from a lack of precision, but also do not allow users to specify all possible positions and orientations.

A third category of distant manipulation techniques scales down the world (or objects in the world) to allow users to reach and manipulate objects directly using the hand. Techniques such as World in Miniature (Stoakley et al. 1995), Voodoo Dolls (Pierce et al. 1999), and Scaled World Grab (Mine et al. 1997) all provide expressive distant manipulation, but still suffer from precision issues due to scaling.

Scaling can also be used to increase precision. For example, desktop GUIs use scaling of mouse motion (Mackenzie and Riddersma 1994) to allow rapid movements across the entire desktop and controlled movements in small areas, based on the velocity of the device’s movement. For 3D manipulation, the PRISM technique (Frees et al. 2007) proposed to use scaling in a similar way to improve the usability of simple virtual hand techniques. In PRISM, as long as the user’s hand is moving with a velocity above a certain threshold, the virtual hand follows the physical hand exactly. When the physical hand velocity drops below this threshold, however, the velocity of the virtual hand is scaled down. Thus, users can achieve higher precision by making slow, controlled movements. This technique, however, does not allow at-a-distance manipulation and limits the interaction to the near environment. We seek to attain the same benefits as PRISM with a distant manipulation technique, and to combine scaling down for precision with scaling up for expressiveness in the same technique.

Finally, some research has attempted to improve the precision of 3D manipulation by adding constraints, such as snap-to-grid or snap-to-object (Bier 1990). Such techniques improve performance, but are not applicable to all situations. Our technique allows objects to be placed in any 3D position and orientation.

# 3. Design of the Scaled HOMER Technique

The Scaled HOMER interaction technique seeks to improve upon the original HOMER by applying scaling. There are two main motivations for this work. First, we wish to extend the previous work on scaling to distant manipulation techniques. Second, although the original HOMER technique has been found highly usable in many situations, it has some significant limitations that can be addressed with the addition of scaling.

# 3.1 Original HOMER technique

HOMER is a hybrid 3D manipulation technique that combines the most effective components for the selection and manipulation sub-tasks. It uses ray-casting (Bowman and Hodges 1997) for selection and a virtual hand for manipulation. Empirical studies have demonstrated the efficiency and accuracy of the HOMER technique in a wide range of task scenarios (Bowman et al. 1999).

At the time of selection, the technique calculates the ratio of two distances: the initial distance between the user’s torso and hand $( \mathrm { D } _ { \mathrm { h a n d } } )$ , and the initial distance between the user’s torso and the

selected object $\left( \mathrm { D _ { o b j e c t } } \right)$ . This ratio is used to calculate the distance of the virtual hand from the user’s body $\left( \mathrm { D } _ { \mathrm { v i r t h a n d } } \right)$ . Specifically, the current distance between the user’s torso and hand $\mathrm { ( D _ { c u r r h a n d } ) }$ is multiplied by the ratio calculated at the time of selection to obtain the new depth of the virtual hand (Equation 1 – Note that throughout this paper we use D to represent distance, P to represent a 3D position, V to represent a 3D vector, and S to represent a scaled factor).

In addition, HOMER assumes that the object lies on a ray defined by the user’s torso and hand. But this is not strictly true if the user’s hand is not pointing along this ray at the time of selection. Thus, an offset vector is also computed by calculating the object’s expected position along this ray and subtracting this position from the object’s actual initial position (Equation 2). During manipulation, then, the object’s position is calculated by first moving $\mathrm { D } _ { \mathrm { v i r t h a n d } }$ units along the body-hand ray, and then adding the offset vector (Equation 3).

The user can rotate the selected object directly by rotating his hand; rotations are calculated relative to the initial orientation of the object.

$$
D _ {\text {v i r t h a n d}} = D _ {\text {c u r r h a n d}} * \left(\frac {D _ {\text {o b j e c t}}}{D _ {\text {h a n d}}}\right)
$$

Equation 1. Calculation of object distance in HOMER.

$$
V _ {\text {o f f s e t}} = P _ {\text {o b j e c t}} - D _ {\text {o b j e c t}} * \frac {\left(P _ {\text {h a n d}} - P _ {\text {t o r s o}}\right)}{D _ {\text {h a n d}}}
$$

Equation 2. Calculation of offset vector in HOMER.

$$
P _ {\text {o b j e c t}} = D _ {\text {v i r t h a n d}} * \frac {\left(P _ {\text {h a n d}} - P _ {\text {t o r s o}}\right)}{D _ {\text {c u r r h a n d}}} + V _ {\text {o f f s e t}}
$$

Equation 3. Calculation of object position in HOMER.

One feature of this design is that a user can bring even distant objects very near simply by moving her hand very near to her body. In addition, users can rotate objects to any orientation without moving the objects from their original positions. Finally, large-scale movements can be achieved simply by turning around, since the object’s position follows a ray between the user’s torso and hand.

However, HOMER also has several limitations. First, a user’s reach is limited by the length of his arm (even though this length is scaled), so it is difficult for users to move objects farther away. This requires users to think about how they should position their hand before selection to facilitate moving the object. Second, HOMER does not support precise positioning at a distance, since users’ hand movements are scaled, and since any jitter in the system will also be scaled. Thus, although HOMER is very expressive (allows objects to be placed in almost any position), it lacks fine-grained control of objects at a distance.

# 3.2 Velocity-Based Scaling Function

Scaled HOMER seeks to provide the expressiveness of the original HOMER technique with the added benefit of fine-grained

control at a distance. We also want Scaled HOMER to have the same “feel” as HOMER so that its use is transparent to the user.

Scaled HOMER works exactly like the HOMER technique described above, but adds velocity-based scaling to provide more precise object control. Once the object is selected, Scaled HOMER scales the translation of the hand (in all directions) by dividing the velocity of the hand by a scaling constant to obtain a scaling factor, which is then multiplied by the actual velocity to obtain the scaled velocity. Thus, when the hand is moving quickly, the scaled velocity will be equal to or greater than the actual velocity; when the hand is moving slowly, the scaled velocity will be less than the actual velocity, providing finegrained control of object position.

Figure 1 illustrates the way scaled velocity is calculated in the scaled HOMER technique. Each frame, the technique calculates the distance moved by the user’s hand $\mathrm { ( D _ { h a n d } ) }$ . This distance is scaled to obtain a scaled distance $( \mathrm { S D } _ { \mathrm { h a n d } } )$ . Note that $\mathrm { { S D } _ { \mathrm { { h a n d } } } }$ is equal to the scaled velocity of the virtual hand in terms of distance per frame. The scaling factor depends on the actual velocity of the hand $\mathrm { ( V _ { h a n d } ) }$ . This velocity is divided by a constant (SC) to calculate the scaling factor, so that the scaling factor is 1.0 when $\mathrm { \Delta V _ { h a n d } }$ equals SC. We allowed the scaling factor to go above 1:1 to help mitigate the limitation of HOMER’s reach being limited to the length of the user’s arm. We capped this factor at 1.2, since pilot tests showed that scaling up more than 1.2:1 was difficult for users to handle. In addition, we define a minimum velocity below which we set $\mathrm { S D } _ { \mathrm { h a n d } }$ to zero, causing the object to remain still.

![](images/f131a7dac6b70b5e3901238c6e6869e2bacca039d757d5bf14327834e84624ba.jpg)

![](7613bc9fa3affa44f40b799fb931b556184bd3abe9b6c2d8b8d9f3a0f6fa2c66.jpg)  
Figure 1. Calculation of scaled velocity in the scaled HOMER technique.(Bottom figure adapted from (Frees et al. 2007)).

# 3.3 Combining the two functions

A challenge in implementing scaled HOMER stems from the fact that HOMER already scales up the motion of the object. How do we allow our scaling and HOMER’s scaling to intuitively work in conjunction with one another, and in what order do we apply the scaling functions? Do we apply scaling to the hand’s velocity before or after using HOMER to find the location of the virtual hand?

Initially, we created a technique that simply combined the HOMER scaling function and the velocity scaling function into a single calculation. When the user moved his physical hand, we

first took the vector between the beginning and end points of the motion. Then we scaled that vector based on the product of two scaling factors – one calculated by our scaling function (Figure 1) and one calculated by HOMER (Equation 1) – and added the scaled vector to the object’s position. This proved to be very unusable because the user did not feel like they had direct control over the object and the technique did not feel anything like the original HOMER. In particular, this technique did not maintain the object’s position close to the ray between the user’s torso and hand, as the original HOMER technique does.

In our second attempt we combined the two functions by first applying our scaling function (Equation 4) to the physical hand to obtain a scaled position (Equation 5). We then used the HOMER technique directly to move the object according to this scaled position (Equation 6). This approach allows us to generalize the use of velocity-based scaling in conjunction with any manipulation technique, because the scaling function is applied separately, resulting in a hand position that can be used as input for the technique.

$$
S D _ {h a n d} = \min  \left(\frac {\text {V e l o c i t y} _ {h a n d}}{S C}, 1. 2\right) * D _ {h a n d}
$$

Equation 4. Apply the scaling function to the hand

$$
S P _ {h a n d} = S D _ {h a n d} * \left(\frac {V _ {h a n d \_ m o v e}}{\| V _ {h a n d \_ m o v e} \|}\right) + P _ {h a n d}
$$

Equation 5. Scaled position of the hand

$$
D _ {\text {v i r t h a n d}} = S P _ {\text {h a n d}} * \left(\frac {D _ {\text {o b j e c t}}}{D _ {\text {h a n d}}}\right)
$$

Equation 6 Combined position function

# 3.4 Motion of Objects with Scaled HOMER

One of the goals of Scaled HOMER was to give the user an experience very similar to the original HOMER, with the added benefit of being able to accomplish tasks that require high levels of precision. When we demonstrated the Scaled HOMER technique, users familiar with the original HOMER technique stated that they could not immediately tell the difference between the two techniques. It was not until they tried to perform a finegrained motion that they were able to feel the difference. This is the effect we desired, because we do not want the scaling to hinder coarse movements.

Figure 2 demonstrates the difference in object motion between the two techniques. In HOMER (left part of the figure) the object remains on the ray between the user’s torso and hand. In Scaled HOMER (right), this is also true when the user moves at speed SC (as illustrated in the figure between points a and b). However, when the user moves her hand at a lower speed (e.g., SC/2, as illustrated in the figure between points b and c), the object will only move only a fraction of the distance that it would in HOMER, which gives the user more precision. Although this means that the object is no longer on the torso-hand ray, this is not usually noticeable, since it only occurs at low velocities, and since an offset may already exist due to the implementation of HOMER (Equation 2 and Equation 3).

![](d829ef078f31e230f7dc83df2fdc352f75cb50471c208e9a6edd898ec656765b.jpg)  
Figure 2. Left: Object motion in HOMER. Right: Object motion in scaled HOMER.

# 4. Experiment

We performed an experiment in order to evaluate the effectiveness of applying velocity-based scaling to HOMER for positioning tasks.

# 4.1 Goals

The object of this research was to improve upon HOMER by applying velocity-based scaling to the technique. To measure the effectiveness of Scaled HOMER, we used the original HOMER technique as a baseline and compared user performance with the two techniques. We conducted a formal experiment, and measured task completion time for users to perform a variety of manipulation tasks, some of which required high levels of precision. We hypothesized that Scaled HOMER would improve upon the original HOMER by giving the user more control during precise interaction without limiting coarse movements.

# 4.2 Design

We conducted a within-subjects experiment across five independent variables: technique, size of target, distance of user to target, distance users had to move object, and direction of movement. There were two techniques: HOMER and Scaled HOMER. There were two different sizes of targets: small (.381 m x .4572 m) and medium $( 1 . 0 6 6 8 \mathrm { ~ m ~ x ~ } 1 . 5 2 4 \mathrm { ~ m ~ } )$ . We had two distances: near $( 1 5 . 2 4 \mathrm { ~ m } )$ and far $( 6 0 . 9 6 \mathrm { ~ m } )$ . There were three directions of movement: to the left, towards the user, and away from the user. Lastly, there were two distances users had to move the objects: short $( 7 . 6 2 \mathrm { ~ m } )$ and long $( 3 8 . 1 \mathrm { ~ m } )$ . To simplify the experiment, all movements were constrained to a 2D horizontal ground plane; vertical movements were not considered. Thus, for each interaction technique, there were 24 conditions (2x2x3x2).

The dependent variable was task completion time. Task completion time was measured from the time the user initially selected the object until the object was successfully dropped into the specified target.

# 4.3 Apparatus

We implemented the HOMER and scaled HOMER techniques as described in section 3, with the exception that vertical movements of the input device were ignored. We used DIVERSE (Kelso et al. 2002) for our software implementation, which makes use of the

SGI Performer library for graphics rendering. The software was written in $\mathrm { C } { + + }$ .

The experiment took place in a four-screen CAVE display (Cruz-Neira et al. 1993). The CAVE had a front wall, two side walls, and a floor screen, and was $1 0 \mathrm { x } 1 0 \mathrm { x } 9$ feet. The front and side walls were rear-projected, while the floor was front-projected. All screens displayed active stereoscopic imagery, which users viewed through Liquid Crystal shutter glasses.

The user’s head and dominant hand were tracked using an Intersense IS-900 position tracker, which provides six degree-offreedom data. The head tracking data was used to render the scene from the user’s point of view. The hand tracker was a “wand” that provides four buttons and a two degree-of-freedom analog joystick.

One of the wand buttons was used to select and manipulate objects in the scene; users could press the button to select an object via ray-casting, hold the button while manipulating the object, and release the button to drop the object at its current location. A second wand button was used to advance from one trial to the next.

# 4.4 Procedure

Participants completed three trials in each of the 24 conditions for both HOMER and Scaled HOMER, for a total of 144 trials. To avoid confusing the participant we separated the experiment into two blocks, one for each technique. The order of presentation of the techniques was counterbalanced to offset any possible learning effect.

Each trial involved the participant selecting an object and releasing the object into the target. The object was a 3D model of a cactus, and the target was a square on the ground (see Figure 3). The participants were required to select the object and move it along the ground until the object was positioned within the boundaries of the target. The system would then highlight the object to inform the participant that the object was in the correct position, and the participant was instructed to release the object while it was in the correct position.

Participants first gave their informed consent and took a general survey asking for basic demographic information. Then participants were brought into the CAVE where a practice environment was loaded to introduce the technique. In the practice environment there were many objects and a target. We trained participants in the use of the first technique in this practice environment. Participants were then given the freedom to practice selecting and dropping objects until they stated that they were comfortable with the technique and felt that they understood how the technique worked.

After the practice session the first block of trials began. To begin a trial the participant would push a button to make a new object and a new target appear. The participant would then select the object and attempt to place the object within the target as quickly as possible. The timer would start when the participant initially selected the object and run until the object was successfully dropped into the target, or until a 30-second time limit expired. If the 30-second time limit expired, the trial was included as a successful attempt the took 30-seconds to complete. Once the trials were completed for the first technique the participants were given an opportunity for a rest break before repeating the same procedure for the other technique. 14 people participated in the experiment; 1 was female. Participants were recruited from undergraduate classes in computer science.

![](3ee6ce4f32613c6a92c2e41a6ba4971123e312234f95b334de8a74a3d89064b7.jpg)  
Figure 3. Example object and target location

# 5. Results

We measured time to completion of each trial as the dependent variable. To create a normal distribution, we normalized the times by taking the log of time. We ran a five-factor analysis of variance (ANOVA) on the lognormal of times. We found significant main effects of all of the independent variables (Table 1). In particular, the performance of Scaled HOMER (mean $= 5 . 1 7$ seconds) was better than that of HOMER (mean $= 9 . 9 3$ seconds), which was significant $\mathrm { ( p < 0 . 0 0 1 }$ ).

Table 1. Main effects found in the experiment.   

<table><tr><td>Independent variable</td><td>F-statistic</td><td>Significance level</td></tr><tr><td>Technique</td><td>F(1,531) = 339.32</td><td>p &lt; 0.0001</td></tr><tr><td>Target size</td><td>F(1,531) = 674.30</td><td>p &lt; 0.0001</td></tr><tr><td>Target distance</td><td>F(1,531) = 777.10</td><td>p &lt; 0.0001</td></tr><tr><td>Movement direction</td><td>F(2,531) = 10.81</td><td>p &lt; 0.0001</td></tr><tr><td>Movement distance</td><td>F(1,531) = 198.15</td><td>p &lt; 0.0001</td></tr></table>

The ANOVA also revealed multiple significant interactions between variables; we only report here the interactions involving the technique variable, since the experiment was designed to test the differences between the two techniques in various task conditions. There were significant interactions between technique and target distance $( \mathrm { F } ( 1 , 5 3 1 ) = 6 8 . 5 8 $ , $\mathsf { p } < 0 . 0 0 0 1$ , see Figure 4), and between technique and target size $\left( \mathrm { F } ( 1 , 5 3 1 ) = 3 4 . 2 6 \right)$ , $\mathrm { ~ p ~ } <$ 0.0001, see Figure 5). There was a three-way interaction between technique, target distance, and movement distance $( \mathrm { F } ( 2 , 5 3 1 ) =$ 8.94, $\mathsf { p } < 0 . 0 0 0 2$ , see Figure 6). Note that error bars are too small to be seen.

![](images/9fcae429c680f983389b824a78bab606d664e158161c8009fad51e88212a38bd.jpg)  
Figure 4. Interaction between target distance and technique

![](images/76930364f6e50ae87cb201448b5a9de74c34c2055a3c6340ff08c422af093382.jpg)  
Figure 5. Interaction between target size and technique

![](images/b2075e89d145cc1b337975d0b9d176d1a9490f5e1fd5e251d3ca3c6703f89fe8.jpg)

![](a7f8018298df2efca9f4d18ec98be1afd4c34e9d3675f320417e8d3b6834eacc.jpg)  
Figure 6. Three-way interaction between target distance, movement distance, and technique

# 5.1 Discussion

Overall, scaled HOMER offers a significant improvement over the basic HOMER technique. In this experiment the mean task time for HOMER was almost double the task time for Scaled HOMER, indicating that velocity-based scaling has a benefit in a wide range of task conditions.

As the target is positioned farther from the user, the benefits of Scaled HOMER are more pronounced, as demonstrated in the interaction between technique and target distance (Figure 4). This is probably due to the fact that the HOMER technique scales up hand movements for distant manipulation, thereby magnifying any jitter or overshoot caused by the user or the tracking system. Scaled HOMER reduces the effects of jitter or overshoot when the user moves his hand slowly, thus easing the manipulation task.

The interaction between technique and target size (Figure 5) suggests that the benefits of Scaled HOMER increase when targets are very small. This further supports our hypothesis that Scaled HOMER improves performance on tasks requiring a high level of precision, without decreasing performance for other tasks.

The three-way interaction between technique, target distance, and movement distance (Figure 6) suggests that although scaled HOMER is clearly beneficial when targets are far away, it also provides significant benefits when targets are nearby but objects must be moved a long distance. In situations where the object is initially far away, the scale factor in HOMER will be large, making it difficult to place the object even in a nearby target. With scaled HOMER, again, users can achieve high precision in such tasks simply by slowing their hand movements when the object approaches the target.

# 6. Conclusions and Future Work

In this research, we have shown that it is feasible to use the concept of velocity-based scaling with an existing at-a-distance 3D manipulation technique in an immersive VE. We combined the scaled manipulation found in the PRISM technique (Frees et al. 2007) with a best-practice manipulation technique HOMER (Pierce and Pausch 2002). In a formal study, we demonstrated that scaled HOMER improves performance over the basic HOMER technique in a wide variety of task conditions, and especially in those tasks that require a high level of precision for object placement, object placement at a distance, or a large movement distance, without compromising performance on easier manipulation tasks. The scaled HOMER technique allows the user to enjoy the benefits of HOMER, such as at-a-distance manipulation, while also allowing them to achieve greater levels of precision. Scaled HOMER also mitigates the known limitations of HOMER.

One of our major design goals was to combine PRISM and HOMER transparently to the user. Under most conditions in our experiment, the participant could not tell the scaling was being applied until precise control was required. This suggests that velocity-based scaling can be seamlessly combined with other distant manipulation techniques without major drawbacks. Future research could test how well velocity-based scaling can be integrated with other manipulation techniques to increase usability. This will allow applications to provide more expressive and precise interaction in complex virtual environments.

# References

Bier, E. A. (1990). "Snap-dragging in three dimensions." Proceedings of the 1990 symposium on Interactive 3D graphics, ACM, Snowbird, Utah, United States.   
Bowman, D. A., and Hodges, L. F. (1997). "An evaluation of techniques for grabbing and manipulating remote objects in immersive virtual environments." Proceedings of the 1997 symposium on Interactive 3D graphics, ACM, Providence, Rhode Island, United States.   
Bowman, D. A., Johnson, D. B., and Hodges, L. F. (1999). "Testbed evaluation of virtual environment interaction techniques." Proceedings of the ACM symposium on Virtual reality software and technology, ACM, London, United Kingdom.   
Bowman, D. A., Kruijff, E., LaViola, J. J., and Poupyrev, I. (2004). 3D User Interfaces: Theory and Practice, Addison-Wesley, Boston.   
Bowman, D. A., Setareh, M., Pinho, M. S., Ali, N., Kalita, A., Lee, Y., Lucas, J., Gracey, M., Kothapalli, M., Zhu, Q., Datey, A., and Tumati, P. (2003). "Virtual-SAP: An Immersive Tool for Visualizing the Response of Building Structures to Environmental Conditions." Proceedings of the IEEE Virtual Reality 2003, IEEE Computer Society.   
Brooks, F. P., Ouh-Young, M., Batter, J. J., and Kilpatrick, P. J. (1990). "Project GROPEHaptic displays for scientific visualization." Proceedings of the 17th annual conference on Computer graphics and interactive techniques, ACM, Dallas, TX, USA.   
Cruz-Neira, C., Sandin, D. J., and DeFanti, T. A. (1993). "Surround-screen projection-based virtual reality: the design and implementation of the CAVE." Proceedings of the 20th annual conference on Computer graphics and interactive techniques, ACM.   
Frees, S., Kessler, G. D., and Kay, E. (2007). "PRISM interaction for enhancing control in immersive virtual environments." ACM Trans. Comput.-Hum. Interact., 14(1), 2.   
Kelso, J., Arsenault, L. E., Kriz, R. D., and Satterfield, S. G. (2002). "DIVERSE: A Framework for Building Extensible and Reconfigurable Device Independent Virtual Environments." Proceedings of the IEEE Virtual Reality Conference 2002, IEEE Computer Society.   
Lindsey, P., and McLain-Kark, J. (1998). "A Comparison of Real World and Virtual World Interior Environments." Journal of Interior Design, 24(1), 27-39.   
Mackenzie, I. S., and Riddersma, S. (1994). "Effects of output display and control-display gain on human performance in interactive systems." Behaviour & Information Technology, 13(5), 328 - 337.   
Mine, M. R., Brooks, F. P., and Sequin, C. H. (1997). "Moving objects in space: exploiting proprioception in virtualenvironment interaction." Proceedings of the 24th annual conference on Computer graphics and interactive techniques, ACM Press/Addison-Wesley Publishing Co.   
Pierce, J. S., and Pausch, R. (2002). "Comparing voodoo dolls and HOMER: exploring the importance of feedback in virtual environments." Proceedings of the SIGCHI conference on Human factors in computing systems: Changing our world, changing ourselves, ACM, Minneapolis, Minnesota, USA.   
Pierce, J. S., Stearns, B. C., and Pausch, R. (1999). "Voodoo dolls: seamless interaction at multiple scales in virtual environments." Proceedings of the 1999 symposium on Interactive 3D graphics, ACM, Atlanta, Georgia, United States.

Poupyrev, I., Billinghurst, M., Weghorst, S., and Ichikawa, T. (1996). "The go-go interaction technique: non-linear mapping for direct manipulation in VR." Proceedings of the 9th annual ACM symposium on User interface software and technology, ACM, Seattle, Washington, United States.

Stoakley, R., Conway, M. J., and Pausch, R. (1995). "Virtual reality on a WIM: interactive worlds in miniature." Proceedings of the SIGCHI conference on Human factors in computing systems, ACM Press/Addison-Wesley Publishing Co., Denver, Colorado, United States.