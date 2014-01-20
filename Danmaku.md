Danmaku System
==============

Danmaku (or bullet curtain/bullet hell) describes the gameplay whereby it is necessary to evade an enormous amount of bullets

This document describes how the engine generates Danmaku and how it should be designed.

Objects
-------

There are a limited number of objects available from the scripts:

Overall gameplay:
* Chapter
    This is the top object. It could be seen as a 'game' but since games might continue with their story in other chapters, it's officially a chapter.
* Stage
    This is a stage in a chapter which defines how the stage progresses. It contains a couple of Scene objects.
    In Touhou:
    * Intro
    * Travel 1
    * Pre Boss
    * Travel 2
    * Boss
* Scene
    This describes what happens in a certain scene. It has the ability to trigger certain things based on events.
* BGM
    This describes what game music needs to be loaded.
    
Game objects:
* Drawable
    This is the object that defines how something should look, it is bound to bullets and/or enemies
* Action
    This object can be triggered and does something.
* Bullet
    This is the object that defines how a bullet behaves. In the engine it is loaded as a BulletTemplate which is used to create the real Bullet objects.
* Enemy
    This is the object that defines how a enemy behaves. It can spawn bullets from itself and keeps it's own stats like lives. In the engine
    it is loaded as a EnemyTemplate which is used to create the real Enemy objects.
* Boss
    This is the object that defines how a boss behaves. It is a special kind of enemy since it doesn't use bullets directly, also has several
    phases/lives which it goes through. It uses SpellCard objects to spawn bullets. In the engine it is loaded as a BossTemplate which is used to create
    the real Boss objects.
* SpellCard
    This is the object that is used by a boss to define how it attacks. It is like the enemy object but it does not have lives.
* Pattern
    This is the object that is used by SpellCard and Enemy objects and define a series of how bullets are created.
    
Logic
-----

Spell card:

SpellCard objects will define what patterns will be started and when.
You can choose to start a pattern and define that it should wait until this pattern is finished, or you can choose to start a pattern and do
the next action on the next frame. There are special actions that simply wait for an amount of frames in order to delay different patterns.

Pattern:

In order to create a pattern, you would define a Pattern object. This Pattern object will define how bullets are generated (using emitters).
While it seems unusual, the pattern object actually defines what happens when bullets "die" and to which ones. It also defines what bullets
do during their lifetime.

Bullet:

In order to create a bullet, you would define a Bullet object. This Bullet object will define how it reacts during its lifetime. The object has the
(variable) lifetime of the bullet, it's path logic and what it reacts to and what happens when it does that. Bullets can reference other bullets and
what they inherit from those bullets.

Movement:

Movement is controlled by its physics settings. Bullets can either be directly controlled by changing it's (x,y) values, by changing speed and direction
, by changing the acceleration in a direction or by setting force objects. (So yeah, it's basically real life simplified physics)

Internal workings
-----------------

The game keeps a record of Template objects. The Templates are simple static objects and define how the game is run. Once the the game starts.
The Template object generate their non Template versions of themselves who actually interact with the world and do things. Every object has
a link to their Template object to be aple

Ownership
---------

In general, objects hold strong references to their own Template objects. Every Template object holds strong references to other Template objects.

* Scene objects hold strong references to Boss objects.
* Boss objects hold strong references to SpellCard objects and Drawable objects.
* SpellCard objects hold weak references to Boss objects and strong references to Pattern objects.
* Pattern objects hold weak references to SpellCard objects and strong references to Bullet objects.
* Bullet objects hold strong references to Drawable objects.

Your first spell card
---------------------

You would start by creating a SpellCard object. In that SpellCard object, you choose what patterns are used and in which order. By default, once
the sequence has been run, it will start from the top again, you might add a delay to the end of the sequence. Also, you choose the duration of the
SpellCard and define what happens at the end.

Then you would create several Pattern objects. In those Pattern objects, you choose what special emitter is used and when. There are several emitters:
* Simple emitters:
    * Box
        Randomly distributes bullets in a box. Can also distribute items in a horizontal or vertical line
    * Circle
        Randomly distributes bullets in a circle.
    * Line
        Randomly distributes bullets on a line.
* Advanced emitters:
    * Arc
        Creates bullets from an arc, which is evenly distributed. The start and end points can be dependent on the players position. It also sets the
        initial speed to a fixed value and the angle to diverge from the center
    * Point
        Creates bullets from a point. It sets the inital speed to a fixed value and evenly distributes the angles over the bullets to diverge from the center
    * Star
        Creates bullets from a point. It sets the inital speed to a variable value based on the angle and evenly distributes the angles over the bullets to
        diverge from the center

When an emitter is used, it can optionally set various properties. All simple emitters can set the speed and angle to a fixed value or to a value based
on the location of the bullet (diverging from center). With the advanced emitters, this is always the case.

Patterns allow for emitters to be bound on start, on moving bullets and on dying bullets.

Now, for the pattern to be effective, you need to design bullets. The Bullet object defines that. You choose it's lifetime, perhaps base the bullet on
other bullets and choose the Drawable that defines the bullet. You also set it's hit rules. Either by hit circle or hit line. When you define a hit
circle, the character is hit when the distance between the bullet and character is less than the sum of the two circles. If you define a hit line, the
distance to the line may not be less than the characters hit circle plus the set width.

Finally, to make the bullets visible (if wanted), you define Drawable objects. These objects define how things are drawn. It is here that you define
your special effects and sprites. There are several special transform settings to make special bullets:
* Angle
    This will rotate the sprite to the orientation of the bullet.
* Length
    This will rotate and stretch the sprite to form a line, this is used for beam like bullets.
* Trail
    This will draw several (different) sprites where the bullet was. Used for laser like effects.

File formats
------------

The engine uses a special ogg background music file. It is an ogg file with special tags defining what should be repeated.