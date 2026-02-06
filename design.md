# Design Document

## System Overview

Pebble is an event-driven web application that observes user learning behavior in real-time. The system captures behavioral signals on the client side, processes them asynchronously, and provides optional AI assistance through a managed LLM inference service.

The architecture prioritizes non-blocking operations, minimal latency, and user control. Behavioral data flows from the client through processing pipelines that detect struggle patterns and generate context-aware nudges. Users receive assistance only when needed and only if they choose to engage.

Pebble operates as a calm computing system: unobtrusive, adaptive, and respectful of user flow.

## High-Level Architecture

### Client Layer

The Client Layer runs in the user's web browser and handles all direct user interactions. It captures behavioral events such as clicks, keystrokes, mouse movements, and navigation actions. This layer renders the learning interface, displays content, and presents optional AI nudges. It operates independently of the backend for core learning activities, ensuring responsiveness and offline resilience.

### Application Layer

The Application Layer manages business logic and orchestrates system components. It processes incoming behavioral telemetry, maintains session state, and coordinates between the intelligence layer and data storage. This layer implements stateless services that can scale horizontally. It handles user authentication, session management, and API routing.

### Intelligence Layer

The Intelligence Layer analyzes behavioral patterns to detect struggle signals and generate assistance. It applies rule-based heuristics to identify hesitation, confusion, and cognitive overload. When struggle is detected, it constructs context-aware prompts and invokes a managed LLM inference service to generate helpful nudges. This layer operates asynchronously to avoid blocking user interactions.

### Data Layer

The Data Layer persists user information, session logs, and behavioral telemetry. It stores user preferences, learning history, and struggle patterns. The layer ensures data security through encryption and access controls. It provides efficient querying for progress insights and historical analysis.

## Component Design

### Frontend

#### Activity Monitor

The Activity Monitor is a client-side JavaScript module that captures user interaction events. It listens for mouse movements, clicks, keyboard input, scroll actions, and focus changes. Events are timestamped and batched for efficient transmission to the backend. The monitor operates with minimal performance overhead and respects user privacy by capturing only behavioral metadata, not content.

#### Learning Interface

The Learning Interface presents learning materials, exercises, and problems to users. It provides a clean, distraction-free environment optimized for focus. The interface adapts to different content types including text, code, and interactive exercises. It maintains state locally to ensure responsiveness and supports offline learning when possible.

#### Nudge UI

The Nudge UI displays optional AI assistance in a subtle, non-intrusive manner. Nudges appear as small, dismissible cards or tooltips positioned to avoid obscuring learning content. Users can expand nudges for more detail, dismiss them immediately, or adjust nudge frequency through settings. The UI uses calm visual design with minimal animation and muted colors.

### Backend Services

#### Telemetry Stream Processor

The Telemetry Stream Processor receives batched behavioral events from clients and routes them to appropriate analysis components. It validates incoming data, enriches events with session context, and forwards them to the Struggle Classifier. The processor operates asynchronously and handles high-frequency event streams efficiently. It implements rate limiting and error handling to ensure system stability.

#### Session Manager

The Session Manager tracks active learning sessions and maintains session state. It associates behavioral events with specific users and learning activities. The manager handles session creation, updates, and termination. It provides session context to other components and persists session data for historical analysis. Sessions are stateless on the server side, with state reconstructed from stored data as needed.

### AI / Intelligence Components

#### Struggle Classifier

The Struggle Classifier applies rule-based heuristics to detect behavioral patterns indicating struggle. It analyzes time spent on tasks, repeated actions, hesitation markers, and navigation patterns. The classifier uses configurable thresholds to balance sensitivity and false positive rates. When struggle is detected, it triggers the Context Engine to generate assistance. The classifier operates in real-time with minimal latency.

#### Context Engine

The Context Engine constructs context-aware prompts for the managed LLM inference service. It gathers information about the current learning activity, user level, recent interactions, and detected struggle patterns. The engine formats this context into effective prompts that guide the LLM to generate helpful, appropriate hints. It manages API calls to the managed LLM inference service and handles responses. The engine implements retry logic and fallback strategies for service failures.

### Data Storage

#### User Store

The User Store persists user accounts, authentication credentials, and preferences. It stores minimal personal information required for system operation. User preferences include nudge frequency settings, privacy controls, and learning goals. The store implements encryption at rest and secure access patterns.

#### Session Logs

Session Logs store historical behavioral data, struggle events, and AI interactions. Logs enable progress tracking, pattern analysis, and system improvement. Data is retained according to user preferences and privacy policies. Logs are structured for efficient querying and aggregation.

