---
toc: true
layout: post
description: Tips and tricks for integrating Kotlin into your FTC projects.
categories: [FTC, Kotlin]
title: FTC Kotlin Tips & Tricks
---

<h>Neil Mehra</h>  
<h>August 19, 2021</h>
<p>
</p>

Kotlin is a statically typed, modern, concise JVM language that is designed to be fully interoperable with Java. In FTC, the most popular motion planning library, <a href=https://learnroadrunner.com/>Road Runner,</a> is written in Kotlin. 

## Enabling Kotlin

First, we have to enable the Kotlin plugin in our root level gradle files. Navigate to <code>./build.gradle</code> and add this line to your buildscript

<code>ext.kotlin_version = '1.5.21'</code>

**_Note:_** - at the time of writing 1.5.21 is the latest Kotlin release.

This creates an environment variable with name 'kotlin_version' and value '1.5.21' so we don't have to re-type the release version.  

Next, add the Kotlin plugin to the dependencies block with this line  
<code>classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"</code>

If you haven't already, make sure the language level of the Java source code and version of the generated Java bytecode are 1.8 (JDK 8). Open up <code>build.common.gradle</code> and check if your <code>compileOptions</code> block looks like this

~~~ 
compileOptions {  
        sourceCompatibility JavaVersion.VERSION_1_8  
        targetCompatibility JavaVersion.VERSION_1_8  
} 
~~~

Once that's finished, all that's left to do is to enable the <code>kotlin-android</code> and the <code>kotlin-stdlib-jdk8</code> plugins. Head over to the <code>build.gradle</code> of any modules you want to use Kotlin in. 

Personally, I have Kotlin enabled in both the TeamCode and FTCRobotController modules as I converted the Java sample code that the FTC SDK provides to Kotlin to introduce my jr programmer to the SDK and Kotlin at the same time. Add the following definition to the top of your <code>build.gradle</code> file.
<code>apply plugin: 'kotlin-android'</code>  

Then add this line to your dependencies block.  
<code>implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlin_version"</code>  
  
Sync your gradle changes and then you're all good. 

<p>
</p>

## Converting your codebase to Kotlin  

If you're apprehensive about the how long it'll take to rewrite everything, don't worry. As Kotlin was designed to be fully functional with Java, Java source code can actually be converted directly into readable Kotlin code. Head over to Android Studio and press <code>ctrl+alt+shift+K</code> while a Java file is open to convert it to Kotlin.  

Now that you have Kotlin files to work with, let me give some tips so that I wish I knew back when I started developing in Kotlin.

## Tips & Tricks
1. ### Data Classes  
    Whenever you have classes that are only for holding data, such as a <code>Vector</code> or <code>Point</code> class, turn them into a <code>data class</code>. Data classes contain automatically generated functions that are derived from the constructor fields, which makes life simpler. You can also do cool things with data classes like this:  
    <code> val (x, y) = Point(1.0, 2.0)</code>  

2. ### Custom getters/setters  
    I remember during one of my first days after switching to Kotlin, I spent an entire meeting trying to debug a StackOverflowError that made no sense at all. It was erroring at a 'simple' variable that should never have errored, right? Or so I thought. The error was pointing to this simple line:  
    <code>val dbNormalize = Point(-y, x)</code>  
    At first glance it looks like a completely valid piece of code, but the field was actually declared in the <code>Point</code> data class. The Java equivalent of this would be perfectly valid, but for this you need to specify a custom getter like this:  
    <code> val dbNormalize get() = Point(-y, x)</code>  
    The full form of any Kotlin field is as follows:  
    ~~~ 
    val foo:type = type(a, b, c...)  
        get() = field  
        set(value) {
        field = value
    }
    ~~~ 

3. ### Extension Functions  
    Extension functions were the thing that really made me feel good about switching to Kotlin. The amount of flexibility it offers is amazing. They provide you to define custom functions for any class but "extending" it. This also applies to fields/properties. For example, look at the following example. In FTC angle math is used a lot, and we always want to make sure that the angles we use are between some specified bounds, usually between -π and π.  
    ~~~
    fun Double.wrap(): Double {  
        angle = this
        while(angle >= PI)
            angle -= PI * 2
        while(angle < -PI)
            angle += PI * 2
        return angle
    }

    val Double.toRadians: Double = Math.toRadians(this)
    val Double.toDegrees: Double = Math.toDegrees(this)
    ~~~

    Now I can call this method on any double, like this.

    ~~~
    val myAngleDegrees = 180.0
    val myAngleRad = myAngleDegrees.toRadians
    println(myAngleRad.wrap().toDegrees)
    
    Output: -180.0
    ~~~

