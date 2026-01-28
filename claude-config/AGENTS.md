
## Subagents

- ALWAYS wait for all subagents to complete before yielding.
- Spawn agents automatically when:
  - Parallelizable work (e.g. install + verify, npm test + typecheck, multiple tasks from plan)
  - Long-running tasks or blocking tasks where a worker can run independently
  - Isolation for risky changes or checks
