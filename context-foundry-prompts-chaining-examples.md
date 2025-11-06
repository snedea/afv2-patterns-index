# Context Foundry Build Prompts: Chaining Pattern Examples

**Purpose**: Autonomous workflow generation for LinkedIn blog post examples
**Target**: Flowise AgentFlow v2 workflows
**Pattern**: Chaining (Sequential Intelligence)

---

## How to Use These Prompts

Each prompt below can be used with Context Foundry's autonomous build system to generate complete, production-ready Flowise workflows.

### Using the MCP Function

```python
# Example invocation
result = autonomous_build_and_deploy(
    task="[PASTE PROMPT BELOW]",
    working_directory="invoice-processing-chain",
    github_repo_name="flowise-example-invoice-chain",
    mode="new_project"
)
```

### Recording GIFs/Videos

After building each workflow:

1. **Import to Flowise**: Load the generated JSON file
2. **Configure API keys**: Set Anthropic credentials
3. **Test the workflow**: Run with sample data
4. **Record**:
   - **GIF**: Use LICEcap (Mac) or ScreenToGif (Windows)
   - **Video**: QuickTime (Mac) or OBS (cross-platform)
   - Keep recordings 30-120 seconds
   - Focus on node-by-node execution
   - Show Flow State updates

---

## Prompt 1: Document Processing Chain (Invoice Example)

### Build Command

