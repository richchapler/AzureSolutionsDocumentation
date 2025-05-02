# Does the Secondary Replica in a Failover Group Have to Be Read-Only?

## ✅ Short Answer
**No.**  
When configuring a failover group for **Azure SQL Managed Instance**, the secondary **does not have to be read-only**. You can choose whether it is:

- **Read-only** (accessible for queries)
- Or **standby** (completely inaccessible unless failover occurs)

---

## 🔍 Details

When setting up a failover group (FOG), you're pairing:
- A **primary managed instance** in one region
- With a **secondary managed instance** in another region

You then choose how the **secondary replica** behaves:

### Option 1: **Read-Only Secondary**
- The secondary is **queryable**
- Intended for **read scale-out**, reporting, analytics, etc.
- Clients connect using a special **read-only listener endpoint**
- Useful if you want to extract value from the replica before a failover ever happens

### Option 2: **Standby Secondary**
- The secondary is **not accessible** at all during normal operation
- Comes online **only after failover**
- Not used for reporting, backups, or queries
- Preferred for **critical transactional systems** where read-access might introduce risk

---

## 🧠 Why Choose One Over the Other?

| Mode       | Readable? | Used Before Failover? | Common Use Case                          |
|------------|-----------|------------------------|-------------------------------------------|
| Read-Only  | ✅ Yes     | ✅ Yes                 | BI/reporting, active-active style DR      |
| Standby    | ❌ No      | ❌ No                  | Transaction systems, strict DR isolation  |

> ⚠️ Choosing **standby** means your secondary replica contributes no value during normal operation — it's a pure DR strategy.

---

## 🔁 Related: Built-In High Availability (Not the Same)

Don’t confuse failover groups with **built-in high availability** in the **Business Critical** tier:

- Business Critical MI has **read-only secondaries** within the same region (1 primary + 3 secondaries)
- These are **always read-only**, no configuration choice
- General Purpose tier has **no read-only secondaries** at all

Failover groups are **cross-region** and offer **configurable access** to the secondary.

---

## 🧾 Conclusion

If you're using **failover groups** in Azure SQL Managed Instance, **you choose** whether the secondary is **read-only** or **standby**.  
It is **not required** to be read-only — and in many cases, you may explicitly want it to be **standby** only.

Let me know if you'd like a diagram or YAML example for how this is configured in code.
