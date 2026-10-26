# RCM Application - Data Requirements for Calculations

This document outlines the data required for each RCM feature to perform accurate calculations.

---

## 1. Risk Management

### Purpose
Assess and mitigate operational risks associated with equipment failures.

### Required Data

| Field | Type | Description | Scale/Options |
|-------|------|-------------|---------------|
| `risk_id` | string | Unique identifier | Auto-generated |
| `risk_description` | string | Description of the risk | Free text |
| `probability` | string | Likelihood of occurrence | Low, Medium, High |
| `impact` | string | Severity of consequences | Minor, Moderate, Severe, Critical |
| `risk_level` | string | **Calculated** from probability × impact | Low, Medium, High |
| `mitigation_strategy` | string | Actions to reduce risk | Free text |
| `risk_category` | string | Classification of risk type | Safety, Environmental, Operational, Financial |
| `fmea_id` | string | Link to FMEA analysis (optional) | Reference |
| `asset_id` | string | Associated equipment | Reference |

### Calculation Logic
```
Risk Level = f(Probability, Impact)

If Probability=High AND Impact=Critical/Severe → High Risk
If Probability=Low AND Impact=Minor/Moderate → Low Risk
Otherwise → Medium Risk
```

### Additional Data for Enhanced Analysis
- Historical incident data
- Industry benchmarks
- Regulatory requirements
- Insurance requirements

---

## 2. Failure Mode & Effects Analysis (FMEA)

### Purpose
Identify potential failure modes, their causes, effects, and prioritize based on Risk Priority Number (RPN).

### Required Data

| Field | Type | Description | Scale/Range |
|-------|------|-------------|-------------|
| `fmea_id` | string | Unique identifier | Auto-generated |
| `asset_id` | string | Equipment reference | Reference |
| `component` | string | Part being analyzed | Free text |
| `failure_mode` | string | How the part fails | e.g., Seizure, Leakage, Overheating |
| `failure_cause` | string | Root cause of failure | e.g., Wear, Corrosion, Fatigue |
| `failure_effect` | string | Impact on system | e.g., Complete Shutdown, Reduced Efficiency |
| `severity` (S) | number | Impact of failure | **1-10** (10=catastrophic) |
| `occurrence` (O) | number | Frequency of failure | **1-10** (10=very frequent) |
| `detection` (D) | number | Ability to detect before failure | **1-10** (10=cannot detect) |
| `rpn` | number | **Calculated**: S × O × D | 1-1000 |
| `risk_level` | string | **Calculated** from RPN | Low, Medium, High |
| `recommended_action` | string | **Generated** mitigation strategy | Text |

### RPN Calculation
```
RPN = Severity × Occurrence × Detection

Risk Level Classification:
- High Risk:   RPN ≥ 300
- Medium Risk: RPN 100-299
- Low Risk:    RPN < 100
```

### Severity Scale (1-10)
| Rating | Effect | Description |
|--------|--------|-------------|
| 1 | None | No effect |
| 2-3 | Minor | Slight inconvenience |
| 4-5 | Low | Some dissatisfaction |
| 6-7 | Moderate | Reduced performance |
| 8 | High | Inoperable with warning |
| 9 | Very High | Inoperable without warning |
| 10 | Hazardous | Safety hazard |

### Occurrence Scale (1-10)
| Rating | Probability | Failure Rate |
|--------|-------------|--------------|
| 1 | Remote | <1 in 1,500,000 |
| 2-3 | Low | 1 in 150,000 |
| 4-6 | Moderate | 1 in 15,000 |
| 7-8 | High | 1 in 2,000 |
| 9-10 | Very High | >1 in 20 |

### Detection Scale (1-10)
| Rating | Detection | Description |
|--------|-----------|-------------|
| 1 | Almost Certain | Controls will detect |
| 2-3 | High | High chance of detection |
| 4-6 | Moderate | May detect |
| 7-8 | Low | Unlikely to detect |
| 9-10 | Remote/None | Cannot detect |

### Recommended Action Logic
```
If Severity ≥ 9 → Predictive Maintenance required
If RPN ≥ 400 → Immediate action required
If RPN ≥ 300 → Planned preventive maintenance
If RPN ≥ 100 → Scheduled maintenance
If RPN < 100 → Run-to-failure acceptable
```

---

## 3. Operating Costs

### Purpose
Track, analyze, and optimize maintenance operating costs.

### Required Data