```
Build a Flowise AgentFlow v2 workflow for intelligent invoice processing using the Chaining pattern.

WORKFLOW NAME: Document Processing Chain

WORKFLOW TYPE: flowise_flow (Chaining pattern - sequential agent pipeline)

FLOW STRUCTURE:
Start → Extract Data → Validate → Enrich → Risk Check → Route → Process → Confirm

NODE SPECIFICATIONS:

1. Start Node
   - Type: Start
   - Name: "invoiceProcessingStart"
   - Form inputs:
     * invoiceFile: file upload
     * vendorName: text (optional)
   - Flow State initialization:
     * extracted_data: {}
     * validation_result: {}
     * enriched_data: {}
     * risk_score: 0
     * processing_status: "pending"

2. Extract Data Agent
   - Type: Agent
   - Name: "Agent.DataExtractor"
   - Model: claude-sonnet-4-5-20250929
   - System Prompt:
     "You are a data extraction specialist for invoice processing.

     TASK: Extract structured data from the invoice file.

     Extract:
     - Invoice number
     - Vendor name
     - Total amount
     - Line items (description, quantity, unit price)
     - Date
     - Payment terms

     Identify flags:
     - First-time vendor
     - Rush payment requested
     - Unusual amounts
     - Missing information

     OUTPUT FORMAT (JSON):
     {
       \"invoice_number\": \"string\",
       \"vendor\": \"string\",
       \"total\": number,
       \"line_items\": [{\"description\": \"string\", \"qty\": number, \"unit_price\": number}],
       \"date\": \"YYYY-MM-DD\",
       \"payment_terms\": \"string\",
       \"confidence\": 0.0-1.0,
       \"flags\": [\"array of strings\"]
     }

     Store result in Flow State as: extracted_data"
   - Tools: currentDateTime, searXNG
   - Memory: 10 messages

3. Validate Agent
   - Type: Agent
   - Name: "Agent.Validator"
   - Model: claude-sonnet-4-5-20250929
   - System Prompt:
     "You are a validation specialist for invoice processing.

     TASK: Validate the extracted invoice data.

     Extracted Data: {{$flow.state.extracted_data}}

     Perform checks:
     1. Vendor in approved list (simulate: if vendor name contains 'Corp' or 'Inc' = approved)
     2. Format validation (all required fields present)
     3. Amount reasonableness (< $10,000 = reasonable)
     4. Date validity

     OUTPUT FORMAT (JSON):
     {
       \"vendor_check\": \"approved\" | \"needs_review\",
       \"format_check\": \"pass\" | \"fail\",
       \"amount_check\": \"pass\" | \"warning\",
       \"all_checks_passed\": boolean,
       \"confidence\": 0.0-1.0,
       \"issues\": [\"array of strings\"]
     }

     Store result in Flow State as: validation_result"
   - Tools: currentDateTime
   - Memory: 10 messages

4. Enrich Agent
   - Type: Agent
   - Name: "Agent.Enricher"
   - Model: claude-sonnet-4-5-20250929
   - System Prompt:
     "You are a data enrichment specialist.

     TASK: Enrich invoice data with additional context.

     Extracted: {{$flow.state.extracted_data}}
     Validation: {{$flow.state.validation_result}}

     Add context:
     - Vendor history (simulate: new vendor if no history)
     - Budget impact (simulate: department budget $50K)
     - Approval chain needed
     - Risk assessment

     OUTPUT FORMAT (JSON):
     {
       \"vendor_history\": {
         \"is_new\": boolean,
         \"total_invoices\": number,
         \"avg_amount\": number | null
       },
       \"budget_impact\": {
         \"department\": \"Operations\",
         \"remaining\": number,
         \"percentage_used\": number
       },
       \"approval_chain\": [\"Manager\", \"Finance Director\"],
       \"risk_score\": 1-10
     }

     Store as: enriched_data and risk_score"
   - Tools: currentDateTime
   - Memory: 10 messages

5. Risk Router (Condition Node)
   - Type: Condition
   - Name: "riskRouter"
   - Scenarios:
     * Scenario 1: "highRisk"
       - Condition: risk_score > 7
       - Description: "High risk - needs manual review"
     * Scenario 2: "lowRisk" (default)
       - Description: "Low risk - auto-process"

6a. Human Review (Human Input Node) - High Risk Path
   - Type: HumanInput
   - Name: "manualReview"
   - Title: "Invoice Review Required"
   - Description: "Risk Score: {{$flow.state.risk_score}}/10
                   Vendor: {{$flow.state.extracted_data.vendor}}
                   Amount: ${{$flow.state.extracted_data.total}}

                   Please review and approve."
   - Approval message: "Review the invoice details above"

6b. Auto Process Agent - Low Risk Path
   - Type: Agent
   - Name: "Agent.AutoProcessor"
   - Model: claude-3-haiku-20240307
   - System Prompt:
     "You are an automated invoice processor.

     TASK: Process the approved invoice automatically.

     Invoice: {{$flow.state.extracted_data}}

     Simulate these actions:
     1. Update ERP system
     2. Schedule payment (15 days from today)
     3. Send vendor confirmation email
     4. Create audit log entry

     OUTPUT: Confirmation message with action summary"
   - Tools: currentDateTime
   - Memory: 5 messages

7. Final Confirmation Agent
   - Type: Agent
   - Name: "Agent.Report"
   - Model: claude-3-haiku-20240307
   - System Prompt:
     "Generate final processing confirmation.

     Invoice: {{$flow.state.extracted_data}}
     Risk Score: {{$flow.state.risk_score}}

     Create confirmation message with:
     - Invoice number
     - Vendor
     - Amount
     - Payment schedule
     - Processing time

     Store as: final_report"
   - Tools: currentDateTime
   - Memory: 5 messages

8. Direct Reply (Terminal Node)
   - Type: DirectReply
   - Name: "processingComplete"
   - Message: "{{$flow.state.final_report}}"

EDGE CONNECTIONS:
- Start → Agent.DataExtractor
- Agent.DataExtractor → Agent.Validator
- Agent.Validator → Agent.Enricher
- Agent.Enricher → riskRouter
- riskRouter (highRisk) → manualReview
- riskRouter (lowRisk) → Agent.AutoProcessor
- manualReview → Agent.Report
- Agent.AutoProcessor → Agent.Report
- Agent.Report → processingComplete

SUCCESS CRITERIA:
- 8 nodes total (1 Start, 5 Agents, 1 Condition, 1 HIL, 1 Direct Reply)
- Flow State tracks: extracted_data, validation_result, enriched_data, risk_score
- Conditional routing based on risk score
- Human review only for high-risk invoices
- Complete audit trail in Flow State

TESTING:
1. Test Case 1 (Low Risk):
   - Invoice: "Acme Corp, $2,500"
   - Expected: Auto-process (no human review)

2. Test Case 2 (High Risk):
   - Invoice: "Unknown Vendor LLC, $8,500"
   - Expected: Route to human review
```

---

## Prompt 2: Cascade Pattern (Multi-Tier Model Routing)

### Build Command

