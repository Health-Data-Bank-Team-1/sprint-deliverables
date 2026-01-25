### Client Requests
- voice to text
- text to speech
- potentially have images in future, accessibility considerations for those
- simple point and click UI

### Web Content Accessibility Guidelines Four Principles
1. **Perceivable**: Information and user interface components must be presentable to users in ways they can perceive.
    - This means that users must be able to perceive the information being presented (it can't be invisible to all of their senses)
2. **Operable**: User interface components and navigation must be operable.
    - This means that users must be able to operate the interface (the interface cannot require interaction that a user cannot perform)
3. **Understandable**: Information and the operation of user interface must be understandable.
    - This means that users must be able to understand the information as well as the operation of the user interface (the content or operation cannot be beyond their understanding)
4. **Robust**: Content must be robust enough that it can be interpreted reliably by a wide variety of user agents, including assistive technologies.
    - This means that users must be able to access the content as technologies advance (as technologies and user agents evolve, the content should remain accessible)

### Perceivable Information and User Interface
- **Text alternatives for non-text content**: text alternatives convey the purpose of an image or function to provide an equivalent experience.
- **Examples**:
    - short equivalents for images, including icons, buttons, and graphics
    - description of data represented on charts, diagrams, and illustrations
    - brief descriptions of non-text content such as audio and video files
    - labels for form controls, input, and other user interface components
- **More Info and Examples**: [https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Understanding_WCAG/Text_labels_and_names](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Understanding_WCAG/Text_labels_and_names)
- **Captions and other alternatives for multimedia**: people who cannot hear audio or see video need alternatives.
- **Examples**:
    - text transcripts and captions for audio content, such as recordings of a radio interview
    - audio descriptions, which are narrations to describe important visual details in a video
    - sign language interpretation of audio content, including relevant auditory experiences
- **Content can be presented in different ways**: for users to be able to change the presentation of content, it is necessary that:
    - headings, lists, tables, input fields, and content structures are marked-up properly
    - sequences of information or instructions are independent of any presentation
    - browsers and assistive technologies provide settings to customize the presentation
- **Content is easy to easier to see and hear**: distinguishable content is easier to see and hear. Such content includes:
    - color is not used as the only way of conveying information or identifying content
    - default foreground and background color combinations provide sufficient contrast
    - when users resize text up to 400% or change text spacing, no information is lost
    - text reflows in small windows (“viewports”) and when users make the text larger
    - images of text are resizable, replaced with actual text, or avoided where possible
    - users can pause, stop, or adjust the volume of audio that is played on a website
    - background audio is low or can be turned off, to avoid interference or distraction

### Operable User Interface and Navigation
- **Functionality is available from a keyboard**: keyboard access to all functionality including form controls, input, and other user interface components. Keyboard accessibility includes:
    - all functionality that is available by mouse is also available by keyboard
    - keyboard focus does not get trapped in any part of the content
    - web browsers, authoring tools, and other tools provide keyboard support
- **More Info**: [https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Understanding_WCAG/Keyboard](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Understanding_WCAG/Keyboard)
- **Users can easily navigate, find content, and determine where they are**: well organized content helps users to orient themselves and to navigate effectively. Such content includes:
    - pages have clear titles and are organized using descriptive section headings
    - there is more than one way to find relevant pages within a set of web pages
    - users are informed about their current location within a set of related pages
    - there are ways to bypass blocks of content that are repeated o multiple pages
    - the keyboard focus is visible, and the focus order follows a meaningful sequence
    - the purpose of a link is evident, ideally even when the link is viewed on its own
- **Users can use different input modalities beyond keyboard**: input modalities beyond keyboard, such as touch activation, voice recognition, and gestures make content easier to use for many people. Particular design considerations maximize the benefit of these input modalities. This includes:
    - gestures that require dexterity or fine movement have alternatives that do not require high dexterity
    - components are designed to avoid accidental activation, for example providing undo functionality
    - labels presented to users match corresponding object names in the code, to support activation by voice
    - functionality that is activated by movement can also be activated through user interface components
    - buttons, links, and other active components are large enough to make them easier to activate by touch

### Understandable Information and User Interface
- **Text is readable and understandable**: ensure that text content is readable and understandable to the broadest audience possible, including when it is read aloud by text-to-speech. Such content includes:
    - identifying the primary language of a web page
    - identifying the language of text passages, phrases, or other parts of a web page
    - providing definitions for unusual words, phrases, idioms, and abbreviations
    - using the clearest and simplest language possible, or providing simplified versions
- **Content appears and operates in predictable ways**: many people rely on predictable user interfaces and are disoriented or distracted by inconsistent appearance or behavior. Examples of making content more predictable include:
    - navigation mechanisms that are repeated on multiple pages appear in the same place each time
    - user interface components tat are repeated on web pages have the same labels each time
    - significant changes on a web page do not happen without the consent of the user
- **Users are helped to avoid and correct mistakes**: forms and other information can be confusing or difficult to use for many people, and, as a result, they may be more likely to make mistakes. Examples of helping users to avoid and correct mistakes include:
    - descriptive instructions, error messages, and suggestions for correction
    - context-sensitive help for more complex functionality and interaction
    - opportunity to review, correct, or reverse submissions if necessary

### Robust Content and Reliable Interpretation
- **Content is compatible with current and future user tools**: content is compatible with different browsers, assistive technologies, and other user agents. Examples of how this can be achieved include:
    - ensuring markup can be reliably interpreted
    - providing a name, role, and value for non-standard user interface components

### Forms
- **Labeling Groups**: use the `<label>` element, and, in specific cases, other mechanisms (e.g. WAI-ARIA, `title` attribute etc.), to identify each form control
- **Grouping Controls**: use the `<fieldset>` and `<legend>` elements to group and associate related form controls
- **Form Instructions**: provide instructions to help users understand how to complete the form an individual form controls
- **Validating Input**: validate input provided by the user and provide options to undo changes and confirm data entry
- **User Notifications**: notify users about successful task completion, any errors, and provide instructions to help them correct mistakes
- **Multi-Page Forms**: divide long forms into multiple smaller forms that constitute a series of logical steps or stages and inform users about their progress
- **Custom Controls**: use stylized form elements and other progressive enhancement techniques to provide custom controls

### Additional Resources
- [Accessibility Tools](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/Tooling)
- [Accessible HTML](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/HTML)
- [WAI-ARIA Basics](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility/WAI-ARIA_basics)
- [ARIA Guides](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Guides)
- [Mobile Accessibility Checklist](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Mobile_accessibility_checklist)

### References
- [Web Content Accessibility Guidelines](https://www.w3.org/TR/WCAG21/)
- [Web Accessibility Initiative Forms Tutorial](https://www.w3.org/WAI/tutorials/forms/)
