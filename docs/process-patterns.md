# LogicPrimitives Process Patterns

This document describes common patterns of primitive operations that work together to accomplish specific research and analytical goals. Rather than using primitives in isolation, these patterns chain them together in specific sequences optimized for different use cases.

## Core Process Patterns

| # | Process nickname | Primitive chain | What it accomplishes | Quick use-case |
|---|------------------|-----------------|----------------------|----------------|
| 1 | **Signal → Concept** | Observe → Distinguish → Define | Turns raw noise into named parts so the team is talking about the same thing | Scoping a brand-new domain |
| 2 | **Trend Lens** | Observe → Sequence → Compare → Infer | Lines up events chronologically, spots changes, extracts "what's really moving" | Detect demand shifts in weekly metrics |
| 3 | **Causal Map** | Observe → Sequence → Infer → Synthesize | Builds a cause-and-effect chain, then fuses it into a coherent narrative | Explaining why a rollout spiked churn |
| 4 | **Decision Sprint** | Observe → Compare → Infer → Decide | Rapid pattern check → conclusion → action | Fire-fighting an outage |
| 5 | **Hypothesis Loop** | Observe → Ask → Infer → Reflect → Adapt | Tests an idea, questions gaps, adjusts model in real time | A/B testing a new pricing page |
| 6 | **Strategic Weave** | Distinguish → Compare → Synthesize → Decide | Separates similar signals, contrasts them, braids insights into a strategy pick | Choosing market entry among close geos |
| 7 | **Feedback Flywheel** | Decide → Ask → Reflect → Adapt | After acting, solicit feedback, critique results, tweak course—keeps plans alive | Continuous product-market fit tuning |
| 8 | **Anomaly Hunter** | Observe → Compare → Distinguish → Ask | Spots outliers, checks if they matter, digs with pointed questions | Detecting fraud spikes in transaction logs |
| 9 | **Scenario Forge** | Define → Sequence → Infer → Synthesize → Adapt | Frames variables, orders them, projects futures, stress-tests and revises | Building best-/worst-case supply-chain plans |
| 10 | **Meta-Mirror** | Observe → Reflect → Ask → Synthesize | Looks at the *reasoning itself*, challenges blind spots, consolidates a sharper lens | Post-mortem on a failed forecast |

## Detailed Pattern Explanations

### 1. Signal → Concept

**Chain:** `Observe → Distinguish → Define`

This pattern transforms raw, unstructured information into clear conceptual frameworks and definitions. It's the foundation for establishing shared understanding in new or complex domains.

**Process:**
1. **Observe**: Collect raw data without interpretation
2. **Distinguish**: Identify meaningful differences and patterns in the data
3. **Define**: Establish clear boundaries, categories and terminology

**Example Application:**
```markdown
# Signal → Concept: Emerging Privacy Regulation Framework

## Observe
- Collected 37 privacy regulations from different jurisdictions
- Gathered implementation timelines and scope definitions
- Documented enforcement mechanisms and penalties

## Distinguish
- Separated regulations by geographic scope (national, regional, sectoral)
- Identified differences in data subject rights
- Categorized by enforcement approach (fines, audits, certification)

## Define
- Created "Privacy Regulation Framework" with 5 key dimensions
- Established clear terminology for compliance requirements
- Defined boundary conditions for applicability to different organizations
```

### 2. Trend Lens

**Chain:** `Observe → Sequence → Compare → Infer`

This pattern detects meaningful patterns in time-series data, establishing not just what happened, but the trajectory and significance of changes.

**Process:**
1. **Observe**: Collect time-stamped data points
2. **Sequence**: Arrange events chronologically
3. **Compare**: Identify differences between time periods
4. **Infer**: Draw conclusions about underlying trends

**Example Application:**
```markdown
# Trend Lens: Mobile App Engagement Metrics

## Observe
- Collected daily active users, session length, and conversion rates
- Gathered feature usage statistics across 6 months
- Documented app store ratings and reviews

## Sequence
- Arranged metrics in weekly and monthly views
- Aligned feature releases with engagement metrics
- Created timeline of marketing campaigns and engagement spikes

## Compare
- Identified 27% drop in session length after UI redesign
- Found conversion rate improvements correlating with onboarding changes
- Noted demographic shifts in user base over time

## Infer
- Core users responding negatively to simplified navigation
- New user acquisition improving but retention declining
- Competitor releases coinciding with engagement drops
```

### 3. Causal Map

**Chain:** `Observe → Sequence → Infer → Synthesize`

This pattern builds explanatory models by establishing cause-and-effect relationships and integrating them into coherent narratives.

**Process:**
1. **Observe**: Collect relevant data points
2. **Sequence**: Establish chronological order of events
3. **Infer**: Identify potential causal connections
4. **Synthesize**: Integrate causal links into a comprehensive explanation

