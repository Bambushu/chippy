# Runner Preamble

This block is prepended to every chip prompt at forge time. The spawned session reads it at startup and runs the ritual below before executing the actual task.

**Copy the entire block between the dashed lines below into the chip prompt, at the very top.**

--------------------------------------------------------------------------------

> **Runner ritual (Chippy) — run this before any other work.**
>
> You were spawned as a chip by a parent session. Do not skip these steps. They exist because chips fail when they start executing before confirming understanding.
>
> 1. **Echo check.** In one sentence, state what you understand this chip is doing. If the prompt is ambiguous, or you'd need to guess between two reasonable interpretations, STOP. Report the ambiguity back as your final message and do NOT execute.
>
> 2. **Plan.** Break the work into 3–7 concrete steps using the TodoWrite tool. Each step should be checkable ("test X passes", "file Y updated"), not vague ("improve Z").
>
> 3. **Verify starting state.** Confirm the files, branches, and tests the prompt references actually exist and are in the expected state. Run the "starting pointers" commands from the prompt if provided. If anything is missing or unexpected, STOP and report.
>
> 4. **Treat "Observed so far" as provisional.** The parent session listed things it thought were true. If your own reading of the code contradicts any observation, trust your reading, update your plan, and continue.
>
> 5. **Execute with TDD** where tests exist for the touched code. Write failing test → minimal fix → passing test → continue.
>
> 6. **Exit check.** Before declaring done, verify every success criterion from the prompt. Run the exact verification commands. Paste the output into your final report.
>
> 7. **Final report.** Include: what you changed (files + one-line summary each), the verification output, any scope-fence items you hit that forced a stop, any assumptions you made.
>
> If at any point the work goes beyond the scope fence, STOP and report rather than expanding scope.

--------------------------------------------------------------------------------

(The actual chip prompt begins below the second dashed line.)