4. ### Null Safety  
    Kotlin is supposed to be safe, so what does it do with the dreaded null-types? Simple, nothing can be null unless specifically stated, and even if it is null, there are compile-time checks to make sure nothing goes haywire. To declare a variable as possibly null, you add the <code>?</code> to the end of the type declaration, like this.  
    <code>var foo: String? = "abc"</code>  
    But what happens if it is null? At compile time if you ever access <code>foo</code> without checking if it's null, Kotlin will throw a "can possibly be null" error. The correct way to access <code>foo</code> would be like this.  
    <code>val length = foo?.length ?: -1</code>  
    This is equivalent to the following, more verbose expression.  
    <code>val l: Int = if(foo != null) foo.length else -1</code>  
    And if you really do love having Null Pointer Exceptions, you can assert that a field is not null with <code>!!</code>. Ex:  
    <code>val l = b!!.length</code>

5. ### kotlin.math  
    This is a short one, but the built in extension functions in the kotlin.math class feel really good to use, such as the <code>Double.sign</code> extension field or the <code>Double.exp</code> function.

6. ### Operator Overloading  
    Operator overloading is a must know for anyone who frequently uses any sort of data class such as the aforementioned <code>Vector</code> or <code>Point</code> data classes. Basic unary/binary operators such as the increment operator or the adding operator can be overloaded with your custom definition. Let me just show how much cleaner it is.  
    ~~~
    data class Point(var x: Int, var y: Int) {
        operator fun plus(p: Point) = Point(x + p.x, y + p.y)
        operator fun times(n: Double) = Point(x * n, y * n)
        operator fun unaryMinus(p: Point) = this.times(-1.0)

        override fun toString() = x + ", " + y
    }

    fun main() {
        val foo = Point(2,3)
        val bar = Point(3,2)
        val foobar = foo + bar
        println(foobar.toString())
        val minusfoobar = -foobar
        println(minusfoo.toString()
    }

    Output: 
    5, 5
    -5, -5
    ~~~

7. ### Infix Functions  
    Can't talk about operator overloading without mentioning infix functions. Infix functions are functions that can be called without the dot. I'm not that much of a fan of this due to the possible danger of making stuff so simple that it gets way too overcomplicated, but it does have its uses. Sometimes when I want to approximate something equal to another, I would previously have to do this: <code>Math.abs(a-b) < 1e6 </code>, but with infix notation it becomes a lot cleaner.
    ~~~
    infix fun Double.epsilonEquals(a: Double): Boolean {
        return if(this.isInfinite()) 
            this == a
        else 
            (this - a).absoluteValue < EPSILON
    }

    fun main() {
        val foo = 1.0
        val bar = foo * 1.00000000000001
        println(foo epsilonEquals bar)
    }

    Output: true
    ~~~

8. ### When Statement  
    Using the when statement is like saying goodbye to the <code>if</code> and <code>switch</code> (well, Kotlin doesn't have a switch statement anyway) statements: you'll never want to go back again. Just look at this example in my FTC Angle data class and you'll understand what I mean.
    ~~~
    val deg: Double
        get() = when(unit) {
            AngleUnit.RAD -> angle.toDegrees
            AngleUnit.DEG -> angle
            AngleUnit.RAW -> angle
        }
    ~~~  
    Or when determining the metadata of a <code>Waypoint</code>:
    ~~~  
    val skip: Boolean = when(waypoint) {
        is StopWaypoint -> ....
        is TurnWaypoint -> ....
        is LateTurnWaypoint -> ....
        else -> ....
    }
    ~~~  
    See what I mean?

9. ### Break labels  
    Oftentimes I found myself annoyed with Java not having a specific break when inside nested statements/loops. Kotlin solves this annoyance with <code>labels</code>. Labels can annotate an expression, and breaks/returns can then jump straight to that label when that label is called.
    ~~~
    mainLoop@ for(i in 1..100) {
        for(j in 1..100) {
            if(...) break@mainLoop
        }
    }
    ~~~
    Upon that nested <code>if</code> being fulfilled, the program will break out of both for loops.

10. ### Java Interoperability  
    Last but not least is Kotlin being fully compatible with Java. Any libraries written in Java can be used in Kotlin without any trouble, and any Kotlin classes can be used in Java with some extra annotations. Any Kotlin fields that are being used in a Java class must be labled with the <code>@JVMField</code> annotation, or with the <code>@JVMStatic</code> annotation if it's a static field.  



Thanks for reading, if you have any comments or concerns you can reach me at <a href = "mailto: neilmehra@outlook.com">my email</a>.
    