```
Build a Flowise AgentFlow v2 workflow demonstrating the Cascade pattern - routing queries through progressively more powerful models.

WORKFLOW NAME: Cascade Pattern Demo

WORKFLOW TYPE: flowise_flow (Chaining pattern with model escalation)

FLOW STRUCTURE:
Start → Small Model Triage → Confidence Check → [High Confidence → Reply | Low Confidence → Large Model → Reply]

NODE SPECIFICATIONS:

1. Start Node
   - Type: Start
   - Name: "cascadeStart"
   - Form inputs:
     * userQuery: text area
   - Flow State:
     * query: ""
     * triage_confidence: 0
     * triage_answer: ""
     * final_answer: ""
     * model_used: ""

2. Fast Triage Agent (Small Model)
   - Type: Agent
   - Name: "Agent.FastTriage"
   - Model: claude-3-haiku-20240307
   - System Prompt:
     "You are a fast triage agent using a lightweight model.

     TASK: Answer the user's query if you can with HIGH CONFIDENCE.

     User Query: {{$flow.state.query}}

     If the query is simple (facts, basic calculations, straightforward questions):
     - Provide your best answer
     - Set confidence HIGH (0.90-1.0)

     If the query is complex (requires reasoning, analysis, expertise):
     - Provide a brief attempt
     - Set confidence LOW (0.0-0.70)

     OUTPUT FORMAT (JSON):
     {
       \"answer\": \"your answer here\",
       \"confidence\": 0.0-1.0,
       \"reasoning\": \"why this confidence level\",
       \"escalation_needed\": boolean
     }

     Store as: triage_result"
   - Tools: currentDateTime, searXNG
   - Memory: 5 messages

3. Confidence Router (Condition Node)
   - Type: Condition
   - Name: "confidenceRouter"
   - Scenarios:
     * Scenario 1: "highConfidence"
       - Condition: triage_result.confidence >= 0.90
       - Description: "High confidence - use triage answer"
     * Scenario 2: "lowConfidence" (default)
       - Description: "Low confidence - escalate to large model"

4a. Triage Answer Direct Reply (High Confidence Path)
   - Type: DirectReply
   - Name: "triageReply"
   - Message: "✅ **Answered by Fast Model (Haiku)**

                {{$flow.state.triage_result.answer}}

                Confidence: {{$flow.state.triage_result.confidence}}
                Cost: ~$0.0002"

4b. Deep Analysis Agent (Low Confidence Path - Large Model)
   - Type: Agent
   - Name: "Agent.DeepAnalysis"
   - Model: claude-sonnet-4-5-20250929
   - System Prompt:
     "You are an expert analyst using a powerful model.

     TASK: Provide comprehensive analysis for this complex query.

     User Query: {{$flow.state.query}}
     Previous Attempt: {{$flow.state.triage_result.answer}}

     Provide a thorough, well-reasoned answer with:
     - Detailed analysis
     - Supporting evidence
     - Multiple perspectives if applicable
     - Clear conclusion

     Take your time and use your full capabilities.

     Store as: deep_analysis_result"
   - Tools: currentDateTime, searXNG
   - Memory: 15 messages

5. Deep Analysis Reply (Terminal Node)
   - Type: DirectReply
   - Name: "deepAnalysisReply"
   - Message: "✅ **Answered by Large Model (Sonnet 4.5)**

                {{$flow.state.deep_analysis_result}}

                (Escalated from fast model due to complexity)
                Cost: ~$0.0032"

EDGE CONNECTIONS:
- Start → Agent.FastTriage
- Agent.FastTriage → confidenceRouter
- confidenceRouter (highConfidence) → triageReply
- confidenceRouter (lowConfidence) → Agent.DeepAnalysis
- Agent.DeepAnalysis → deepAnalysisReply

SUCCESS CRITERIA:
- 70% of simple queries answered by fast model
- Cost reduction: 3x lower average cost
- Response time: 2x faster for simple queries
- Quality maintained for complex queries

TESTING:
1. Simple Query: "What's 2+2?"
   - Expected: Fast model answers directly

2. Complex Query: "Analyze the economic implications of distributed AI systems on labor markets"
   - Expected: Escalates to large model
```

---

## Prompt 3: Data Enrichment Pipeline

### Build Command

