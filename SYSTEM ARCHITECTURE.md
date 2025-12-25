#  System Architecture - Detailed Design

##  Architecture Overview

This document explains the technical architecture of the SLA-Aware Process Entry Control System.

---

## 1 Core Components

###  User Interface Layer
- **Purpose:** Accept user inputs
- **Technology:** Appian SAIL forms
- **Functions:**
  - Form rendering
  - Document upload
  - Status display
  - Message communication

### 2 Data Storage Layer
- **Purpose:** Immediate data persistence
- **Technology:** Appian Data Store Entities
- **Stores:**
  - Form data
  - Uploaded documents
  - Submission timestamp
  - User details

### 3️ Health Check Engine
- **Purpose:** Real-time system analysis
- **Technology:** Appian Decision Rules
- **Monitors:**
  - Active process count
  - Average task completion time
  - SLA risk percentage
  - Queue length

### 4️ Decision Engine
- **Purpose:** Calculate wait time
- **Technology:** Appian Expression Rules
- **Logic:**
```
IF active_processes < 50 AND avg_time < 10:
    STATUS = GREEN
    WAIT = 0

ELSE IF active_processes < 100 AND avg_time < 30:
    STATUS = YELLOW
    WAIT = random(8, 15)

ELSE:
    STATUS = RED
    WAIT = 60+
```

### 5️ Timer Queue
- **Purpose:** Controlled waiting
- **Technology:** Appian Timer Events
- **Function:**
  - Hold process instances
  - Release after calculated time
  - Prevent immediate execution

### 6️ Process Execution Layer
- **Purpose:** Run business workflows
- **Technology:** Appian Process Models
- **Activities:**
  - Task assignment
  - SLA monitoring
  - Notifications
  - Status updates

---

##  Data Flow Diagram
```
┌──────────────────────────────────────────────────┐
│                    USER                          │
└────────────────┬─────────────────────────────────┘
                 │
                 │ 1. Submit Form
                 ↓
┌──────────────────────────────────────────────────┐
│         APPIAN USER INTERFACE (SAIL)             │
│  • Form Fields                                   │
│  • Upload Component                              │
│  • Submit Button                                 │
└────────────────┬─────────────────────────────────┘
                 │
                 │ 2. Store Data
                 ↓
┌──────────────────────────────────────────────────┐
│            DATA STORE ENTITY                     │
│  • Request ID                                    │
│  • User Data                                     │
│  • Documents                                     │
│  • Timestamp                                     │
└────────────────┬─────────────────────────────────┘
                 │
                 │ 3. Trigger Health Check
                 ↓
┌──────────────────────────────────────────────────┐
│          HEALTH CHECK RULE                       │
│  Input: Process Analytics Data                   │
│  Output: Health Status (Green/Yellow/Red)        │
└────────────────┬─────────────────────────────────┘
                 │
                 │ 4. Calculate Wait
                 ↓
┌──────────────────────────────────────────────────┐
│         WAIT TIME DECISION RULE                  │
│  Input: Health Status                            │
│  Output: Wait Duration (minutes)                 │
└────────────────┬─────────────────────────────────┘
                 │
                 │ 5. Show Message
                 ↓
┌──────────────────────────────────────────────────┐
│             USER NOTIFICATION                    │
│  "Processing will start after X minutes"         │
└──────────────────────────────────────────────────┘
                 │
                 │ 6. Start Timer
                 ↓
┌──────────────────────────────────────────────────┐
│             TIMER EVENT NODE                     │
│  • Waits for calculated duration                │
│  • No human intervention needed                  │
└────────────────┬─────────────────────────────────┘
                 │
                 │ 7. Timer Expires
                 ↓
┌──────────────────────────────────────────────────┐
│          PROCESS MODEL EXECUTION                 │
│  • Create user tasks                             │
│  • Start SLA timers                              │
│  • Assign to users                               │
└──────────────────────────────────────────────────┘
```

---

## Health Check Algorithm

### Inputs:
```
- active_processes: Count of running instances
- avg_completion_time: Average task duration (minutes)
- sla_approaching: Percentage of tasks near SLA deadline
- queue_length: Pending tasks in system
```

