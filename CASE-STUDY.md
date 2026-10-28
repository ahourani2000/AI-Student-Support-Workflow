# The Complete Workflow Architecture

## AI Workflow Design

I separated the AI workflow into dinstinct stages rather than asking a single prompt to interpret the inquiry, retrieve information, generate a response, and make routing decisions.

### Stage 1 - Inquiry Classification

Purpose: Understand the student's request before attempting to answer it.

An AI classifier produces structured outputs:
* Category: The subject of the inquiry
* Priority: Low, Normal, High, Urgent
* Summary: A concise description of the student's request
* Human review requirements: Whether staff involvement is necessary
* Search terms: Terms that help identify relevant knowledge

Example: 

Student inquiry: "What is the minimum age for the Associate Program?"

The classifier identifies the inquiry as program-related, summarizes the question, and produces search terms related to eligibility and age requirements.

The classifier does not generate the student-facing answer. Separating classification from response generation makes the workflow easier to inspect, test, and troubleshoot.

### Stage 2 - Human-Review Routing

Before retrieval, the workflow evaluates whether the inquiry should be escalated.

Examples of escalation conditions include:

* Individual admissions decisions
* Requests for exceptions
* Account-specific questions
* Situations requiring staff judgment
* Sensitive student circumstances

If review is required, the inquiry is marked for human review and routed to the staff queue.

This prevents the system from treating every question as an opportunity to generate an answer.

### Stage 3 - Approved Knowledge Retrieval

For inquiries that can proceed, Make searches Airtable for knowledge records whose status is Approved.

The retrieved records are collected and passed to a separate AI relevance-matching stage.

The relevance matcher identifies the knowledge record or records most relevant to the student's question and returns structured identifiers and a retrieval-confidence signal.

The workflow then retrieves the selected knowledge record so the response generator can use the actual approved answer.

This design follows a retrieval-grounded approach: Institutional information is supplied to the model rather than relying on the model's general knowledge as the source of institutional truth.

**Important limitation:** Retrieval and grounding reduce opportunities for unsupported answers, but do not guarantee factual correctness. The knowledge itself must be accurate, current, and appropriately approved, and the retrieval and response stages must be evaluated.

### Stage 4 - Knowledge-Found Decision

The workflow checks whether relevant approved knowledge was identified.

* Relevant knowledge found: Continue to response generation.
* No relevant knowledge found: Escalate the inquiry and avoid generating an unsupported answer.

This creates an explicit fallback path rather than forcing the AI to answer every question.

### Stage 5 - Grounded Response Generation

The response generator receives the original inquiry and the selected approved knowledge.

Its instructions require it to:
* Use the supplied institutional information
* Avoid inventing policies, deadlines, prices, or eligibility requirements
* Avoid using outside information to fill gaps
* Avoid making institutional decisions
* Escalate when approved information is insufficient
* Produce a concise, professional, student-facing draft

The output is saved as Suggested Response.

It is not automatically treated as the final response.

### Stage 6 - Human Review and Resolution

The inquiry is displayed in the Support Inbox, where staff can review the AI assessment, knowledge source, and suggested response.

Staff choose one of 4 decisions:
* **Approve:** Accept the suggested response.
* **Edit:** Modify the response before finalizing it.
* **Reject:** Decline the suggested response and leave the inquiry unresolved for further handling.
* **Escalate:** Route the inquiry for additional human attention.

For approval, the suggested is copied into Final Response. For an edit, the staff-edited response becomes the final response.

The staff-resolution automation was built and tested separately. Because the prototype's Make free plan limits the number of simultaneously active scenarios, it is not represented as continuously active alongside the intake and AI-processing scenarios.

### Stage 7 - Automated Response Delivery

I added an Airtable automation that sends the student's email when the inquiry's Status becomes Resolved.

The email uses the student's email address and the Final Response field.

This removes the repetitive step of manually composing an email and copying the approved response into it.

The distinction is intentional:

AI drafts -> Staff finalizes -> System delivers.

The automation streamlines delivery without transferring final response approval to AI.

## Knowledge Governance and Data Model

The knowledge base is a central product component rather than a collection of text pasted into an AI prompt.

Each knowledge record contains structured information such as:
* Knowledge ID
* Category and topic
* Question
* Approved answer
* Source name and URL
* Approval status
* Last-reviewed date
* Escalation requirement
* Search terms
* Internal notes