**Example Application:**
```markdown
# Causal Map: Customer Churn Spike Analysis

## Observe
- Documented 43% increase in churn rate over 2 months
- Collected exit survey responses from churned customers
- Gathered product usage data before cancellation

## Sequence
- Created timeline of product changes and churn rate
- Mapped customer journey events leading to cancellation
- Established sequence of competitor actions in the market

## Infer
- Pricing change triggered reassessment among 60% of churned customers
- UX friction points contributed to abandonment during key workflows
- Competitor promotions provided clear exit opportunity

## Synthesize
- Comprehensive churn narrative connecting pricing, experience, and market factors
- Model explaining how different customer segments responded differently
- Integrated explanation of how multiple factors compounded churn effect
```

### 4. Decision Sprint

**Chain:** `Observe → Compare → Infer → Decide`

This pattern enables rapid, evidence-based decision making in time-sensitive situations, focusing on quick pattern recognition and decisive action.

**Process:**
1. **Observe**: Gather immediately available data
2. **Compare**: Evaluate options against key criteria
3. **Infer**: Draw immediate conclusions
4. **Decide**: Make a clear choice with explicit rationale

**Example Application:**
```markdown
# Decision Sprint: Service Outage Response

## Observe
- Error rates spiked to 37% on authentication service
- Database connection pool showing 98% utilization
- Recent deployment introduced new authentication flow

## Compare
- Option 1: Rollback deployment (quick, conservative, disrupts users again)
- Option 2: Scale database resources (addresses symptoms, not root cause)
- Option 3: Fix connection leak in code (addresses root cause, requires dev time)

## Infer
- Authentication code change likely introduced connection pool leak
- Problem will worsen as more users authenticate
- Temporary mitigation needed while proper fix developed

## Decide
- Implement database resource scaling immediately
- Begin deployment rollback process as backup
- Assign development team to diagnose and fix connection management
```

### 5. Hypothesis Loop

**Chain:** `Observe → Ask → Infer → Reflect → Adapt`

This pattern implements a scientific method approach to testing ideas and refining models based on empirical feedback.

**Process:**
1. **Observe**: Collect baseline data
2. **Ask**: Formulate testable questions
3. **Infer**: Make predictions based on hypothesis
4. **Reflect**: Compare results to predictions
5. **Adapt**: Refine hypothesis based on findings

**Example Application:**
```markdown
# Hypothesis Loop: Pricing Page Optimization

## Observe
- Current conversion rate: 2.3% on pricing page
- Average time on page: 1:45
- 68% of visitors view features comparison

## Ask
- Will emphasizing annual discount increase annual plan selection?
- Does removing feature comparison increase or decrease conversion?
- Does social proof placement affect conversion rate?

## Infer
- Predict 15% increase in annual plan selection
- Predict increased conversion but lower average contract value without comparison
- Predict 5-10% lift from testimonial placement above pricing

## Reflect
- Annual emphasis increased annual selection by 23% (better than expected)
- Removing comparison decreased overall conversion by 17% (contrary to hypothesis)
- Testimonial placement showed no significant impact (null result)

## Adapt
- Refine hypothesis: Feature comparison is decision-critical content
- Adjust model to emphasize transparent feature comparison with annual discount
- Design new experiment for social proof that targets specific objections
```

### 6. Strategic Weave

**Chain:** `Distinguish → Compare → Synthesize → Decide`

This pattern separates complex options, evaluates them systematically, integrates multiple considerations, and leads to strategic choices.

**Process:**
1. **Distinguish**: Separate options and evaluation criteria
2. **Compare**: Evaluate options across key dimensions
3. **Synthesize**: Integrate findings into coherent strategic options
4. **Decide**: Select optimal strategy with clear rationale

**Example Application:**
```markdown
# Strategic Weave: Market Expansion Analysis

## Distinguish
- Separated market attributes: size, growth rate, competition, regulatory complexity
- Identified 5 potential target markets with distinct profiles
- Established evaluation criteria: short-term revenue, long-term potential, entry cost

## Compare
- Market A: Highest immediate revenue, strong competition, medium regulatory complexity
- Market B: Moderate growth, underserved demand, high entry costs
- Market C: Highest growth potential, low current revenue, minimal regulation

## Synthesize
- Created integrated market opportunity profiles
- Developed resource requirement projections for each entry strategy
- Mapped competitive positioning options in each market

## Decide
- Selected Market B for expansion
- Rationale: Best alignment with core capabilities, sustainable advantage opportunity
- Implementation: Phased entry strategy with localization partnership
```

### 7. Feedback Flywheel

**Chain:** `Decide → Ask → Reflect → Adapt`

This pattern creates a continuous improvement cycle that maintains alignment with changing conditions and requirements.

**Process:**
1. **Decide**: Make an initial decision or implementation
2. **Ask**: Solicit specific feedback on outcomes
3. **Reflect**: Critically evaluate results and implications
4. **Adapt**: Make systematic adjustments based on feedback

