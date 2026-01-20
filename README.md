# Lia - Take-Home Project: Intelligent Action Item Extraction
Welcome! We're excited to see how you approach problems. This exercise is designed to simulate a real-world task you might encounter as part of our team. It gives you a chance to showcase your technical design skills, coding practices, and how you communicate your work.

## 1. The Product: Lia
Lia is a solution that automatically joins your calendar meetings (Google Meet, Zoom, etc.) to transcribe, summarize, and analyze them. Our goal is to turn conversational data into structured knowledge, saving teams hundreds of hours and unlocking powerful insights.

## 2. The Context & Problem
In modern remote work, teams generate hours of meeting audio every day. While we can transcribe this audio into text, the raw value lies in extracting commitments and tasks.

Blindly sending hour-long transcripts to Large Language Models (LLMs) is expensive, slow, and prone to hallucinating tasks that don't exist. We need a hybrid approach: a deterministic/probabilistic filter to identify high-potential segments, followed by an LLM to refine and format them.

## 3. The Challenge
Your mission is to build a pipeline that takes a raw meeting transcript and produces a clean, structured "To-Do List" by filtering noise and synthesizing intent.

This project is divided into two main parts:

- The RFC (Request for Comments): A document where you outline your analysis of the problem and your proposed technical solution. This is where you do your thinking and planning.

- The Prototype: A working piece of code that implements your proposed solution.

The goal is to produce a well-reasoned RFC and a clean, well-documented prototype that demonstrates your approach to the problem.

## 4. Provided Files
- README.md: This file.
- rfc_template.md: The template you should use to document your approach.
- transcripts.json: A file containing 5 synthetic meeting transcripts to help you test your solution.

## 5. Step-by-Step Instructions
1. Setup: Clone this repository. Please keep your fork private to ensure a fair process for all applicants. Do not create a pull request back to the original repository.
2. Analysis & RFC: Before writing any code, thoroughly analyze the problem. Consider different approaches, their trade-offs, potential edge cases, and how you would measure success. Document your thoughts by filling out the `rfc_template.md` file. Rename it to `rfc.md` when you are done.
3. Prototyping: Implement a prototype. This involves three parts:
    *   **Part 1: The "Actionability" Heuristic**: Develop a mechanism to analyze transcript segments and identify those likely to contain action items. You should aim to detect intent efficiently. **Constraint**: You cannot send the entire transcript to an LLM; you must minimize token usage.
    *   **Part 2: The LLM Refiner**: Process the selected segments with an LLM (using a provider of your choice or a mock function) to extract specific tasks, owners, and deadlines.
    *   **Part 3: Visualization & Analysis**: Create a Jupyter Notebook that demonstrates your pipeline. It should load data, run your model, visualize the results (e.g., where tasks appear in the meeting), and display the final output.
   - Please use Python.
   - Your code should be clean, readable, and well-documented.
4. Testing & Data: Use the `transcripts.json` file provided to test your solution. It contains 5 meeting transcripts. We strongly encourage you to generate more synthetic data to test different edge cases.

   Example JSON structure:
   ```json
   {
     "meeting_id": "mtg_12345",
     "participants": ["Alice", "Bob", "Charlie"],
     "segments": [
       { "id": 1, "speaker": "Alice", "text": "Okay, let's kick this off. Has anyone seen the Q3 report?", "timestamp": 12.5 },
       { "id": 2, "speaker": "Bob", "text": "I haven't finished it yet.", "timestamp": 15.0 },
       { "id": 3, "speaker": "Bob", "text": "I will send the draft to everyone by 5 PM today.", "timestamp": 18.2 },
       { "id": 4, "speaker": "Charlie", "text": "Great. Alice, can you schedule the follow-up meeting?", "timestamp": 22.0 }
     ]
   }
   ```
5. Documentation & Git Hygiene: How you work is as important as the final output.
6. Your git history should tell a story of your development process. Use clear, conventional commit messages that explain the "why" behind your changes.
   For example: feat: Implement heuristic scoring logic, docs: Draft initial RFC with feature engineering ideas, fix: Handle delegation edge case.

## 6. Deliverables
Your final submission should be a private GitHub repository containing:

- A completed `rfc.md` file detailing your proposed solution.
- The source code for your prototype.
- A Jupyter Notebook showing your thought process and results.
- A clean, descriptive Git history that reflects your process.
- (Optional) Any additional test data files you created.
- (Optional) A brief NOTES.md file for any thoughts, observations, or ideas you had that didn't fit elsewhere.

## 7. Evaluation Criteria
We'll be looking at:

- Creativity in Approach: How effectively did you identify potential action items?
- Prompt Design: Is your LLM prompt structured to handle edge cases?
- Code Quality & Documentation: The readability, efficiency, and structure of your prototype code and its accompanying comments.
- Product Thinking: Did you handle different types of tasks (e.g., self-commitment vs. delegation)?
- Methodology & Communication: Your ability to break down a problem into logical steps and communicate your work clearly through your commit history and RFC.

## 8. Submission
When your project is complete, give the user @vanyabrucker read access to your private repository.

Send an email to vanya@aintegrator.ch to let us know you have completed the task, including a link to your private repository in the email.

Good luck, and we look forward to seeing your work!
