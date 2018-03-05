---
layout:     post
title:      "LayoutInflater详解"
date:       2016-7-6 18:59:25
author:     "afayp"
catalog:    true
tags:
    - Android
---



`LayoutInflater`主要是用于加载布局、动态添加view的时候,中文翻译成`填充`我觉得十分的形象，意味着把一个xml布局填充成view。

<!--more--> 

LayoutInflater其实就是使用Android提供的pull解析方式来解析布局文件的，根据节点名来创建View对象的。先是创建出一个根布局view，然后循环遍历这个根布局下的子元素，每次创建出一个子view就添加到对应的父布局中。最后解析完成返回这个根布局。

获取LayoutInflater实例有两种方法：
```java
LayoutInflater layoutInflater = LayoutInflater.from(context); 

LayoutInflater layoutInflater = (LayoutInflater) context  .getSystemService(Context.LAYOUT_INFLATER_SERVICE);  
```
第一种其实是第二种的简单封装。

得到LayoutInflater的实例之后就可以调用它的inflate()方法来填充加载布局。有下面四个方法：
```java
inflate(resourceId, root)   
inflate(resourceId, root, attachToRoot) 
inflate(XmlPullParser, root)  
inflate(XmlPullParser, root, boolean) 
```
一般用到前面两个，解释如下：
`layoutInflater.inflate(resourceId, root);`
第一个参数就是要加载的布局id，第二个参数是指给该布局的外部再嵌套一层父布局，如果不需要就直接传null。

`inflate(int resource, ViewGroup root, boolean attachToRoot)`
几种情况解释如下：

1. 如果root为null，attachToRoot将失去作用，设置任何值都没有意义。也可以说root为null，attachToRoot即默认为false。
2. 如果root不为null，attachToRoot设为true，则会给加载的布局文件的指定一个父布局，即root。
3. 如果root不为null，attachToRoot设为false，则会将布局文件最外层的所有layout属性进行设置，当该view被添加到父view当中时，这些layout属性会自动生效。
4. 在不设置attachToRoot参数的情况下，如果root不为null，attachToRoot参数默认为true。
5. 注意attachToRoot设置为true或false的区别在于，true表示加载出来的布局将自动附上一个父布局(即root);而false的意思是inflate完毕后并没有自动添加一个父布局(因为你的root为null)，此时最外层的所有layout属性都不生效，而当你后面手动给这个加载出来的布局设置父布局后，layout属性才会生效。

不管你是使用的哪个inflate()方法的重载，最终都会调用到LayoutInflater的如下代码中
```java
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

            final Context inflaterContext = mContext;
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            Context lastContext = (Context) mConstructorArgs[0];
            mConstructorArgs[0] = inflaterContext;
            View result = root;

            try {
                // Look for the root node.
                int type;
                while ((type = parser.next()) != XmlPullParser.START_TAG &&
                        type != XmlPullParser.END_DOCUMENT) {
                    // Empty
                }

                if (type != XmlPullParser.START_TAG) {
                    throw new InflateException(parser.getPositionDescription()
                            + ": No start tag found!");
                }

                final String name = parser.getName();
                
                if (DEBUG) {
                    System.out.println("**************************");
                    System.out.println("Creating root view: "
                            + name);
                    System.out.println("**************************");
                }

                if (TAG_MERGE.equals(name)) {
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }

                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                    // Temp is the root view that was found in the xml
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }

                    if (DEBUG) {
                        System.out.println("-----> start inflating children");
                    }

                    // Inflate all children under temp against its context.
                    rInflateChildren(parser, temp, attrs, true);

                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }

            } catch (XmlPullParserException e) {
                InflateException ex = new InflateException(e.getMessage());
                ex.initCause(e);
                throw ex;
            } catch (Exception e) {
                InflateException ex = new InflateException(
                        parser.getPositionDescription()
                                + ": " + e.getMessage());
                ex.initCause(e);
                throw ex;
            } finally {
                // Don't retain static reference on context.
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;
            }

            Trace.traceEnd(Trace.TRACE_TAG_VIEW);

            return result;
        }
    }
```
`LayoutInflater`其实就是使用Android提供的pull解析方式来解析布局文件的。会调用了`createViewFromTag()`这个方法，并把节点名和参数传了进去。这个方法是用于根据节点名来创建View对象的。在`createViewFromTag()`方法的内部又会去调用`createView()`方法，然后使用反射的方式创建出View的实例并返回。这样就创建了根布局，然后会调用`rInflate()`或者`rInflateChildren`方法来循环遍历这个根布局下的子元素，代码如下：
```java
void rInflate(XmlPullParser parser, View parent, Context context,
            AttributeSet attrs, boolean finishInflate) throws XmlPullParserException, IOException {

        final int depth = parser.getDepth();
        int type;

        while (((type = parser.next()) != XmlPullParser.END_TAG ||
                parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {

            if (type != XmlPullParser.START_TAG) {
                continue;
            }

            final String name = parser.getName();
            
            if (TAG_REQUEST_FOCUS.equals(name)) {
                parseRequestFocus(parser, parent);
            } else if (TAG_TAG.equals(name)) {
                parseViewTag(parser, parent, attrs);
            } else if (TAG_INCLUDE.equals(name)) {
                if (parser.getDepth() == 0) {
                    throw new InflateException("<include /> cannot be the root element");
                }
                parseInclude(parser, context, parent, attrs);
            } else if (TAG_MERGE.equals(name)) {
                throw new InflateException("<merge /> must be the root element");
            } else {
                final View view = createViewFromTag(parent, name, context, attrs);
                final ViewGroup viewGroup = (ViewGroup) parent;
                final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
                rInflateChildren(parser, view, attrs, true);
                viewGroup.addView(view, params);
            }
        }

        if (finishInflate) {
            parent.onFinishInflate();
        }
    }
```
同样是调用`createViewFromTag()`方法来创建View的实例，然后调用`rInflateChildren`方法，该方法又会调用`rInflate()` 。 就这样递归调用`rInflate()`方法来查找这个View下的子元素，每次递归完成后则将这个View添加到父布局当中。最终把整个布局文件都解析完成后就形成了一个完整的DOM结构，最后会把最顶层的根布局返回，至此inflate()过程全部结束。

任何一个Activity中显示的界面其实主要都由两部分组成，标题栏和内容布局。标题栏就是在很多界面顶部显示的那部分内容，比如刚刚我们的那个例子当中就有标题栏，可以在代码中控制让它是否显示。而内容布局就是一个FrameLayout，这个布局的id叫作content，我们调用setContentView()方法时所传入的布局其实就是放到这个FrameLayout中的，这也是为什么这个方法名叫作setContentView()，而不是叫setView()。
![](http://oeiu2t0ur.bkt.clouddn.com/20131218231254906.png)