**Example Application:**
```markdown
# Feedback Flywheel: Product Launch Refinement

## Decide
- Initial product launch with core feature set
- Targeting early adopter segment
- Premium pricing strategy

## Ask
- Conducted user interviews with initial customers
- Gathered feature request patterns
- Measured usage patterns across different segments

## Reflect
- Core value proposition resonating with target audience
- Unexpected use case emerging in adjacent market
- Pricing perceived as barrier by high-potential segment

## Adapt
- Refined messaging to highlight discovered use case
- Created segment-specific pricing tier for high-potential users
- Adjusted roadmap to prioritize features supporting emerging use patterns
```

### 8. Anomaly Hunter

**Chain:** `Observe → Compare → Distinguish → Ask`

This pattern identifies outliers and investigates their significance, turning unusual data points into valuable insights.

**Process:**
1. **Observe**: Collect comprehensive baseline data
2. **Compare**: Identify deviations from expected patterns
3. **Distinguish**: Separate meaningful anomalies from noise
4. **Ask**: Formulate specific questions to investigate findings

**Example Application:**
```markdown
# Anomaly Hunter: Payment Processing Analysis

## Observe
- Collected 3 months of payment processing data
- Gathered transaction volumes, success rates, and processing times
- Documented error codes and frequency

## Compare
- Identified 3.2% higher decline rate on Tuesdays
- Found processing time spikes between 2-3am UTC
- Detected unusual error code frequency in specific geographic region

## Distinguish
- Tuesday decline pattern statistically significant (p<0.01)
- Processing spikes correlated with maintenance window
- Regional errors showing consistent pattern, not random distribution

## Ask
- Is the payment processor running batch operations on Tuesdays?
- Are maintenance windows properly configured for minimal impact?
- Is there a region-specific integration issue affecting these transactions?
```

### 9. Scenario Forge

**Chain:** `Define → Sequence → Infer → Synthesize → Adapt`

This pattern creates structured future projections that account for uncertainty and support contingency planning.

**Process:**
1. **Define**: Establish key variables and boundaries
2. **Sequence**: Order potential events and dependencies
3. **Infer**: Project likely outcomes in different scenarios
4. **Synthesize**: Create coherent scenario narratives
5. **Adapt**: Refine scenarios based on new information

**Example Application:**
```markdown
# Scenario Forge: Supply Chain Resilience Planning

## Define
- Key variables: shipping costs, material availability, lead times
- Critical dependencies: key supplier relationships, transportation routes
- Boundary conditions: regulatory changes, extreme weather events

## Sequence
- Mapped cascading impacts of port disruptions
- Ordered supplier failure risks by criticality
- Created timeline of potential regulatory changes

## Infer
- Best case: 5-7% cost increase with minor delays
- Expected case: 12-18% cost increase with 2-week lead time extension
- Worst case: 30%+ cost increase with 4+ week delays and material shortages

## Synthesize
- Developed three integrated scenarios with specific triggers and impacts
- Created narrative describing interconnected effects across supply network
- Established clear indicators for scenario identification

## Adapt
- Updated worst-case scenario based on new sustainability regulations
- Refined expected case with latest supplier capacity information
- Added emerging scenario for technology-driven logistics transformation
```

### 10. Meta-Mirror

**Chain:** `Observe → Reflect → Ask → Synthesize`

This pattern examines the research and decision process itself, improving analytical quality through metacognitive awareness.

**Process:**
1. **Observe**: Gather information about the reasoning process
2. **Reflect**: Critically examine assumptions and methods
3. **Ask**: Formulate questions about blind spots and biases
4. **Synthesize**: Create improved analytical frameworks

**Example Application:**
```markdown
# Meta-Mirror: Sales Forecast Accuracy Analysis

## Observe
- Collected past 8 quarters of sales forecasts and actuals
- Documented forecasting methodology and inputs
- Gathered decision records based on previous forecasts

## Reflect
- Identified systematic 15-20% optimism bias in forecasts
- Found recency bias affecting seasonal adjustment calculations
- Noted stakeholder influence patterns in forecast revisions

## Ask
- How can we incorporate historical accuracy into confidence intervals?
- What independent measures could serve as forecast validation?
- How might we separate analytic and political aspects of forecasting?

## Synthesize
- Created new forecast governance model with explicit bias corrections
- Developed confidence interval framework based on historical accuracy
- Established independent validation metrics for forecast quality
```

## Implementing Process Patterns

These patterns can be implemented in research workflows by:

1. **Identifying pattern match**: Determine which pattern best fits your research need
2. **Following the chain**: Execute each primitive in sequence with appropriate documentation
3. **Storing artifacts**: Preserve the outputs of each step for traceability
4. **Iterative refinement**: Use Meta-Mirror pattern periodically to improve process quality

For optimal results, maintain explicit documentation of which pattern you're using and why it's appropriate for the specific research context.