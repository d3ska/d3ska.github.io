  ---
Blog Post Ideas from "97 Things Every Programmer Should Know"

1. "The Myth of the 10x Developer: Why Working Harder Won't Save You"

Source essays: Hard Work Does Not Pay Off, Put the Mouse Down and Step Away from the Keyboard, Do Lots of Deliberate
Practice, The Guru Myth

Angle: Dismantle the hustle-culture myth in tech. Working 60-hour weeks makes you worse, not better. Real expertise
comes from deliberate practice (10,000 hours of the right kind), stepping away to let your creative brain solve
problems, and accepting that "gurus" are just disciplined humans. Include the regex story from the book — a developer
refactored 30 lines into 1 after sleeping on it.

  ---
2. "Your Code Is a Love Letter to the Future (Write It Like One)"

Source essays: A Message to the Future, Write Code As If You Had to Support It for the Rest of Your Life, Only the
Code Tells the Truth, Ubuntu Coding for Your Friends, You Gotta Care About the Code

Angle: Code is not just instructions for a machine — it's a message to the next developer (often your future self).
The Ubuntu philosophy ("a developer is a developer through other developers") and the story of Joe writing
impenetrable code his brother Phil would inherit make this deeply personal. Your code is your reputation trail.

  ---
3. "When Comments Lie: The Art of Letting Code Speak for Itself"

Source essays: A Comment on Comments, Comment Only What the Code Cannot Say, Code in the Language of the Domain, Code
Layout Matters

Angle: The heated debate between "always comment" and "comments are a code smell." Both sides have a point. Comments
rot, become wrong, and survive forever. The solution: make code so expressive (domain types, good names, clean layout)
that comments become unnecessary — then only comment what the code cannot say. Include the career-limiting move story
of pasting an email into a code comment.

  ---
4. "Less Code, Better Software: The Counterintuitive Art of Subtraction"

Source essays: Improve Code by Removing It, Simplicity Comes from Reduction, Beauty Is in Simplicity, The Boy Scout
Rule

Angle: The most impactful code change you can make is deletion. Feature bloat, YAGNI violations, and "fun" extras slow
systems down and breed bugs. Stefan deleting his student's code line by line. The Boy Scout Rule: leave code cleaner
than you found it. Simplicity isn't the starting point — it's the destination after reduction.

  ---
5. "Technical Debt Is a Loan — And You're Paying Compound Interest"

Source essays: Act with Prudence, The Longevity of Interim Solutions, Before You Refactor, Don't Be Afraid to Break
Things

Angle: Technical debt isn't just a metaphor — it follows real financial dynamics. Deliberate debt (shortcuts under
deadline pressure) compounds silently. "Interim" solutions become permanent because they work, acquiring inertia
nobody wants to fight. Cover when to refactor (incremental, not "one big shebang"), when to leave things alone, and
when to be the surgeon who cuts boldly.

  ---
6. "The Dark Side of DRY: When Code Reuse Becomes a Trap"

Source essays: Don't Repeat Yourself, Beware the Share, WET Dilutes Performance Bottlenecks, Convenience Is Not an
-ility

Angle: Provocative take — DRY is gospel, but blindly eliminating duplication can increase coupling and hide
performance bottlenecks. The story of the eager junior developer who shared code between unrelated business domains —
tying their shoelaces together. WET code spreads a 30% CPU bottleneck into ten invisible 3% fragments. Sometimes 3
similar lines are better than 1 premature abstraction.

  ---
7. "A Philosophy of Failure: Why Error Handling Is the Most Important Code You'll Write"

Source essays: Don't Ignore That Error!, Don't Nail Your Program into the Upright Position, Distinguish Business
Exceptions from Technical, Verbose Logging Will Disturb Your Sleep, Prevent Errors

Angle: The broken-leg analogy (walking on a fracture makes it worse), the "nailing the corpse in the upright position"
horror story of nested try/catch, and why swallowing exceptions is the moral equivalent of ignoring a fire alarm.
Distinguish business exceptions (insufficient funds) from technical ones (null pointer). And if your error log wakes
you at 3 AM, it better be for a real reason.

  ---
8. "Designing APIs People Actually Love: The Golden Rules"

Source essays: The Golden Rule of API Design, Make Interfaces Easy to Use Correctly and Hard to Use Incorrectly,
Convenience Is Not an -ility, Prefer Domain-Specific Types to Primitive Types

Angle: The $327.6 million Mars Climate Orbiter crashed because of a unit mismatch that stronger typing would have
caught. APIs should make the "pit of success" the path of least resistance. The Golden Rule: write unit tests for code
that uses your API, not just the API itself. walk(true) vs run() — vocabulary matters. parser.processNodes(text,
false) is meaningless without docs.

  ---
9. "Master Your Tools or They'll Master You"

Source essays: Know Your IDE, Know How to Use Command-Line Tools, The Unix Tools Are Your Friends, Choose Your Tools
with Care, Step Back and Automate, Automate, Automate

