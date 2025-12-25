Exercise C — On-Device Light Parsing
Overview

This repository contains my solution for Exercise C: On-Device Light Parsing as part of the Ashwam Backend Engineering Internship take-home assignment.

The goal of this exercise is to build a deterministic, privacy-first parsing system that extracts food and symptom signals from messy, free-text journal entries without using LLMs, cloud APIs, or heavy NLP libraries.

The system is designed to be:

Lightweight and local

Deterministic (same input → same output)

Easy to reason about and test

Runnable entirely via a CLI

High-Level Design

The solution is intentionally split into three explicit components, each with a single responsibility.

1. FoodParser

Extracts only food-related signals from text.

Responsibilities:

Detect food items (including Hinglish terms)

Extract quantities when present

Infer meal context (breakfast / lunch / dinner / unknown)

Normalize plural forms

Handle skipped meals

2. SymptomParser

Extracts only physical symptoms from text.

Responsibilities:

Detect symptom names

Handle negation (e.g. “no headache”)

Extract severity (e.g. 6/10)

Detect time hints (morning, evening, night, after meal)

Ignore vitals, steps, and unrelated numeric data

3. LightParsePipeline

A thin orchestrator that:

Runs FoodParser and SymptomParser independently

Merges their outputs into a single structured result

Adds metadata such as parser version

CLI

The CLI is intentionally minimal and is responsible only for file I/O and streaming.
All parsing logic lives inside the parsers and pipeline.

Project Structure
```
light-parse/
├── src/
│   ├── parsers/
│   │   ├── FoodParser.js
│   │   └── SymptomParser.js
│   ├── pipeline/
│   │   └── LightParsePipeline.js
│   └── cli.js
│
├── tests/
│   ├── foodParser.test.js
│   ├── symptomParser.test.js
│   └── pipeline.test.js
│
├── data/
│   └── entries.jsonl
│
├── output/
│   └── parsed.jsonl   (generated)
│
├── package.json
├── jest.config.js
└── README.md
```
Input Format

Input is provided as JSONL (JSON Lines).

Each line represents a single journal entry:
```
{"entry_id":"e_001","text":"2 eggs + toast. Cramps started by noon 😣"}
```
Output Format

For each input entry, the system produces a structured JSON object:
```
{
  "entry_id": "e_001",
  "foods": [],
  "symptoms": [],
  "parse_errors": [],
  "parser_version": "v1"
}

```
Output is written in JSONL format, one entry per line.

How to Run
Install dependencies
```
npm install
```
Run tests
```
npm test
```
Run the parser (recommended)
```
npm run dev
```

npm run dev is a convenience script that runs the CLI with the required input and output arguments.

Equivalent raw command
```
node src/cli.js --in data/entries.jsonl --out output/parsed.jsonl
```
CLI Output (Human-Readable Preview)

While running, the CLI prints a short preview for each entry to the terminal:
```
[e_001]
  Foods: egg(2), toast
  Symptoms: cramp
```

This preview is intended for debugging and demos only.
The canonical output is the structured JSONL written to output/parsed.jsonl.

Testing Strategy

Tests are included for:

FoodParser

food detection

quantity extraction

skipped meals

SymptomParser

negation handling

severity extraction

time hints

LightParsePipeline

integration of both parsers

All logic is deterministic and independently testable.

Design Decisions & Tradeoffs

Parsing is intentionally rule-based and conservative

The system prefers under-parsing over over-inference

No diagnoses or emotional states are inferred

Vitals (BP), steps, and unrelated numbers are ignored

Logic is kept simple, explainable, and extensible

Known Limitations

Some overlapping symptoms (e.g. “pain” and “back pain”) may both appear

Quantity and unit extraction is basic

Negation handling is limited to simple patterns (e.g. “no X”)

Meal detection relies on keyword heuristics

Confidence scores are heuristic-based, not learned

These limitations are intentional to keep the system lightweight and predictable.

Future Improvements

With more time or additional tooling, this system could be extended to:

Deduplicate overlapping symptoms

Improve negation scope handling

Add richer unit extraction

Expand normalization for Hinglish and synonyms

Make lexicons configurable

Notes

This project is implemented as a local CLI utility only.
No frontend or API layer is included, as the focus is on backend parsing logic, clarity of design, and deterministic behavior.
