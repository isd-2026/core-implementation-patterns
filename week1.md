# Agile, Requirements & Planning, Setups

## Agile

### Introduction to Workshop Case Study
- Read and analyse the project brief

### Tool for Documentation
- Miro board - Theories
- Set up a Miro account with your UTS email
- Constraints and Risks for agile project management
- You may create a Miro team for your group assignment

### User Story

User stories are the basis of agile projects. They are expressed from the end users' perspective and focus on the what not the how. The key attributes of user stories are

- Template: as a `<someone>`, I want to `<perform an action>`, so that I can `<achieve a goal>`
- Acceptance criteria
- Estimate
- Priority
- Release & Iteration
- Feature
- Develop user stories for the workshop case study

> Good user story criteria

| Criterium | Description |
|---|---------|
| Indepenent | Can be developed and delivered on its own, without depending on other stories |
| Negotiable | The goal should be clear; the implementation details are negotiable |
| Valuable | Provides real value to the end user or customer |
| Estimable | Can be reasonably sized and estimated for effort and complexity |
| Properly Sized | Small enough to complete in a single iteration |
| Testable | With clear acceptance criteria |

### Non-Functional Requirements (NFRs)

- Expressed as user stories
- Develop user stories to capture non-functional requirements for the workshop case study

### User Story Map

- Map user stories to features/tasks
- Plan releases/iterations based on user story priority
- Create the user story map

## Software Design
### Object Oriented Programming (OOP) - A Quick Recap

Assumes prior learning in OOP. Used in backend design (Python) with limitted coverage, as we will only need a few classes.

- Encapsulation
- Abstraction
- Polymorphism
- Inheritance

### UML Class Diagram

UML stands for Unified Modeling Language. It is a standardized, general-purpose visual modeling language used by software developers and engineers to visualize, specify, construct, and document the structure and behavior of software systems.

UML class diagram is a foundational tool created specifically for object-oriented programming (OOP). A UML class diagram uses classes to represent entities or objects in the system, uses relationships and multiplicity to visualize the system structure. It serves as a blueprint for coding.

> Classes

- Classname: CamelCase
- Attributes: name, type, visibility (public, protected, private, package)
- Methods: name, parameters, return type, visibility

> Relationships

| Relationship | Description | Illustration | Implementation (Typical) | 
|---|--------|----------|-------|
| Association | One class knows about or uses (holds a reference to) another | Solid line; arrow points to the referenced class; generally uni-directional | Class attributes |
| Dependency | Temporary usage | Dashed line; arrow points from depenent to supplier | Method parameter, local variable, static call |
| Generalization/Inheritance | "Is-a" relationship | Solid line; hollow triangle arrowhead; arrow points to the parent | Class inheritance |
| Realization | Interface implementation | Dashed line; hollow triangle arrowhead; arrow points to the interface | Interface |
| Aggregation | Weak "has-a" relationship; parts can live without the whole | Solid line; hollow diamond arrowhead; arrow points to the whole | Independently created objects that are referenced by another class |
| Composition | Strong "has-a" relationship; parts cannot exist without the whole; whole controls parts' lifecycle | Solid line; filled diamond arrowhead; arrow points to the whole | Objects created and owned by another object, with their lifecycle bound to that object (created and destroyed together) |

> Multiplicity

| Notation | Meaning |
|---|--------|
| 1	| Exactly one |
| 0..1 | Zero or one (optional) |
| *	| Many (unbounded) |
| 0..* | Zero or many |
| 1..* | One or more |
| n	| Exactly n |
| m..n | Between m and n |

> Diagramming Tools

- Visual Paradiagm (Community Version) (good alignment with UML terms)
- Other diagramming tools are acceptable

## GitHub & GitHub Projects
- Create GitHub account with your UTS email
- Setup a shared repo for the workshop
- Create a project linked to this repo
- Add the user stories as issues (plan user stories)
- Setup a shared repo in GitHub Classroom for your assignment (invites to be sent out in Week 4)

## Practice
- Familiarize yourself with these tools 
