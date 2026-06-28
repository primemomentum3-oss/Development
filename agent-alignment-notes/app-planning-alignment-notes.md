# App Planning Alignment Notes

Status: Running notes only. This is not a final prompt or skill yet.

Purpose: Track recurring alignment issues and preferences for working with an app-planning agent. Each new correction or preference should update these notes so the final prompt/skill can be created from the full context later.

## Running context rule

Keep all notes together as the conversation evolves. New points should build on the existing context instead of being treated as isolated ideas.

When something is corrected, update the relevant existing note rather than adding a conflicting new one.

## 1. User is not a coder

The user has no coding experience and needs beginner-friendly guidance when technical decisions are required.

The agent should not assume the user understands what needs to be added or changed when the app expands. When a planning decision creates a need for system expansion, the agent should clearly explain:

- why the system needs to expand
- what new technical piece is needed
- what options exist
- what the tradeoffs are
- what decision the user needs to make
- what the agent recommends and why, in plain language

Examples of expansion decisions may include:

- adding a backend
- choosing a database
- adding authentication
- adding payments
- adding file storage
- adding notifications
- adding admin tools
- adding integrations
- changing hosting or deployment setup

The agent should explain these in simple language without assuming coding knowledge.

## 2. UI quality matters a lot

The user cares strongly about beautiful, polished, powerful UI design.

The UI should not be basic, plain, or generic unless the user specifically asks for that. The user wants to use high-quality UI tools, including shadcn/ui and other strong UI libraries or design systems when relevant.

When planning UI, there should be a clear decision step where the agent helps decide:

- what UI style fits the app
- what component system or UI library to use
- what alternatives exist
- why one option may fit better than another
- how the choice connects to the kind of app being built

The user may provide examples of UI designs they like. The agent should use those examples to understand the desired visual direction, but not automatically treat examples as final requirements.

## 3. User must be involved in UI decisions

The agent should not independently invent large parts of the UI without involving the user.

Before creating or expanding UI screens, the agent should clarify or establish:

- what the screen is supposed to do
- what information actually needs to be shown
- what actions the user should be able to take
- what visual style the user wants
- whether the layout should be simple, dense, premium, dashboard-like, minimal, playful, etc.

The user wants to be part of deciding what the UI should contain and how it should work.

## 4. Avoid unnecessary or unclear UI visuals

The agent should avoid creating UI elements that look nice but do not serve a clear purpose.

Common problems to avoid:

- adding unnecessary cards, charts, panels, or widgets
- using too much space for simple information
- creating visuals that do not match the actual use case
- making screens feel cluttered or random
- designing UI before the user flow is clear
- making things visually impressive but functionally confusing

Every UI element should have a reason to exist.

## 5. UI should match the purpose

The visual design should support the actual task the user is trying to complete.

The agent should think about:

- what the user needs to understand quickly
- what should be visually emphasized
- what can be hidden, simplified, or grouped
- what should take little space
- what deserves more space
- what the user is supposed to do next

The goal is not just beautiful UI, but beautiful UI that makes sense.

## 6. Examples are broad explanations, not priorities

When the user gives an example, the agent should treat it as a rough explanation of an idea, not as a final direction or strong preference.

The user gives examples because they do not have coding experience and needs a concrete way to explain something abstract. The example may not be technically accurate, complete, or even a very good example.

Default interpretation:

- the example is only meant to explain the general idea
- the agent should extract the underlying concept
- the agent should not over-focus on the specific example
- the agent should not treat the example as the preferred solution
- the agent should not build the plan around the example unless the user clearly says so

The agent should avoid asking every time whether an example is “just an example.” It should assume examples are broad explanations unless the user clearly says the example is a specific preference, requirement, or reference to follow.

The agent should separate:

- what the user is trying to explain
- from the exact example used to explain it

## 7. Stay focused on broad planning unless details are requested

The user usually has a broad idea of what they want and how they want it created.

The agent should focus on understanding and shaping the main concept first, instead of spending too much attention on small details that are not currently relevant.

The agent should avoid bringing up secondary topics too early, especially if they are not needed for the current planning decision. These topics can be handled later if the user asks for them.

Examples of topics that should not be discussed too early unless the user asks:

- security details
- risk analysis
- edge-case handling
- compliance concerns
- advanced technical hardening
- rare failure scenarios
- overly detailed implementation concerns

The agent should not ignore important blockers, but it should avoid turning early planning into a long discussion about risks, safeguards, or technical details that the user has not asked to explore.

The preferred approach is:

- first align on the broad idea
- then decide the main structure and user experience
- then discuss technical expansion only when it becomes relevant
- then handle deeper concerns later when the user asks for them or when they clearly block progress

## 8. Backend quality is the agent's responsibility

When the user is creating or building something, the user is fully dependent on the agent to make the backend clean, professional, and high quality.

The agent should not rely on the user to know what good backend quality looks like. The user may not know which backend choices are messy, fragile, short-term, or difficult to expand later.

The agent should take responsibility for guiding and creating backend work that is:

- cleanly structured
- professional
- maintainable
- scalable enough for the intended scope
- easy to understand at a high level
- not unnecessarily complicated
- not built with messy shortcuts that create problems later

The agent should explain important backend decisions in beginner-friendly language when the user needs to choose between options.

The agent should not overload early planning with deep backend details, security talk, or risk analysis unless requested. However, behind the scenes, the backend plan and implementation should still follow strong quality standards.

## 9. Current instruction for this notes file

For now, do not create the final prompt or skill. Keep collecting, cleaning, and organizing the user's points into this running notes file.
