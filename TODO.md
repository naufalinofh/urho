
Findings
========

Struct vector3 can be passed by ref to an unmanaged method that will take that as a Vector3&

Binding
=======

Missing Bindings
----------------

PODVector
---------

This comes in three flavors, "PODVector<X>", "PODVector<X>&" and
"const PODVector<X>&", it is used as vector that can be blit, but they
do use it to store classes (see documentation about the differences in
which ctors are called in PODVector vs Vector).

THe ones that we seem to be interested are the second two, the ones
that are passed by reference as they are the most common.  The
non-const version is used in a few cases to return new values, so
perhaps we can bind those differently.

Ownership
---------

Currently there is no object destruction of any kind, although there
is some supporting code to queue finalizers to invoke destruction on
the main thread.

But we do not have a way of tracking whether the unmanaged code is
keeping a reference to our code around.  This might not be an issue,
as long as we do not add managed state and prevent users from
subclassing our framework classes (because then, we can relinquish our
reference safely, as we can always recreate it if needed).


C# code Registration
--------------------

The C++ code uses an idiom where classes must be registered with the
context before they can be used by the various "Create" methods, for example

	context->RegisterFactory<Rotator> ()

Which creates an ObjectFactoryImpl that has been initialized with some
of the constant values from the C++ type to track the type, base type
and name.  Once that happens, it is possible to
CreateComponent<Rotator>.

In the scripting world, that is achieved with the ScriptComponent,
which has support to proxy the data and also convenience methods to 
attaching to events to listen to (not needed in our case I think).

What we need to do is implement a subclass of ObjectFactory that
overrides the CreateObject() method with one that uses C# to create the object and returns it.

The sole point of this is to allow the idiom: foo->CreateObject<XX>
instead of doing "new XX(Context)" followed by an AddComponent.

ALTERNATIVE: provide a managed implementation for the registration
of managed code, and invoke code directly.

Blessing of Objects
-------------------

Need some boilerplate to allow C# classes to implement the entire set
of Urho object system methods.  For example, used in registration of
objects and factories.

Style
-----
Input.MouseMove is a property, this does not look good.   It should be jsut a method, so we need a blacklist there.

Scale overloads
---------------

Node's scale overloads does not cope well with the get/set autogenerator

API
---
Produce default parameters

Object System
-------------

Need support most derived object creation in Runtime.LookupObject.

Bind
----
[ ] Input/inputEvents constant definitions.


WorkItem issues
---------------
structs that subclass RefCounted should be bound as classes, see
WorkItem (which is currently surfaced as a struct)

Generator
=========

Serializer, Deserializer
------------------------
They are only used in a couple of places as base classes, and the type
is in general not used to pass data around.   So we should inline the
methods from those classes in the classes that adopt them.

(File, MemoryBuffer, VectorBuffer)


Build Issues
============

Currently we hardcode /cvs/Urho3D as well as hardcoding URHO3D_OPENGL,
and the reality is that we should instead install Urho3D into a
prefix, and use pkg-config to fetch the flags for the install (so we
actually get the configured flags correctly, and we do not need to
hardcode URHO3D_OPENGL everywhere)


Optimizations
=============

Events
------

Object.cs/ObjectCallback: do we even need the stringHash in the event?

Seems like we do not, since we proxy everything.

API improvements
================

Node's SetBoundingBox should have a GetBoundingBox and surface a BoundingBox property