```
Build a Flowise AgentFlow v2 workflow demonstrating progressive data enrichment - each step adds value to customer data.

WORKFLOW NAME: Data Enrichment Pipeline

WORKFLOW TYPE: flowise_flow (Chaining pattern with progressive enrichment)

FLOW STRUCTURE:
Start → Clean → Enrich → Analyze → Recommend → Report → Reply

NODE SPECIFICATIONS:

1. Start Node
   - Type: Start
   - Name: "enrichmentStart"
   - Form inputs:
     * customerEmail: text
     * customerName: text (optional)
     * signupDate: date (optional)
   - Flow State:
     * raw_data: {}
     * cleaned_data: {}
     * enriched_data: {}
     * analyzed_data: {}
     * recommendations: {}

2. Data Cleaning Agent
   - Type: Agent
   - Name: "Agent.DataCleaner"
   - Model: claude-3-haiku-20240307
   - System Prompt:
     "You are a data cleaning specialist.

     TASK: Clean and validate input data.

     Input:
     - Email: {{$flow.state.raw_data.email}}
     - Name: {{$flow.state.raw_data.name}}
     - Signup Date: {{$flow.state.raw_data.signup_date}}

     Clean and validate:
     1. Email format validation
     2. Extract domain from email
     3. Standardize name formatting
     4. Validate date format

     OUTPUT FORMAT (JSON):
     {
       \"email\": \"lowercase@domain.com\",
       \"email_valid\": boolean,
       \"domain\": \"domain.com\",
       \"name\": \"Proper Case\",
       \"signup_date\": \"ISO 8601 format\",
       \"data_quality_score\": 0.0-1.0
     }

     Store as: cleaned_data"
   - Tools: currentDateTime
   - Memory: 5 messages

3. Data Enrichment Agent
   - Type: Agent
   - Name: "Agent.DataEnricher"
   - Model: claude-sonnet-4-5-20250929
   - System Prompt:
     "You are a data enrichment specialist.

     TASK: Enrich customer data with additional context.

     Cleaned Data: {{$flow.state.cleaned_data}}

     Use the domain to infer:
     - Company name (from domain)
     - Industry (simulate based on domain keywords)
     - Company size (simulate: small/medium/large)
     - Location (simulate based on TLD or common patterns)

     Example enrichment logic:
     - *.edu = Education, Large, Academic
     - *.gov = Government, Large, Public Sector
     - Contains 'tech' = Technology, varies
     - Generic domains = Unknown

     OUTPUT FORMAT (JSON):
     {
       ...cleaned_data,
       \"company\": \"string\",
       \"industry\": \"string\",
       \"company_size\": \"small\" | \"medium\" | \"large\",
       \"location\": \"string\",
       \"enrichment_confidence\": 0.0-1.0
     }

     Store as: enriched_data"
   - Tools: currentDateTime, searXNG
   - Memory: 10 messages

4. Data Analysis Agent
   - Type: Agent
   - Name: "Agent.DataAnalyzer"
   - Model: claude-sonnet-4-5-20250929
   - System Prompt:
     "You are a customer data analyst.

     TASK: Analyze enriched data to derive insights.

     Enriched Data: {{$flow.state.enriched_data}}

     Analyze and determine:
     - Customer segment (e.g., Enterprise, Mid-Market, SMB)
     - Estimated lifetime value (simulate based on company size)
     - Churn risk (simulate: new customers = low risk)
     - Engagement score (simulate: 1-10 based on data quality)

     OUTPUT FORMAT (JSON):
     {
       ...enriched_data,
       \"customer_segment\": \"string\",
       \"lifetime_value_estimate\": number,
       \"churn_risk\": \"low\" | \"medium\" | \"high\",
       \"engagement_score\": 1-10,
       \"analysis_timestamp\": \"ISO 8601\"
     }

     Store as: analyzed_data"
   - Tools: currentDateTime
   - Memory: 10 messages

5. Recommendation Engine Agent
   - Type: Agent
   - Name: "Agent.RecommendationEngine"
   - Model: claude-sonnet-4-5-20250929
   - System Prompt:
     "You are a recommendation engine specialist.

     TASK: Generate actionable recommendations based on analysis.

     Analyzed Data: {{$flow.state.analyzed_data}}

     Generate recommendations for:
     - Recommended pricing tier
     - Upsell opportunities
     - Next best action (e.g., product demo, trial extension)
     - Estimated conversion probability
     - Suggested outreach timing

     OUTPUT FORMAT (JSON):
     {
       \"recommended_plan\": \"string\",
       \"upsell_opportunities\": [\"array\"],
       \"next_best_action\": \"string\",
       \"conversion_probability\": 0.0-1.0,
       \"suggested_contact_date\": \"YYYY-MM-DD\",
       \"reasoning\": \"string\"
     }

     Store as: recommendations"
   - Tools: currentDateTime
   - Memory: 10 messages

6. Final Report Generator
   - Type: Agent
   - Name: "Agent.ReportGenerator"
   - Model: claude-3-haiku-20240307
   - System Prompt:
     "Generate comprehensive enrichment report.

     Data:
     - Raw: {{$flow.state.raw_data}}
     - Cleaned: {{$flow.state.cleaned_data}}
     - Enriched: {{$flow.state.enriched_data}}
     - Analyzed: {{$flow.state.analyzed_data}}
     - Recommendations: {{$flow.state.recommendations}}

     Create markdown report showing:
     1. Data quality improvements (raw → cleaned)
     2. Enrichment additions
     3. Analysis insights
     4. Actionable recommendations

     Store as: final_report"
   - Tools: currentDateTime
   - Memory: 5 messages

7. Direct Reply (Terminal Node)
   - Type: DirectReply
   - Name: "enrichmentComplete"
   - Message: "{{$flow.state.final_report}}"

EDGE CONNECTIONS:
- Start → Agent.DataCleaner
- Agent.DataCleaner → Agent.DataEnricher
- Agent.DataEnricher → Agent.DataAnalyzer
- Agent.DataAnalyzer → Agent.RecommendationEngine
- Agent.RecommendationEngine → Agent.ReportGenerator
- Agent.ReportGenerator → enrichmentComplete

SUCCESS CRITERIA:
- 7 nodes total
- Each step adds measurable value
- Progressive enrichment visible in Flow State
- Final report shows complete data journey

TESTING:
Test Input:
- Email: "john@techcorp.com"
- Name: "john doe"
- Signup Date: "2025-11-06"

Expected Enrichment:
- Clean: Proper email, domain extraction
- Enrich: Company = TechCorp, Industry = Technology
- Analyze: Segment, LTV estimate, engagement score
- Recommend: Plan tier, next actions
```