### Approval controls

The knowledge base distinguishes between records marked Approved and records that still require review.

Only approved records are retrieved by the AI workflow. This creates a defined boundary between information available for automated responses and information that should not yet be used.

Because this is an independent prototype, Approved means approved within the prototype's workflow after review against public information. It does not imply formal approval by Miftaah Institute.

### Why traceability matters

The system records the selected Knowledge Source on the inquiry.

This allows staff to inspect the information used to generate a response rather than seeing an answer without context.

For a production implementation, I would add formal knowledge ownership, source validation, review intervals, expiration handling, and a process for resolving conflicting information.

## Staff Interface and Operational Visibility

I created an Airtable interface with 3 main operational pages and an intake form.

### Support Dashboard

The dashboard provides a high-level view of inquiry volume and workflow activity.

It includes 4 KPI cards:
* Total Inquiries
* Awaiting Review
* Human Escalations
* Resolved

It also includes a chart showing the number of inquiries marked as requiring human review.

These are prototype activity indicators, not evidence of measured improvements in efficiency or student satisfaction.

### Support Inbox

The Support Inbox is the main staff workspace. It displays inquiries in review and organizes information into 4 sections:
1. Student and Inquiry
2. AI Assessment
3. AI Response
4. Staff Review

Staff can inspect the original inquiry, category, priority, summary, human-review flag, knowledge source, suggested response, and final response. Editable fields and decision buttons support the staff workflow.

The inbox also supports filtering and sorting so staff can focus on high-priority inquiries or specific categories.

### Knowledge Base

The Knowledge Base interface exposes the structured institutional information used by the AI. Staff can browse records by category and inspect their status and other available fields. 

The knowledge base is intended to serve as the governed source of truth for automated responses.

### Student Intake Form

A prototype form demonstrates how students could submit their questions through a simplified intake experience. The current intake automation uses Google Forms and Google Sheets; the Airtable interface form is an alternative front-end concept, not a second connected intake route.

## Product Testing and Quality Assurance

I designed test cases around both successful retrieval and situations where the AI should not answer.

The focus was not only on whether the workflow could generate a response, but also whether it could recognize its limitations.

| Test Scenario | Expected Behavior | Result |
| --- | --- | --- |
| Known factual question about Associate Program eligibility | Retrieve relevant approved knowledge, generate a grounded draft, and log the source | Passed | 
| Request for an exception after a registration deadline | Require human review rather than decide the exception | Passed | 
| Question about a program absent from approved knowledge | Escalate without inventing an answer | Passed |
| Student asks about their individual application decision | Escalate rather than make an individual decision | Passed |
| Ambiguous or potentially conflicting schedule information | Avoid unsupported certainty and route for review | Passed | 

These results refer to the individual workflow and quality-control tests performed on the prototype. They do not establish production reliability or measured performance at institutional scale.

### Testing principles

The tests reinforced 3 requirements:
1. A known answer should be grounded in the correct approved source.
2. Missing or ambiguous knowledge should trigger a safe fallback.
3. Questions requiring institutional judgment should remain under human control.

The final live intake-to-resolution path should also be validated as a complete integrated scenario before the prototype is described as fully end-to-end verified.

## Product Decisions and Trade-offs

### Why no-code?

Airtable and Make made it possible to prototype the data model, AI workflow, staff interface, and automation without building a custom application.

This reduced implementation overhead and made the workflow easier to inspect and modify. Whether these tools would remain suitable for production would depend on institutional security requirements, scale, integration needs, and operating costs.

### Why separate classification, retrieval, and generation?

Separating these stages creates clearer responsibilities and makes errors easier to isolate.

If the wrong category is assigned, the classification stage can be investigated. If the wrong knowledge record is selected, retrieval can be evaluated. If the response misrepresents the approved answer, generation can be tested independently.

### Why require approved knowledge?

Institutional policies and program information should not be inferred from general model knowledge. Restricting retrieval to approved records provides a more controlled starting point for generating responses.

### Why human oversight?

The consequences of an incorrect answer vary by inquiry type. A routine factual question differs from an individual admissions decision or a request for an exception.

The workflow therefore treats human escalation as an intentional product capability, not as a failure to automate.

