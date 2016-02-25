转自：https://possiblemobile.com/2013/05/layout-inflation-as-intended/

##Layout inflation is the term used within the context of Android to indicate when an XML layout resource is parsed and converted into a hierarchy of View objects. 
It’s common practice in the Android SDK, but you may be surprised to find that there is a wrong way to use LayoutInflater, and your application might be one of the offenders. If you’ve ever written something like the following code using LayoutInflater in your Android application: 
```Java
inflater.inflate(R.layout.my_layout, null);
```
PLEASE read on, because you’re doing it wrong and I want to explain to you why.
```Java
Get to Know LayoutInflater
```
Let’s first take a look at how LayoutInflater works. There are two usable versions of the inflate() method for a standard application: 
```Java
inflate(int resource, ViewGroup root)
```
```Java
inflate(int resource, ViewGroup root, boolean attachToRoot)
```
 The first parameter points to the layout resource you want to inflate. The second parameter is the root view of the hierarchy you are inflating the resource to attach to. When the third parameter is present, it governs whether or not the inflated view is attached to the supplied root after inflation.

It is these last two parameters that can cause a bit of confusion. With the two parameter version of this method, LayoutInflater will automatically attempt to attach the inflated view to the supplied root. However, the framework has a check in place that if you pass null for the root it bypasses this attempt to avoid an application crash.

Many developers take this behavior to mean that the proper way to disable attachment on inflation is by passing null as root; in many cases not even realizing that the three parameter version of inflate() exists. By doing things this way, we also disable another very important function the root view has…but I’m getting ahead of myself.
