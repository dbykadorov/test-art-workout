ArtWorkout is a mobile app that teaches drawing through step-by-step, game-like lessons that help people 
learn, relax, and build a creative habit. We ship fast, measure everything (feature flags & A/B tests), 
and keep quality high with automation. The product runs on iOS and Android with a TypeScript-centric stack.

We’re hiring a Senior Full-Stack Engineer (TypeScript · React · NestJS) to design and ship end-to-end 
features (including internal tools), shape our data and content pipelines, and help us scale 
real-time and experimentation capabilities.

## 1 Minimal Data Model (Based on the Current App)  
Describe how you would design a data model for ArtWorkout.

Outline key entities and relationships.

Provide an example (e.g., a short JSON snippet or table sketch).

### Answer

[q1_answer](docs/q1_answer.md)

## Real-Time Multiplayer: Architecture
Imagine a mode where one player can see what another is drawing as it happens.

Describe the frontend and backend architecture at a high level.

How would you handle state sync, conflict resolution, and late joins/reconnects?

### Answer

[q2_answer](docs/q2_answer.md)

## Technologies, Protocols, and Constraints

ArtWorkout already has multiplayer. Your task is to inspect how the current multiplayer works in the l
ive app and then answer the questions below.

Which technologies/protocols would you use for the real-time channel and payloads?
Also specify the React-side technologies you would use to implement the core client–server interaction 
and state/data management.

Justify each choice with concrete reasons tied to ArtWorkout’s needs.

Where relevant, reference how the current ArtWorkout mechanics (e.g., step validation, scoring, 
undo behavior) influenced your design choices above.

Briefly outline the data-flow architecture. Include a simple end-to-end example of ArtWorkout 
showing how state updates propagate (client storage <> client <> server <>  db) in real time.
What constraints or calculations would you consider?

If you’ve built something similar (real-time collaboration/graphics, lesson pipelines, 
or event-driven apps), briefly share that experience and one lesson you’d apply here.

### Answer

[q3_answer](docs/q3_answer.md)