Angle: "We expect a plumber to know his blowtorch." Yet most developers use 10% of their IDE's power. Learn keyboard
shortcuts (Ctrl+Shift+I to inline a variable without breaking flow). But also learn the command line — IDEs come and
go, Unix tools are eternal. The word processor line-counting horror story. Automation misconceptions debunked (5 of
them). Choose tools carefully — vendor lock-in, lifecycle mismatches, and XML config hell are real risks.

  ---
10. "Writing Tests That Actually Prove Something"

Source essays: Testing Is the Engineering Rigor of Software Development, Test for Required Behavior Not Incidental
Behavior, Test Precisely and Concretely, Write Tests for People, Write Small Functions Using Examples

Angle: Testing is the closest thing we have to structural engineering calculations for software. But most tests are
poorly written — testing implementation details (asserting -1 instead of "negative"), testing too vaguely (sorted +
same length but not a permutation), or being unreadable. Tests are documentation. Write them for the next developer,
not the compiler. Use concrete examples: [3,1,4,1,5,9] sorted is [1,1,3,4,5,9] — accept no substitutes.

  ---
11. "Learn a Language You'll Never Use at Work (It'll Make You Better at Everything)"

Source essays: Know Well More Than Two Programming Languages, Don't Just Learn the Language Understand Its Culture,
Apply Functional Programming Principles, Reinvent the Wheel Often

Angle: Each new paradigm rewires your brain. Moving from C to Haskell teaches you referential transparency. Learning
Ruby improves your C# delegates. You can write Fortran in any language — but why would you? Learning culture, not just
syntax, is key. And sometimes reinventing the wheel (building your own linked list, memory manager) gives you
understanding that reading a textbook never will.

  ---
12. "Programming Is a Team Sport (Even If You Hate Meetings)"

Source essays: Pair Program and Feel the Flow, Two Heads Are Often Better Than One, Code Reviews, When Programmers and
Testers Collaborate, News of the Weird: Testers Are Your Friends, Start from Yes

Angle: The "lone wolf" programmer stereotype is outdated and harmful. Pair programming reduces the "truck factor,"
catches dead-end approaches early, and spreads knowledge. Testers aren't the enemy — Margaret the secretary found
every bug in moments. Code reviews should share knowledge, not judge. And "starting from yes" instead of "no"
transforms technical leadership. Include the 40% effectiveness increase stat from pair programming research.

  ---
13. "Your Build Is Code Too: Stop Treating It Like a Second-Class Citizen"

Source essays: Own (and Refactor) the Build, Keep the Build Clean, One Binary, Deploy Early and Often, Automate Your
Coding Standard, Let Your Project Speak for Itself

