# 🔥 High-Probability Questions & Answers

---

## 1. Why are we using parallel execution here?

**Answer:**
“We have 20 independent data blocks. Instead of processing them sequentially, we execute them in parallel using executor.invokeAll. This reduces total request latency significantly.”

---

## 2. Why invokeAll() instead of Future.get()?

**Answer:**
“invokeAll() handles bulk execution with timeout and returns only when all tasks complete or timeout occurs. It simplifies control flow and avoids managing individual Future.get calls.”

---

## 3. How do we ensure all tasks completed successfully?

**Answer:**
“We validate using validateFutureList. It checks:
- All tasks are done
- None are cancelled
- No executionErrorMsg is present in PcsmRequest”

---

## 4. Where is aggregation happening?

**Answer:**
“There is no separate aggregation layer. All tasks directly update the shared PcsmRequest object, which contains DAJSONDocument.”

---

## 5. Is PARAMTER fetched from DB for every request?

**Answer (important):**
“No. PARAMTER is loaded once at startup using @PostConstruct in SdsParameterMapperImpl and stored in PcsmInitDataBlock. During request, SdsParameterTask reads from this cache.”

---

## 6. Why do we use parameterChangeDateTime?

**Answer:**
“It ensures the correct version of PARAMTER is used based on request context. The task uses this value to fetch appropriate cached data.”

---

## 7. What happens if PARAMTER CSV and DB are not aligned?

**Answer:**
“It leads to incorrect mapping and wrong decision output, because mapper expects exact structure matching DB data.”

---

## 8. Why is PARAMTER mapper generated manually?

**Answer:**
“Because PARAMTER has 5302 fields. MapStruct fails to generate implementation for such a large block, so we use custom generator scripts.”

---

## 9. What is the purpose of warmUP()?

**Answer:**
“It preloads JSON parsing, utility methods, and executes a sample PCSM call to avoid first-request latency.”

---

## 10. Why is MDC used?

**Answer:**
“MDC stores request-level identifiers like productAppId and correlationId, enabling consistent log tracing across parallel threads.”

---

## 11. What happens if EXPECTED_TASK_COUNT is wrong?

**Answer (critical):**
“The system throws DES002 error because task count validation fails. This is a common failure when adding new blocks.”

---

## 12. Where is PCSM called?

**Answer:**
“Inside DecisionServiceImpl using NTJSEMJSONInterface from sp5 jar.”

---

## 13. What happens after PCSM execution?

**Answer:**
“We perform:
- Business error validation
- Extract generation
- Timeout validation
- Store response in DB
- Build final response”

---

## 14. What is checkBusinessControlErrors doing?

**Answer:**
“It checks OCONTROL block for error count. If errors exist, it throws DES003 with error codes.”

---

## 15. Why do we validate request time?

**Answer:**
“To ensure total execution time does not exceed configured timeout. If exceeded, request is failed.”

---

## 16. What is stored in DB after execution?

**Answer:**
“We store:
- Output JSON
- Status (SUCCESS/ERROR)
- Metadata like sequence number and timestamps”

---

## 17. What is addEntryToIf024JsonDtls?

**Answer:**
“It stores audit data for the request if configuration flag is enabled.”

---

## 18. What is SMPOJO?

**Answer:**
“It’s a generated jar from SM team JSON, containing DAJSONDocument and all block models used in request/response.”

---

## 19. What changes are required when adding a new block?

**Answer:**
“We need updates in:
- SMPOJO
- ApiConstants (including EXPECTED_TASK_COUNT)
- PcsmConfiguration
- CSV mapping
- Mapper
- Sequence
- Task”

---

## 20. What is the biggest risk area in this system?

**Answer:**
“PARAMTER block — due to its size, manual mapper, and dependency on CSV and DB alignment.”

---

# ⚡ Bonus: Trick Questions

---

## Q: Is execution sequential or parallel?

**Answer:**
“Parallel using executor.invokeAll.”

---

## Q: Is there any caching in the system?

**Answer:**
“Yes — PARAMTER is cached at startup in PcsmInitDataBlock.”

---

## Q: Where is business logic implemented?

**Answer:**
“In DecisionServiceImpl and block-level tasks, not in controller.”

---

# 💬 Final Tip

If you don’t know an answer:

“Let me quickly confirm that — but as per current understanding…”

(Then answer confidently based on flow)