### Why automate the email separately?

Finalizing a response and delivering it are different tasks.

Staff should retain responsibility for the response itself, while a workflow automation can handle the repetitive delivery step once the inquiry is resolved.

## Success Metrics

The prototype establishes a foundation for measurement, but it has not yet demonstrated quantified business outcomes.

For a production pilot, I would define a baseline and evaluate the following metrics.

| Dimension | Metric | What it measures |
| --- | --- | --- |
| Efficiency | Median time to resolution | Whether the workflow reduces time spent handling inquiries |
| Efficiency | Staff handling time per inquiry | Whether routine work becomes less labor-intensive | 
| AI quality | Retrieval accuracy | Whether the correct approved knowledge is selected|
| AI quality | Grounded-response rate | Whether generated responses accurately reflect their sources |
| AI quality | Staff correction rate | How frequently staff need to materially edit responses |
| Safety | Escalation precision and recall | Whether inquiries requiring human review are correctly identified |
| Safety | Unsupported-answer rate | How often responses contain claims not supported by approved knowledge |
| Student experience | Student satisfaction | Whether the support experience improves |
| Student experience | Repeat inquiry rate | Whether students receive sufficiently clear and complete answers |
| Operations | Automated delivery success rate | Whether finalized responses are successfully sent |

### Evaluation Approach

I would begin with a controlled pilot using a defined set of low-risk inquiry categories and a staff-reviewed test set.

The pilot would compare AI-assisted handling with the existing process, track errors and corrections, and expand only when quality and safety criteria are met.

The objective would not be to maximize automation at any cost. It would be to improve operational efficiency while maintaining reliable responses and appropriate human oversight.

## Limitations and Production Considerations

This project is a no-code prototype, not a production student-support system.

Several areas would need additional work before institutional deployment.

### Knowledge Quality and Ownership

Institutional staff would need to validate the knowledge records, resolve inconsistencies, assign owners, and establish review and expiration policies.

### Retrieval and Response Evaluation 

The current relevance-matching approach needs a larger evaluation set, measured retrieval accuracy, and systematic assessment of response grounding. A model-generated confidence value should not be treated as a calibrated probability of factual correctness.

### Privacy and Access Controls

A production deployment would require an assessment of student-data sensitivity, data retention, access permissions, and the suitability of each connected service. Account-specific support would require appropriate authentication and secure integrations rather than relying on the public information prototype.

### Operational Reliability

The prototype uses scheduled workflows and is constrained by free-tier automation limits. Production use would require validated latency expectations, failure handling, monitoring, duplicate prevention, and reliable notifications.

### Email Delivery Safeguards

The email automation should be tested for missing email addresses, blank final responses, duplicate sends, delivery failures, and changes to a resolved record.

### Institutional Integration

The prototype does not connect to Miftaah's private systems or student records. A production design would require stakeholder discovery to determine the appropriate integrations, ownership, and security requirements.

## Future Roadmap

### Phase 1 — Prototype and workflow validation

* Build intake and AI-processing workflows.
* Establish the structured knowledge base.
* Implement classification, retrieval, response generation, and escalation.
* Create the staff interface and dashboard.
* Test known-answer, unknown-answer, exception, and account-specific scenarios.
* Validate the integrated workflow from intake through email delivery.

### Phase 2 — Stakeholder and knowledge validation

* Interview support staff to identify frequent inquiry categories.
* Validate institutional answers and source ownership.
* Define which inquiry types are eligible for automation.
* Establish escalation policies and review procedures.
* Agree on success metrics and acceptance criteria.

### Phase 3 — Controlled pilot

* Start with high-volume, low-risk inquiry categories.
* Evaluate retrieval and response quality against a staff-reviewed dataset.
* Monitor staff corrections and escalation accuracy.
* Validate privacy, access, email delivery, and operational reliability.
* Compare pilot results with the existing support process.

### Phase 4 — Production integration and improvement

* Integrate approved intake channels and institutional systems.
* Establish monitoring, notifications, and failure recovery.
* Analyze recurring staff edits and unanswered questions.
* Propose knowledge improvements for staff review.
* Expand automation gradually based on measured performance.

AI-generated feedback would not automatically modify institutional knowledge. Proposed changes would require appropriate human review and approval.
