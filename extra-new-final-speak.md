# 1. Overview

“This API acts as an orchestration layer between the database and PCSM.”

“It fetches request data from DB, builds DAJSONDocument, executes all blocks in parallel, calls PCSM, processes the response, and returns a summarized output.”

“The core class handling everything is DecisionServiceImpl.”

---

# 2. Controller Layer

“Controller is very thin.”

“It validates request type using DesType.isRequest, extracts productAppId, starts SLA timing, sets MDC for logging, and calls the service layer.”

“MDC helps in tracking logs across parallel execution.”

“No business logic is handled in controller.”

---

# 3. End-to-End Flow

“Request comes into controller and goes to DecisionServiceImpl.”

“First we fetch data from DB and validate it.”

“Then we convert Blob to JSON map.”

“Next we create PcsmRequest object.”

“Then we create tasks for all blocks and execute them in parallel using executor.invokeAll.”

“After all tasks complete, we build input JSON and call PCSM.”

“Once response is received, we validate business errors, generate extract, validate execution time, store response, and return final output.”

“Extract is generated using ExtractService — detailed logic is documented separately.”

---

# 4. Build Phase

“During build, we load two main dependencies.”

“SMPOJO jar contains DAJSONDocument and all block models.”

“sp5 jar provides NTJSEMJSONInterface for PCSM execution.”

“MapStruct generates most mappers, but PARAMTER mapper is manually generated due to its large size.”

---

# 5. Application Startup & Warmup

“At startup, Spring initializes all beans.”

“Then @PostConstruct warmUP runs.”

“It preloads JSON parsing, utility methods, and executes a sample PCSM call.”

“This ensures no latency during first request.”

---

# 6. PARAMTER Caching

“This is a critical part.”

“At application startup, SdsParameterMapperImpl loads PARAMTER data from vec_sdsparam table.”

“It maps it and stores it in PcsmInitDataBlock in memory.”

“During request processing, SdsParameterTask reads from this cache using parameterChangeDateTime.”

“It does not hit the database again.”

---

# 7. Parallel Execution

“We create 20 Callable tasks, one for each block.”

“These are executed using executor.invokeAll with a timeout.”

“There is no Future.get or aggregation.”

“All tasks directly update the shared PcsmRequest object.”

“After execution, we validate all tasks are completed successfully.”

---

# 8. Data Blocks

“There are 20 blocks configured in PcsmConfiguration.”

“This count must always match EXPECTED_TASK_COUNT.”

“If mismatch happens, system throws DES002 error.”

---

# 9. JSON Structures

“shortJSON comes from DB and is the initial input.”

“inputJSON is DAJSONDocument built by our system.”

“outputJSON is returned by PCSM and used for response and DB storage.”

---

# 10. Block Sizes

“These sizes come from PCSM output structure.”

“PARAMTER is the largest block with 5302 fields.”

“RESULTS and SUMMARY are also large and important.”

---

# 11. PARAMTER Deployment

“PARAMTER values come from CSV files.”

“These are deployed via GitLab pipeline into vec_sdsparam table.”

“At startup, this data is loaded into memory.”

“If CSV, DB, and mapper are not aligned, it leads to incorrect decision results.”

---

# 12. PARAMTER Mapper Generation

“Since PARAMTER has 5302 fields, mapper is generated using scripts.”

“We run generator Java file to create mapper interface.”

“Then run another script to generate implementation class.”

“We never modify generated code manually.”

---

# 13. Exception Handling

“All exceptions are handled centrally.”

“We use StrategicDecisionException with specific error codes.”

“Errors are logged and stored in DB.”

“This ensures consistent error handling.”

---

# 14. Database Layer

“We interact with DB to fetch request, store response, and store audit data.”

“Main methods are getDesRequest, storeDesResponce, and addEntryInIf024JsonDtls.”

---

# 15. SMPOJO

“SMPOJO jar is generated from JSON provided by SM team.”

“It contains DAJSONDocument and all block models.”

“When new blocks are added, SMPOJO must be regenerated.”

---

# 16. Adding New Block

“Adding a new block requires changes in multiple places.”

“The most critical one is updating EXPECTED_TASK_COUNT.”

“If missed, system fails with DES002 error.”

---

# 17. Summary

“Overall flow is: shortJSON → parallel execution → cached PARAMTER → PCSM → response.”

“The system is optimized using parallel processing and caching.”

“PARAMTER is the most critical block in the entire flow.”