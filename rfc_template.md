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
My approach uses a two-stage pipeline that combines fast pattern matching with intelligent LLM processing:
The Big Picture
Instead of sending everything to an LLM, I first use a heuristic scorer to identify which parts of the transcript are likely to contain action items. Only those high-probability segments (plus their surrounding context) go to the LLM. This cuts token usage by 70-85% while still capturing the important stuff.
Raw Transcript → Score Segments → Filter High Scores → Add Context → LLM Extraction → Structured Tasks
(500 segments)    (pattern matching)  (keep ~20-30%)    (±2 segments)   (GPT/Claude)      (clean JSON)

Stage 1: Smart Filtering with Heuristics
I built a scoring system that looks for linguistic patterns that signal action items. Each segment gets scored 0-10 based on what it contains:
Strong Signals (high points):

Commitment language: "I'll", "I will", "going to" → +3 points
Delegation: "can you", "please", "you should" → +2.5 points
Deadlines: "by Friday", "tomorrow", "next week" → +2 points

Supporting Signals:

Action verbs: "send", "create", "schedule", "review" → +1 point
Task words: "report", "document", "meeting", "ticket" → +1 point

Red Flags (penalties):

Past tense: "did", "was", "already done" → -1 point
Hypotheticals: "maybe", "what if", "perhaps" → -0.5 points

Why this works: If someone says "I'll send the report by Friday", that hits commitment language (+3), action verb (+1), task noun (+1), and deadline (+2) = 7 points. That's clearly worth checking. But "That's an interesting idea" scores near zero.
The Context Window Trick: When I find a high-scoring segment, I grab the 2 segments before and 2 after it (5 total). This preserves the conversation flow. For example:
Alice: "Can you review the design doc?"     (context)
Bob: "Sure, I'll do it by tomorrow"        (HIGH SCORE - the key commitment)
Alice: "Great, send it to the team after"  (context)
This way, the LLM sees the full exchange and knows who's doing what.
Stage 2: LLM Extraction with Guard Rails
For the filtered segments (now only 20-30% of the original), I send them to an LLM with a carefully designed prompt that:
Tells the LLM what to extract:

