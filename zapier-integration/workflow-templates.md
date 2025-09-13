# Zapier Workflow Templates & Configuration Guide

## **Core Workflow Templates**

### **1. Investor Relationship Management Workflows**

#### **Workflow 1A: New Investor Lead Processing**
```yaml
Name: "New Investor Lead → CRM Pipeline Setup"
Trigger: 
  App: Salesforce
  Event: New Contact Created
  Filter: Contact.Type = "Investor"
  
Actions:
  1. HubSpot - Create/Update Contact
     Fields:
       - Email: {{contact.email}}
       - Company: {{contact.account_name}}
       - Lead_Source: "Investor Network"
       - Lifecycle_Stage: "Investor Prospect"
  
  2. Google Sheets - Add Row
     Sheet: "Investor Pipeline Tracker"
     Data:
       - Name: {{contact.first_name}} {{contact.last_name}}
       - Company: {{contact.account_name}}
       - Email: {{contact.email}}
       - Status: "New Lead"
       - Date_Added: {{zap.utc_iso_string}}
  
  3. Slack - Send Channel Message
     Channel: #investor-pipeline
     Message: "🎯 New investor lead: {{contact.first_name}} {{contact.last_name}} from {{contact.account_name}}"
  
  4. Calendly - Create Scheduling Link
     Event_Type: "Investor Pitch Meeting"
     Duration: 60 minutes
     
  5. Gmail - Send Email
     To: {{contact.email}}
     Subject: "Following up on your interest in AuditORSEC"
     Template: investor_intro_template.html
```

#### **Workflow 1B: Investor Meeting Follow-up Automation**
```yaml
Name: "Investor Meeting Complete → Follow-up Sequence"
Trigger:
  App: Calendly
  Event: Invitee Created
  Filter: Event_Type contains "Investor"
  
Actions:
  1. Salesforce - Update Opportunity
     Fields:
       - Stage: "Initial Meeting Complete"
       - Next_Step: "Send Pitch Deck"
       - Last_Meeting_Date: {{start_time}}
  
  2. Asana - Create Task
     Project: "Series A Execution"
     Task: "Follow up with {{invitee.name}} - Send pitch deck"
     Assignee: CEO
     Due_Date: +1 day from meeting
  
  3. Gmail - Send Follow-up Email
     Template: post_meeting_followup.html
     Attachments: pitch_deck_latest.pdf
  
  4. Google Calendar - Create Follow-up Event
     Title: "Second meeting with {{invitee.name}}"
     Date: +1 week from initial meeting
     Duration: 45 minutes
```

#### **Workflow 1C: Due Diligence Request Automation**
```yaml
Name: "Due Diligence Stage → Document Preparation"
Trigger:
  App: Salesforce  
  Event: Opportunity Updated
  Filter: Stage = "Due Diligence"
  
Actions:
  1. Google Drive - Create Folder
     Name: "DD - {{account.name}} - {{current_date}}"
     Parent_Folder: "Series A Due Diligence"
     
  2. Asana - Create Project
     Name: "Due Diligence - {{account.name}}"
     Template: "DD Checklist Template"
     Assignees: [Legal Team, Finance Team, CEO]
  
  3. Slack - Send Message
     Channel: #legal-team
     Message: "🔍 Due diligence initiated for {{account.name}}. Folder: [link] | Project: [asana_link]"
     
  4. Airtable - Create Record
     Table: "DD Tracking"
     Fields:
       - Investor: {{account.name}}
       - Status: "In Progress"
       - Started: {{current_date}}
       - Folder_Link: {{drive_folder_url}}
```

### **2. Customer Success Automation Workflows**

#### **Workflow 2A: Success Story Interview Pipeline**
```yaml
Name: "Customer Success → Interview Scheduling"
Trigger:
  App: Salesforce
  Event: Account Updated
  Filter: Customer_Success_Score > 8 AND Success_Story_Candidate = True
  
Actions:
  1. Gmail - Send Interview Request
     To: {{account.primary_contact_email}}
     Subject: "Share your AuditORSEC success story?"
     Template: success_story_request.html
     
  2. Calendly - Send Booking Link
     Event_Type: "Customer Success Interview"
     Duration: 45 minutes
     
  3. Asana - Create Task
     Project: "Customer Success Stories"
     Task: "Conduct success interview - {{account.name}}"
     Assignee: Customer Success Manager
     
  4. Airtable - Create Record
     Table: "Success Story Pipeline"
     Fields:
       - Customer: {{account.name}}
       - Contact: {{account.primary_contact}}
       - Status: "Interview Requested"
       - Potential_Value: {{account.arr_value}}
```

