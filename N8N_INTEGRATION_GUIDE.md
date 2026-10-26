# n8n Integration Guide for RCMAPP

This guide explains how to integrate your RCMAPP with n8n automation workflows.

## Overview

The RCMAPP follows this simple flow:
1. **Login/SignUp** → User authentication
2. **Subscription** → User selects a plan
3. **Projects** → User creates/selects projects
4. **Asset Addition** → User inputs asset data (sent to n8n)
5. **Dashboard** → Displays analysis results from n8n

## n8n Webhooks Setup

You need to create TWO n8n workflows with webhooks:

### Webhook 1: Receive Asset Data (Input)

**Purpose**: Receives asset data from the Asset Addition page

**Webhook Configuration**:
- **Method**: POST
- **Path**: `/rcm/asset-input`
- **Authentication**: None (or configure as needed)

**Expected Payload**:
```json
{
  "project_id": "string",
  "asset_data": {
    "equipment_name": "string",
    "tag_number": "string",
    "system": "string",
    "manufacturer": "string",
    "model": "string",
    "criticality_level": "High" | "Medium" | "Low",
    "installation_date": "YYYY-MM-DD",
    "operating_hours_per_day": number,
    "created_at": "ISO 8601 timestamp"
  }
}
```

**Example Workflow**:
1. Webhook Trigger (receives data)
2. Process asset data (your RCM analysis logic)
3. Store results in database/storage
4. Return success response

---

### Webhook 2: Fetch Analysis Results (Output)

**Purpose**: Returns analysis results to the Dashboard

**Webhook Configuration**:
- **Method**: GET
- **Path**: `/rcm/analysis-results`
- **Query Parameter**: `project_id`
- **Authentication**: None (or configure as needed)

**Expected Response**:
```json
{
  "asset_info": {
    "equipment_name": "Centrifugal Pump",
    "tag_number": "PUMP-001",
    "criticality_level": "High"
  },
  "risk_assessment": {
    "overall_risk": "High",
    "risk_priority_number": 240,
    "failure_modes": 12
  },
  "maintenance_recommendations": {
    "frequency": "Monthly",
    "type": "Predictive + Preventive",
    "priority": "High"
  },
  "cost_analysis": {
    "estimated_annual_cost": 45000,
    "potential_savings": 18000
  },
  "analysis_timestamp": "ISO 8601 timestamp"
}
```

---

## Configuration Steps

### Step 1: Create n8n Workflows

1. **Open n8n** and create a new workflow for asset input
2. Add a **Webhook** node:
   - Set HTTP Method to `POST`
   - Set Path to `/rcm/asset-input`
   - Enable "Respond Immediately" if you want instant response
3. Add your **analysis logic** (function nodes, HTTP requests, database operations, etc.)
4. Save and activate the workflow
5. Copy the webhook URL (e.g., `https://your-n8n-instance.com/webhook/rcm/asset-input`)

6. Create another workflow for results:
   - Add Webhook node with GET method
   - Set Path to `/rcm/analysis-results`
   - Add logic to fetch and return analysis data
   - Save and activate
   - Copy the webhook URL

### Step 2: Update RCMAPP Configuration

Update the following files in your RCMAPP:

**File**: `src/pages/AssetManagement.tsx`

Find this line (around line 66):
```typescript
const n8nWebhookUrl = 'YOUR_N8N_WEBHOOK_URL_HERE';
```

Replace with your actual webhook URL:
```typescript
const n8nWebhookUrl = 'https://your-n8n-instance.com/webhook/rcm/asset-input';
```

---

**File**: `src/pages/Dashboard.tsx`

Find this line (around line 53):
```typescript
const n8nResultsWebhookUrl = 'YOUR_N8N_RESULTS_WEBHOOK_URL_HERE';
```

Replace with your actual webhook URL:
```typescript
const n8nResultsWebhookUrl = 'https://your-n8n-instance.com/webhook/rcm/analysis-results';
```

### Step 3: Test the Integration

