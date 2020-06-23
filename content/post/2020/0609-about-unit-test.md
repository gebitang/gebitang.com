+++
title = "关于单测哪些事儿"
description = ""
tags = [
]
date = "2020-06-09"
topics = [
    "BB"
]
+++

[writing-standards-for-unit-testing](https://stackoverflow.com/questions/507000/writing-standards-for-unit-testing)

Unit Test的三A原则: Arrange, Act, Assert
- Arrange any input data and processing classes needed for the test
- Perform the action under test
- Test the results with one or more asserts. Yes, it can be more than one assert, so long as they all work to test the action that was performed.

或者说[5个原则](https://medium.com/@abstarreveld/the-5-unit-testing-guidelines-f21d39c33e0b)

- Unit tests have one assert per test.
- Avoid if-statements in a unit test.
- Unit tests only “new()” the unit under test.
- Unit tests do not contain hard-coded values unless they have a specific meaning.
- Unit tests are stateless.

[不是Unit Test](https://www.artima.com/weblogs/viewpost.jsp?thread=126923)
- It talks to the database
- It communicates across the network
- It touches the file system
- It can't run at the same time as any of your other unit tests
- You have to do special things to your environment (such as editing config files) to run it.

[Test Driven Development FAQ](http://pantras.free.fr/tddfaq.html) 

[微软的单测实践](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)


--- 

八卦时间—— 

>[Michael Feathers]是世界级面向对象技术专家，以丰富的软件项目开发经验著称。~~目前在世界顶尖的软件咨询公司~~从事敏捷方法/极限编程、测试驱动开发、重构、面向对象设计、Java、C#和C++等方面的培训和项目指导。他是著名测试框架CppUnit和FitCpp的开发者，已经主持了三次面向对象界盛会OOPSLA上的CodeFest比赛。他还是[《修改代码的艺术》](http://product.china-pub.com/36363)（Working Effectively with Legacy Code）一书的作者。

>[Michael Feathers](https://michaelfeathers.silvrback.com/bio) is the founder and Director of [R7K Research & Conveyance](http://www.r7krecon.com/), a
company specializing in software and organization design. Michael is also the author
of the book Working Effectively with Legacy Code (Prentice Hall, 2004).


[wiki Jim_Coplien](https://en.wikipedia.org/wiki/Jim_Coplien)在2014年发表了一篇：[Most Unit Testing Is Waste (2014) [pdf]](https://rbcs-us.com/documents/Why-Most-Unit-Testing-is-Waste.pdf) 然后一石激起千层浪：

- [在yc news上的讨论](https://news.ycombinator.com/item?id=20309815)   
- [在reddit上的讨论](https://www.reddit.com/r/programming/comments/4lqni5/why_most_unit_testing_is_waste/) 8个板块都有讨论
- [twitter上Michael Feathers的回复 ](https://twitter.com/mfeathers/status/441598005515669504) 引用了08年自己的一篇文章[The Flawed Theory Behind Unit Testing](https://michaelfeathers.typepad.com/michael_feathers_blog/2008/06/the-flawed-theo.html) 
- [Henrik Warne在个人博客上的response](https://henrikwarne.com/2014/09/04/a-response-to-why-most-unit-testing-is-waste/)，跟Coplien在评论区又讨论了很多。
- 上面这个回复也在reddit上进行了[讨论](https://www.reddit.com/r/programming/comments/2fflvo/a_response_to_why_most_unit_testing_is_waste/)

然后Coplien大佬出了[第二季](https://rbcs-us.com/documents/Segue.pdf)


[Uncle Bob and Jim Coplien’s debate on TDD](https://medium.com/@mena.meseha/uncle-bob-and-jim-copliens-debate-on-tdd-b33006f171b4)提到更早之前(2008年2月)，Cope大老讨论[Coplien and Martin Debate TDD, CDD and Professionalism](https://www.infoq.com/interviews/coplien-martin-tdd)另外一位[Uncle Bob](https://blog.cleancoder.com/uncle-bob/2017/03/03/TDD-Harms-Architecture.html)

>Bob Martin is an Agile Manifesto author, and author of books on Agile Programming, XP, UML, O-O Programming, and C++. He is CEO and president of Object Mentor www.objectmentor.com/ Jim Coplien is a software pioneer in o-o programming and C++ and multi-paradigm design. He appreciates the human side of design, and has written critically acclaimed books on design and development.