#### **Workflow 2B: Reference Network Management**
```yaml
Name: "Interview Complete → Reference Activation"
Trigger:
  App: Calendly
  Event: Invitee Created
  Filter: Event_Type = "Customer Success Interview"
  
Actions:
  1. Airtable - Update Record
     Table: "Success Story Pipeline"
     Filter: Customer = {{invitee.account_name}}
     Update: Status = "Interview Complete"
     
  2. Typeform - Send Follow-up Survey
     To: {{invitee.email}}
     Survey: "Reference Willingness Assessment"
     
  3. Asana - Create Task
     Task: "Process interview - extract quotes and metrics"
     Project: "Customer Success Stories"
     Assignee: Content Manager
     Due: +2 days
     
  4. Slack - Notify Team
     Channel: #customer-success
     Message: "✅ Success interview completed with {{invitee.name}} from {{invitee.account_name}}"
```

#### **Workflow 2C: Customer Reference Request Router**
```yaml
Name: "Reference Request → Customer Matching"
Trigger:
  App: Typeform
  Event: New Entry
  Form: "Reference Request Form"
  
Actions:
  1. Airtable - Find Records
     Table: "Reference Network"
     Filter: 
       - Industry = {{typeform.industry}}
       - Use_Case = {{typeform.use_case}}
       - Available = True
     Sort: Last_Reference_Date (oldest first)
     
  2. Gmail - Send Reference Request
     To: {{airtable.customer_email}}
     Subject: "Reference request - {{typeform.requester_company}}"
     Template: reference_request.html
     
  3. Calendly - Create Booking
     Event_Type: "Customer Reference Call"
     Participants: [{{airtable.customer_email}}, {{typeform.requester_email}}]
     
  4. Salesforce - Log Activity
     Related_To: {{airtable.account_id}}
     Subject: "Reference provided for {{typeform.requester_company}}"
     Type: "Reference"
```

### **3. Marketing Automation Workflows**

#### **Workflow 3A: Lead Qualification & Routing**
```yaml
Name: "Website Lead → Qualification & Distribution"
Trigger:
  App: HubSpot
  Event: New Form Submission
  Filter: Form_Name = "Contact Us" OR "Demo Request"
  
Actions:
  1. Code by Zapier - Lead Scoring
     Script: |
       // Lead scoring logic
       let score = 0;
       if (inputData.company_size > 100) score += 25;
       if (inputData.industry.includes('Finance')) score += 20;
       if (inputData.title.includes('CTO') || inputData.title.includes('CEO')) score += 15;
       if (inputData.budget > 100000) score += 30;
       return {score: score, qualified: score > 50};
       
  2. Paths by Zapier
     Path A (Qualified Lead - Score > 50):
       - Salesforce - Create Lead
         Fields:
           - Lead_Score: {{score}}
           - Lead_Source: "Website"
           - Status: "MQL"
       - Slack - Alert Sales Team
         Channel: #sales-alerts
         Message: "🔥 Hot lead: {{first_name}} {{last_name}} from {{company}} (Score: {{score}})"
           
     Path B (Nurture Lead - Score ≤ 50):
       - HubSpot - Add to Workflow
         Workflow: "Lead Nurture Sequence"
       - Airtable - Add to Nurture List
         Table: "Lead Nurture Pipeline"
```

#### **Workflow 3B: Content Engagement Tracking**
```yaml
Name: "Content Download → Engagement Scoring"
Trigger:
  App: HubSpot
  Event: Form Submission
  Filter: Form_Name contains "Download"
  
Actions:
  1. HubSpot - Update Contact Property
     Property: Last_Content_Download
     Value: {{form_name}}
     
  2. Code by Zapier - Engagement Score Update
     Script: |
       // Update engagement score based on content type
       const contentScores = {
         'whitepaper': 10,
         'case_study': 15,
         'demo_video': 20,
         'roi_calculator': 25
       };
       
       let contentType = inputData.form_name.toLowerCase();
       let scoreIncrease = 0;
       
       for (let type in contentScores) {
         if (contentType.includes(type)) {
           scoreIncrease = contentScores[type];
           break;
         }
       }
       
       return {
         score_increase: scoreIncrease,
         content_category: contentType
       };
       
  3. HubSpot - Update Contact Score
     Property: Engagement_Score
     Value: {{current_score}} + {{score_increase}}
     
  4. Filter by Zapier
     Continue if: {{engagement_score}} > 75
     
  5. Slack - Sales Alert
     Channel: #sales-hot-leads
     Message: "🎯 High engagement lead: {{email}} downloaded {{content_type}} (Total score: {{engagement_score}})"
```

