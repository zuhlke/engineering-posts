---
title: Limit your code line lenth
subtitle: Why we should still enforce limits to the length of lines in 2023
domain: campzulu.hashnode.dev
tags: development, code-style, lint,
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1673264332496/a9-qPZTKf.jpg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=webp
hideFromHashnodeCommunity: false
publishAs: culas
---

"Everyone has widescreen monitors nowadays."
This is what I often hear when bringing up enforcing a maximum line length in our code.
And with that, the discussion is often dismissed and done.
I argue this is a fairly ignorant stance and misses the point completely.
Let me explain why we should at least discuss a sensible limit in 2023.

# Readability

We developers are often focused on _writing_ code.
In reality we are mostly _reading_ it.
Whether it's our code from before the coffee break, the pull request from a colleague, some dusty corners in the legacy codebase or simply a StackOverflow answer.
Each line of code is written once but read many times.
Code that is easy to read is essential, which is partially why tools like Prettier and linters exist in the first place.

Shorter lines of code make it easier to scan through and understand the code without having to scroll horizontally or mentally parse long lines.
Ever jumped to the wrong next line after finishing reading a really long one?
Even with wide screen websites limit the width of text too.
Based on an [article by Edward Scott of the Baymard Institute](https://baymard.com/blog/line-length-readability) the optimal line length for body text is between 50 and 75 characters.
Now code is not exactly like English prose.
With additional syntactic symbols and a larger word length of variables and functions we can argue that the optimum is higher for code.
The [PEP 8 style guide for Python](https://peps.python.org/pep-0008/#maximum-line-length) recommends a limit of 79 characters, Google sets it at [80 for JavaScript](https://google.github.io/styleguide/jsguide.html#formatting-column-limit) and [100 for Java](https://google.github.io/styleguide/javaguide.html#s4.4-column-limit).
The ideal is probably somewhere in that range of 80 to 100.

# Maintainability

Debugging is a necessary part of software development.
When code is written with shorter lines, it becomes easier to debug as it is much easier to identify the exact line that may be causing issues.
Otherwise it can be hard to pinpoint the exact location of the error even with the line numbers in the stacktrace.
I find it much easier to set breakpoints and step through the program, when multiple statements and function calls are put on different lines.
IDEs help with it but they can't take the entire mental overhead from you.

# Collaboration

In any non-trivial software project, collaboration among the team of developers is key to success.
Effective communication is critical, and techniques like pair programming and code reviews are essential in facilitating communication.
When shorter lines of code are used, it's easier to refer to specific locations in the code during these collaborative activities.

Statistically speaking the chances of merge conflits should decrease when the code is split into more lines.
In my experience diffing tools manage changes on different lines much better than if they were on the same line.

# Accessibility

Accessability is an important consideration in software development not just when we talk about end users.
Shorter lines are more accessible to people with visual impairments.
You might sit comfortably in front of your 4K wide screen with font size 14, ignoring line lengths.
But accessibility is about all of us, whether permanently or temporarily impaired.
Your own long lines might come back to you to haunt you: while fixing a bug on the notebook during a train ride, after putting on presentation mode for a talk, while trying to resolve a merge conflict in split view.

Many developers, including myself, find it beneficial to have multiple files open next to each other.
This allows for seamless navigation between files and aids in working on multiple aspects of a project simultaneously.
For instance, editing HTML side by side with the associated CSS file allows the developer to align changes in both more easily.
Similarly, when working on related files with code that is dependent on each other, having them open next to each other can make it easier to understand the relationships between them.
By having multiple files open next to each other, the famed wide screens are used more efficiently, instead of allowing line for lazy developers.

# Code Quality

I argue that limiting the line length results in better code quality.
We should consider long lines as a code smell.
Maybe that line is doing too many different things.
In which case the readability is impacted, as the mental overhead for each line is larger.
If the line isn't shorter because of long variable and function names, you might question whether _they_ do too much.
Furthermore, multiple levels of indentations push the limits sooner which is good as you should limit those too.

# Conclusion

<!-- TODO: arguments wrapping of editors is no alternative -->
