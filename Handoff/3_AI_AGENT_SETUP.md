# 3_AI_AGENT_SETUP.md

## READYPROSPECT AI AGENT - SETUP & OPTIMIZATION GUIDE

### TABLE OF CONTENTS
1. [Agent Architecture](#agent-architecture)
2. [Initial Setup & Configuration](#initial-setup--configuration)
3. [Knowledge Base Creation](#knowledge-base-creation)
4. [Deployment to Twilio](#deployment-to-twilio)
5. [Script Management](#script-management)
6. [A/B Testing Framework](#ab-testing-framework)
7. [Performance Monitoring](#performance-monitoring)
8. [Optimization Loop](#optimization-loop)
9. [Troubleshooting](#troubleshooting)

---

## AGENT ARCHITECTURE

### What the Agent Does

```
INCOMING CALL
    ↓
Twilio routes to our API
    ↓
Cloud Function triggers
    ↓
Agent Context Prepared
├─ Customer info (industry, services)
├─ Caller info (if identified)
├─ Current script version
├─ Recent successful responses
└─ A/B test variant (10% new vs 90% control)
    ↓
Claude API Call
├─ System prompt: "You are a helpful sales assistant for [Company]"
├─ Context: Customer's services, pricing, objections
├─ Instruction: "Answer this caller's question naturally"
├─ Streaming: Real-time response to Twilio
└─ Cost: ~$0.01-0.03 per API call
    ↓
Agent Speaks to Caller
├─ Twilio Text-to-Speech: Converts response to audio
├─ Streaming: Real-time playback
└─ Natural: Human-like conversation
    ↓
Caller Responds
    ↓
Speech-to-Text (Google Cloud Speech API)
├─ Transcribe caller's response
├─ Extract intent (price question, objection, etc.)
├─ Sentiment analysis (happy, frustrated, etc.)
└─ Cost: ~$0.001 per minute
    ↓
Analyze & Respond
├─ Determine objection type
├─ Select best script (based on A/B test results)
├─ Generate response
└─ Repeat until call ends
    ↓
Call Recording & Analysis
├─ Store call duration
├─ Store transcript
├─ Record outcome (demo scheduled, info left, etc.)
├─ Analyze success
└─ Update scripts if improvement needed
```

### System Prompt (The Foundation)

```
You are [Company Name]'s AI Sales Assistant.

YOUR ROLE:
You answer incoming calls for [Company Name]. Your job is to:
1. Answer the caller's questions about our services
2. Understand their pain points and needs
3. Determine if they're a good fit for our services
4. Schedule a demo if they're interested
5. Leave contact info if they're not ready

YOUR KNOWLEDGE:
- Services: [List services with 1-2 sentence descriptions]
- Pricing: [Show tier 1, tier 2, tier 3 pricing]
- Unique advantages:
  - Faster response time than competitors (2 min vs industry avg 5 min)
  - Industry-specific AI training
  - 24/7 availability
  - ROI-focused approach
- Industries served: [List with success rates]

YOUR COMMUNICATION STYLE:
- Professional but friendly
- Conversational (not robotic)
- Ask clarifying questions
- Listen more than you talk
- Acknowledge concerns before dismissing them
- Use customer's language (if they say "emergency," mirror it)

COMMON OBJECTIONS & HOW TO HANDLE:
1. "How much does it cost?"
   → "Our pricing starts at $599/month. But first, let me understand your situation better. How many calls do you get per month?"
   
2. "We don't need this right now"
   → "I totally understand. Can I ask—what would trigger you to reconsider in the next 30-60 days?"
   
3. "We already have a solution"
   → "Got it. What solution are you using? [Listen] Here's what makes us different..."
   
4. "I need to think about it"
   → "Absolutely. What's the one thing you'd need to see to move forward? [Listen] Let's schedule a quick call with our specialist to address that."

YOUR CONSTRAINTS:
- You CANNOT schedule demos without getting:
  - Name, email, phone, company name
- You CANNOT make promises about pricing or discounts
- You MUST collect contact info if they're interested
- You MUST be honest if you don't know something

TONE GUIDELINES:
- Enthusiastic but not pushy
- Helpful and problem-solving focused
- Respectful of their time
- Personalized (use their name, reference their industry)
```

### Context Window Management

```
Keep in context (always available to agent):
├─ Customer company name & industry
├─ List of services (3-5 key ones)
├─ Pricing tiers (high level)
├─ Top 3 unique advantages
├─ Top 3 common objections + responses
└─ Demo scheduling info

Don't include in context (too much):
├─ Full customer service documentation
├─ Detailed pricing for 50+ options
├─ Complete customer list
├─ Internal metrics
└─ Confidential information

Token budget: ~500 tokens per response
├─ System prompt: ~200 tokens
├─ Context: ~150 tokens
├─ User message: ~100 tokens
├─ Remaining: ~50 tokens for output buffer

Cost per call: ~$0.02-0.03
├─ Average: 10-15 exchanges per call
├─ Duration: 3-5 minutes per call
└─ Total cost: $0.20-0.45 per completed call
```

---

## INITIAL SETUP & CONFIGURATION

### Step 1: Gather Information

Create a setup document with:

```
COMPANY INFORMATION:
├─ Company name: [e.g., "ReadyProspect"]
├─ Industry: [e.g., "AI lead generation"]
├─ Primary offering: [e.g., "ProspectID + ProspectMax"]
├─ Company website: [URL]
└─ Phone number: [Customer's phone for routing]

SERVICES (List 3-5 main ones):
├─ Service 1: [Name]
│  ├─ Description: [2 sentences]
│  ├─ Pricing: $XX/month
│  └─ Who it's for: [Target customer]
├─ Service 2: [Name]
│  └─ ...
└─ Service 3: [Name]
   └─ ...

UNIQUE ADVANTAGES (Top 3):
├─ #1: [e.g., "Fastest response time in industry (2 min avg)"]
├─ #2: [e.g., "Industry-specific AI training"]
└─ #3: [e.g., "24/7 availability at fraction of answering service cost"]

INDUSTRIES SERVED:
├─ Industry 1: [e.g., "Plumbing"]
│  ├─ Typical lead volume: XX/month
│  ├─ Success rate: XX%
│  └─ Key pain points: [List 3]
├─ Industry 2: [e.g., "Dental"]
│  └─ ...
└─ Industry 3: [e.g., "HVAC"]
   └─ ...

COMMON QUESTIONS & OBJECTIONS:
├─ Q1: "How much does this cost?"
│  └─ Answer: [Customized response]
├─ Q2: "How long until we see results?"
│  └─ Answer: [Customized response]
├─ Q3: "Is this a real human or AI?"
│  └─ Answer: "I'm an AI assistant trained on [Company]'s knowledge..."
└─ Q4: [Customer-specific objection]
   └─ Answer: [Customized response]

DEMO SCHEDULING:
├─ Calendly URL (if available): [URL]
├─ Alternative: [Email or phone for manual booking]
└─ Demo duration: [15 min / 30 min]

TONE & PERSONALITY:
├─ Professional level: [Corporate / Casual / Mix]
├─ Humor level: [None / Light / Moderate]
├─ Geographic accent: [Standard / Regional]
└─ Example conversation style: [Describe how your team would answer]
```

### Step 2: Create Knowledge Base Document

**File: `agent-knowledge-base.md`**

```markdown
# ReadyProspect AI Agent Knowledge Base

## About Us
[Company description - 100-150 words]

## What We Do
[Main problem you solve - 50-75 words]

## Our Services

### Service 1: ProspectID
- **What it does:** Identifies who's shopping for your services before they contact you
- **How it works:** [2-3 sentences]
- **Price:** $299/month add-on
- **Best for:** Companies with 20+ leads/month

### Service 2: ProspectMax
- **What it does:** AI answers calls 24/7, responds to forms, follows up automatically
- **How it works:** [2-3 sentences]
- **Price:** $599/month (Starter), $1,299/month (Professional)
- **Best for:** All company sizes

## Why Choose Us?

### Advantage 1: Speed
- Average response time: 2 minutes
- Industry average: 5+ minutes
- Impact: Converts 30% more leads

### Advantage 2: Accuracy
- Industry-specific training
- 95% call capture rate
- Real-time optimization

### Advantage 3: Affordability
- Cheaper than hiring: $2-5k/month vs our $599-1,299
- Better results than answering service
- ROI-positive in 30 days

## Results by Industry

### Plumbing Companies
- Average leads: 20 → 60/month (+200%)
- Revenue impact: +$45k/month
- Timeline: 90 days

### Dental Practices
- Calls answered: 60% → 95%
- Revenue impact: +$36k/month
- Timeline: 60 days

### HVAC Companies
- Response time: 2 hours → 2 minutes
- Revenue impact: +$52.5k/month
- Timeline: 90 days

## Pricing

### Starter - $599/month
- AI call answering (24/7)
- Form submission responses
- Basic reporting

### Professional - $1,299/month
- Everything in Starter
- ProspectID (lead identification)
- Advanced reporting
- Priority support

### Enterprise - Custom
- Everything in Professional
- Custom integrations
- Dedicated support
- White-label options

## Common Questions

**Q: Is this a real human?**
A: No, I'm an AI trained on [Company]'s data. But I'm designed to sound natural and helpful!

**Q: How long until results?**
A: Response time improves immediately. Lead quality improvements show in 30 days. Revenue impact: 60-90 days.

**Q: What about my existing CRM?**
A: We integrate with HubSpot, Salesforce, Pipedrive, and 100+ others. I can look up which one you use.

**Q: Can you handle objections?**
A: Absolutely. I'm trained to understand concerns and address them with real, specific value.

**Q: What if you can't answer something?**
A: I'll be honest and collect your contact info so our team can follow up with the answer.
```

### Step 3: Auto-Generate Initial Script (Using Claude)

**Prompt to send to Claude:**

```
Using this knowledge base [paste above], generate a conversational, 
friendly AI agent script that:

1. Answers customer questions about [Company]'s services
2. Handles common objections with evidence-based responses
3. Asks qualifying questions to identify if customer is a good fit
4. Smoothly transitions to scheduling a demo
5. Collects contact info if they're interested

Format: Conversation examples (caller says X, agent says Y)

Generate 10 conversation examples covering:
- Initial greeting
- Service inquiry
- Price question
- "I'm already using a competitor" objection
- "I need to think about it" objection
- Ready to schedule demo
- Not ready but interested
- Not interested at all

Make responses natural, friendly, and conversational (not robotic).
```

**Claude Output:** Pre-built script ready to use

---

## KNOWLEDGE BASE CREATION

### Option 1: Automated (Recommended)

**Process:**
```
Customer provides website URL
    ↓
Web scraper (Puppeteer) downloads website
    ├─ Extracts: Services, pricing, testimonials, FAQs
    ├─ Generates: Initial knowledge base
    ├─ Cleans: Removes redundant content
    └─ Stores: In BigQuery
    ↓
Claude API: Summarize & Organize
    ├─ Input: Raw website content
    ├─ Output: Structured knowledge base
    ├─ Format: Markdown, ~1000 words
    └─ Cost: ~$0.02
    ↓
Customer Review
    ├─ Reads generated knowledge base
    ├─ Adds missing information
    ├─ Corrects any errors
    └─ Approves for deployment
    ↓
Agent Training
    ├─ Load knowledge base into vector DB
    ├─ Create embeddings (for semantic search)
    └─ Ready to use in agent context
```

**Implementation:**

```python
# Pseudocode - actual implementation in Node.js

async function generateKnowledgeBase(websiteUrl) {
    // Step 1: Scrape website
    const content = await scrapeWebsite(websiteUrl);
    
    // Step 2: Send to Claude for summarization
    const knowledgeBase = await claude.messages.create({
        model: "claude-opus",
        max_tokens: 2000,
        messages: [
            {
                role: "user",
                content: `Using this website content, create a concise knowledge base for an AI sales assistant:\n\n${content}\n\nFormat as markdown with sections: About Us, Services, Pricing, Common Questions.`
            }
        ]
    });
    
    // Step 3: Store in database
    await storeKnowledgeBase(customerId, knowledgeBase.content[0].text);
    
    // Step 4: Create embeddings
    const embeddings = await createEmbeddings(knowledgeBase.content[0].text);
    
    return {
        knowledgeBase: knowledgeBase.content[0].text,
        embeddings: embeddings,
        status: "ready_for_review"
    };
}
```

### Option 2: Manual (Customizable)

**Create markdown file directly:**

1. Copy the template above
2. Fill in company-specific information
3. Add industry-specific objections
4. Include specific case studies
5. Customize tone to match company

---

## DEPLOYMENT TO TWILIO

### Step 1: Provision Phone Number

```javascript
// Using Twilio SDK (Node.js)

const twilio = require('twilio');

const client = twilio(ACCOUNT_SID, AUTH_TOKEN);

async function provisionPhoneNumber(customerId) {
    // Search for available numbers in customer's area code
    const numbers = await client.availablePhoneNumbers
        .local
        .list({
            areaCode: '608', // Wisconsin (example)
            limit: 5
        });
    
    // Buy the first available number
    const phoneNumber = await client
        .incomingPhoneNumbers
        .create({
            phoneNumber: numbers[0].phoneNumber,
            friendlyName: `${customerId} Business Line`,
            smsUrl: 'https://api.readyprospect.ai/sms',
            voiceUrl: 'https://api.readyprospect.ai/voice',
            voiceFallbackUrl: 'https://api.readyprospect.ai/fallback',
            voiceMethod: 'POST'
        });
    
    // Store in database
    await storePhoneNumber(customerId, phoneNumber.sid, phoneNumber.phoneNumber);
    
    return phoneNumber;
}
```

### Step 2: Create Twilio Webhook Handler

```javascript
// Express.js endpoint that Twilio calls

const express = require('express');
const app = express();

app.post('/voice', async (req, res) => {
    const twiml = new twilio.twiml.VoiceResponse();
    const customerId = req.body.Caller; // Phone number
    
    // Look up customer from phone number
    const customer = await getCustomerByPhoneNumber(customerId);
    
    if (!customer) {
        twiml.say("Sorry, this number is not registered.");
        res.type('text/xml');
        res.send(twiml.toString());
        return;
    }
    
    // Start recording
    twiml.record({
        playBeep: true,
        timeout: 30,
        action: `/record-callback?customer_id=${customer.id}`,
        method: 'POST'
    });
    
    res.type('text/xml');
    res.send(twiml.toString());
});

app.post('/record-callback', async (req, res) => {
    const recordingUrl = req.body.RecordingUrl;
    const customerId = req.query.customer_id;
    
    // Transcribe the recording
    const transcript = await transcribeAudio(recordingUrl);
    
    // Determine caller intent
    const intent = await classifyIntent(transcript);
    
    // Get appropriate response from Claude API
    const response = await getAgentResponse(
        customerId,
        intent,
        transcript
    );
    
    // Convert to speech and play back
    const twiml = new twilio.twiml.VoiceResponse();
    twiml.say(response);
    
    res.type('text/xml');
    res.send(twiml.toString());
});
```

### Step 3: AI Agent Response Handler

```javascript
async function getAgentResponse(customerId, intent, transcript) {
    // Get customer knowledge base
    const knowledgeBase = await getKnowledgeBase(customerId);
    
    // Get A/B test variant
    const variant = await getABTestVariant(customerId);
    
    // Get agent script (select based on intent)
    const script = getScriptForIntent(intent, variant);
    
    // Call Claude API for dynamic response
    const response = await claude.messages.create({
        model: "claude-opus-4-20250514",
        max_tokens: 200,
        system: `You are a helpful sales assistant for ${knowledgeBase.companyName}.
        
${knowledgeBase.systemPrompt}

Answer this customer's question naturally and conversationally.`,
        messages: [
            {
                role: "user",
                content: `Customer said: "${transcript}"\n\nHow should you respond?`
            }
        ]
    });
    
    // Extract response text
    const agentResponse = response.content[0].text;
    
    // Log for learning
    await logCall({
        customerId,
        transcript,
        intent,
        agentResponse,
        variant,
        timestamp: new Date()
    });
    
    return agentResponse;
}
```

### Step 4: Test the Agent

```bash
# Make a test call to your provisioned number
# Verify:
1. Phone rings
2. Agent answers with correct greeting
3. Agent responds to questions
4. Responses are natural and accurate
5. Call is recorded
6. Data logged to database
```

---

## SCRIPT MANAGEMENT

### Version Control

```
Agent Script Versions:
├─ v1.0: Initial script (baseline)
├─ v1.1: Improved objection handling for "price"
├─ v1.2: Improved objection handling for "need to think"
├─ v2.0: Complete rewrite based on 500 call analysis
├─ v2.1: Industry-specific scripts (plumbing vs dental)
└─ v3.0: Real-time dynamic response (using Claude)

Each version tracked:
├─ Creation date
├─ Key changes
├─ Performance metrics (close rate, call duration)
├─ Rollout percentage
└─ Author notes
```

### A/B Testing Scripts

```
Test Setup:
├─ Control: Current best-performing script (90% of traffic)
├─ Variant A: New objection handler (10% of traffic)
├─ Variant B: New greeting style (separate test, 10% of traffic)
└─ Duration: Until 500 calls per variant (2-3 weeks typical)

Measurement:
├─ Primary metric: Demo scheduled rate
├─ Secondary: Call duration
├─ Tertiary: Customer satisfaction (post-call survey)
└─ Significance: 95% confidence required before rollout

Example Results:
Control: 28% close rate
Variant A: 31% close rate (+10.7% improvement)
Variant B: 27% close rate (-3.6% worse)
Winner: Variant A → Deploy to 100%
```

### Script Versioning System

```python
class AgentScriptManager:
    def __init__(self):
        self.scripts = {}
        self.variants = {}
        self.performance = {}
    
    def create_version(self, name, content, baseVersion=None):
        """Create a new script version"""
        newVersion = {
            'version': self.getNextVersion(),
            'name': name,
            'content': content,
            'baseVersion': baseVersion,
            'createdAt': datetime.now(),
            'status': 'draft'
        }
        self.scripts[newVersion['version']] = newVersion
        return newVersion['version']
    
    def deploy(self, version, percentage=10):
        """Deploy script to percentage of traffic"""
        script = self.scripts[version]
        script['status'] = 'active'
        script['percentage'] = percentage
        
        # Start A/B test
        self.startABTest(version, percentage)
    
    def rollout(self, version, percentage):
        """Gradually roll out successful variant"""
        script = self.scripts[version]
        script['percentage'] = percentage
    
    def analyze(self, version, minSamples=500):
        """Analyze performance of variant"""
        calls = self.getCallsForVersion(version)
        
        if len(calls) < minSamples:
            return {'status': 'insufficient_data', 'calls': len(calls)}
        
        metrics = {
            'closeRate': self.calculateCloseRate(calls),
            'avgDuration': self.calculateAvgDuration(calls),
            'satisfaction': self.calculateSatisfaction(calls),
            'confidence': self.calculateConfidence(calls),
            'significance': self.calculateSignificance(calls)
        }
        
        return metrics
```

---

## A/B TESTING FRAMEWORK

### Test Planning

```
Monthly Testing Calendar:
├─ Week 1: Plan tests (identify highest-impact experiments)
├─ Week 2-3: Run experiments (10% traffic to variants)
├─ Week 4: Analyze results, deploy winners, plan next month
└─ Pattern: 2-3 tests running simultaneously (different aspects)

High-Impact Experiments (ordered by expected ROI):
├─ #1: Price objection responses (highest frequency objection)
├─ #2: Greeting tone (affects all calls)
├─ #3: Call-to-action wording (affects conversion)
├─ #4: Follow-up sequence (affects nurture)
└─ #5: Qualifying questions (affects lead quality)
```

### Statistical Significance

```python
def calculate_significance(control_conversions, control_total,
                         variant_conversions, variant_total):
    """Calculate statistical significance using Z-test"""
    
    # Calculate proportions
    p1 = control_conversions / control_total
    p2 = variant_conversions / variant_total
    
    # Calculate pooled proportion
    p_pool = (control_conversions + variant_conversions) / (control_total + variant_total)
    
    # Calculate standard error
    se = sqrt(p_pool * (1 - p_pool) * (1/control_total + 1/variant_total))
    
    # Calculate Z-score
    z = (p1 - p2) / se
    
    # Get p-value from Z-score
    p_value = 2 * (1 - norm.cdf(abs(z)))
    
    # Is it significant? (p < 0.05 = 95% confidence)
    is_significant = p_value < 0.05
    
    return {
        'pValue': p_value,
        'isSignificant': is_significant,
        'confidenceLevel': 1 - p_value,
        'recommendation': 'Deploy' if is_significant else 'Continue Testing'
    }
```

### Test Example: Price Objection

```
HYPOTHESIS: 
Customers object to price because we don't show value first.
Adding "$47k average recovery" before price will increase close rate.

CONTROL SCRIPT:
Customer: "How much does this cost?"
Agent: "Our pricing starts at $599/month."

VARIANT A SCRIPT:
Customer: "How much does this cost?"
Agent: "Great question. Most of our customers see about $47,000 in recovered revenue in the first month. Our cost is $599/month. So basically, we pay for ourselves 78 times over in the first month. Does that make sense?"

VARIANT B SCRIPT (Alternative):
Customer: "How much does this cost?"
Agent: "Before I answer that, let me ask—how much revenue would you say you're currently losing to missed calls and slow follow-ups?"
[Listen to answer]
"That's pretty common. Our customers typically recover all of that. And our cost is just $599/month. So the ROI is huge. Does that sound interesting?"

TEST SETUP:
├─ Duration: 2 weeks (until 500 calls per variant)
├─ Traffic split: 90% Control, 5% Variant A, 5% Variant B
├─ Metric: Close rate (demo scheduled)
├─ Acceptable improvement: +5% (28% → 33%)
└─ Rollout decision: Implement highest performer

EXPECTED OUTCOME:
├─ Control: 28% close rate
├─ Variant A: 34% close rate (+21% improvement)
├─ Variant B: 31% close rate (+11% improvement)
├─ Winner: Variant A → Deploy to 100%
├─ Impact: +6 demos per 100 calls
└─ Revenue impact: +$15k/month for typical customer
```

---

## PERFORMANCE MONITORING

### Key Metrics Dashboard

```
Daily Agent Performance:

CALL METRICS
├─ Total calls: 47
├─ Average duration: 4:23
├─ Calls answered: 46 (97.9%)
├─ Abandoned calls: 1 (2.1%)
└─ Hang-ups per minute: 0.1 (excellent)

CONVERSION METRICS
├─ Demos scheduled: 8 (17%)
├─ Info collected but no demo: 22 (47%)
├─ Interested but asked to follow up: 12 (26%)
├─ Not interested: 5 (11%)
└─ Conversion rate: 17% (target: 20%)

QUALITY METRICS
├─ Customer satisfaction (post-call): 4.2/5
├─ Clarity score: 92%
├─ Naturalness score: 88%
├─ Handled objection well: 91%
└─ Would recommend: 85%

SCRIPT PERFORMANCE
├─ Current script version: v2.3
├─ Performance vs baseline: +12%
├─ A/B test active: Yes (Variant A)
├─ Test performance: +8% (trending positive)
└─ Next rollout decision: 3 days

TREND ANALYSIS
├─ Close rate trend (past 7 days): ↑ +2.1%
├─ Avg duration trend: ↓ -0:18 (getting faster)
├─ Satisfaction trend: ↑ +0.3 points
└─ Overall health: EXCELLENT
```

### Alerting System

```
Automated alerts when:
├─ Close rate drops 5%+ vs average
├─ Call abandonment > 10%
├─ Customer satisfaction < 3.5/5
├─ New objection appears (>20% of calls)
├─ Agent response time > 5 seconds
└─ Any error in logging/transcription

Alert destinations:
├─ Slack: Real-time notifications
├─ Email: Daily summary
├─ Dashboard: Visual indicator
└─ Database: Historical log
```

---

## OPTIMIZATION LOOP

### Daily Optimization

```
EVERY NIGHT @ 11 PM:

1. Analyze All Calls (Past 24 hours)
   └─ Extract features:
      ├─ Objections detected
      ├─ Success rate per objection
      ├─ Sentiment trends
      ├─ Call duration distribution
      └─ New patterns
   
2. Identify Improvement Opportunity
   └─ Which objection had lowest success rate?
      Example: "Need to think about it" (52% success vs 68% average)
   
3. Generate Improved Response
   └─ Claude API analysis:
      Input: Transcripts of failed calls + successful calls
      Output: Improved response that handles objection better
   
4. A/B Test Variant
   └─ Deploy to 10% of traffic
      ├─ Control: Current script (90%)
      ├─ Variant: New response (10%)
      └─ Monitor: For 500 calls
   
5. Deploy Winner
   └─ If variant beats control:
      ├─ Deploy to 100%
      ├─ Archive old version
      ├─ Log improvement
      └─ Repeat process
```

### Weekly Optimization

```
EVERY MONDAY @ 9 AM:

1. Weekly Performance Review
   ├─ Average close rate: XX%
   ├─ Best-performing day: [Day]
   ├─ Best objection handler: [Objection]
   └─ Biggest improvement: +X%

2. Segment Analysis
   ├─ By industry:
   │  ├─ Plumbing: 28% close rate
   │  ├─ Dental: 22% close rate
   │  └─ HVAC: 19% close rate
   │
   ├─ By time of day:
   │  ├─ 8-10 AM: 31% (BEST)
   │  ├─ 10-2 PM: 24%
   │  └─ 2-5 PM: 22%
   │
   └─ By call length:
      ├─ 1-3 min: 12%
      ├─ 3-5 min: 28%
      ├─ 5-10 min: 35%
      └─ 10+ min: 32%
   
3. Optimization Recommendations
   ├─ Create dental-specific script (lower than plumbing)
   ├─ Extend call time (5-10 min segment has best conversion)
   ├─ Schedule callbacks for 8-10 AM window (highest close rate)
   └─ Test new HVAC messaging (weakest vertical)

4. Script Evolutions
   ├─ Generate 3 new variants:
   │  ├─ Variant 1: Longer call approach (5-10 min)
   │  ├─ Variant 2: Dental-specific messaging
   │  └─ Variant 3: HVAC-specific messaging
   │
   ├─ Deploy variants (10% traffic each)
   └─ Measure impact over next week

5. Team Review
   ├─ Share insights with customer
   ├─ Discuss optimization strategy
   ├─ Plan for next week
   └─ Document decisions
```

### Monthly Optimization

```
FIRST MONDAY OF MONTH:

Comprehensive Analysis:

1. Month-over-Month Performance
   ├─ Close rate: 28% (vs 25% last month) +12%
   ├─ Avg call duration: 4:23 (vs 4:50 last month) -9%
   ├─ Customer satisfaction: 4.2/5 (vs 3.9 last month) +8%
   └─ Script versions deployed: 6

2. Script Evolution Timeline
   ├─ Start of month: v2.0 (28% close rate)
   ├─ Week 1: v2.1 (+1% improvement)
   ├─ Week 2: v2.2 (+2% improvement)
   ├─ Week 3: v2.3 (+3% improvement)
   └─ End of month: v2.4 (+4% improvement) = 32% close rate

3. Most Effective Script Elements
   ├─ Greeting: "Hi, is this a good time to talk?" (reduces hang-ups)
   ├─ Price frame: Value-first approach (+8% conversion)
   ├─ Objection: "Need to think" → Use timeline question (+12%)
   ├─ CTA: "I'll have my specialist follow up" (vs "Schedule now")
   └─ Closing: Soft close (suggest vs demand)

4. Industry-Specific Scripts
   ├─ Plumbing: 32% close rate (script: v2.4)
   ├─ Dental: 26% close rate (script: v1.8)
   └─ HVAC: 22% close rate (script: v1.5)
   
   Action: Create custom variants for each industry
   
5. Strategic Recommendations
   ├─ Next focus: "Interested but not ready" segment
   │  └─ Currently: 26% of calls. If convert 10% more: +$X/month
   ├─ Test: Callback scheduling system
   │  └─ Currently: Manual follow-up. If automated: +Y%
   └─ Plan: Multi-language support (if needed)

6. Customer Communication
   ├─ Generate insights report
   ├─ Show performance improvements
   ├─ Share script evolution story
   ├─ Recommend next month's focus
   └─ Discuss ROI impact
```

---

## TROUBLESHOOTING

### Common Issues & Solutions

**Issue 1: Agent sounds robotic**

```
Symptom:
- Customers comment on unnatural responses
- Low satisfaction scores (< 3.5/5)
- Lower conversion rates than expected

Solution:
1. Review transcripts for repetitive patterns
2. Add variability: Multiple response options for same intent
3. Include filler words: "Um...", "So...", "Actually..."
4. Add acknowledging statements: "I totally understand...", "Yeah, that makes sense..."
5. Personalize: Use caller's name, reference their company
6. Test with humans: Have team members call and give feedback

Example improvement:
ROBOTIC: "Our pricing starts at $599/month for the Starter plan."
NATURAL: "So for the Starter plan, we're at $599 a month. For that you get the AI call answering and form responses. Does that fit your budget range?"
```

**Issue 2: Agent doesn't understand complex requests**

```
Symptom:
- Agent gives irrelevant responses
- Customers ask to be transferred to human
- Multiple clarifying questions needed

Solution:
1. Check knowledge base: Is information missing?
2. Improve intent classification: Train better at detecting intent
3. Add fallback: "I didn't quite catch that. Can you rephrase?"
4. Increase token budget: More context window for complex questions
5. Human transfer: If agent confidence < 60%, transfer to human

Example improvement:
POOR: [Customer asks complex multi-part question] → Agent picks one part
GOOD: [Customer asks complex question] → Agent asks clarifying question → Addresses all parts
```

**Issue 3: High abandonment rate (customers hang up)**

```
Symptom:
- Many calls < 30 seconds
- High abandonment in first minute
- Customers not staying through greeting

Solution:
1. Review greeting: Is it too long? Too salesy?
2. Call speed: Can agent respond faster?
3. Caller detection: Can we identify and personalize?
4. Hold music: Add calming background music if wait
5. Transparency: "This is an AI assistant, but I can help. What's your question?"

Example improvement:
OLD GREETING: "Good morning, thank you for calling ReadyProspect. This is our automated system. Please state your inquiry."
[ABANDONMENT: 35%]

NEW GREETING: "Hi there! This is Alex from ReadyProspect. How can I help you today?"
[ABANDONMENT: 8%]
```

**Issue 4: Low demo booking rate (< 15%)**

```
Symptom:
- Agent answers well but doesn't convert
- Customers interested but not booking
- Missing soft close techniques

Solution:
1. Test soft closes: Instead of "Schedule a demo?", try "Can I have [name] call you Thursday?"
2. Remove friction: Can customer book directly from call?
3. Add urgency: "I have one slot available Thursday at 2 PM..."
4. Clarify next steps: "So here's what happens next: You'll get a calendar link..."
5. Address hesitation: "What would make you feel comfortable scheduling?"

Example improvement:
WEAK CTA: "Would you like to schedule a demo?"
STRONG CTA: "Perfect. So here's how this works: I'm going to send you a calendar link in the next 2 minutes. You just pick a time that works—Tuesday or Wednesday work best for our team. Does Tuesday at 2 PM work?"
```

**Issue 5: CRM sync failures**

```
Symptom:
- Leads captured but not appearing in CRM
- Data mismatches between systems
- Duplicate leads created

Solution:
1. Check API credentials: Are they valid?
2. Check rate limits: Are you hitting limits?
3. Check field mapping: Are fields mapped correctly?
4. Log errors: Store failed syncs for replay
5. Manual reconciliation: Weekly check for orphaned leads

Example debug:
```python
# Check what's failing
failed_syncs = db.query("SELECT * FROM crm_sync_log WHERE status='failed' ORDER BY created_at DESC")

for sync in failed_syncs[:10]:  # Check last 10 failures
    print(f"Lead: {sync.lead_name}")
    print(f"Error: {sync.error_message}")
    print(f"Retry: {sync.retry_count}")
    
    # Attempt retry
    if sync.retry_count < 3:
        retry_crm_sync(sync.id)
```
```

---

## NEXT STEPS

1. **Gather customer information** (Step 1 of Initial Setup)
2. **Create knowledge base** (Automated or manual)
3. **Deploy to Twilio** (Provision number + webhook)
4. **Test thoroughly** (Make test calls)
5. **Start A/B testing** (Plan first 3 experiments)
6. **Monitor daily** (Check dashboard)
7. **Optimize weekly** (Review segment performance)
8. **Report monthly** (Share results with customer)

**Success metrics (3-month trajectory):**
- Week 1: Agent working, baseline established
- Week 4: Close rate +5-8% from optimization
- Week 8: Close rate +12-15% from accumulated improvements
- Week 12: Close rate +20-30% from full optimization

---

**Next document:** `4_DATA_PIPELINE.md` (How to set up BigQuery, tracking, and analytics)
