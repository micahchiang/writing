Quality Assurance is a responsibility that all members of the Design System Team are beholden to and share in. The process isn't one that begins when code is written and a pull request is submitted, rather one that starts with the inception of design. With that, this RFC proposes a multipart QA process beginning with design work and ending when a pull request is merged for new features, and a sub workflow for iterative work done on existing components and design system code.

### New Features / Components

#### Step 1: Design Specification QA

Engineers working on new components or features rely on design specifications that provide accurate information and maintain consistency with existing standards. Design specs should address the following questions:

1. Does the new component contain text? If so, does it adhere to the [http://VA.gov](http://VA.gov) [content style guide](https://design.va.gov/content-style-guide/)? If not, has the reason for divergence been documented somewhere?
    
2. Does the new component adhere to the guidance on [creating components in Figma?](https://design.va.gov/about/designers/creating-components)
    
3. Have the designs for the new component been created with accessibility in mind? Do the colors used maintain proper contrast?
    
4. How many variations of the new component are there? Does each variation have a corresponding design spec?
    
5. Are there design specs for mobile and desktop screen sizes?
    
6. Is there a list of all design tokens used in a spec that developers can easily reference?
    

When all of the above questions have been addressed, a link to the mockups of the component should be added to the corresponding engineering ticket so developers can reference them while working.

Collaboration between design and engineering should be an ongoing conversation. Engineers should rely on the mockups provided by a designer and should seek clarification on anything that isn’t clear while the component / feature is being built. Ideally, when a pull request is submitted, design review shouldn’t surface any major discrepancies because those have been addressed through consistent communication during construction.

#### Step 2: Engineering QA

An engineer is responsible for ensuring their work aligns with the design specification provided to them on the ticket they are working on. In completing required work an engineer should be answering these questions:

1. Does the new component maintain 1:1 parity with the provided design specification? Are all variations provided in the mocks accounted for?
    
2. Is text content consistent with what's been provided in the mockup?
    
3. Does the new component behave as expected across all breakpoints?
    
4. Has an accessibility expert been consulted throughout the development process when questions about accessibility arise?
    
5. Do tab order and focus state flow and work as expected?
    
6. How do screen readers handle the new component? Engineers should do what they can to test their code themselves. Usually this means VoiceOver and Safari/Chrome.
    
7. Is the new component covered by end to end tests?
    
8. **Does the component function as expected in vets-website?** It isn't enough for our components to work in Storybook or through Chromatic since they are used most meaningfully in vets-website. During local development, engineers should link their local copy of component library to their local copy of vets-website and verify changes work there, using [this guide](https://vfs.atlassian.net/wiki/spaces/DST/pages/1929379847/Component+library#How-to-Link-component-library-and-vets-website-repos).
    

#### Step 3: Engineering Code Review

Code review is a way we as a team build trust, hold each other accountable to excellence and quality, and ensure we are putting forth the best work possible so our customers can use our product with a high degree of confidence.

Code reviews are **not** about being right or pointing out where someone is wrong. They are about cultivating healthy and constructive conversations, ensuring standards are respected and upheld, and learning from one another. Furthermore, pull requests in Github serve as a written history for our work, and conversations associated with them allow people who come after us the benefit of additional context.

When an engineer submits a pull request they should be including all of these things:

1. Screenshots that show the component at different breakpoints which cover the design spec
    
2. A summary of how things were tested. This includes accessibility testing with a screen reader, keyboard navigation, etc. This may include a screenshot verifying that aXe has been run against the current component and no issues were surfaced as a result.
    
3. The name of the vets-website branch where this was tested so reviewers can clone and verify.
    
4. A descriptive title for release notes
    
5. Correct label assigned (patch, minor, major, ignore-for-release)
    
6. A link to the original Github issue it closes.
    
7. A link to the design spec
    
8. A Chromatic Link that contains the component updates
    

**Note**: An accessibility expert should be tagged on every pull request that includes a new component, and every pull request where an existing component has undergone changes that may impact accessibility. Accessibility review and sign off are required before a pull request is merged in. This looks like our own internal review, similar to what we do with the collaboration cycle and staging review.

An engineer reviewing a pull request should verify expected functionality and accurate design implementation by asking checking these things:

1. Does the PR description include screenshots at different breakpoints? Do those screenshots accurately reflect the design spec associated with the work?
    
2. Do things work as expected in Chromatic?
    
3. Do things work as expected in vets-website?
    
    - Provided engineers have linked their local instances of component library and vets-website using [this guide](https://vfs.atlassian.net/wiki/spaces/DST/pages/1929379847/Component+library#How-to-Link-component-library-and-vets-website-repos), reviewing engineers should check component functionality in vets-website.
        
    - If the implementation engineer has done this themselves, then they should include their branch name on the pull request so reviewers can clone the work down to check it out. If the implementing engineer hasn't done this yet, reviewers should request changes on the PR.
        
4. Has an accessibility expert signed off on the work being submitted?
    
5. Do tests cover all functionality?
    
6. Has documentation been updated properly?
    

In addition to answering the above list, engage with the pull request! See something you're impressed by? Leave an encouraging comment about it. Something you don't understand? It's okay to say as much! Ask for an explanation, it's a learning opportunity for you, and a teaching opportunity for the other person. There's no such thing as a dumb question, and if you have the courage to ask it, chances are good someone else had the same question in mind.

### Iterating on Existing Components

Quality assurance on existing component iteration follows a similar template to the one laid out above with one important addition:

**Decisions on iteration have to be documented somewhere. Why is this change being made? Who or what governing body approved the change? How was consensus reached?**

Much of our current tech debt is tied to differing opinions on how things should be done. Our feature iteration should include evidence of a conclusive decision we can point to when an issue is submitted that counteracts or undoes that iteration.

Coupled with the above, pull requests should include many of the same things listed in the requirements for new components: Links to updated design specifications, a summary of testing, and sign off from an accessibility expert, etc.

Testing in vets-website may not be necessary, depending on the level of iteration: Is a PR just a minor refactor? A small update to existing logic? Testing in vets-website is probably not necessary for these things. Conversely, is a pull request introducing breaking changes? Those may need to be tested in vets-website to ensure existing usages aren't impacted.