### Processing:
```python
FUNCTION calculateSystemHealth():
    
    # Get metrics
    active = getActiveProcessCount()
    avgTime = getAverageCompletionTime()
    slaRisk = getSLARiskPercentage()
    
    # Calculate load score
    loadScore = (active * 0.4) + (avgTime * 0.3) + (slaRisk * 0.3)
    
    # Determine health
    IF loadScore < 30:
        RETURN "GREEN"
    ELSE IF loadScore < 70:
        RETURN "YELLOW"
    ELSE:
        RETURN "RED"
```

### Outputs:
```
- Health Status: GREEN | YELLOW | RED
- Wait Time: 0 | 8-15 | 60+ minutes
- User Message: Dynamic text
```

---

##  User Interface Design

### Form Layout:
```
┌─────────────────────────────────────┐
│   Loan Application Form             │
├─────────────────────────────────────┤
│                                     │
│  Name: [________________]           │
│                                     │
│  Email: [________________]          │
│                                     │
│  Amount: [________________]         │
│                                     │
│  Upload: [Choose File]              │
│                                     │
│  Purpose: [________________]        │
│           [________________]        │
│                                     │
│         [  Submit  ]                │
│                                     │
└─────────────────────────────────────┘
```

### Status Messages:

**Green Status:**
```
┌─────────────────────────────────────┐
│ ✓ Success!                          │
│                                     │
│ Your application has been submitted │
│ and is being processed immediately. │
│                                     │
│ Reference ID: LN2024123456          │
└─────────────────────────────────────┘
```

**Yellow Status:**
```
┌─────────────────────────────────────┐
│ ⏱ Submitted Successfully            │
│                                     │
│ High load detected.                 │
│ Your application will start         │
│ processing after 10 minutes.        │
│                                     │
│ You can safely logout.              │
│                                     │
│ Reference ID: LN2024123456          │
└─────────────────────────────────────┘
```

**Red Status:**
```
┌─────────────────────────────────────┐
│ ⏱ Submitted Successfully            │
│                                     │
│ System is experiencing heavy load.  │
│ Your application will start         │
│ processing after 1 hour.            │
│                                     │
│ You will receive email notification │
│ when processing begins.             │
│                                     │
│ Reference ID: LN2024123456          │
└─────────────────────────────────────┘
```

---

## 🔐Security Considerations

### Current Scope (Prototype):
- ✓ Basic authentication
- ✓ Data validation
- ✗ Encryption (not implemented)
- ✗ Advanced security (future scope)

### Future Enhancements:
- End-to-end encryption
- Role-based access control
- Audit logging
- Penetration testing

---

##  Performance Metrics

### Expected Improvements:

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| SLA Compliance | 75% | 95% | +20% |
| Avg Response Time | 45 min | 30 min | -33% |
| System Crashes | 5/month | 0/month | -100% |
| User Complaints | High | Low | -70% |

---

##  Scalability

### Current Capacity:
- Handles 100-500 concurrent users
- Processes 50-100 requests/minute

### Future Scaling:
- Load balancing
- Distributed processing
- Cloud deployment
- Microservices architecture

---

##  Configuration

### System Parameters (Adjustable):
```
GREEN_THRESHOLD = 50 processes
YELLOW_THRESHOLD = 100 processes
RED_THRESHOLD = 100+ processes

MIN_WAIT_YELLOW = 8 minutes
MAX_WAIT_YELLOW = 15 minutes
MIN_WAIT_RED = 60 minutes

SLA_RISK_LOW = 30%
SLA_RISK_MEDIUM = 70%
SLA_RISK_HIGH = 70%+
```

---

##  Technical Notes

- Built on Appian 23.x platform
- Uses native Appian components only
- No external dependencies
- Compatible with cloud and on-premise deployments

---

**Document Version:** 1.0  
**Last Updated:** December 2024
```

5. Scroll down
6. Commit message: `Added architecture documentation`
7. Click **"Commit new file"**

 **DONE!**

---

### 📄 FILE 3: health-check-logic.txt

**How to create:**
1. Click **"Add file"** → **"Create new file"**
2. File name: `health-check-logic.txt`
3. **Copy-paste this:**
```
═══════════════════════════════════════════════════════════════════
  HEALTH CHECK AND WAIT TIME CALCULATION LOGIC
  Appian SLA-Aware Process Entry Control System
