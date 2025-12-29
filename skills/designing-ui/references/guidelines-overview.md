# Human Interface Guidelines: Complete Index

This document provides a complete index of all 100 Human Interface Guidelines from Sociomedia. Guidelines are organized by category for easy reference.

## Contents
- [Simplification & Ease of Use](#simplification--ease-of-use)
- [Cognitive Principles](#cognitive-principles)
- [User Communication](#user-communication)
- [Design Optimization](#design-optimization)
- [Functionality](#functionality)
- [Object-Based Design](#object-based-design)
- [Operation Patterns](#operation-patterns)
- [Automation & Positioning](#automation--positioning)
- [Navigation & Icons](#navigation--icons)
- [Data & Interaction](#data--interaction)
- [Input Design](#input-design)
- [UI States & Input](#ui-states--input)
- [Preview & Error Handling](#preview--error-handling)
- [Usage Focus](#usage-focus)
- [Navigation & Progress](#navigation--progress)
- [Transitions & Feedback](#transitions--feedback)
- [Progressive Disclosure](#progressive-disclosure)
- [Localization](#localization)
- [Conventions](#conventions)
- [Spatial & Memory](#spatial--memory)
- [Touch & Mobile](#touch--mobile)
- [Design Principles](#design-principles)
- [Interaction Quality](#interaction-quality)
- [User Experience](#user-experience)
- [Design Philosophy](#design-philosophy)

---

## Simplification & Ease of Use

### 1. Simplify
Reduce features and information to essentials. Minimize interface elements.

### 2. Make it Easy
Streamline usage and minimize steps required to achieve goals. Reduce cognitive effort.

## Cognitive Principles

### 3. Mental Model
Design structures and movements that match users' imagined usage model. Use metaphors to convey concepts.

### 4. Signifiers
Make interactive elements visible and self-explanatory. Use familiar expressions and intuitive shapes.

### 5. Mapping
Clarify relationships between controls and their effects. Use positioning, shape, color, symbols as visual cues.

### 6. Consistency
Apply consistent rules in color, shape, layout, and behavior across the interface.

### 7. User Control
Let users control the system. Minimize forced operations. Allow users to choose based on their intentions.

### 8. Direct Manipulation
Create sensation of directly touching/manipulating on-screen objects with real-time feedback.

### 9. Modeless
Minimize modes in the interface. Allow free-order task completion. Avoid context-dependent operation meanings.

### 10. Visual Gestalt
Utilize perception's tendency to recognize whole patterns. Use proximity, similarity, and closure in layout.

## User Communication

### 11. Use User Language
Use common user expressions. Avoid technical or internal jargon.

### 12. Don't Rely on User Memory
Provide necessary information at point of need. Don't assume users remember previous details.

### 13. Constraints
Intentionally limit user actions to prevent errors. Guide towards effective usage.

## Design Optimization

### 14. Precomputation
Use pre-existing optimal values. Leverage proven solutions and presets.

### 15. Error Avoidance
Prevent errors before they occur. Distinguish confusing elements. Disable contextually meaningless operations.

### 16. Fitts's Law
Larger, closer targets are easier to point at. Smaller, distant targets are harder to interact with.

### 17. Hick's Law
Selection time increases with number of choices. More options mean more time to decide.

### 18. Complexity Conservation
Complexity can't be reduced, only shifted. Move complexity from user to system side.

### 19. Task Coherence
Users likely to repeat yesterday's tasks. Remember and predict user behavior patterns.

## Functionality

### 20. Optimize for Major Tasks
Prioritize features used by majority (80/20 rule). Focus on most common user needs.

### 21. Persuasion
Use motivational techniques to guide user behavior through recommendations, comparisons, simulations.

### 22. Provide Shortcuts
Offer faster methods for experienced users. Include keyboard shortcuts, bookmarks, gestures.

## Object-Based Design

### 23. Object-Based Design
Extract objects from user requirements. Allow direct interaction with key objects.

### 24. Views Represent Objects
Interface composed of object representation views. Objects can have multiple view representations.

### 25. Object State Visualization
Objects must visually and continuously embody their own state. Users interact to move toward goal state.

## Operation Patterns

### 26. Noun→Verb Operation Order
Users first select an object (noun), then choose an action (verb). Avoid verb-first patterns.

### 27. Graphical Tone & Manner Consistency
Align graphic elements' visual characteristics: color tone, saturation, gradients, borders, shadows, line styles.

### 28. Communicate Information, Not Just Data
Users want meaning, not raw numbers. Show disk space as percentage, not exact bytes.

## Automation & Positioning

### 29. Automate Single-Path Operations
If only one input method exists, system should automate it. Minimal user input indicates automation opportunity.

### 30. Position Tools Near Work Context
Place tools close to their processing target. Enable mode-less, immediate interaction with visible results.

### 31. Visual + Textual Description
Use icons to represent object nature. Use labels to clarify details. Combine graphical and numeric representations.

## Navigation & Icons

### 32. Present Preconditions Early
Show prerequisite conditions before complex procedures. Avoid wasting user effort by revealing requirements upfront.

### 33. Use Nouns for Navigation
Navigation items should be noun-based. Avoid verb-based labels that obscure content.

### 34. Nouns/Adjectives for Icons
Icons should represent objects or resulting states. Avoid depicting processes/actions in icons.

## Data & Interaction

### 35. Data Binding
Synchronize object states across multiple views. Enable real-time, bidirectional updates.

### 36. Zero-One-Infinity Principle
Avoid arbitrary numeric limitations. Allow 0, 1, or infinite instances of elements.

### 37. Meaningful Interactive Elements
Every interactive element must have task-relevant meaning. Disable or hide elements without purpose.

### 38. User-Owned Inputs
Save all user-entered values and settings. Allow users to modify and delete their inputs.

### 39. Prevent Inconsistent Operations
Avoid actions that could compromise information integrity. Don't request redundant or conflicting inputs.

## Input Design

### 40. Input Form Storytelling
Group input fields meaningfully. Order items from simple to complex.

### 41. Create Interaction Flow
Use visual grouping to guide user actions. Position action buttons to suggest navigation path.

### 42. Provide Good Defaults
Use high-probability default values. Consider risk, commonality, neutrality, current state.

### 43. Use Constrained Controls
Implement selection controls: radio buttons, dropdown menus, sliders, pickers.

### 44. Use Positive Language for Choices
Frame selection labels positively. Especially important for on/off controls.

### 45. Select Results, Not Values
Prefer selection over manual input. Show potential outcomes instead of parameter details.

### 46. Structured Input Fields
Divide input fields to match expected format. Suggest input, improve efficiency, reduce errors.

### 47. Specific Verbs for Default Buttons
Use action-specific verbs on primary buttons. Avoid generic "OK" or "Yes".

### 48. Input Control Distinctions
Radio buttons: Single selection with default. Checkboxes: Independent on/off or multi-select.

## UI States & Input

### 49. Flip-Flop Problem Avoidance
Separate state representation from labels to clarify button state changes.

### 50. User Input Flexibility
Reduce system-imposed input format constraints. Automatically handle input variations like half/full-width characters.

### 51. Input Suggestions
Provide selectable options while typing. Help users choose rather than manually entering data.

### 52. Scroll Screen Design
Create visual cues suggesting more content below the fold.

## Preview & Error Handling

### 53. Property Selection Preview
Show preview of style/tool changes before application.

### 54. Fail-Safe over Fool-Proof
Design for recoverable mistakes rather than preventing all errors. Make operations reversible.

### 55. Constructive Error Messaging
Provide clear, helpful error messages. Explain what happened and how to proceed.

## Usage Focus

### 56. Possibility vs Probability
Focus on main use cases. Don't overcomplicate interface for rare scenarios.

### 57. Silent Execution
Provide feedback through screen changes. Confirm only critical, irreversible actions.

## Navigation & Progress

### 58. Wayfinding
Help users navigate information spaces. Provide clear navigation schemes.

### 59. Escape Hatch
Always provide quick return to home/starting screen.

### 60. Immediate Gratification
Enable quick success within first few seconds of use.

### 61. Delayed Decision-Making
Minimize upfront information requirements. Allow partial engagement without complete setup.

## Transitions & Feedback

### 62. Transition Animations
Use 0.1-0.5 second animations for screen state changes. Animate both forward and reverse transitions.

### 63. Responsive Interaction
Respond to user actions within 0.1 seconds. Show progress for longer processes.

### 64. Contextual Feedback
Provide system status updates near point of interaction.

## Progressive Disclosure

### 65. Progressive Disclosure
Hide advanced features initially. Reveal complexity as user skill increases.

## Localization

### 66. Culturally Neutral Icons
Avoid culture-specific symbols in international interfaces.

### 67. Multilingual UI Considerations
Account for label length variations across languages.

### 68. Symbol Usage Caution
Avoid culturally specific symbols like ○ × △.

## Conventions

### 69. Conventional over Intuitive
Prioritize familiar interaction patterns. Design learnable new interactions.

### 70. Platform-Specific Button Placement
Follow platform conventions for positive/negative buttons.

## Spatial & Memory

### 71. Menu Item Positioning
Do not change the position of menu items. Users spatially memorize locations.

### 72. Highlight Design
Use only one visual element to highlight. Avoid excessive changes.

### 73. Optical Illusions
Consider visual illusions when designing. Manually adjust designs to counteract perceptual distortions.

### 74. Spatial Memory
Allow users to place objects in 2D space. Preserve object locations between sessions.

### 75. Prospective Memory
Enable users to mark content for future reference. Support bookmarks, flags, draft saving.

## Touch & Mobile

### 76. Touch Interface Design
Touchscreen buttons should be 7-10mm square. Space elements to prevent accidental touches.

### 77. Gesture Design
Use direct manipulation gestures. Avoid arbitrary command-based gestures.

### 78. Navigation Hierarchy
Drill down: top-to-bottom, left-to-right. Adjust for right-to-left language systems.

### 79. Screen Transition
Left represents "back" (past). Right represents "forward" (future). Reverse for right-to-left languages.

### 80. Mobile vs Desktop Design
Desktop: comprehensive overview. Mobile: hierarchical, limited information.

## Design Principles

### 81. Simplicity Principle
Keep simple functions easily accessible. Prevent complex interfaces from obscuring basic features.

### 82. Visual Restraint
Limit color and font variations. Avoid cluttered designs.

### 83. Layout Consistency
Use grid-based layouts. Maintain consistent margins and alignments. Create visual stability.

### 84. Customization Caution
Avoid over-relying on customization. Focus on optimal default design.

### 85. User-Centered Design
Design systems as user tools. Focus on user empowerment.

### 86. Flexibility
Create mode-less interfaces. Allow users to develop unique workflows.

### 87. Self-Explanatory Design
Make interfaces intuitively understandable. Avoid extensive instructions.

## Interaction Quality

### 88. UI Responsiveness
Prevent UI lockups. Ensure quick response to user actions.

### 89. Remove Game-Like Elements
Avoid randomness, complexity. Eliminate speed/skill requirements. Focus on task completion.

### 90. Interaction Refinement
Add subtle "play" in user interactions. Use experience-based corrections.

### 91. Interaction Targets
Expand clickable/tappable areas. Maintain visual balance.

### 92. Accessibility
Support screen readers. Provide alternative text. Allow text size adjustment. Design independent of color.

## User Experience

### 93. Pace of User Work
Allow users to work at their own pace. No time limits. Do not require high reflex skills.

### 94. User Illusion
Create a virtual world that conceals system mechanics. Provide illusions that help users focus on tasks.

## Design Philosophy

### 95. Design Meaning
Understand user needs. Analyze behavior patterns. Propose new perspectives.

### 96. Positive Human Impact
Design to tolerate human shortcomings, enhance strengths, and enrich life.

---

## Cross-Category Relationships

Many guidelines work together:

- **Simplify** (1) + **Mental Model** (3) + **Signifiers** (4): Core foundation for intuitive design
- **Consistency** (6) + **Tone & Manner** (27): Visual coherence
- **User Control** (7) + **Direct Manipulation** (8) + **Modeless** (9): User empowerment
- **Error Avoidance** (15) + **Constraints** (13) + **Constructive Error Messaging** (55): Error handling strategy
- **Accessibility** (92) + **User Language** (11) + **Culturally Neutral Icons** (66): Inclusive design

## Related Reference Files

- **Cognitive Principles**: Deep dive into Mental Model, Signifiers, Mapping, Consistency
- **Simplification**: Detailed guidance on Simplify and Make it Easy principles
- **Visual Design**: Visual Gestalt, Tone & Manner, Layout guidelines
- **Information Architecture**: Wayfinding, Navigation, Object-Based Design
- **Interaction Patterns**: User Control, Direct Manipulation, Input Forms
- **Accessibility & Localization**: Complete accessibility and multilingual design guidance
- **Platform-Specific**: Mobile vs Desktop, Touch interface guidelines
