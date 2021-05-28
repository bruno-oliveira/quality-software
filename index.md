## Writing quality code

There are endless books and blog posts, articles and videos of lots of respected people in the industry telling you all about code quality and what it takes to write good code.

While all of these are extremely useful in its own right, I feel that sometimes it's all too much on the realm of the theoretical, it's all about arbitrarily large projects, customers demanding features that seem to be a perfect fit to illustrate certain design patterns or refactoring approaches. 

This is rarely true in the real world, where good code is tightly linked to good practices within a development team, besides the single individual refactoring the code to patterns.

I will present what I've gathered after around ~5 years of professional software development on several levels. There won't be a specific order to any of the sections and they can all be read independently of one another. This will be focused mostly on the standard Java, Spring Boot, Docker stack. I will try to otherwise keep all of these as generic as possible, to be useful for the widest possible audience.

### Process and company culture 

Process and company culture can affect code quality much more than any single individual contributions. There is a law, called Conway's law, that states that organizations design systems and structure their code in a way that mirrors their own organizational structure, so, this immediately implies that the culture and company that you work for, will affect the way technical problems are handled by you, as an active part of such culture.

It can be very hard to change such a culture from the inside-out, and, sometimes, it may even be impossible, so, it's important to be aware that there is only so much you can do as an individual.

A good way to approach this is to tap on your previous experiences and knowledge, and, trying to leverage it near where there are higher chances it will be listened to and acted upon: near senior developers who can be naturally more in touch with managers, so, your own ideas can ripple up in the organizational structure by first being "seeded" near where they can have the most impact: people who write code and are very aware of current limitations, but, at the same time, can exert some pressure near management. This is typically senior developers who know the company business well, have spent countless hours on the codebase, and, can have influence over management to slightly tip the scales towards better approaches.

### Write maintainable code

TODO add intro

#### Keep classes and methods short

TODO

#### Leverage your tools: using functional programming in Java

TODO

#### Naming is important

TODO

#### Codebase-wide consistency 

TODO

### Testing for modern web apps

TODO

#### Docker

TODO

#### Testcontainers

TODO

#### Meaningful test data

TODO

### Importance of CI/CD

TODO

### Closing remarks

TODO



### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/bruno-oliveira/quality-software/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
