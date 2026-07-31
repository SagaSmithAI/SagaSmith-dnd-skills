# Combat turns and choices

Resolve pending owned choice and reaction windows before ending or advancing a
turn. Ready declarations identify trigger, intended response, target rules,
resource commitment, and concentration implications; release occurs only
through the matching server window.

`combat_choice` may resolve a real open choice, source-bound on-hit ruling, or
first-use custom content plan. It must validate the current attack/event,
operator, revision, and pending window. It is not a general free-form mutation
tool.

End the turn only after required action costs, saves, ongoing effects, death,
and concentration consequences are settled.