---

## Prompt 4: Customer Onboarding Chain (Complete Implementation)

### Build Command

```
Build a complete, production-ready Flowise AgentFlow v2 workflow for customer onboarding with intelligent risk-based routing.

WORKFLOW NAME: Customer Onboarding Chain

WORKFLOW TYPE: flowise_flow (Chaining pattern with conditional routing and HIL gate)

FLOW STRUCTURE:
Start → Identity Verify → Risk Assessment → Route → [Low Risk → Auto Approve | High Risk → Manual Review] → Welcome → Complete

NODE SPECIFICATIONS:

1. Start Node
   - Type: Start
   - Name: "onboardingStart"
   - Form inputs:
     * customerId: text (required)
     * customerEmail: email (required)
     * customerName: text (required)
     * documentType: select (ID, Passport, Driver License)
   - Flow State:
     * customer_id: ""
     * customer_email: ""
     * verification_status: "pending"
     * risk_score: 0
     * risk_category: ""
     * onboarding_path: ""
     * account_created: false

2. Identity Verification Agent
   - Type: Agent
   - Name: "Agent.IdentityVerifier"
   - Model: claude-sonnet-4-5-20250929
   - System Prompt:
     "You are an identity verification specialist for customer onboarding.

     TASK: Verify customer identity and documents.

     Customer: {{$flow.state.customer_id}}
     Email: {{$flow.state.customer_email}}
     Document Type: {{$flow.state.document_type}}

     Simulate verification checks:
     1. Document validity (assume valid if document type provided)
     2. Name matching (assume matches if name provided)
     3. Sanctions screening (assume clear unless name contains 'blocked')
     4. Address verification (simulate based on email domain)

     OUTPUT FORMAT (JSON):
     {
       \"verification_status\": \"verified\" | \"needs_review\" | \"rejected\",
       \"confidence\": 0.0-1.0,
       \"checks_passed\": [\"array of checks that passed\"],
       \"checks_failed\": [\"array of checks that failed\"],
       \"risk_flags\": [\"array of risk indicators\"],
       \"recommendation\": \"string\"
     }

     Store result in Flow State as: verification_result"
   - Tools: currentDateTime, searXNG
   - Memory: 10 messages

3. Risk Assessment Agent
   - Type: LLM
   - Name: "riskAssessment"
   - Model: claude-sonnet-4-5-20250929
   - System Prompt:
     "You are a risk assessment specialist for customer onboarding."
   - User Message:
     "Assess onboarding risk for this customer:

     Customer ID: {{$flow.state.customer_id}}
     Email: {{$flow.state.customer_email}}
     Verification: {{$flow.state.verification_result}}

     Determine risk score (1-10) based on:
     - Verification confidence
     - Email domain (corporate = lower risk, free email = higher risk)
     - Any risk flags from verification

     Provide:
     - Risk score (1-10)
     - Risk category (low/medium/high)
     - Risk factors
     - Recommended tier (standard/enhanced/restricted)"
   - JSON Output Format:
     {
       "riskScore": "number 1-10",
       "riskCategory": "low | medium | high",
       "riskFactors": "array of strings",
       "recommendedTier": "standard | enhanced | restricted",
       "reasoning": "string"
     }
   - Update Flow State:
     * risk_score: {{output.riskScore}}
     * risk_category: {{output.riskCategory}}

4. Risk Router (Condition Node)
   - Type: Condition
   - Name: "riskRouter"
   - Scenarios:
     * Scenario 1: "highRisk"
       - Condition: risk_score > 7
       - Description: "High risk - manual review required"
     * Scenario 2: "mediumRisk"
       - Condition: risk_score between 4 and 7
       - Description: "Medium risk - enhanced verification"
     * Scenario 3: "lowRisk" (default)
       - Description: "Low risk - auto-approve"

5a. Auto-Approval Agent (Low/Medium Risk Path)
   - Type: Agent
   - Name: "Agent.AutoApprove"
   - Model: claude-3-haiku-20240307
   - System Prompt:
     "You are an automated account provisioning agent.

     TASK: Provision customer account automatically.

     Customer: {{$flow.state.customer_id}}
     Tier: {{$flow.state.risk_assessment.recommendedTier}}

     Simulate account creation:
     1. Create account record
     2. Assign tier
     3. Generate welcome credentials
     4. Log onboarding event

     OUTPUT: Confirmation message with account details

     Store result and set: account_created = true, onboarding_path = 'auto_approved'"
   - Tools: currentDateTime
   - Memory: 5 messages

5b. Manual Review (High Risk Path - HIL Gate)
   - Type: HumanInput
   - Name: "manualReview"
   - Title: "High Risk Customer - Manual Review Required"
   - Description:
     "Customer Onboarding Review Needed

     Customer: {{$flow.state.customer_email}}
     Risk Score: {{$flow.state.risk_score}}/10
     Risk Category: {{$flow.state.risk_category}}
     Risk Factors: {{$flow.state.risk_assessment.riskFactors}}

     Verification Status: {{$flow.state.verification_result.verification_status}}

     Please review and make decision:"
   - Approval Message: "Review customer details and decide"

6. Account Provisioning After Manual Review
   - Type: Agent
   - Name: "Agent.ManualApprove"
   - Model: claude-3-haiku-20240307
   - System Prompt:
     "You are an account provisioning agent following manual review.

     TASK: Provision account after human approval.

     Customer: {{$flow.state.customer_id}}
     Manual Decision: {{manualReview.approved}}

     Create account and set: account_created = true, onboarding_path = 'manual_approved'"
   - Tools: currentDateTime
   - Memory: 5 messages

7. Welcome Email Agent
   - Type: Agent
   - Name: "Agent.WelcomeEmail"
   - Model: claude-3-haiku-20240307
   - System Prompt:
     "Generate personalized welcome message for new customer.

     Customer: {{$flow.state.customer_email}}
     Tier: {{$flow.state.risk_assessment.recommendedTier}}
     Onboarding Path: {{$flow.state.onboarding_path}}

     Create warm, professional welcome message with:
     - Greeting
     - Account details
     - Next steps
     - Support contact

     Store as: welcome_message"
   - Tools: currentDateTime
   - Memory: 5 messages

8. Direct Reply (Terminal Node)
   - Type: DirectReply
   - Name: "onboardingComplete"
   - Message: "✅ **Customer Onboarding Complete**

                Customer: {{$flow.state.customer_id}}
                Email: {{$flow.state.customer_email}}
                Tier: {{$flow.state.risk_assessment.recommendedTier}}
                Path: {{$flow.state.onboarding_path}}
                Risk Score: {{$flow.state.risk_score}}/10

                {{$flow.state.welcome_message}}

                Account Status: Active ✓
                Processing Time: {{$flow.executionTime}} seconds"

EDGE CONNECTIONS:
- Start → Agent.IdentityVerifier
- Agent.IdentityVerifier → riskAssessment
- riskAssessment → riskRouter
- riskRouter (lowRisk) → Agent.AutoApprove
- riskRouter (mediumRisk) → Agent.AutoApprove
- riskRouter (highRisk) → manualReview
- Agent.AutoApprove → Agent.WelcomeEmail
- manualReview → Agent.ManualApprove
- Agent.ManualApprove → Agent.WelcomeEmail
- Agent.WelcomeEmail → onboardingComplete

SUCCESS CRITERIA:
- 10 nodes total (1 Start, 5 Agents, 1 LLM, 1 Condition, 1 HIL, 1 Direct Reply)
- Risk-based routing (85% auto-approve, 15% manual review)
- Complete audit trail in Flow State
- Human review only for high-risk cases

TESTING:
1. Low Risk Test:
   - Email: "john@bigcorp.com"
   - Expected: Auto-approve path

2. High Risk Test:
   - Email: "unknown@freemail.com"
   - Expected: Manual review required
```