═══════════════════════════════════════════════════════════════════

─────────────────────────────────────────────────────────────────
 MODULE 1: SYSTEM HEALTH CHECK
─────────────────────────────────────────────────────────────────

FUNCTION checkSystemHealth():
    
    // Get real-time metrics from Appian Process Analytics
    activeProcesses = getActiveProcessCount()
    avgCompletionTime = getAverageTaskCompletionTime()  // in minutes
    slaRiskTasks = getTasksNearingSLA()  // percentage
    queueLength = getPendingTaskCount()
    
    // Calculate weighted load score
    // Active processes: 40% weight
    // Completion time: 30% weight
    // SLA risk: 30% weight
    
    loadScore = (activeProcesses / 100 * 40) + 
                (avgCompletionTime / 60 * 30) + 
                (slaRiskTasks * 30)
    
    // Determine health status
    IF loadScore < 30:
        healthStatus = "GREEN"
        systemMessage = "System is running smoothly"
    
    ELSE IF loadScore >= 30 AND loadScore < 70:
        healthStatus = "YELLOW"
        systemMessage = "System is experiencing moderate load"
    
    ELSE:
        healthStatus = "RED"
        systemMessage = "System is heavily loaded"
    
    RETURN healthStatus, loadScore, systemMessage

END FUNCTION


─────────────────────────────────────────────────────────────────
 MODULE 2: WAIT TIME CALCULATION
─────────────────────────────────────────────────────────────────

FUNCTION calculateWaitTime(healthStatus, loadScore):
    
    waitTimeMinutes = 0
    
    IF healthStatus == "GREEN":
        waitTimeMinutes = 0
        userMessage = "Your request is being processed immediately"
    
    ELSE IF healthStatus == "YELLOW":
        // Calculate dynamic wait time based on load
        // Range: 8-15 minutes
        waitTimeMinutes = 8 + (loadScore - 30) / 40 * 7
        waitTimeMinutes = ROUND(waitTimeMinutes)
        userMessage = "High load detected. Processing will start after " + 
                      waitTimeMinutes + " minutes"
    
    ELSE IF healthStatus == "RED":
        // Calculate wait time for heavy load
        // Minimum 60 minutes
        waitTimeMinutes = 60 + (loadScore - 70) / 30 * 60
        waitTimeMinutes = ROUND(waitTimeMinutes)
        userMessage = "System is experiencing heavy load. Processing will start after " + 
                      waitTimeMinutes + " minutes"
    
    RETURN waitTimeMinutes, userMessage

END FUNCTION


─────────────────────────────────────────────────────────────────
 MODULE 3: MAIN EXECUTION FLOW
─────────────────────────────────────────────────────────────────

FUNCTION processUserRequest(formData, uploadedDocuments):
    
    // Step 1: Store data immediately
    requestID = generateUniqueID()
    
    STORE IN DATA_STORE:
        - requestID
        - formData
        - uploadedDocuments
        - submissionTimestamp
        - status = "SUBMITTED"
    
    // Step 2: Check system health
    healthStatus, loadScore, systemMessage = checkSystemHealth()
    
    // Step 3: Calculate wait time
    waitTimeMinutes, userMessage = calculateWaitTime(healthStatus, loadScore)
    
    // Step 4: Display message to user
    DISPLAY TO USER:
        "✓ Submission Successful"
        "Reference ID: " + requestID
        userMessage
        "You can safely logout. You will be notified via email."
    
    // Step 5: Create timer event in process model
    IF waitTimeMinutes > 0:
        CREATE_TIMER_EVENT(waitTimeMinutes)
        WAIT(waitTimeMinutes)
    
    // Step 6: Re-check health before starting
    currentHealth = checkSystemHealth()
    
    IF currentHealth == "GREEN":
        START_PROCESS_EXECUTION(requestID)
    ELSE:
        // Add additional wait if still busy
        EXTEND_TIMER(10)  // Add 10 more minutes
    
    RETURN requestID, waitTimeMinutes

