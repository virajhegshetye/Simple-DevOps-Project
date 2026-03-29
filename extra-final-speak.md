🎤 Decision Engine API – KT Speaking Script

---

1. Introduction

Hi everyone, today I’ll walk you through the Decision Engine API.
I’ll keep this simple and follow the exact execution flow so it’s easy to understand.

At a high level, this service takes input, prepares a structured JSON for PCSM, calls the PCSM engine, and returns a processed decision response.

---

2. High-Level Flow

Let me start with the overall flow.

A request comes from the client and hits the controller.
From there it goes to the service layer.

Inside the service, we first fetch data from the database. This is what we call shortJSON.

Then we process different blocks using sequence tasks.

After that, we build the full DAJSONDocument, which is the required input format for PCSM.

We call the PCSM API using NTJSEMJSONInterface.

Once we receive the response, we process it and send a final response back to the client.

---

3. shortJSON vs inputJSON vs outputJSON

Now let’s understand the three important JSONs.

First is shortJSON.
This is a partial JSON stored in DB. It does not contain all blocks.

Second is inputJSON.
This is the full JSON we send to PCSM. Here we add missing blocks like PARAMTER, RESULTS, SUMMARY, etc.

Third is outputJSON.
This is the response we get from PCSM. It has updated values, especially in SUMMARY and RESULTS.

So basically:
shortJSON → enriched → inputJSON → PCSM → outputJSON

---

4. Block Processing (Sequence)

Each block is processed using tasks.

We use Callable-based tasks, so execution is modular and can be parallel if needed.

Each task handles one responsibility.

For example, SdsParameterTask handles PARAMTER block.

This keeps the system clean and scalable.

---

5. PARAMTER Block (Most Important)

Now I’ll focus on the most critical part, which is PARAMTER.

This block has around 5302 fields, so it’s very large.

We don’t get this from request. Instead, we fetch it from DB using VecSdsparam table.

The flow is:

Repository → DAO → Mapper → Paramter object → set into DAJSONDocument

This block is mandatory before calling PCSM.

---

6. Mapper Problem & Solution

Because PARAMTER has 5302 fields, MapStruct fails to generate implementation.

So instead of automatic generation, we used a different approach.

We generate the mapper interface using a CSV-based script.

Then we manually generate the Impl class using another script.

We also comment out @Mapping annotations to avoid compile issues.

So final setup is:
Generated interface + manually generated Impl

---

7. CSV Driven Mapping

All PARAMTER fields are maintained in a CSV file.

This CSV contains:
logicalName, externalName, type, array size, etc.

We use this CSV to generate mapping code automatically.

Steps are:
Update CSV → run generator → generate mapper → run impl generator → get final class

This avoids writing 5000+ fields manually.

---

8. PCSM Integration

Once all blocks are ready, we call PCSM using NTJSEMJSONInterface.

We pass DAJSONDocument as input.

PCSM processes the request and returns updated JSON.

Important outputs come in:
SUMMARY, RESULTS, MQSUMMARY

We use these to build final response.

---

9. Exception Handling

Exception handling is centralized using ControllerExceptionHandler.

If anything fails in flow:
It is caught and mapped to a standard error response.

We use StrategicDecisionErrors enum for consistent error codes.

We also log correlationId for tracing.

---

10. Logging

We use MDC for logging.

We log:
productAppId
correlationId
execution time

This helps in debugging and tracking request flow.

---

11. Database Usage

We use repositories for:

Fetching request JSON
Saving request/response
Fetching PARAMTER data

Key repositories include:
If024JsonDtlsRepository
ThmdeaudRepository
ThmderesRepository
VecSdsparamRepository

---

12. SMPOJO Jar

DAJSONDocument and related classes come from SMPOJO jar.

This jar is built using GitLab pipeline and added as dependency.

It defines the structure required by PCSM.

---

13. Adding New Block

If we need to add a new block:

We update DAJSONDocument
Add mapping
Create task
Update sequence config
Update CSV if required

So changes are spread across multiple layers.

---

14. Final Summary

To summarize everything:

We fetch shortJSON from DB
Build full input JSON
Add PARAMTER and other blocks
Call PCSM
Process output JSON
Return final response

That’s the complete flow of the system.

---

15. Closing Line

That’s all from my side.
I can take questions or deep dive into any specific part if needed.

---