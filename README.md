转自：https://possiblemobile.com/2013/05/layout-inflation-as-intended/

##Layout inflation is the term used within the context of Android to indicate when an XML layout resource is parsed and converted into a hierarchy of View objects. 
It’s common practice in the Android SDK, but you may be surprised to find that there is a wrong way to use LayoutInflater, and your application might be one of the offenders. If you’ve ever written something like the following code using LayoutInflater in your Android application: 
```Java
inflater.inflate(R.layout.my_layout, null);
```
PLEASE read on, because you’re doing it wrong and I want to explain to you why.
###Get to Know LayoutInflater
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
###Examples from the Framework
Let’s examine some situations in Android where the framework expects you as a developer to interactively inflate portions of the view.
**Adapters** are the most common case for using LayoutInflater is custom ListView adapters overriding getView(), which has the following method signature:
```Java
getView(int position, View convertView, ViewGroup parent)
```
**Fragments** also use inflation often when creating views via onCreateView(); notice its method signature:
```Java
onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState)
```
Have you noticed that every time the framework wants you to inflate a layout, they also pass you the parent ViewGroup it will eventually be attached to? Notice also that in most cases (including the above two examples), it will throw an Exception later on if LayoutInflater is allowed to automatically attach the inflated view to the root.

So why do you suppose we are given this ViewGroup if we are not supposed to attach to it? It turns out the parent view is a very important part of the inflation process because it is necessary in order to evaluate the LayoutParams declared in the root element of the XML being inflated. Passing nothing here is akin to telling the framework “I don’t know what parent this view will be attached to, sorry.”

The problem with this is android:layout_xxx attributes are always be evaluated in the context of the parent view. **As a result, without any known parent, all LayoutParams you declared on the root element of your XML tree will just get thrown away**, and then you’ll be left asking “why is the framework ignoring the layout customizations I defined? I’d better check SO and then file a bug.”

Without LayoutParams, the ViewGroup that eventually hosts the inflated layout is left to generate a default set for you. If you are lucky (and in many cases you are) these default parameters are the same as what you had in XML…masking the fact that something is amiss.
###Application Example
So you claim you’ve never seen this happen in an application? Take the following simple layout that we want to inflate for a ListView row:
####R.layout.item_row
```Java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
　　android:layout_width="match_parent"
　　android:layout_height="?android:attr/listPreferredItemHeight"
　　android:gravity="center_vertical"
　　android:orientation="horizontal">
　　<TextView
　　　　android:id="@+id/text1"
　　　　android:layout_width="wrap_content"
　　　　android:layout_height="wrap_content"
　　　　android:paddingRight="15dp"
　　　　android:text="Text1" />
　　<TextView
　　　　android:id="@+id/text2"
　　　　android:layout_width="0dp"
　　　　android:layout_height="wrap_content"
　　　　android:layout_weight="1"
　　　　android:text="Text2" />
</LinearLayout>
```
We want to set the height of our row to be a fixed height, in this case the preferred item height for the current theme…seems reasonable.

However, when we inflate this layout the wrong way
```Java
public View getView(int position, View convertView, ViewGroup parent) {
　　if (convertView == null) {
　　　　convertView = inflate(R.layout.item_row, null);
　　}
 
　　return convertView;
}
```
we end up with a result that looks like this<br/>
![github](http://www.doubleencore.com/wp-content/uploads/2013/05/Image11-300x187.png "github")  
What happened to the fixed height we set?? This is usually where you end up setting the fixed height on all your child views, switching the root elements height to wrap_content, and move on without really knowing why it broke (you may have even cursed at Google in the process).

If we instead inflate the same layout this way
```Java
public View getView(int position, View convertView, ViewGroup parent) {
　　if (convertView == null) {
　　　　convertView = inflate(R.layout.item_row, parent, false);
　　}
 
　　return convertView;
}
```
we end up with what we expected in the first place.<br/>
![github](http://www.doubleencore.com/wp-content/uploads/2013/05/Image21-300x187.png "github")  
Hooray!
###Every Rule Has An Exception
There are of course instances where you can truly justify a null parent during inflation, but they are few. One such instance occurs when you are inflating a custom layout to be attached to an AlertDialog. Consider the following example where we want to use our same XML layout but set it as the dialog view:
```Java
AlertDialog.Builder builder = new AlertDialog.Builder(context);
View content = LayoutInflater.from(context).inflate(R.layout.item_row, null);
 
builder.setTitle("My Dialog");
builder.setView(content);
builder.setPositiveButton("OK", null);
builder.show();
```
 The issue here is that AlertDialog.Builder supports a custom view, but does not provide an implementation of setView() that takes a layout resource; so you must inflate the XML manually. However, because the result will go into the dialog, which does not expose its root view (in fact, it doesn’t exist yet), we do not have access to the eventual parent of the layout, so we cannot use it for inflation. It turns out, this is irrelevant, because AlertDialog will erase any LayoutParams on the layout anyway and replace them with match_parent.

So the next time your fingers are tempted to just type null into inflate(), you should stop and ask yourself “do I really not know where this view will end up?”

Bottom line, you should think of the two parameter version of inflate() as a convenience shortcut to omit true as the third paramter. You should not think of passing null as a convenience shortcut to omit false.

For future insights, please subscribe to our email list at the bottom of the page (below the comments)!
