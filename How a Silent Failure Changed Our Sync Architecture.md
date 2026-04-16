## TL;DR

We had a subtle but critical payroll bug: deductions were getting silently dropped due to timezone mismatches between us (Rippling) and a partner system (Employee Navigator). What looked like a simple `effective_date == pst_now()` check turned into a case study in why real-world systems need to be resilient to time, contracts, and tacit assumptions. We solved it with a ScheduledDeductions model and a batch job system. Here's the story.

---

## Background

At Rippling, our 3P Ben Admin team integrates with external benefit platforms like Employee Navigator to sync deduction data with our internal payroll engine. The catch? Rippling Payroll **doesn't support future-dated deductions.**

So we had an agreement with our partner: they'll send us deduction updates only **on the day** those deductions are meant to take effect. Our logic was simple:

```python
if deduction.effective_date == pst_now().date():
    sync_to_payroll(deduction)
```

And for a while — it worked. Until it didn’t.

---

## The Bug: When "Today" Isn’t the Same Day

We started noticing missing deductions. No crashes, no retries, no alerts — just... missing deductions in payroll.

After digging through logs, we found a recurring pattern:

- The partner sent the data at **1:55 AM CST** on `2025-04-14`.
    
- But `pst_now()` on our end still read **2025-04-13, 11:55 PM PST**.
    
- So our equality check failed, and we skipped syncing the deduction.
    

Yup — we were off by one hour, but it cost us payroll data.

The partner **was** technically sending data on the right date. But **in their timezone**, not ours.

---

## The Fix: ScheduledDeductions + Batch Processing

We realized that our implementation was too brittle. Timezone edge cases shouldn’t cause silent data loss. So we moved to an architecture built for this.

### 🔧 Enter: `ScheduledDeductions`

Instead of syncing deductions immediately, we now:

1. Store every incoming deduction update in a `ScheduledDeductions` model
    
2. Add metadata like `effective_time`, `companyId`, `employeeId`, etc.
    
3. Set the status to `"Scheduled"`
    

### 🕒 Batch Job FTW

Every 6 hours, a cron job runs:

- Fetches deductions where `effective_time <= pst_now() - timedelta(hours=6)` and `status = 'Scheduled'`
    
- Applies them in batch to `EmployeeDeductionType`
    
- Marks success as `COMPLETED`, and collects failures into an `errors` bucket
    

### 🧠 Why This Works

- **Handles future-dated deductions** reliably
    
- **Decouples from timezone-sensitive assumptions**
    
- **Adds observability**: we now know how many deductions succeeded or failed
    
- **Scales**: 1,000+ deductions processed in seconds
    

---

## Architecture Diagram

![[Pasted image 20260416104535.png]]

Payroll EngineEmployeeDeductionTypeCron Job (Every 6 hours)ScheduledDeductions DBBen Admin Service3P VendorPayroll EngineEmployeeDeductionTypeCron Job (Every 6 hours)ScheduledDeductions DBBen Admin Service3P VendorCron runs every 6 hoursalt[Success][Failure]loop[For each eligible deduction]alt[If all succeeded][If any errors]Send deduction requestSave as Scheduled with effective_time and status = "Scheduled"Query deductions where effective_time ≤ now - 6h and status = "Scheduled"Return eligible deductionsUpdate EmployeeDeductionTypeMark status as "COMPLETED"Add to error_detailsChanges picked up in next payrunTrigger alert with errors

---

## Lessons Learned

1. **Equality checks on time are dangerous.** Always normalize or buffer.
    
2. **Tacit contracts break easily.** Make assumptions explicit.
    
3. **Timezones are where bugs go to hide.**
    
4. **Batch systems are more resilient than real-time syncs** when dealing with external inputs.
    

---

## Final Thoughts

This wasn’t the flashiest bug I’ve fixed, but it was the most _quietly devastating_. And solving it felt like a turning point — not just technically, but in how I thought about system design.

We didn't just fix a bug. We turned a fragile sync system into a robust, scalable pipeline.