| Field | Type | Description | Unit |
|-------|------|-------------|------|
| `cost_id` | string | Unique identifier | Auto-generated |
| `asset_id` | string | Equipment reference | Reference |
| `period` | string | Time period | Month/Year |
| `planned_cost` | number | Budgeted amount | $ (Currency) |
| `actual_cost` | number | Actual spending | $ (Currency) |
| `variance` | number | **Calculated**: Actual - Planned | $ (Currency) |

### Cost Breakdown Categories

| Category | Description | Typical % |
|----------|-------------|-----------|
| `labor_cost` | Technician wages, overtime | 30-40% |
| `spare_parts_cost` | Replacement parts, consumables | 20-30% |
| `downtime_cost` | Production losses during repairs | 15-25% |
| `contractor_cost` | External service providers | 10-15% |
| `overhead_cost` | Tools, equipment, facilities | 5-10% |

### Asset-Level Cost Data (from `CostData` type)
```typescript
{
  cost_id: string;
  asset_id: string;
  downtime_cost_per_hour: number;  // $/hr lost production
  repair_cost: number;              // Average repair cost
  replacement_cost: number;         // New equipment cost
  spare_parts_cost: number;         // Inventory value
  labor_cost_per_hour: number;      // Technician rate
}
```

### Calculations

**Annual Operating Cost:**
```
Annual Cost = Operating Hours × Operating Cost/Hour 
            + Maintenance Cost 
            + Downtime Cost 
            + Failure Cost

Where:
- Maintenance Cost = (Preventive + Predictive + Corrective)
- Downtime Cost = Downtime Hours × Downtime Cost/Hour
- Failure Cost = Failure Frequency × Cost Per Failure
```

**Cost Per Failure:**
```
Cost Per Failure = Repair Cost 
                 + (MTTR × Downtime Cost/Hour) 
                 + Spare Parts Cost
```

**Maintenance Efficiency Ratio:**
```
Efficiency = Planned Maintenance Cost / Total Maintenance Cost × 100%
Target: >80%
```

---

## 4. Optimal Replacement Analysis

### Purpose
Determine the economically optimal time to replace equipment vs. continuing maintenance.

### Required Data

| Field | Type | Description | Unit |
|-------|------|-------------|------|
| `analysis_id` | string | Unique identifier | Auto-generated |
| `asset_id` | string | Equipment reference | Reference |
| `equipment_name` | string | Name of equipment | Text |
| `tag_number` | string | Asset tag | Text |
| `current_age` | number | Current age of asset | Years |
| `optimal_replacement_age` | number | Calculated optimal age | Years |
| `replacement_cost` | number | New equipment cost | $ |
| `years_remaining` | number | **Calculated**: Optimal - Current | Years |
| `status` | string | **Calculated** recommendation | Good, Monitor, Replace Soon |

### Additional Data for Analysis

| Field | Type | Description |
|-------|------|-------------|
| `installation_date` | date | Original installation |
| `operating_hours_per_day` | number | Daily usage |
| `mtbf_hours` | number | Mean Time Between Failures |
| `mttr_hours` | number | Mean Time To Repair |
| `failure_frequency_per_year` | number | Annual failure count |
| `repair_cost_trend` | string | Cost trajectory: Increasing/Stable/Decreasing |
| `salvage_value` | number | End-of-life value |
| `depreciation_rate` | number | Annual depreciation % |

### Maintenance History Data
```typescript
{
  history_id: string;
  asset_id: string;
  mtbf_hours: number;           // Mean Time Between Failures
  mttr_hours: number;           // Mean Time To Repair
  failure_frequency_per_year: number;
}
```

### Calculations

**Status Determination:**
```
Years Remaining = Optimal Age - Current Age

If Years Remaining ≤ 0 → "Replace Soon"
If Years Remaining ≤ 2 → "Monitor"
If Years Remaining > 2 → "Good"
```

**Equivalent Annual Cost (EAC) Method:**
```
EAC = (Purchase Cost - Salvage Value) × CRF + Annual Operating Cost

Where:
CRF = Capital Recovery Factor = i(1+i)^n / [(1+i)^n - 1]
i = Interest rate
n = Economic life in years

Decision:
If EAC(New) < EAC(Existing) → Replace
```

**Optimal Replacement Age:**
```
Minimize: Total Annual Cost = Ownership Cost + Operating Cost + Maintenance Cost

Optimal age occurs when:
- Annual maintenance costs start exceeding capital recovery costs
- Failure frequency increases significantly
- Efficiency drops below acceptable threshold
```

