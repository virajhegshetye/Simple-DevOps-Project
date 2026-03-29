🎯 KT Q&A – What Team Might Ask (with Answers)

---

1. Why are we using shortJSON instead of sending full JSON directly?

Answer:

We use shortJSON to reduce payload size and avoid passing unnecessary data from upstream systems.
Only required data is stored in DB, and the remaining structure is built internally before calling PCSM.

This gives us:

- Better performance
- More control over data enrichment
- Flexibility to inject PARAMTER and other blocks

---

2. Why is PARAMTER not part of shortJSON?

Answer:

PARAMTER is not request-specific.
It is configuration-driven and stored in DB.

Since it contains around 5300+ fields, sending it in every request is not efficient.

So we fetch it separately and inject it during processing.

---

3. Why did MapStruct fail for PARAMTER mapping?

Answer:

MapStruct generates code at compile time, but PARAMTER has 5300+ fields.

Because of this:

- Generated class size becomes too large
- Compilation fails or becomes unstable

So we switched to:

- CSV-based code generation
- Manual Impl class

---

4. Why are we using CSV for mapping?

Answer:

CSV acts as a single source of truth for all PARAMTER fields.

Benefits:

- Easy to maintain
- No need to write 5000+ mappings manually
- Can regenerate code anytime

---

5. What happens if a new field is added in PARAMTER?

Answer:

We update the CSV file with the new field.

Then:

- Run mapper generator
- Run impl generator

This automatically updates mapping code without manual effort.

---

6. What happens if a block is missing in shortJSON?

Answer:

If a required block is missing:

- It may be handled in sequence tasks
- Or it can result in validation error

Critical blocks like SPAREFIELD are validated early.

If missing, we log error and stop processing.

---

7. What is the role of SPAREFIELD?

Answer:

SPAREFIELD contains metadata like parameter change datetime.

This is used to:

- Fetch correct PARAMTER configuration
- Ensure we use latest or relevant parameter set

---

8. Why are we using Callable tasks instead of simple methods?

Answer:

Callable allows:

- Parallel execution (if needed)
- Better modularity
- Independent block processing

It improves scalability and maintainability.

---

9. What is NTJSEMJSONInterface?

Answer:

It is the PCSM interface used to execute decision logic.

We pass DAJSONDocument as input, and it returns processed JSON with decisions.

---

10. Which blocks are most important in output?

Answer:

The most important blocks are:

- SUMMARY → final decision
- RESULTS → detailed output
- MQSUMMARY → additional decision data

These are used to build response.

---

11. How do we handle errors from PCSM?

Answer:

If PCSM fails:

- Exception is caught
- Wrapped in StrategicDecisionException
- Mapped using StrategicDecisionErrors
- Returned as standardized response

---

12. Why do we use correlationId?

Answer:

correlationId helps in tracing a request across systems.

It is used in:

- Logs
- Error responses
- Performance tracking

---

13. What is stored in DB tables like If024JsonDtls?

Answer:

These tables store:

- Request JSON
- Response JSON
- Execution details

Used for:

- Audit
- Debugging
- Reprocessing if needed

---

14. What happens if PARAMTER mapping fails?

Answer:

If mapping fails:

- PARAMTER block will be null or incomplete
- PCSM call may fail or give incorrect output

So we log error and mark execution failure.

---

15. Can we optimize PARAMTER loading?

Answer:

Yes, possible improvements:

- Cache PARAMTER in memory
- Load only once instead of every request

But currently it is DB-driven for flexibility.

---

16. What changes are needed to add a new block?

Answer:

We need to update:

- DAJSONDocument
- Mapper
- Task class
- Sequence config
- Possibly CSV (if parameter-related)

---

17. Why is Impl class manually generated?

Answer:

Because MapStruct cannot handle very large mappings.

Manual Impl gives:

- Full control
- Avoids compilation issues
- Stable execution

---

18. Is this system scalable?

Answer:

Yes, because:

- Modular task-based execution
- CSV-driven mapping
- Clear separation of concerns

Only heavy part is PARAMTER, which can be optimized using caching.

---

19. Where is most complexity in system?

Answer:

PARAMTER block.

Because:

- Very large (5302 fields)
- Requires custom mapping solution
- Impacts PCSM execution

---

20. If something breaks, where do we debug first?

Answer:

Start with:

1. Logs (correlationId)
2. shortJSON from DB
3. PARAMTER mapping
4. Input JSON sent to PCSM
5. PCSM response

This helps isolate issue quickly.

---

🔚 Closing Tip

If you don’t know an answer, say:

“I’ll need to check that in detail, but based on current understanding…”

This sounds confident and honest.

---