### **4. Internal Operations Workflows**

#### **Workflow 4A: Project Status Communication**
```yaml
Name: "Asana Project Update → Team Notification"
Trigger:
  App: Asana
  Event: Task Updated
  Filter: Project = "Series A Execution" AND Status changed
  
Actions:
  1. Slack - Send Channel Message
     Channel: #series-a-updates
     Message: |
       📋 Project Update: {{task.name}}
       Status: {{task.status}}
       Assignee: {{task.assignee}}
       Due Date: {{task.due_date}}
       Notes: {{task.notes}}
       
  2. Google Sheets - Update Dashboard
     Sheet: "Series A Progress Tracker"
     Action: Update row where Task_ID = {{task.gid}}
     Fields:
       - Status: {{task.status}}
       - Last_Updated: {{current_timestamp}}
       - Completion_Percentage: {{project.completion_percentage}}
       
  3. Gmail - Status Email (if milestone)
     Filter: Task contains "Milestone"
     To: board@auditorsec.com
     Subject: "Series A Milestone Update: {{task.name}}"
     Template: milestone_update.html
```

#### **Workflow 4B: Meeting Notes Distribution**
```yaml
Name: "Meeting Notes → Action Item Creation"
Trigger:
  App: Google Drive
  Event: New File in Folder
  Filter: Folder = "Meeting Notes" AND File type = "Google Doc"
  
Actions:
  1. Code by Zapier - Extract Action Items
     Script: |
       // Parse document for action items
       const doc_content = inputData.document_content;
       const action_items = [];
       
       // Look for patterns like "ACTION:", "TODO:", "@person"
       const actionPatterns = [
         /ACTION:\s*(.+?)(?=\n|$)/gi,
         /TODO:\s*(.+?)(?=\n|$)/gi,
         /@(\w+)\s+(.+?)(?=\n|$)/gi
       ];
       
       actionPatterns.forEach(pattern => {
         let matches = doc_content.match(pattern);
         if (matches) {
           matches.forEach(match => {
             action_items.push({
               text: match.trim(),
               assigned_to: extractAssignee(match),
               priority: extractPriority(match)
             });
           });
         }
       });
       
       return {action_items: action_items};
       
  2. Looping by Zapier
     Loop Over: {{action_items}}
     
  3. Asana - Create Task (for each action item)
     Name: {{action_item.text}}
     Assignee: {{action_item.assigned_to}}
     Due Date: +3 days
     Project: "Meeting Follow-ups"
     
  4. Slack - Send Summary
     Channel: #meeting-updates
     Message: |
       📝 Meeting notes processed: {{document_name}}
       Action items created: {{action_items.length}}
       Link to notes: {{document_url}}
```

### **5. Error Handling & Monitoring Workflows**

#### **Workflow 5A: Error Alert System**
```yaml
Name: "Workflow Error → Immediate Alert"
Trigger:
  App: Zapier
  Event: Zap Error
  Filter: Error Type = "App Error" OR "Timeout"
  
Actions:
  1. Slack - Send Alert
     Channel: #system-alerts
     Priority: High
     Message: |
       🚨 WORKFLOW ERROR
       Zap: {{zap_name}}
       Error: {{error_message}}
       Time: {{error_time}}
       Data: {{error_data}}
       
  2. Gmail - Send to Admin
     To: admin@auditorsec.com
     Subject: "URGENT: Zapier Workflow Failure - {{zap_name}}"
     Priority: High
     
  3. Airtable - Log Error
     Table: "System Error Log"
     Fields:
       - Zap_Name: {{zap_name}}
       - Error_Type: {{error_type}}
       - Error_Message: {{error_message}}
       - Timestamp: {{current_timestamp}}
       - Status: "Open"
```