Task description (what needs to be done)
Assignee (who's responsible - and this is the person who commits, not who asks!)
Deadline (when it's due, if mentioned)
Confidence level (how sure are we this is real)
Priority (inferred from urgency language)
Status (is this upcoming or already done?)

Sets strict rules:

Only extract actual commitments, not ideas or hypotheticals
Don't make stuff up - if there's no clear deadline, mark it as null
If a task is already completed, flag it so we can filter it out
For ambiguous situations (like "someone should do this"), mark confidence as low

Example prompt structure:
You are extracting action items from meeting transcripts.

[conversation segments with timestamps and speakers]

Extract as JSON. For each action item:
- task: clear description
- assignee: who committed (not who asked)
- deadline: exact time mentioned or null
- confidence: high/medium/low
- priority: high/medium/low based on language
- status: upcoming/completed/cancelled

Only extract real commitments. Return [] if none exist.
The LLM returns structured JSON that I can immediately parse and use.
How It All Fits Together

Load the transcript (JSON format with segments)
Run the heuristic scorer on every segment
Keep only segments scoring above 3.0
Add context windows around each one
Remove duplicate windows (if segments are close together)
Send to LLM in batches
Parse the JSON responses
Remove duplicate action items (sometimes the same task is mentioned multiple times)
Return a clean list of structured action items

Technical Implementation
I'm building this in Python with four main modules:

models.py: Data classes for Segment, ActionItem, and Meeting
heuristic.py: The scoring logic and pattern matching
extractor.py: LLM integration (supports OpenAI, Anthropic, or a mock for testing)
pipeline.py: Orchestrates everything and handles the end-to-end flow

The design is modular so I can easily swap out the LLM provider or adjust the heuristic weights based on real-world performance.

# 4. Evaluation & Metrics
I need to measure success in a few key dimensions:
Primary Success Metrics
Recall (Target: >85%)

Did we catch most of the actual action items?
Measured by: Manually label test transcripts, compare to what the system extracts
Why it matters: Missing tasks defeats the whole purpose

Precision (Target: >75%)

Are the extracted items actually legitimate action items?
Measured by: Review extracted items and mark false positives
Why it matters: False positives waste people's time

Token Efficiency (Target: >70% reduction)

How much did we reduce LLM token usage?
Measured by: Count tokens in filtered segments vs. full transcript
Why it matters: This directly impacts cost at scale

Assignee Accuracy (Target: >80%)

Did we correctly identify who's responsible?
Measured by: Compare extracted assignee to ground truth
Why it matters: Wrong assignee = task goes to wrong person

How I'll Test This
Test Data: Start with the 5 provided transcripts, then create a few more edge cases:

A meeting with no action items (pure brainstorming)
A meeting with tons of action items (sprint planning)
A meeting with soft/ambiguous commitments
A meeting with completed vs. upcoming tasks

Evaluation Process:

Manually go through test transcripts and mark all actual action items (ground truth)
Run the pipeline on these transcripts
Compare what it found vs. what I marked
Calculate recall (did we catch them?), precision (were they real?), and F1 score
Look at failures to understand what went wrong

Iterative Tuning:

If recall is low: Lower the threshold or add more patterns
If precision is low: Tighten the LLM prompt or raise the threshold
Track which patterns work best and adjust weights accordingly

What Good Looks Like
A successful extraction on a test meeting would:

Catch at least 9 out of 10 real action items
Have no more than 2 false positives per 10 items
Process only 20-30% of segments through the LLM
Correctly assign tasks 8+ times out of 10
Complete processing in under 30 seconds

# 5. Risks & Open Questions

Major Risks I'm Worried About
Risk #1: The filter might be too aggressive

What if the heuristic misses action items with unusual phrasing?
Mitigation: Start with a conservative threshold (2.5-3.0) and tune based on real data. Keep track of false negatives to add patterns.

Risk #2: The LLM might hallucinate tasks

What if it extracts things that sound plausible but weren't actually said?
Mitigation: Very strict prompt with explicit rules. Use low temperature (0.1). Add confidence scores so users can review low-confidence items.

Risk #3: Context windows might not be enough

What if the commitment and details are far apart in the conversation?
Mitigation: Make window size configurable. Could do ±3 instead of ±2 if needed. Track cases where we miss context.

Risk #4: It might still be too expensive

What if even 20-30% of segments is too many tokens at scale?
Mitigation: Use cheaper models (GPT-4o-mini instead of GPT-4). Could batch multiple windows in one API call. Add caching for similar segments.

Risk #5: Handling speaker attribution could be tricky

What if speaker labels are wrong or "you" is ambiguous?
Mitigation: Trust the labels but mark confidence as low when unclear. Support multiple assignees for group tasks.

Questions I Need to Answer

What's the optimal threshold? I'm starting at 3.0, but this needs testing with real data. Might vary by meeting type (standup vs. planning vs. retrospective).
How do I handle recurring tasks? If someone says "I'll send the weekly report as usual", should I link it to previous weeks? That's probably v2.
Should I track completion across meetings? If a task is assigned Monday and marked done Wednesday, should the system connect those? Complex but valuable.
What about time zones? "By 5 PM" - whose timezone? Probably need meeting metadata.
How do I collect feedback efficiently? Need a simple way for users to mark "good" or "bad" extractions to improve the system over time.
When should I switch to ML? The heuristic is interpretable and fast, but eventually a fine-tuned classifier might be better. When is that worth it?




# 6. Prototype Implementation Plan


Given the time constraints, here's my realistic plan:
What I'm Building (Core MVP)
Hour 1: Foundation

 Set up project structure
 Create data models (Segment, ActionItem, Meeting)
 Implement heuristic scorer with pattern matching
 Test scorer on sample segments

Hour 2: Integration

 - Design and test LLM prompt
 - Implement extractor (with mock option for testing)
 - Build pipeline to connect everything
 - Test end-to-end on provided transcripts

Hour 3: Polish

 Create demo script showing results
 - Add basic visualizations (score distribution, timeline)
 - Write this RFC
 - Test thoroughly and document

What's Working
The pipeline successfully:
- Scores all segments based on linguistic patterns
- Filters to ~20-30% of segments
- Extracts structured action items with all required fields
- Provides confidence levels and priorities
- Exports to JSON for downstream use

