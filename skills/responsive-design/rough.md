# Introduction

Responsive design is an approach to designing interfaces that adapt to the size of the screen they are being viewed on. This means that an app will look good and be easy to use no matter what device it is being viewed on, whether it is a desktop computer, laptop, tablet, or smartphone.

Responsive design is achieved by using a combination of media queries and fluid grids. Media queries allow you to specify different CSS styles for different screen sizes, while fluid grids allow you to create layouts that automatically adjust to the size of the screen.

There are a number of benefits to building with a responsive approach. These include:

    Better user experience: Responsive design ensures that our app is easy to use and navigate on any device. This can improve the user experience and lead to increased engagement.

    Reduced development costs: By adopting a responsive-first approach, we can eliminate the need to create separate versions of our app for each device or breakpoint. This strategy will save us time and money in the long run.

    Future readiness: As technology continues to evolve and new devices with diverse screen sizes and resolutions emerge, responsive design offers a flexible and future-proof solution. It seamlessly adapts to changing user behaviors and preferences, ensuring that your website remains relevant and usable across a wide range of devices, without necessitating significant redesigns or redevelopment.



# Responsive Design Layouts

With interfaces, we mainly focus on three broad categories of responsive layouts. These patterns, reiterated in a Google Developers post, give designers a head start when deciding how and why to set elements on a page. Remember that there is no “ultimate responsive layout.” Each of these options has its own merits, and the best choice depends on the type of screen you are designing for.

    Column Drop Layout

    Fluid Design Layout

    Off Canvas Pattern

There is no one-design-fits-all solution that covers every situation. While exceptions to these patterns may exist, they cover most situations that occur across the use-cases. Designers can choose from these three patterns to give their layouts direction before they design—a real head start in their work.

### 01. Column Drop Layout
Another popular pattern starts with a multi-column layout and ends up with a single column layout, dropping columns along the way as screen sizes get narrower. Unlike the Mostly Fluid pattern, the overall size of elements in this layout tend to stay consistent. Adapting to various screen sizes instead relies on stacking columns (illustrated below).


When and how is each column is stacked at different resolution breakpoints differs for each design, but generally either navigation or content is placed at the top of narrow screens.

### 02. Fluid Design Layout
The most popular pattern was perhaps surprisingly simple: a multi-column layout that introduces larger margins on big screens, relies on fluid grids and images to scale from large screens down to small screen sizes, and stacks columns vertically in its narrowest incarnations (illustrated below).

"Mostly fluid" because the core structure of the layout really doesn't change until the smallest screen width. Instead, the design mostly relies on fluid grids to adapt to a variety of screen sizes.

### 03. Off Canvas Pattern
This approach should only be used for global patterns like Sidebar and Context bar.

The general idea behind the Off Canvas pattern is that on small screens, additional elements are a click away, and as screen size expands, they become visible until no clicks are required to access these components.

---


# Responsive Design Workflow
Achieving the “Responsive" goal requires us to hone our thinking into a flexible mindset. We need to reassess what we think we know, to design for small devices first, to test on real devices, and to realize that our designs are not fixed, but fluid.

Below are some key design process optimizations, best practices, and references that we can apply to provide a high-quality responsive user experience.

- Setting Up Breakpoints: The most popular pattern was perhaps surprisingly simple: a multi-column layout that introduces larger margins on big screens, relies on fluid grids and images to scale from large screens down to small screen sizes, and stacks columns vertically in its narrowest incarnations.
- Content-First Workbench
- Visual Hierarchy
- Designer’s Checklist