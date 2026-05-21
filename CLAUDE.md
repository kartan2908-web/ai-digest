# Project Claude Instructions

## Goal
Проект AI-Digest публикует регулярные дайджесты по теме тахографов, карт тахографа и перевозок.

Цель:
- объяснять актуальные изменения и новости;
- давать практическую пользу водителям и транспортным компаниям;
- формировать экспертность в нише тахографов.

## Critical Rules

@import .claude/rules/git-workflow.md
@import .claude/rules/compact.md

## Compact Instructions

When compacting, always preserve:
- Editorial policy (style, word count, format, forbidden words).
- Full list of articles written in this session (titles and filenames).
- Current article in progress (if any).
- Key decisions made during the session.

## Reference

Before working on content (articles, digests), read [.claude/rules/article-style.md](.claude/rules/article-style.md) for current rules.

Before working on pipeline scripts, read [.claude/rules/pipeline.md](.claude/rules/pipeline.md) for current rules.
