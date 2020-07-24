---
title: "Factories breed complexity"
layout: 'post'
author: bo
---

Having maintainable code is great. Maintainable code allows you to deliver improvements faster, happier, and more reliably. 

Furthermore, the measures that developers need to take and the strategies that we have to employ to achieve maintainable code have been understood for years, if not decades. Especially in the realm of object-oriented programming, but certainly not exclusively, most of these principles boil down to reducing coupling and system complexity. A system whose parts are coupled as loosely as possible is a modular system; the parts know little of each other and a lot about themselves and they have thin and specific interfaces between each other.

Test-driven development is one of the many tools at a developer's disposal to achieve code quality. Unfortunately, there is a lot of naïveté around the benefits of TDD. A lot of developers see TDD as primarily a tool for verifying system correctness. While TDD does of course offer this benefit, and arguably better than retroactive automated testing, the real benefit of TDD is that it offers short feedback loops that guide the design/architecture of the system.

Since it is accepted that a loosely-coupled modular system is a simpler system, it stands that tools, such as TDD, which guide a design towards modularity and simplicity are good tools. A module that is tightly coupled to another is not easily tested in isolation. However, if the isolated tests are written first, it is difficult to write a passing implementation for that module that maintains such a low degree of coupling. Therefore, good TDD should guide you towards a simpler design (though it is certainly not the only way).

Unfortunately, factories work against this goal. Factories debilitate TDD's ability to give you feedback into the complexity of your design.

To be clear, I am not talking about the [Factory Method Pattern](http://en.wikipedia.org/wiki/Factory_method_pattern)
or the [Abstract Factory Pattern](http://en.wikipedia.org/wiki/Abstract_factory_pattern) — both of which can be described
as ways to <q src="http://en.wikipedia.org/wiki/Object_creation#Creating_objects">decouple a particular implementation
of an object from code for the creation of such an object</q> (<cite>[Wikipedia](http://en.wikipedia.org/wiki/Object_creation#Creating_objects)</cite>). Instead, I am talking about the "factories" for replacing fixtures in tests — something which has seemingly obsessed the Ruby (Rails, especially) community. The two primary Ruby libraries for factory-based fixture replacement are [factory_girl](https://github.com/thoughtbot/factory_girl) and [Machinist](https://github.com/notahat/machinist).

A tool such as Machinist or factory_girl <q>generates data for the attributes you don't care about, and constructs any necessary associated objects, leaving you to specify only the fields you care about in your test</q> (from Machinist's own [README file](https://github.com/notahat/machinist/blob/master/README.markdown)). This sounds nice at first, because it makes your tests more readable and relevant. However, behind the scenes, these tools are still creating other objects and entities and introducing them into your test environment. By having data and objects in tests that are irrelevant to the functionality that is being tested (in *isolation*, remember), a developer creates an environment that permits, if not invites, silent dependencies to creep into an implementation.

Furthermore, and perhaps more significantly, by creating objects (and usually entire *hierarchies* of objects) with such ease and opacity, you are outright masking the dependencies (\*cough* complexity \*cough* coupling) between your implementation and those entities. If forced to stub out all those intricacies, the system complexity would be screamingly obvious and a developer would quickly avail herself of a rewrite to reduce complexity or thin out the interface. 

Instead of having feedback that guides a developer to simplicity, fixture factories seem to guide developers to complexity by masking dependencies as one-line simplicity. In fact, that one (or five, whatever) line setup is a shotgun blast of environmental dependencies that are hidden from the architect. That complexity will come back for revenge after being ignored for so long.

It seems that factory_girl and Machinist exist to make testing components more convenient. This is, at face value, an admirable and desirable goal. However, in unit tests, the cost is too high for any system of considerable size. 

Please, do the right thing and avoid the convenience and "fun" of the factory_girl temptress. You will trend towards a simpler system and as a bonus (in fact, an incredible one) your test suite will likely be exponentially faster which, in turn, will breed simplicity by letting you have more feedback more often.

P.S. It shouldn't go without mention that factories can be absolutely awesome for integration tests. Integration tests aren't used for guiding system design nor testing in isolation so the drawbacks of these tools drop away. However, both factory_girl and Machinist use RSpec as some of their very first usage examples and this troubles me deeply.

P.P.S. A lot of these arguments can be applied to fixtures too. However, they usually don't create hordes of objects invisibly and litter your environment with them. Also, they aren't as slow. But yes, the fewer factories *and* fixtures in a test, the better.

**Update for clarity**: Firstly, I am absolutely just talking about unit tests. If you are testing code that *integrates* with ActiveRecord or number of levels of your stack, then factories and fixtures are certainly defensible (though I still prefer to steer clear). Secondly, I've tried to be careful about where I use the words simple, easy, complex, and difficult. For the definitions that I intend, please watch (at least the first 10 minutes of) [Simple Made Easy](http://www.infoq.com/presentations/Simple-Made-Easy). 
