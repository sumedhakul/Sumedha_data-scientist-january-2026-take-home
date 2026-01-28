# RFC: Meeting Transcript To-Do List Pipeline

# 1. Purpose
The purpose of this project is  developing a pipeline that converts raw meeting transcripts into a structured to-do list by identifying and synthesizing actionable items, thus making the process more efficient and insightful for teams.

# 2. Background & Problem Statement
Teams in remote work often generate long meeting transcripts that are unstructured, making it difficult to extract actionable tasks and commitments. Currently, transcription services capture all content but fail to highlight key tasks, leading to inefficiency and missed action items.

The goal is to automate the extraction of tasks, commitments, and deadlines from transcripts into a structured to-do list. This would reduce manual effort, improve task tracking, and save time.

Core Challenge :
Transform raw meeting transcripts into structured, actionable to-do lists while:
##### Minimizing LLM token costs by filtering before processing
##### Avoiding hallucinated tasks that weren't actually discussed
##### Capturing nuanced commitments (self-assignments, delegations, soft commitments)
##### Maintaining context necessary for understanding who should do what by when


# 3. Proposed Solution
This is the core of the RFC. Describe your proposed solution in detail.

# 4. Evaluation & Metrics
How will you define and measure the success of your prototype? What specific metrics will you use to determine if your solution is effective?

# 5. Risks & Open Questions
What are the biggest risks or unknowns with your proposed approach? What questions would you need to answer to de-risk the project?

# 6. Prototype Implementation Plan
Briefly outline the steps you will take to build the prototype. This can be a simple checklist.