#### **Workflow 5B: Performance Monitoring**
```yaml
Name: "Daily Performance Report"
Trigger:
  App: Schedule by Zapier
  Frequency: Daily at 9:00 AM
  
Actions:
  1. Zapier - Get Zap Statistics
     Timeframe: Last 24 hours
     Metrics: [success_rate, execution_time, error_count]
     
  2. Google Sheets - Update Dashboard
     Sheet: "Zapier Performance Metrics"
     Action: Add new row
     Data:
       - Date: {{current_date}}
       - Total_Executions: {{total_executions}}
       - Success_Rate: {{success_rate}}
       - Average_Runtime: {{avg_runtime}}
       - Errors: {{error_count}}
       
  3. Code by Zapier - Generate Report
     Script: |
       // Generate performance summary
       const metrics = {
         executions: inputData.total_executions,
         success_rate: inputData.success_rate,
         avg_runtime: inputData.avg_runtime,
         errors: inputData.error_count
       };
       
       let status = 'GREEN';
       if (metrics.success_rate < 95) status = 'RED';
       else if (metrics.success_rate < 98) status = 'YELLOW';
       
       return {
         status: status,
         summary: `Performance Status: ${status}\nSuccess Rate: ${metrics.success_rate}%\nTotal Executions: ${metrics.executions}\nAverage Runtime: ${metrics.avg_runtime}s\nErrors: ${metrics.errors}`
       };
       
  4. Slack - Daily Report
     Channel: #system-health
     Message: |
       📊 Daily Zapier Performance Report
       {{performance_summary}}
       
       View detailed metrics: [Dashboard Link]
```

---

## **Workflow Configuration Examples**

### **Field Mapping Templates**

#### **Salesforce ↔ HubSpot Contact Sync**
```json
{
  "salesforce_to_hubspot": {
    "FirstName": "firstname",
    "LastName": "lastname", 
    "Email": "email",
    "Phone": "phone",
    "Company": "company",
    "Title": "jobtitle",
    "LeadSource": "hs_lead_source",
    "Industry": "industry",
    "AnnualRevenue": "annualrevenue",
    "NumberOfEmployees": "numberofemployees"
  },
  "custom_fields": {
    "Quantum_Interest__c": "quantum_interest_level",
    "AI_Use_Cases__c": "ai_use_cases",
    "Decision_Timeline__c": "decision_timeline",
    "Budget_Range__c": "budget_range"
  }
}
```

#### **Customer Success Metrics Tracking**
```json
{
  "customer_metrics": {
    "account_fields": {
      "Customer_Health_Score__c": "health_score",
      "NPS_Score__c": "nps_score",
      "ARR__c": "arr_value",
      "Implementation_Date__c": "implementation_date",
      "Last_Success_Review__c": "last_review_date"
    },
    "success_indicators": {
      "Platform_Usage_Hours__c": "usage_hours_monthly",
      "Features_Adopted__c": "features_count",
      "Support_Tickets__c": "support_ticket_count",
      "ROI_Achieved__c": "roi_percentage"
    }
  }
}
```

### **Conditional Logic Examples**

#### **Lead Routing Logic**
```javascript
// Lead routing based on multiple criteria
function routeLead(leadData) {
  let routing = {
    assignee: '',
    priority: 'Medium',
    followUpDate: new Date(),
    nurturePath: ''
  };
  
  // Enterprise routing
  if (leadData.employees > 1000 && leadData.budget > 500000) {
    routing.assignee = 'enterprise_sales_rep';
    routing.priority = 'High';
    routing.followUpDate.setHours(routing.followUpDate.getHours() + 2);
  }
  
  // Mid-market routing
  else if (leadData.employees >= 100 && leadData.budget >= 100000) {
    routing.assignee = 'midmarket_sales_rep';
    routing.priority = 'Medium';
    routing.followUpDate.setDate(routing.followUpDate.getDate() + 1);
  }
  
  // SMB or nurture
  else {
    routing.assignee = 'inside_sales_rep';
    routing.priority = 'Low';
    routing.nurturePath = 'smb_nurture_sequence';
    routing.followUpDate.setDate(routing.followUpDate.getDate() + 7);
  }
  
  // Industry-specific routing
  if (leadData.industry === 'Financial Services') {
    routing.assignee = 'fintech_specialist';
    routing.priority = 'High';
  }
  
  return routing;
}
```