---

## Recording Guidelines

### For Each Workflow

**1. Import Phase** (10 seconds)
- Show importing JSON into Flowise
- Quick pan across the workflow canvas
- Highlight node count

**2. Configuration Phase** (20 seconds)
- Show one agent node configuration
- Highlight model selection
- Show system prompt snippet

**3. Execution Phase** (60 seconds)
- Click "Test" button
- Show input form
- Enter test data
- Watch nodes execute sequentially
- Highlight Flow State updates

**4. Results Phase** (10 seconds)
- Show final Direct Reply message
- Display Flow State variables
- Show execution time

### GIF Optimization
- **Resolution**: 1280x720 or 1920x1080
- **Frame rate**: 10-15 fps (GIFs), 30 fps (video)
- **File size**: <10MB for GIFs, <50MB for videos
- **Duration**: 30-120 seconds
- **Format**: GIF for loops, MP4 for linear

### Placeholder Mapping

| Placeholder | Workflow | Duration |
|-------------|----------|----------|
| `[WORKFLOW_1_GIF]` | Document Processing Chain | 90s |
| `[WORKFLOW_1_ROUTING_DEMO]` | Document Processing (routing focus) | 30s |
| `[WORKFLOW_2_CASCADE_GIF]` | Cascade Pattern | 60s |
| `[WORKFLOW_3_ENRICHMENT_DEMO]` | Data Enrichment Pipeline | 90s |
| `[WORKFLOW_4_ONBOARDING_GIF]` | Customer Onboarding Chain | 120s |

---

## Build Order Recommendation

1. **Start with Cascade Pattern** (simplest - 5 nodes)
   - Easiest to record
   - Clear value demonstration
   - Quick win

2. **Document Processing Chain** (8 nodes)
   - Core chaining example
   - Shows conditional routing
   - HIL gate demonstration

3. **Data Enrichment Pipeline** (7 nodes)
   - Progressive value story
   - Flow State progression visible
   - Clean linear flow

4. **Customer Onboarding Chain** (10 nodes)
   - Most complex
   - Production-ready example
   - Full feature demonstration

---

## Next Steps

1. **Build workflows** using Context Foundry prompts above
2. **Import to Flowise** and configure API keys
3. **Test each workflow** with provided test cases
4. **Record GIFs/videos** following guidelines
5. **Replace placeholders** in blog post markdown
6. **Publish on LinkedIn** with embedded media

---

*For questions or issues, check: [Context Foundry Documentation](https://github.com/snedea/context-foundry)*