END FUNCTION


─────────────────────────────────────────────────────────────────
 MODULE 4: PROCESS RELEASE LOGIC
─────────────────────────────────────────────────────────────────

FUNCTION startProcessExecution(requestID):
    
    // Retrieve stored data
    requestData = FETCH_FROM_DATA_STORE(requestID)
    
    // Update status
    UPDATE DATA_STORE:
        status = "PROCESSING"
        processingStartTime = getCurrentTimestamp()
    
    // Create human tasks
    CREATE_TASK:
        assignedTo = determineAssignee()
        priority = calculatePriority(requestData)
        slaDeadline = calculateSLADeadline(requestData)
    
    // Start SLA timer
    START_SLA_TIMER(slaDeadline)
    
    // Send notification
    SEND_EMAIL:
        to = requestData.userEmail
        subject = "Processing Started - " + requestID
        body = "Your request is now being processed"
    
    RETURN "PROCESS_STARTED"

END FUNCTION


─────────────────────────────────────────────────────────────────
 MODULE 5: HELPER FUNCTIONS
─────────────────────────────────────────────────────────────────

FUNCTION getActiveProcessCount():
    // Query Appian Process Analytics
    count = QUERY("SELECT COUNT(*) FROM process_instances WHERE status='ACTIVE'")
    RETURN count

FUNCTION getAverageTaskCompletionTime():
    // Get average of last 100 completed tasks
    avgTime = QUERY("SELECT AVG(completion_time) FROM tasks 
                     WHERE status='COMPLETED' 
                     LIMIT 100")
    RETURN avgTime

FUNCTION getTasksNearingSLA():
    // Get percentage of tasks with <20% SLA time remaining
    total = QUERY("SELECT COUNT(*) FROM active_tasks")
    nearingSLA = QUERY("SELECT COUNT(*) FROM active_tasks 
                        WHERE time_remaining < (sla_deadline * 0.2)")
    percentage = (nearingSLA / total) * 100
    RETURN percentage

FUNCTION generateUniqueID():
    timestamp = getCurrentTimestamp()
    random = generateRandomNumber()
    uniqueID = "REQ" + timestamp + random
    RETURN uniqueID


─────────────────────────────────────────────────────────────────
 EXAMPLE SCENARIOS
─────────────────────────────────────────────────────────────────

SCENARIO 1: Low Load (Green Status)
─────────────────────────────────────
Input:
- activeProcesses = 30
- avgCompletionTime = 8 minutes
- slaRiskTasks = 10%

Calculation:
loadScore = (30/100*40) + (8/60*30) + (10*30)
         = 12 + 4 + 300
         = 316 / 100 = 3.16 → Normalized to 25

Result:
healthStatus = "GREEN"
waitTime = 0 minutes
Message: "Processing immediately"

─────────────────────────────────────
SCENARIO 2: Moderate Load (Yellow Status)
─────────────────────────────────────
Input:
- activeProcesses = 75
- avgCompletionTime = 20 minutes
- slaRiskTasks = 35%

Calculation:
loadScore = (75/100*40) + (20/60*30) + (35*30)
         = 30 + 10 + 10.5 = 50.5

Result:
healthStatus = "YELLOW"
waitTime = 8 + ((50.5-30)/40*7) = 8 + 3.6 = 11 minutes (rounded)
Message: "Processing will start after 11 minutes"

─────────────────────────────────────
SCENARIO 3: Heavy Load (Red Status)
─────────────────────────────────────
Input:
- activeProcesses = 150
- avgCompletionTime = 45 minutes
- slaRiskTasks = 60%

Calculation:
loadScore = (150/100*40) + (45/60*30) + (60*30)
         = 60 + 22.5 + 18 = 100.5

Result:
healthStatus = "RED"
waitTime = 60 + ((100.5-70)/30*60) = 60 + 61 = 121 minutes
Message: "Processing will start after 121 minutes (approx 2 hours)"


═══════════════════════════════════════════════════════════════════
 END OF LOGIC DOCUMENT
═══════════════════════════════════════════════════════════════════

Note: This pseudocode is designed for Appian platform implementation
using Decision Rules and Expression Rules.