### Replacement Recommendation Logic
```
If Current Age > 1.2 × Optimal Age AND Failure Frequency ≥ 4/year → "Immediate Replacement"
If Current Age > Optimal Age OR Repair Cost Trend = "Increasing" → "Planned Replacement"
Otherwise → "Continue Operation"
```

---

## 5. Reporting

### Purpose
Generate comprehensive RCM reports for documentation, auditing, and decision-making.

### Required Data (Aggregated from all modules)

#### Project Information
```typescript
{
  project_id: string;
  project_name: string;
  location: string;
  industry: string;
  description: string;
  start_date: string;
}
```

#### Asset Information
```typescript
{
  asset_id: string;
  equipment_name: string;
  tag_number: string;
  system: string;
  subsystem: string;
  manufacturer: string;
  model: string;
  installation_date: string;
  criticality_level: 'Low' | 'Medium' | 'High';
}
```

#### Executive Summary Data
- Total assets analyzed
- High risk items count
- Completed vs pending tasks
- Total annual cost projection
- Key recommendations

#### FMEA Summary
```typescript
{
  fmea_summary: FMEAResult[];
  // Aggregates:
  total_failure_modes: number;
  high_risk_count: number;
  average_rpn: number;
}
```

#### Risk Assessment Summary
```typescript
{
  risk_assessment: RiskAssessment[];
  // By category:
  safety_risks: number;
  environmental_risks: number;
  operational_risks: number;
  financial_risks: number;
}
```

#### Maintenance Plan
```typescript
{
  maintenance_plan: MaintenanceTask[];
  // By type:
  preventive_tasks: number;
  predictive_tasks: number;
  corrective_tasks: number;
  run_to_failure: number;
}
```

#### Cost Analysis
```typescript
{
  cost_analysis: {
    total_annual_cost: number;
    downtime_cost: number;
    maintenance_cost: number;
    breakdown: {
      preventive: number;
      predictive: number;
      corrective: number;
    };
  };
}
```

#### Replacement Analysis
```typescript
{
  replacement_analysis: ReplacementAnalysis;
  cost_comparison: {
    continue_cost: number;
    replace_cost: number;
  };
}
```

### Report Structure

```
1. Executive Summary
   - Project overview
   - Key findings
   - Recommendations

2. Asset Information
   - Equipment details
   - Operating context
   - Criticality assessment

3. FMEA Analysis
   - Failure modes identified
   - RPN distribution
   - High-risk items list

4. Risk Assessment
   - Risk matrix
   - Mitigation strategies
   - Unacceptable risks

5. Maintenance Strategy
   - Task breakdown by type
   - Scheduling plan
   - Engineering justifications

6. Cost Analysis
   - Historical costs
   - Projected costs
   - ROI calculations

7. Replacement Recommendations
   - Equipment due for replacement
   - Cost-benefit analysis
   - Timeline

8. Appendices
   - Detailed FMEA tables
   - Supporting calculations
   - Reference documents
```

---

## Database Schema Summary

### Core Tables
1. **projects** - Project master data
2. **assets** - Equipment/asset data
3. **fmea_analysis** - FMEA records with RPN
4. **risk_assessments** - Risk register
5. **cost_data** - Cost parameters per asset
6. **maintenance_history** - MTBF, MTTR data
7. **maintenance_tasks** - Scheduled tasks
8. **replacement_analysis** - Replacement decisions

### Relationships
```
projects (1) → (many) assets
assets (1) → (many) fmea_analysis
assets (1) → (1) cost_data
assets (1) → (1) maintenance_history
fmea_analysis (1) → (many) risk_assessments
fmea_analysis (1) → (many) maintenance_tasks
assets (1) → (1) replacement_analysis
```

---

## Data Validation Rules

### Numeric Ranges
- Severity, Occurrence, Detection: 1-10 (integers)
- RPN: 1-1000 (calculated)
- Costs: ≥ 0
- Ages: ≥ 0
- Hours: ≥ 0

### Required Fields
- All IDs (auto-generated)
- asset_id for asset-related records
- At least S, O, D for FMEA calculations

### Calculated Fields (Do Not Input)
- RPN (S × O × D)
- Risk Level (from RPN or probability×impact)
- Years Remaining (Optimal Age - Current Age)
- Variance (Actual - Planned)
- Status (from calculations)