#### **Customer Success Escalation Logic**
```javascript
// Customer health score automation
function assessCustomerHealth(customerData) {
  let healthScore = 100;
  let alerts = [];
  
  // Usage-based scoring
  if (customerData.loginFrequency < 5) {
    healthScore -= 20;
    alerts.push('Low platform usage');
  }
  
  // Support ticket analysis
  if (customerData.supportTickets > 10) {
    healthScore -= 15;
    alerts.push('High support volume');
  }
  
  // Payment and contract health
  if (customerData.paymentDelays > 0) {
    healthScore -= 25;
    alerts.push('Payment issues');
  }
  
  // Engagement scoring
  if (customerData.lastTrainingDate > 90) { // days
    healthScore -= 10;
    alerts.push('Training needed');
  }
  
  // Determine action needed
  let action = 'none';
  if (healthScore < 60) {
    action = 'immediate_intervention';
  } else if (healthScore < 80) {
    action = 'proactive_outreach';
  }
  
  return {
    score: healthScore,
    alerts: alerts,
    recommendedAction: action,
    assignTo: healthScore < 70 ? 'customer_success_manager' : 'automated_nurture'
  };
}
```

---

## **Advanced Workflow Patterns**

### **Multi-Step Approval Workflows**

#### **Investment Decision Approval Chain**
```yaml
Name: "Investment Decision → Board Approval Process"
Trigger:
  App: Salesforce
  Event: Opportunity Updated
  Filter: Amount > 1000000 AND Stage = "Proposal Submitted"
  
Steps:
  1. Initial Review (CEO)
     Action: Send approval request
     Timeout: 24 hours
     
  2. Financial Review (CFO) - If CEO Approves
     Action: Financial analysis request
     Timeout: 48 hours
     
  3. Board Notification - If CFO Approves
     Action: Board meeting agenda item
     
  4. Legal Review - Parallel to Board
     Action: Term sheet legal review
     
  5. Final Approval Collection
     Required: [CEO, CFO, 2 Board Members]
     
  6. Execution Actions - If All Approve
     - Update opportunity status
     - Generate term sheet
     - Schedule signing meeting
     - Notify all stakeholders
```

### **Data Synchronization Patterns**

#### **Bi-Directional CRM Sync**
```yaml
Name: "Salesforce ↔ HubSpot Bidirectional Sync"

Sync_Rules:
  Primary_System: Salesforce (for opportunities)
  Secondary_System: HubSpot (for marketing data)
  
Conflict_Resolution:
  - Most_Recent_Update: Wins
  - Field_Priority: Salesforce > HubSpot for sales data
  - Field_Priority: HubSpot > Salesforce for marketing data
  
Sync_Frequency: Real-time for critical fields, 15 minutes for others

Critical_Fields: [Email, Phone, Company, Deal_Stage, Deal_Value]
Marketing_Fields: [Lead_Score, Last_Campaign, Email_Engagement]
Sales_Fields: [Opportunity_Stage, Close_Date, Deal_Amount]
```

---

## **Performance Optimization**

### **Batch Processing for High-Volume Workflows**

```yaml
Name: "Daily Lead Processing Batch"
Trigger:
  App: Schedule by Zapier
  Frequency: Daily at 2:00 AM
  
Process:
  1. Query HubSpot for leads created in last 24 hours
  2. Batch process leads in groups of 50
  3. Apply lead scoring to entire batch
  4. Route qualified leads to appropriate sales reps
  5. Add unqualified leads to nurture sequences
  6. Generate daily lead report
  
Benefits:
  - Reduced API calls
  - Better performance
  - Lower costs
  - More reliable processing
```

### **Caching and Data Storage**

```yaml
Name: "Customer Data Caching System"
Purpose: Reduce API calls and improve response times

Storage: 
  App: Airtable
  Table: "Customer_Cache"
  TTL: 4 hours
  
Process:
  1. Check cache before making API calls
  2. Use cached data if fresh (<4 hours)
  3. Update cache with new data
  4. Expire old cache entries automatically
  
Benefits:
  - 60% reduction in API calls
  - Faster workflow execution
  - Lower costs
  - Better reliability
```

---

**Implementation Timeline**: 4 weeks for core workflows
**Testing Period**: 2 weeks comprehensive testing
**Training Required**: 8 hours total across all teams
**Maintenance**: 5-10 hours monthly ongoing