1. **Start the app**: `npm run dev`
2. **Navigate through**:
   - Login (or use the demo)
   - Select subscription plan
   - Create a project
   - Add asset data
3. **Check n8n**: Verify the webhook receives the data
4. **View Dashboard**: Check if results are displayed

---

## Example n8n Workflow (Asset Input)

```
┌─────────────┐       ┌──────────────┐       ┌─────────────┐
│   Webhook   │──────▶│   Function   │──────▶│   Storage   │
│  (Receive)  │       │   (Process)  │       │   (Save)    │
└─────────────┘       └──────────────┘       └─────────────┘
```

**Function Node Example** (Process Asset):
```javascript
// Extract data
const assetData = $json.asset_data;
const projectId = $json.project_id;

// Perform RCM analysis
const riskScore = calculateRisk(assetData.criticality_level);
const rpn = calculateRPN(assetData);
const maintenancePlan = determineMaintenance(assetData);

// Return processed data
return {
  json: {
    success: true,
    project_id: projectId,
    analysis: {
      risk_score: riskScore,
      rpn: rpn,
      maintenance_plan: maintenancePlan
    }
  }
};
```

---

## Example n8n Workflow (Results Output)

```
┌─────────────┐       ┌──────────────┐       ┌─────────────┐
│   Webhook   │──────▶│  Database    │──────▶│   Response  │
│  (Request)  │       │   (Fetch)    │       │   (Return)  │
└─────────────┘       └──────────────┘       └─────────────┘
```

**Response Node Example**:
```javascript
// Get project_id from query
const projectId = $('Webhook').item.json.query.project_id;

// Fetch from database (example with mock data)
const analysisData = {
  asset_info: {
    equipment_name: items[0].json.equipment_name,
    tag_number: items[0].json.tag_number,
    criticality_level: items[0].json.criticality_level
  },
  risk_assessment: {
    overall_risk: "High",
    risk_priority_number: 240,
    failure_modes: 12
  },
  maintenance_recommendations: {
    frequency: "Monthly",
    type: "Predictive + Preventive",
    priority: "High"
  },
  cost_analysis: {
    estimated_annual_cost: 45000,
    potential_savings: 18000
  },
  analysis_timestamp: new Date().toISOString()
};

return { json: analysisData };
```

---

## Environment Variables (Optional)

For better security, you can use environment variables:

**File**: `.env.local`
```env
VITE_N8N_INPUT_WEBHOOK=https://your-n8n-instance.com/webhook/rcm/asset-input
VITE_N8N_OUTPUT_WEBHOOK=https://your-n8n-instance.com/webhook/rcm/analysis-results
```

**Usage in code**:
```typescript
const n8nWebhookUrl = import.meta.env.VITE_N8N_INPUT_WEBHOOK;
const n8nResultsWebhookUrl = import.meta.env.VITE_N8N_OUTPUT_WEBHOOK;
```

---

## Troubleshooting

### Data not sending to n8n
- Check the browser console for errors
- Verify webhook URL is correct
- Check CORS settings in n8n
- Ensure n8n workflow is activated

### Dashboard not showing results
- Check if n8n is returning data in the correct format
- Verify the GET endpoint is working (test in browser/Postman)
- Check browser network tab for failed requests
- Ensure project_id is being passed correctly

### CORS Errors
In n8n, enable CORS for your webhooks:
- Go to Settings → Security
- Add your app URL to allowed origins: `http://localhost:5176`

---

## Production Deployment

When deploying to production:

1. **Update webhook URLs** to production n8n instance
2. **Add authentication** to webhooks (API keys, OAuth, etc.)
3. **Enable HTTPS** for all webhook communications
4. **Set up error handling** and logging
5. **Configure rate limiting** to prevent abuse

---

## Additional Resources

- n8n Documentation: https://docs.n8n.io/
- Webhook Node: https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/
- RCM Best Practices: (Add your internal documentation)

---

**Need Help?**
Contact your n8n administrator or refer to the n8n community forum.
