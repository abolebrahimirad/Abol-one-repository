
#!/usr/bin/env python3
"""
prompt_generator.p
Generate multi-layer, debate-style prompts for a two-AI scoring system.

Usage:
    python prompt_generator.py --count 10 --format md --seed 42 > prompts.md

Options:
    --count N       Number of prompts to generate (default: 10)
    --format fmt    Output format: md or json (default: md)
    --seed S        Random seed for reproducibility
"""

import argparse
import json
import random
import textwrap
from datetime import datetime

# ---------- Configurable building blocks ----------
TOPICS = [
    "Privacy laws and the future of open internet",
    "AI's impact on global labor markets",
    "Tokenized economies and airdrop incentives",
    "Ethics of social robotics",
    "Game theory in scoring systems",
    "Security in smart contracts",
    "Disinformation and mitigation strategies",
    "Decentralized compute networks",
    "Governance models in DAOs",
    "Environmental impact of mining"
]

STYLES = [
    "Academic analytical",
    "Debate format (Argue & Counter-argue)",
    "Roleplay simulation",
    "Problem-solving & proposal style",
    "Multi-step reasoning with risk conditions",
    "QA hierarchical ladder"
]

SCALES = [
    "Level 1 — Basic: Simple and clear",
    "Level 2 — Intermediate: Requires analysis",
    "Level 3 — Advanced: Multi-layer reasoning + simulation",
    "Level 4 — Strategic: Includes scenario consequences"
]

STEPS_TEMPLATES = [
    textwrap.dedent("""\
    Step 1 — Problem Definition:
    Write a 2–3 sentence description clearly defining the core proble.

    Step 2 — Base Assumptions:
    Provide 3 assumptions that both AIs must follow in their answers.

    Step 3 — Debate:
    - AI_A: Provide a supporting argument (max 250 words)
    - AI_B: Provide a counter-argument (max 250 words)

    Step 4 — Scenario & Scoring:
    Provide a short scenario (possible outcomes) and define 3 scoring metrics.
    """),
    textwrap.dedent("""\
    Step 1 — Role Assignment:
    Each AI takes a distinct stakeholder role (developer, regulator, end-user, etc).

    Step 2 — Impact Simulation:
    Write 2 short scenarios (positive/negative) and list 3-stage consequences for each.

    Step 3 — Practical Proposal:
    Provide a 5-step actionable solution.
    """),
    textwrap.dedent("""\
    Step 1 — Decision Hierarchy:
    Break the problem into 4 key decisions, each with 2 options.

    Step 2 — Risk-Reward Analysis:
    Provide concise bullet lists evaluating risks and rewards for each option.

    Step 3 — Final Judgement:
    Each AI must give a final recommendation based on the defined criteria.
    """)
]

# ---------- Generator logic ----------
def generate_prompt(idx, rng):
    topic = rng.choice(TOPICS)
    style = rng.choice(STYLES)
    level = rng.choice(SCALES)
    steps = rng.choice(STEPS_TEMPLATES)

    pid = f"P{datetime.utcnow().strftime('%Y%m%d%H%M%S')}-{idx:03d}"
    created = datetime.utcnow().isoformat() + "Z"

    title = f"[{level}] {topic} — {style} (#{pid})"

    instruction = textwrap.dedent(f"""\
    Title: {title}

    Summary:
    Goal: Generate competitive responses focusing on creativity, reasoning, and actionability.
    Audience: Two competing AIs (AI_A and AI_B). Each must follow the output structure below.

    Required Output Format for Each AI:
    1) Summary (max 80 words)
    2) Main Argument (max 250 words)
    3) 3 Actionable Recommendations
    4) 1 Sentence describing potential weakness in their own proposal

    Scoring Weights:
    - Creativity & Innovation: 30%
    - Logical Structure & Accuracy: 40%
    - Practicality & Impact: 30%

    Prompt ID: {pid}
    Generated: {created}

    Task Steps:
    {steps}
    """)

    return {
        "id": pid,
        "title": title,
        "topic": topic,
        "style": style,
        "level": level,
        "created": created,
        "instructions": instruction
    }

def output_md(prompts):
    parts = []
    header = f"# Generated Prompts — {len(prompts)} items\n\nGenerated at {datetime.utcnow().isoformat()}Z\n\n"
    parts.append(header)
    for p in prompts:
        parts.append(f"## {p['title']}\n")
        parts.append(f"**ID:** `{p['id']}`  \n**Topic:** {p['topic']}  \n**Style:** {p['style']}  \n**Level:** {p['level']}\n\n")
        parts.append("### Instructions\n")
        parts.append("```\n" + p['instructions'].strip() + "\n```\n")
        parts.append("---\n")
    return "\n".join(parts)

def output_json(prompts):
    return json.dumps({
        "generated_at": datetime.utcnow().isoformat() + "Z",
        "prompts": prompts
    }, ensure_ascii=False, indent=2)

# ---------- CLI ----------
def main():
    parser = argparse.ArgumentParser(description="Multi-layer prompt generator for 2-AI scoring systems.")
    parser.add_argument("--count", type=int, default=10, help="Number of prompts to generate")
    parser.add_argument("--format", choices=("md","json"), default="md", help="Output format")
    parser.add_argument("--seed", type=int, default=None, help="Random seed")
    args = parser.parse_args()

    rng = random.Random(args.seed)

    prompts = [generate_prompt(i, rng) for i in range(1, args.count + 1)]

    if args.format == "md":
        print(output_md(prompts))
    else:
        print(output_json(prompts))

if __name__ == "__main__":
    main()
