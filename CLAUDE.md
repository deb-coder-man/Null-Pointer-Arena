# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Context Files

Read these at the start of every session:
- `.claude/product-research.md` — market research, MVP features, game loop, ELO system design, UX recommendations, retention mechanics, and launch strategy
- `.claude/pages.md` — full page inventory with URLs and auth requirements

## Project Overview

Competitive Debugging is a web-based 1v1 game where two developers race to find and fix bugs in real code. Each match gives both players the same broken program, an error message without line numbers, and a description of what the code is supposed to do. The first player to make all test cases pass wins.

A lightweight power-up system adds a strategic layer — players can pause to answer a quiz question and earn a power-up (defensive: reveals a failing test case; offensive: delays opponent's error output 15s). The decision to farm power-ups or stay focused on the code is a deliberate trade-off.

**Target users:** Job-seeking developers (interview prep) and competitive programmers (recreational). Desktop-only MVP.

**MVP success metrics:** 40%+ of new users complete first match and start a second, 35%+ 48-hour return rate, Easy matches average 7–12 min, 1,000 accounts within 30 days.

**What ships in v1:** Real-time 1v1 matchmaking (Practice + Ranked queues), code editor, server-side test runner, 30 problems (Easy/Medium/Hard), Python/JavaScript/Java, bot fallback, ELO rating with developer job title tiers (Syntax Error → Junior Dev → Mid-Level → Senior Dev → Staff Engineer → Principal), post-match bug breakdown.

**Out of scope for v1:** Spectator mode, tournaments, daily challenges, mobile, in-match chat, user-submitted problems, B2B/private rooms.

## Setup

<!-- TODO: Add setup/installation steps -->

## Development Commands

<!-- TODO: Add build, lint, test, and run commands -->

## Architecture

<!-- TODO: Describe high-level architecture and key design decisions -->
