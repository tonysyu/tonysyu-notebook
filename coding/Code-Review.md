Code Review
===========

Overview
--------

Code review should be focused on four things:

- Functionality and Security
- Code Clarity and Design

Principles to remember when writing reviews and receiving feedback:

- Code reviews are an opportunity to learn and teach.
    - If it's not obvious, explain _how_/_why_ the change improves clarity or functionality.
- Check your ego at the door ([You are not your code](http://www.hanselman.com/blog/YouAreNotYourCode.aspx))
    - If a junior dev doesn't understand code from a senior dev, most likely it needs to be
      clearer. (Exceptions include language idioms, advanced topics, etc. that the junior dev
      hasn't yet learned.)
    - Reviews tend to be blunt. It's not personal.
- Suggest improvements rather than simply criticizing.


Functionality and Security
--------------------------

Functionality: Obviously, the code should implement the desired feature (or fix the undesired bug).
If we were writing tests, I would call this happy-path testing. When I say security, I mean that
you should consider the sad-path:

- What happens when things fail?
- What happens when arguments are null?
- What vulnerabilities can be exploited?


Code Clarity and Design
-----------------------

Code clarity and design are essentially the small and large scopes of code quality.

On the code clarity side, you should consider the general principles espoused in
[Clean Code](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_2?ie=UTF8&qid=1455681555&sr=8-2&keywords=clean+coder)
and [Pragmatic Programmer](http://www.amazon.com/Pragmatic-Programmer-Journeyman-Master/dp/020161622X/ref=sr_1_1?ie=UTF8&qid=1455681575&sr=8-1&keywords=pragmatic+programmer).
Code clarity also means that you should be adhering to
[standards and conventions](https://www.python.org/dev/peps/pep-0008/) and the idioms of your
specific language (e.g. [Python Cookbook](http://www.amazon.com/Python-Cookbook-Third-David-Beazley/dp/1449340377/ref=sr_1_1?ie=UTF8&qid=1455681599&sr=8-1&keywords=python+cookbook)).

On the design side, you should consider [design patterns](http://gameprogrammingpatterns.com/contents.html)
and architectural patterns (e.g. [Domain-Driven Design](https://www.amazon.com/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577/ref=sr_1_fkmr0_1?s=books&ie=UTF8&qid=1493257769)).

I would argue that code clarity and design are often more important than _complete_ functionality
and _robust_ security. Clean code is easier to understand, debug, and extend. When reading clean
code, it's easier to find gaps in functionality and security vulnerabilites.

In the end, code is read more often than it's written. Time spent writing clearer code will usually
pay off in the long run.
