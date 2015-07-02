---
layout: post
title:  "A Little Style for Our Stylesheets"
author: Laure Fischer
github_username: LaureMFischer
date:   2015-06-29 20:00:00
---

I've written previously on this blog about [CSS best practices](http://americastestkitchen.github.io/css/2014/10/05/css-debates/), and in the eight months since that post, our team has continued to churn out new CSS like there's no tomorrow. At the beginning of this year, we embarked on a massive redesign of one of our sites, and that meant new color schemes, new typography, and, probably most notably, responsive layouts. Given that this also meant we would be able to write brand new stylesheets for essentially brand new pages, and that we would (eventually) be able to scrap most of our old styles, this redesign seemed like the perfect opportunity to set some standards for our CSS going forward in the form of a CSS style guide.

As it turns out, developing a first draft of the style guide was the easy part. There are lots of great examples out there (I give credit to [this post](https://medium.com/@fat/mediums-css-is-actually-pretty-fucking-good-b8e2a6c78b06) for being my personal inspiration for this whole endeavor), and after writing CSS for a certain amount of time, a lot of the pain points become clear, so it didn't take long for us to come up with a list of rules that we wanted to follow.

For a little more context, the redesigned pages were built using React, and we are essentially maintaining a React app within our Rails app in order to segregate these new pages from our legacy views. Since all of our new React code and the associated styles were housed in their own `/react` directory, we were able to use the structure of our React components to drive the organization of our new stylesheets, without having to worry about reorganizing our legacy stylesheets. With the new pages being fully responsive, we got to use some pretty nice gems (`susy` for our grid layout, `breakpoint` for clean media queries), and we also were able to use our different breakpoints to inform stylesheet structure.

With that, I give you our style guide (a working draft, I'll call it).

---

## File Structure
- `main.scss` imports all of our partials and mixins, which are organized into logical blocks with comments.
- Folders:
    - Feature folder with subfolders for breakpoints (e.g. `search/desktop` `search/mobile` `search/tablet`)
    - Mixins folder (includes any polyfills)
    - Layouts folder (e.g. for a `_grid.scss` file)
    - Vendors folder (e.g. for a `_normalize.scss` file)

## SCSS Implementable Rules

### Naming
- Prepend ids and classes being bound by JavaScript with `js-`.
- Prepend ids and classes being bound by React with `react-`.
- Prepend state-based css classes with `is-`.
- Prepend newly created mixins with `m-`.
- Use BEM (block, element, modifier) naming conventions for classes whenever possible.
    - E.g.: `gallery__image--large`

### Selectors
- To avoid issues with specificity, never use an id as the selector for styling an element.
    - We have made a few exceptions to this rule in order to prevent styles for new features from conflicting with legacy styles, with the intention of removing references to ids in stylesheets once legacy styles are sunsetted.
- Use the minimum number of selectors you need to target an element for styling.
- To avoid adding complexity, try to avoid nesting selectors when possible.
    - A good exception to this rule would be the use of pseudo selectors in the vein of `nth-child` and `nth-of-type`.

### Variables
- Make use of our `z-index` scale file. Here's what it looks like:
{% gist be26dedc217a3e08f8c1 %}
- Use our own line-height, font-weight, and letter-spacing scale files.
- Color variables and font variables should be declared in separate files and referenced whenever possible.

### General Style
- Comma separated selectors should each be declared on a new line.
- Include exactly one line break after each selector block.
- Use single quotes for URLs.
- Use the font shorthand when possible (`font: 14px $proximanova $black`).
- `@include`s (for breakpoints) should be declared at the top of a given breakpoint-specific stylesheet, and should wrap all breakpoint-specific styles.
- Always use `rgb` and `rgba` for colors, to avoid mixing hex codes and `rgba` for instances where transparency is needed.

### Grid
- Elements should always be on the established grid (the assumption being that the design team will always place elements in the context of a grid going forward).
- Horizontal padding and margins should be of some factor of our established gutter widths.

### Breakpoints
- Place breakpoint declarations inside a new file and folder `BREAKPOINT_NAME/component_name`, per the file structure above (e.g. `tablet/_load_more_button.scss`).
- As much as possible, avoid duplication of shared styles across different breakpoints.

---

## Lessons Learned

The hard part, maybe not surprisingly, has been actually implementing the rules we prescribed above. So in addition to sharing our style guide, I also wanted to share a few things I learned in implementing it, which will hopefully help anyone out there who is thinking about implementing a CSS style guide of their own.

### Be vigilant

I've learned the hard way the importance of getting CSS code into as good a state as possible before calling a feature "done". We've all said to ourselves "I'm just going to get this to look right, and then I'll come back and refactor later." _Except that you won't. Because **refactoring CSS is terrible**._ Class names tend to be quite "sticky" and resistent to refactoring given the way they get reused, and after giving in to a suboptimal solution in order to fix a bug quickly, changing things can feel precarious. This is why it's worth spending a little extra time up front to make sure you are following your own rules (harder to do than I thought). The same goes for reviewing code from other team members -- it's worth being a little nitpicky on the initial pull request for the sake of everyone's future sanity.

### Make peace with your legacy styles

Despite my idealistic visions of creating perfect, beautiful CSS without the weighty baggage of our legacy styles dragging us down, the reality of the situation was a little different. Even though we were building new pages, we were going to be slowly rolling them out to users, while still maintaining many of the existing pages (and thus stylesheets) for quite some time. Our CSS could still be beautiful, but we needed a strategy to combat legacy style conflicts. Our approach for this has evolved over time, from commenting any lines of code that were written to handle legacy style issues, to temporarily scoping new styles under top-level id selectors (and hating ourselves for it), to creating a `resets` directory to house only styles that were needed to reconcile conflicts between old and new styles. None of these strategies have been ideal, but the main point here is that, even when I thought we had found the perfect time to implement a CSS style guide, with minimal refactoring of legacy code required, it still had (and continues to have) its challenges. So if you're waiting for that perfect time to implement your own style guide, the good news is you don't have to wait anymore! Right now is probably as good a time as any to start.

### Keep iterating

My final takeaway from this process is the importance of regularly reevaluating the utility of a style guide's rules. After all, such a guide is meant to make our lives as programmers easier, in addition to improving our code quality. As we started using our guide, we thought of things we needed to add (a `vendors` directory), exceptions we needed to make (when nesting is okay) and rules we needed to clarify (why are we using this font scale thing again?). Even as I write this, I'm thinking I should ask the team again if they have more ideas for updates. So as much as rules are made to be followed (that's the saying, right?), since they are going to be internal rules for our team, I think there is always room to revisit and iterate. And in the process, I think we've made our stylesheets more readable, more logical, and quite a bit more bearable to work with.



