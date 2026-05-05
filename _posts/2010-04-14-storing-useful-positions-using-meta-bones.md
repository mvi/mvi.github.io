# Storing useful positions using meta-bones

In the development of many computer games, it is often important and necessary to store positions and meta data relating to 3d models and scenes. For example, you may have a model of a car and wish to place a number of meta-points on the model. Such as a point on the exhaust for a smoke emitter, points on the front for the headlights to project from, an object attachment point for that roof mounted turret, local damage helpers and positions inside the car for players to sit and interact with the steering wheel, to name but a few implementations. Rockstar North for example use this system in GTA: San Andreas by setting up dummies in 3d models. An example can be viewed [here](https://www.youtube.com/watch?v=M1Z9SkNiO2A) of a car from the game.

A method to address such a need is in placing pre-identified (essentially meta) bones throughout the model. There are many names for creatable objects in graphics packages which will export to bones, 3ds max has dummies, Maya has locators and most other software packages call them nulls (such as Lightwave, Softimage, Houdini, Blender and Cinema4d), systems like Unreal Engine call them pawns. These bones are connected to no geometry, which means its very easy to access them from your bone tree but they won’t visibly be displayed (unless you want them to be) and so are ideal for representing invisible helper points.

Some key uses of meta bones include:
- Object attachment points, e.g. customisable weapon hard-points on a spaceship, or an articulated trailer on the back of a truck.
- Particle emitter locations
- Spawn points
- Lights
- Waypoints
- CTF Locations
- Weapon Pickups
- King of the hill zones
- Anchor points

## How to make them

These objects are very easy to create in most packages, for example in 3ds max simply go to the create tab, then the helper tab and click the “Dummy” button and create the object in one of your viewports. You can accomplish the same in Maya by creating a locator, in Lightwave by creating a dummy and most other graphics packages use the term “null”.

Now that you’ve created one of these object, you now need to encode information about its purpose and relevance, which can be parsed by your game. In this article I describe storing the data and identification by encoding it into the bone name, however in the “Taking it further” section I cover some options for extending the capabilities of this meta-data. Each object should have a name which can identify every piece of information it needs to communicate to the game, such as the type of meta-object it is and some related data. Also note that bone names must be unique, so you may need to include some sort of unique identification number. As an example, you could approach encoding a smoke emitter like so: `<type>-<subType>-<uniqueId>-<size>` such as `emitter-smoke-01-50`. Bear in mind that there will be a comparatively small limit on the length of the name.

Also remember that it can get confusing having a lot of these objects in a scene, so confusing object a that does x with object b that does y is a problem. You may consider picking different colours for different types of objects.

## How to load them and display them

Once exported, your nulls (or equivalent) should now be stored as model bones. You can retrieve a list of the model’s bones by accessing the collection of ModelBones, for example: myModel.Bones. You will then have a list of bones which store the name and world matrix, which is great as you can retrieve the relevant meta-tags stored in the name and decompose the world matrix into its translation and scale vector3s and a quaternion for rotation (you can convert this to your conventional Euler angles (i.e. yaw, pitch, roll) by using skytigercube’s method in this [article](http://forums.xna.com/forums/p/4574/23763.aspx) at Creators Club.) For example:
{% highlight csharp %}
{% raw %}
Vector3 scale, translation, eulerRotation;
Quaternion rotation;
bone.Decompose(out scale, out rotation, out translation);
eulerRotation = QuaternionToYawPitchRoll(rotation);
{% endraw %}
{% endhighlight %}
You may wish to parse the bones in the model when you load the model in and build up the data structures which will rely on the stored meta-data. A quick example, which just runs through bones each Draw call and displays a sphere at all bones that begin with “myNamedTag” follows:

{% highlight csharp %}
{% raw %}
foreach(ModelBone bone in model.Bones)
{
    if(bone.Name.StartsWith(“myNamedTag”)
    {
        spherePrimitive.Draw(bone, view, projection, Color.White);
    }
}
{% endraw %}
{% endhighlight %}
Here any bones that begin with “myNamedTag” such as “myNamedTag01”, “myNamedTag02” and “myNamedTag-Front” will have a sphere drawn using their bone world matrix. Note that this is using the sphere primitive class from the Primitives3D Creators Club sample and relies upon an already sphere primitive.

## Taking It Further

Encoding meta-data into the name of bones works fine for fairly simple implementations, however if an object has a lot of information attached to it this becomes less than ideal to encode everything into the contents of a small text box. You may consider encoding all this data into each object’s user defined properties (in 3ds max, other software will have equivalents.)

In terms of displaying and loading user-defined-properties you can check out the creators club NormalMappingEffect sample (and I’m told the premium robot sample), while <http://xnaengine.com/> and <http://www.catalinzima.com/?page_id=66> appear to do something with 3ds max user-defined properties. There is also some discussion at: <http://forums.xna.com/forums/t/13265.aspx?PageIndex=2>

You could also make a custom tool to achieve more customised functionality or extend your graphics package’s functionality by taking into account its support for plugins and scripting (if it has the support) and coding in what you need.

## And Finally

I hope this article has been informative. Criticism and comments are appreciated. If people want me to put together a quick sample I’ll be happy to do so, just comment.

## References

<http://www.gamedev.net/community/forums/topic.asp?topic_id=279475>
<http://gamecareerguide.com/features/20010324/melax_03.htm>
<http://www.3dcognition.com/theoryGameDev.php>
<http://update.multiverse.net/wiki/index.php/Platform_Tutorial_Importing_Static_Models>
