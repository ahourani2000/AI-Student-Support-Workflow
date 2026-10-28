# AI-Student-Support-Workflow

**Project type:** Independent AI Product Management Case Study

**Role:** Product Strategy, Workflow Design, No-Code Prototyping, AI Evaluation

**Tools:** Airtable, Make, Google Forms, Google Sheets

**Focus areas:** AI product design · Workflow automation · Retrieval-augmented generation concepts · Knowledge governance · Human-in-the-loop systems · Product testing

### Executive Summary

How can an educational institution use AI to streamline routine student inquiries without compromising the accuracy of institutional information or removing human judgment from important decisions?

I explored this question by designing and prototyping a no-code AI student-support workflow. The proposed product accepts student inquiries, classifies their intent, retrieves relevant information from a controlled knowledge base, generates a grounded response, and routes inquiries requiring human judgment to staff.

I built the prototype using Airtable and Make, with a staff-facing interface for reviewing AI-generated responses, approving or editing them, and managing inquiry resolution. I also designed an automated email workflow that sends the finalized response to the student after the inquiry reaches the resolved state.

Rather than treating AI as an autonomous support agent, the product treats AI as a workflow assistant operating within defined boundaries.

**Core product principle:** Automate routine information handling while keeping institutional decisions and final response approval under human control.

This is an independent portfolio project inspired by student-support workflows and publicly available institutional information. It was not commissioned by, deployed by, or officially endorsed by Miftaah Institute.

### 1. The Problem

Educational institutions manage recurring questions about program eligibility, registration, tuition, schedules, learning formats, and student services.

Information may be distributed across program pages, FAQs, student portals, contact forms, and staff communications. Handling these inquiries consistently requires staff to locate information, interpret the student's question, determine whether an exception or individual decision is involved, and communicate an appropriate response.

A generic chatbot could generate answers, but generation alone does not solve the underlying operational problem.

An effective AI support workflow also needs to address:

* Information reliability: How does the system ensure that responses reflect approved institutional information?
* Routing: Which inquiries can be handled routinely, and which require staff judgment?
* Traceability: Can staff identify the knowledge source used to generate a response?
* Human oversight: Can staff review, correct, reject, or escalate an AI-generated response?
* Operational efficiency: Can the system reduce repetitive administrative work without automating decisions that should remain with people?
* Follow-through: How does an approved response reach the student without unnecessary manual steps?

**Product Opportunity:**

Design an AI-assisted support workflow that connects inquiry intake, knowledge retrieval, response generation, human review, and response delivery into one structured process.

The goal was not to automate every student interaction. It was to identify the parts of the workflow where AI could provide value while explicitly defining where automation should stop.

### 2. Product Goals and Requirements

I translated the problem into 6 product requirements.

| Requirement | Product Behavior | 
| --- | --- |
| Understand inquiries | Classify the topic, summarize request, and assign a priority |
| Ground responses | Retrieve information from approved knowledge records |
| Prevent unsupported answers | Escalate when relevant approved knowledge is unavailable |
| Preserve human judgment | Route exceptions, account-specific questions, and individual decisions to staff |
| Support staff operations | Provide a centralized interface for review and resolution | 
| Complete the workflow | Send the finalized response after the inquiry is marked resolved | 

**Non-goals:**

To keep the prototype focused, I deliberately excluded several capabilities:

* Making admissions or other institutional decisions
* Accessing private student accounts or records
* Automatically changing institutional policies or approved knowledge
* Sending unreviewed AI-generated responses directly to students
* Replacing the institution's existing student portal or support channels

These boundaries helped define a manageable prototype and reduced the risk of automating decisions without sufficient context.

### 3. Solution Overview

The product consists of 4 connected layers.

***A. Intake and inquiry management***

Students submit an inquiry through a prototype form. The intake workflow transfers the submission into Airtable, where each inquiry becomes a structured record.

***B. AI processing and knowledge retrieval***

Make orchestrates the AI workflow. The system classifies the inquiry, determines whether it requires human review, searches approved knowledge records, identifies relevant information, and generates a responses grounded in the retrieved content.

***C. Staff review and resolution***

An Airtable interface gives staff visibility into the original inquiry, AI assessment, retrieved source, and suggested responses. Staff can approve, edit, reject, or escalate the response. 

***D. Response delivery***

After the inquiry is resolved and a final response has been recorded, an Airtable automation sends the finalized response to the student's email address.

The architecture separates AI-generated content from the final response approved by staff.

### 4. Technology Choices

* **Google Forms and Google Sheets:** Lightweight prototype intake and submission tracking.
* **Make:** Workflow orchestration, AI module coordination, structured data handling, conditional routing, and Airtable updates.
* **Airtable:** Structured inquiry records, knowledge management, staff interface, operational dashboard, and response-delivery automation.

The architecture is modular. The intake platform could be replaced by a website form or another approved institutional intake channel without fundamentally redesigning the AI workflow. 