Angle: Build scripts are code. They rot, they have bugs, they need refactoring. "Don't touch the build" is as
dangerous as "don't touch that legacy code." Build one binary, promote it through environments — stop recompiling
per-environment. Zero-tolerance for build warnings (they're like broken windows). Deploy from day one, not the last
week. Your project should have a "voice" (extreme feedback devices — lamps, rocket launchers).

  ---
14. "You Are Not Your User: Bridging the Developer-Human Gap"

Source essays: Ask What Would the User Do, Your Customers Do Not Mean What They Say, Prevent Errors, Read the
Humanities, Install Me

Angle: The false consensus bias: you assume users think like you. They don't. Watch real users — they narrow focus
when stuck, muddle through, and what they say they want differs from what they do. Customers speak "customer speak"
not "developer speak." Reading Wittgenstein and Heidegger (yes, really) helps you understand how people experience
tools. And your install process is the first thing customers see — make it frictionless or they'll never see your
code.

  ---
15. "What Professionalism Actually Means in Software"

Source essays: The Professional Programmer, Code Is Design, Learn to Estimate, Know Your Next Commit, Don't Touch That
Code!

Angle: Uncle Bob's surgical analogy — would you want your heart surgeon to "fix it later"? Professionals own their
career, their code quality, and their estimates. They know the difference between estimates, targets, and commitments
(most managers don't). They know their next commit — what they'll change, which files, and when they'll finish. And
they respect environment boundaries: never touch production.

  ---
16. "Performance: Where Developer Intuition Goes to Die"

Source essays: Know Your Limits, Use the Right Algorithm and Data Structure, Interprocess Communication Affects
Application Response Time, The Road to Performance Is Littered with Dirty Code Bombs, Floating-Point Numbers Aren't
Real

Angle: The bank whose tellers waited because of a strlen() in a for loop (turning O(n) into O(n^2)). The memory
hierarchy table (register: <1ns, disk: 10ms — 7 orders of magnitude). Dirty code makes performance tuning impossible
because dependencies cascade. IPC latency dominates modern app performance, not algorithms. And floating-point math
lies to you (2147483647 stored as float becomes 2147483648).

  ---
17. "Object-Oriented Design: The Patterns You're Getting Wrong"

Source essays: Resist the Temptation of the Singleton Pattern, Missing Opportunities for Polymorphism, Encapsulate
Behavior Not Just State, The Single Responsibility Principle, Thinking in States

Angle: Singletons are seductive and almost always harmful (they prevent testing, hide dependencies, break
multithreading). if-then-else chains are often missed polymorphism opportunities. Classes with only getters/setters
have missed the point of OO — encapsulate behavior, not just state. SRP means one reason to change, not one method.
And modeling explicit states (In Progress -> Paid -> Shipped) eliminates entire categories of bugs.

  ---
18. "Speak Your Customer's Language: Domain-Driven Code That Reads Like English"

Source essays: Code in the Language of the Domain, Domain-Specific Languages, Prefer Domain-Specific Types to
Primitive Types, Learn Foreign Languages

Angle: trader.canView(portfolio) vs portfolioIdsByTraderId.get(trader.getId()).containsKey(portfolio.getId()). Same
logic, worlds apart in readability. Using domain types (Velocity_In_Knots instead of float) caught the Mars Orbiter
bug — in Ada at least. DSLs let domain experts read and even write business logic. And learning human languages (not
just programming ones) sharpens the abstract thinking programming requires.

  ---
19. "Making the Invisible Visible: How Transparency Saves Projects"

Source essays: Make the Invisible More Visible, Don't Rely on "Magic Happens Here", The Linker Is Not a Magical
Program, How to Use a Bug Tracker, Put Everything Under Version Control

Angle: Software is "mostly invisible." When your project is "90% done" for 3 months, the invisibility is the problem,
not the schedule. "Magic Happens Here" is dangerous — that department that "ran itself" until the project manager was
removed, then collapsed within 6 months. Linkers aren't magical, builds aren't magical, deployments aren't magical.
Make progress concrete: bulletin boards, unit tests, version control, incremental delivery. What you can see, you can
manage.

  ---
20. "Becoming a Better Developer Without Writing a Single Line of Code"

Source essays: Continuous Learning, Read Code, Fulfill Your Ambitions with Open Source, Read the Humanities, Learn to
Say "Hello World"

Angle: The most underrated developer skill is reading code, not writing it. Study open source projects. Read the
humanities (Wittgenstein on communication, Heidegger on tools, Rosch on categorization). Contribute to open source for
the technology you're passionate about, not just what your employer uses. And sometimes, the most productive thing
you can do is write a tiny tryit.c — Hoppy's "Hello World" approach to understanding.

  ---
Summary Table

┌─────┬─────────────────────────────────────┬────────────────────┬──────────────────────────────┐
│  #  │           Blog Post Title           │ # of Source Essays │          Core Theme          │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 1   │ The Myth of the 10x Developer       │ 4                  │ Work smarter, not harder     │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 2   │ Code Is a Love Letter to the Future │ 5                  │ Code as communication        │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 3   │ When Comments Lie                   │ 4                  │ Expressive code > comments   │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 4   │ The Art of Subtraction              │ 4                  │ Less code = better software  │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 5   │ Technical Debt Is Compound Interest │ 4                  │ Managing debt strategically  │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 6   │ The Dark Side of DRY                │ 4                  │ Reuse vs. coupling tradeoffs │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 7   │ A Philosophy of Failure             │ 5                  │ Error handling done right    │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 8   │ APIs People Actually Love           │ 4                  │ Interface design             │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 9   │ Master Your Tools                   │ 5                  │ Deep tool proficiency        │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 10  │ Tests That Prove Something          │ 5                  │ Testing philosophy           │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 11  │ Learn a Language You'll Never Use   │ 4                  │ Cross-paradigm growth        │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 12  │ Programming Is a Team Sport         │ 6                  │ Collaboration & pairing      │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 13  │ Your Build Is Code Too              │ 6                  │ Build/deploy infrastructure  │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 14  │ You Are Not Your User               │ 5                  │ User empathy                 │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 15  │ What Professionalism Means          │ 5                  │ Developer mindset            │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 16  │ Performance: Intuition Dies Here    │ 5                  │ Data-driven optimization     │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 17  │ OO Patterns You're Getting Wrong    │ 5                  │ Design principles            │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 18  │ Speak Your Customer's Language      │ 4                  │ Domain-driven code           │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 19  │ Making the Invisible Visible        │ 5                  │ Transparency & visibility    │
├─────┼─────────────────────────────────────┼────────────────────┼──────────────────────────────┤
│ 20  │ Better Dev Without Writing Code     │ 5                  │ Learning & reading           │
└─────┴─────────────────────────────────────┴────────────────────┴──────────────────────────────┘

This covers all 97 essays across 20 posts. Each post is a synthesis of 4-6 related chapters with a unique narrative
angle — not a copy of the book, but your own take inspired by its ideas. The strongest ones for learning and
engagement would be #6 (DRY controversy), #7 (error philosophy), #16 (performance intuition), and #1 (10x myth) — they
all challenge conventional wisdom.
