---
layout: post
title: Thoughts - Seperation of Concerns
date: 2020-08-29 17:57:50.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- General
- Programming
tags:
- Thoughts
author: Jonathan Tweedle
permalink: "/2020/08/29/thoughts-separation-of-concerns/"
---
My current workday efforts revolve around a Java based solution. I have been working on refactoring the design that was edging towards something more monolithic in its design to break it up in smaller, easier to maintain packages.

One of the issues was how most of the logic was boxed into a single class. The problem was how the creating of the object relied on being provided another class object, with direct interaction to the private fields. Normally this is not a problem, but I was looking to decompose into multiple class objects. When you have a lot of business logic to work with, this can become time consuming. The goal being to migrate into a POJO (Plain Old Java Object) with no business logic on how to populate the fields, and to push the business logic into a Factory class which knows how to digest any input data in order to create the desired output.

I was looking into the pro's and con's of using the classic Java Getter/Setter method. Some good arguments was how syntactically speaking, the Getter/Setter pretty much does the same thing as declaring a public field which can be accessed easily. Consider the below two examples. They are both basically the same with Example A being a little easier to code and use. It will still work with basic Serialization/Deserialization methods ... so why go to the effort of adding more layers of code on top?

**Example A**
```java
public class Foo {
  public String bar;
}
```

**Example B**
```java
public class Foo {
  private String bar
  public String getBar() { return this.bar; }
  public void setBar(String value) { this.bar = value; }
}
```

One benefit of Example B is the ability to limit access to the field. Like making the field read-only. If you choose to keep the business logic in the class, using the getters and setters to reference the internal fields. This has the benefit if you need to change some logic regarding a field such as making sure a string value matches a desired pattern. With Example A, you need to update the logic in each location to make sure the logic is universally applied which because challenging the bigger the design is. With example B, you simply need to update your logic in either the getter or setter giving a single point to manage the logic.

In terms of serialization/deserialization such as with Jackson (json) you have multiple options on hand. The easiest is to tag the class which can inspect the class to leverage any public fields or getters and setters. The getters ans setters add additional benefits to create unidirectional (de)serialization if needed. You can also tag the getters and setter independently or specify custom serialization on a per field basis. The next option is write a custom serializer but that can be more involved and worse is when you need to update the class to add new fields. Adding fields requires updating your custom serializers. 

## Conclusion

Separation of concerns by spreading the code design around does mean more code and that can also take additional time. This might sound like something you wouldn't want to do if your under pressure. The benefit is making the solution more flexible to changes which sometimes requires redesigning half your solution. Separating the different concerns of your solution by ensuring there is not a lot of tightly coupled components might take a little more time initially but the long term time savings are worth it. The other benefit is each component being easier to test which is crucial when you make changes to ensure you don't break something by mistake.
