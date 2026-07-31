# Problem-Framing Memo
**Project:** IT Asset Management — Predicting Laptop Hardware Failure

## Problem Statement
Company laptops sometimes fail suddenly (battery dies fast, fan breaks, disk starts failing), causing staff downtime and rushed, unplanned replacements. The IT team currently only reacts after a laptop breaks. We want a model that flags laptops likely to fail soon, so IT can service or replace them before they cause a work disruption.

## Target Definition
The model predicts:
- **Will this laptop likely fail within the next 30 days?** (yes / no)

"Fail" means a hardware fault serious enough to need a repair ticket or replacement (not a software issue, not a simple reboot fix). The 30-day window is based on typical repair/replacement lead time, so IT has enough notice to act.

## Safe Feature List
- Laptop age (months since purchase)
- Battery health percentage (from device diagnostics)
- CPU/fan temperature readings over time
- Disk read/write error counts
- Number of past repair tickets for that specific device
- Device model and manufacturer

## Leakage Exclusions
These must NOT be used, because they only become known *after* the failure already happened:
- The date the device was actually replaced
- Repair technician's diagnosis notes written after the failure
- Whether a replacement laptop was ordered
- Any ticket closed with "hardware failure confirmed" as the reason

## Proposed Train / Validation / Test Split
- **Train (70%):** diagnostic history from older time periods
- **Validation (15%):** a later time slice, used to tune the model
- **Test (15%):** the most recent time slice, used once at the end to check real performance

Split by time (not randomly), so the model is tested on data that comes *after* what it trained on — this matches how it will be used: predicting the future, not the past.

## Success Metrics
- **Recall on real failures:** how many actual failures the model correctly catches in advance (this matters most — missing a real failure defeats the purpose)
- **False alarm rate:** how often a laptop is flagged but doesn't actually fail (too high, and IT starts ignoring the alerts)
- **Average lead time:** how many days before failure the model gives warning, on average
- **Cost saved:** fewer emergency replacements vs planned ones

## Top Three Risks & Mitigations

**1. Risk:** The model misses a real failure (false negative), so a laptop still fails unexpectedly and causes downtime.
**Mitigation:** Set the model to be cautious — prefer flagging borderline cases rather than staying silent — and keep a simple manual reporting option for staff to flag laptop issues themselves as a backup.

**2. Risk:** Too many false alarms cause IT to waste time checking laptops that were actually fine, and they start ignoring the alerts over time.
**Mitigation:** Regularly review flagged cases against real outcomes, and adjust the alert threshold so the false alarm rate stays at a level IT can realistically act on.

**3. Risk:** Diagnostic data quality varies (older or non-standard laptop models may not report all sensors), making predictions less reliable for those devices.
**Mitigation:** Clearly mark predictions as "low confidence" when key sensor data is missing, and fall back to normal reactive support for those specific devices instead of trusting the model blindly.
