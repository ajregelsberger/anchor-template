# Transcription Gotchas — {{YOUR_NAME}}'s Anchor

> Known auto-transcription errors for names and terms in {{YOUR_COMPANY}}'s environment.
> Auto-transcription tools (Gemini, Otter, etc.) regularly mangle proper nouns.
> Update this file whenever you catch a consistent error.

---

## People (common mis-transcriptions)

| Actual name | Common error | Notes |
|------------|-------------|-------|
| {{YOUR_NAME}} | [phonetic mangling] | |
| {{TEAM_MEMBER_1}} | [phonetic mangling] | |

> **Add entries here.** When you notice Claude using a wrong name that came from a transcript, add the correction here so future sessions catch it automatically.

---

## Domain Terms

| Correct term | Common error | Notes |
|-------------|-------------|-------|
| [Your term] | [Mis-transcription] | |

> **Example:** In one anchor, "bordereaux" was consistently transcribed as "border row." Add similar domain-specific corrections for your field.

---

## How This File Is Used

The `context-ingestion` and `session-start` skills load this file before processing any transcript. Every name and term in incoming material is cross-referenced here before signal is extracted.

*This file is always loaded — it applies to every session.*