## Data Flow

1. User interacts with the Learning Interface by reading content, attempting exercises, or navigating between resources.

2. Activity Monitor captures interaction events including timestamps, event types, and behavioral metadata. Events are batched locally to reduce network overhead.

3. Batched events are transmitted asynchronously to the Telemetry Stream Processor without blocking user interactions.

4. Telemetry Stream Processor validates and enriches events with session context, then forwards them to the Struggle Classifier.

5. Struggle Classifier analyzes behavioral patterns in real-time. If struggle signals are detected, it triggers the Context Engine.

6. Context Engine retrieves current learning context and constructs a prompt for the managed LLM inference service.

7. Managed LLM inference service generates a context-aware hint based on the provided prompt.

8. Context Engine receives the generated hint and formats it for display.

9. Nudge UI presents the optional hint to the user in a non-intrusive manner.

10. User chooses to engage with the hint, dismiss it, or ignore it. User response is captured as feedback for future improvements.

11. Session Manager persists behavioral data, struggle events, and user interactions to Session Logs for historical analysis.

All steps after step 3 occur asynchronously and do not block the user's learning experience. The user can continue interacting with the Learning Interface regardless of backend processing state.

## Key Design Decisions

### Behavior Over Correctness

Pebble prioritizes observing how users learn rather than evaluating whether answers are correct. This design decision reflects the insight that struggle often precedes failure and can be detected through behavioral signals. By focusing on behavior, Pebble provides earlier, more proactive assistance. This approach also reduces pressure on learners who may fear judgment based on correctness.

### Optional Nudges

All AI assistance is optional and user-controlled. Nudges are presented as suggestions that users can engage with, dismiss, or ignore. This design respects user autonomy and avoids the cognitive load of forced interruptions. Users who prefer independent learning can disable nudges entirely. This decision aligns with calm computing principles and reduces the risk of disrupting flow states.

### Stateless Services

Backend services are designed to be stateless wherever possible. Session state is reconstructed from stored data rather than held in memory. This design enables horizontal scaling, simplifies deployment, and improves reliability. Stateless services can be restarted or replicated without losing user sessions. This decision supports future scalability beyond the hackathon prototype.

### Adaptation Over Accuracy

Pebble uses rule-based heuristics for struggle detection rather than pursuing perfect accuracy through machine learning. This design decision reflects hackathon constraints and the recognition that adaptive systems can improve over time. Simple heuristics are easier to implement, debug, and explain. The system can adapt thresholds and rules based on observed user feedback without requiring model retraining.

## Security & Privacy Considerations

### Data Minimization

Pebble collects only behavioral metadata necessary for struggle detection. The system does not record screen content, typed text, or personally identifiable information beyond what is required for authentication. Event capture is limited to interaction types, timestamps, and element identifiers. This minimizes privacy risk and reduces data storage requirements.

### No Screen Recording

The system explicitly avoids screen recording, keystroke logging, or content capture. Users can learn without concern that their work is being recorded. This design decision builds trust and complies with privacy best practices. Behavioral signals are sufficient for struggle detection without invasive monitoring.

### Encryption

All data transmission uses HTTPS with TLS encryption. Stored user data is encrypted at rest using industry-standard algorithms. Authentication credentials are hashed and salted. API keys for the managed LLM inference service are stored securely and never exposed to clients. Encryption protects user privacy and prevents unauthorized access.

### Explicit User Consent

Users must explicitly consent to behavioral tracking before the Activity Monitor begins capturing events. Consent is obtained through clear, understandable language that explains what data is collected and how it is used. Users can revoke consent at any time and request deletion of their data. This design respects user autonomy and complies with data protection regulations.

## Future Enhancements

### Expanded Language Support

Post-hackathon, Pebble could support multiple natural languages for both learning content and AI nudges. This would make the system accessible to a global audience and support learners in their native languages.

### Mobile Adaptation

The current web-first design could be adapted for mobile devices with touch-optimized interfaces and mobile-specific behavioral signals. Mobile learning is increasingly common, and Pebble's calm computing approach would translate well to smaller screens.

### Social Learning Insights

Future versions could provide anonymized insights into how other learners approach similar problems. This would enable peer learning without compromising privacy. Users could see common struggle patterns and successful strategies without exposing individual data.

### ML-Based Struggle Detection

After gathering sufficient behavioral data, the system could transition from rule-based heuristics to machine learning models for struggle detection. ML models could identify subtle patterns and improve accuracy over time. This enhancement would require careful validation to ensure the system remains explainable and trustworthy.
