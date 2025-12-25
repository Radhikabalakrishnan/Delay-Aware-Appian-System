# 🎯 SLA-Aware Process Entry Control System for Appian

## 📌 Project Overview

This project introduces an intelligent process entry control mechanism for the Appian platform that prevents system overload by dynamically regulating backend workflow initiation based on real-time system load.

---

## 🔴 The Problem

Current Appian systems face these challenges:

- ❌ All process requests start immediately
- ❌ System overload during peak times (festivals, month-end)
- ❌ Task queues build up at user nodes
- ❌ SLA violations detected AFTER delays occur
- ❌ Users resubmit forms creating more load
- ❌ No native control mechanism in Appian

**Result:** Poor user experience + Reactive problem handling

---

## 💡 Our Solution

We implement a **traffic control system for business processes**:

✅ **Accept** user submissions immediately (no blocking)  
✅ **Store** data instantly (prevents duplicates)  
✅ **Analyze** system health in real-time  
✅ **Calculate** optimal wait time  
✅ **Inform** users transparently  
✅ **Release** processes gradually  

### Key Innovation:
**Separate user submission from process execution**

---

## 🔄 How It Works
```
Step 1: User submits form
   ↓
Step 2: Data stored immediately
   ↓
Step 3: System health check
   ↓
Step 4: Wait-time calculation
   ↓
Step 5: User sees message
   ↓
Step 6: Timer-based queue
   ↓
Step 7: Controlled process start
```

---

## 🎨 System Architecture
```
┌─────────────┐
│   USER UI   │
└──────┬──────┘
       │ Submit
       ↓
┌─────────────┐
│ DATA STORE  │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│HEALTH CHECK │
│   RULE      │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│  DECISION   │
│   ENGINE    │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│TIMER QUEUE  │
└──────┬──────┘
       │
       ↓
┌─────────────┐
│  PROCESS    │
│ EXECUTION   │
└─────────────┘
```

---

## 🚦 Health Status Logic

| Status | Condition | Wait Time | User Message |
|--------|-----------|-----------|--------------|
| 🟢 Green | System free | 0 min | "Processing starting immediately" |
| 🟡 Yellow | System busy | 8-15 min | "Processing will start after 10 minutes" |
| 🔴 Red | Overloaded | 60+ min | "Processing will start after 1 hour" |

---

## 🛠️ Technology Stack

- **Platform:** Appian BPM
- **Components:**
  - Decision Rules (Health check logic)
  - Process Models (Workflow design)
  - Timer Events (Delayed execution)
  - Data Store Entities (Queue storage)
  - Process Analytics (Real-time metrics)
  - User Interfaces (Message display)

---

## 📊 Real-World Example

### Scenario: Month-End Loan Processing Rush

**Without our solution:**
- 500 applications at 4 PM
- All start immediately
- System crashes
- 300+ SLA breaches

**With our solution:**
- 500 applications accepted
- 100 start now (Green)
- 200 wait 10 min (Yellow)
- 200 wait 1 hour (Red)
- Users informed clearly
- Zero SLA breaches

---

## 📈 Expected Impact

### For Users:
- 🎯 Clear communication
- 🎯 No confusion
- 🎯 Better experience

### For Organizations:
- 📊 SLA compliance maintained
- 📊 System stability improved
- 📊 Resource optimization

### For Operations:
- ⚡ Proactive management
- ⚡ Predictable load
- ⚡ Reduced escalations



## 🎓 Academic Project Details

**Institution:** [AMRITA VISHWA VIDYAPEETHAM CHENNAI CAMPUS]  
**Team Members:**
- [Name 1] - RADHIKA
- [Name 2] - BHAAVESH

**Course:** [ECE]  
**Semester:** [VI]  
**Year:** 2023-2027

---

## 🔮 Future Enhancements

- AI-based load prediction
- Multi-tenant support
- Advanced analytics dashboard
- Mobile app integration
- Enterprise security features



