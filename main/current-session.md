# Current Session Memory - RAM
*Temporary working memory — resets each session, provides context when AI restarts*

## Session RAM Status
**Current Session**: [Active/New]
**Last Activity**: [Timestamp]
**Active Service**: [Service being worked on]
**Current Task**: [What we're doing right now]
**Context State**: [Development / Debugging / Design / Review]

## Working Memory (RAM)

### Active Context
- **Service Focus**: [Which service we're currently working on]
- **Current Task**: [Specific task being performed]
- **Recent Changes**: [What was just modified or created]
- **Blocked By**: [Any current blockers or issues]
- **Next Steps**: [What comes next]

### Files Recently Modified
| File | Service | Change |
|------|---------|--------|
| [file_path] | [service] | [description of change] |

### Open Questions
- [Question 1 that needs resolution]
- [Question 2 that needs resolution]

## Session Recap (For AI Restart)
*Quick summary when AI loads after close/reopen*

- **Previous Session Summary**: [Key accomplishments from last conversation]
- **Where We Left Off**: [Exact point in work for continuity]
- **Important Context**: [Critical info AI needs for continuing]
- **Active Branch**: [Git branch being worked on]
- **Services Running**: [Which containers are up]
- **Known Issues**: [Current bugs or problems being investigated]

## Debug Context (If Debugging)

### Current Investigation
- **Symptom**: [What's going wrong]
- **Affected Service(s)**: [Which services]
- **Hypothesis**: [Current theory about the cause]
- **Tried So Far**: [What debugging steps were already taken]
- **Logs/Evidence**: [Key log entries or error messages]

### Debug Trail
```
1. [First thing checked] -> [Result]
2. [Second thing checked] -> [Result]
3. [Current investigation step]
```

## Architecture Changes This Session

### Decisions Made
| Decision | Rationale | Affected Services |
|----------|-----------|-------------------|
| [Decision] | [Why] | [Services] |

### Pending Decisions
| Decision Needed | Options | Deadline |
|----------------|---------|----------|
| [Decision] | [Options] | [When] |

## Session Lifecycle

### Session Start
- Load recap from previous session
- Check which services are running
- Resume from last active context

### During Session
- Track active service and task
- Record changes and decisions
- Maintain debug context if debugging
- Queue architecture changes for save

### Session End (On "save")
- Persist important learnings to permanent files
- Update recap section for next session
- Clear detailed working memory
- Note any pending work

## What Gets Cleared Each Session
- Detailed conversation progress
- Temporary debugging observations
- Step-by-step work history
- Session-specific file modification logs

## What Persists (Recap Only)
- Brief summary of last session
- Where work left off
- Critical context for continuity
- Active git branch and running services
- Known issues being investigated

---

**Memory Type**: RAM — Temporary Working Memory
**Persistence**: Brief recap only; detailed content clears each session
**Purpose**: Immediate development context + restart continuity

*This file acts like computer RAM — active during session, provides restart recap, then clears for next session*
