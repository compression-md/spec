# COMPRESSION

> Context compression and summarization protocol for AI agents.
> Spec version: 1.0 | Full specification: https://compression.md

---

## PURPOSE
# Define rules for compressing context when approaching token limits.
# Preserve critical information. Discard redundant history.
# Maintain coherent reasoning after compression.

---

## TRIGGERS
# Conditions that initiate compression.

context_approaching_limit:
  threshold_pct: 0.75             # Begin compression at 75% context capacity
  action: incremental_compress    # Compress oldest turns first

token_budget_exceeded:
  budget_tokens: 100000           # Hard token budget per session
  action: full_compress           # Aggressive compression of full history

scheduled_compression:
  enabled: true
  interval_tokens: 50000          # Compress every 50,000 tokens regardless
  action: checkpoint_compress     # Save checkpoint before compressing

---

## STRATEGY
# Rules for what gets compressed, preserved, and discarded.

preserve_always:
  - system_prompt                 # Never compress the system prompt
  - active_task_context           # Current task instructions and state
  - last_n_turns: 3               # Always keep 3 most recent exchanges verbatim
  - explicit_bookmarks            # Items flagged by agent as important
  - error_states                  # Recent errors and their context
  - pending_actions               # Uncommitted planned actions

compress_aggressively:
  - exploratory_turns             # Thinking-aloud and brainstorming
  - repeated_information          # Information mentioned multiple times
  - verbose_tool_outputs          # Long API responses (keep summary only)
  - superseded_plans              # Plans that were revised or abandoned

discard_safely:
  - completed_subtask_detail      # Finished work with confirmed output
  - redundant_acknowledgements    # OK, understood, got it, etc.
  - intermediate_calculations     # Final answer replaces working steps

compression_ratio_target:
  aggressive: 0.30                # Compress to 30% of original size
  standard:   0.50                # Compress to 50% of original size
  light:      0.75                # Compress to 75% of original size

---

## VERIFICATION
# Checks to run after compression to ensure coherence.

post_compression_checks:
  semantic_consistency: true      # Key facts still present after compression
  task_continuity: true           # Active task context preserved
  error_trail: true               # Recent errors still referenced
  max_acceptable_loss: 0.10       # Fail if >10% of key facts lost

on_verification_failure:
  action: restore_from_checkpoint # Revert to pre-compression state
  notify: true                    # Alert operator
  escalate_to: COLLAPSE.md        # Hand off to collapse prevention

---

## AUDIT

log_file: .compression.log
log_format: jsonl
log_fields:
  - timestamp
  - trigger_type
  - tokens_before
  - tokens_after
  - compression_ratio_achieved
  - verification_passed
  - session_id

---

## METADATA

owner: your-name-or-org
contact: ops@example.com
last_reviewed: 2026-03-11
review_frequency: quarterly
spec_version: "1.0"
spec_url: https://compression.md
