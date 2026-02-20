# Agent Definitions & Orchestration

**Version:** 1.0  
**Last Updated:** 2025-10-04  
**Project:** RoboAgency - Humachine AI Studio

---

## Overview

This document defines the AI agents that power the autonomous agency. Each agent is a specialized component in the pipeline, following **Kasparov's Law**: better process + human oversight > raw compute alone.

**Philosophy:** Agents are tools, not replacements. They handle repetitive, data-heavy tasks while humans provide creativity, judgment, and brand voice.

---

## Table of Contents

1. [Agent Architecture](#agent-architecture)
2. [Agent Catalog](#agent-catalog)
3. [Prompt Engineering](#prompt-engineering)
4. [DSPy Optimization](#dspy-optimization)
5. [Human-in-the-Loop](#human-in-the-loop)
6. [Testing & Validation](#testing--validation)

---

## Agent Architecture

### LangGraph State Machine

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, List

class ProjectState(TypedDict):
    # Input
    project_id: str
    sector: str
    idea: str
    constraints: str | None
    
    # Intermediate outputs
    market_analysis: dict
    prd: str
    wireframe: dict
    prototype_code: str
    demo_script: str
    
    # Final outputs
    prototype_url: str
    demo_video_url: str
    campaign_emails: List[dict]
    
    # Metadata
    metrics: dict
    errors: List[str]

# Build graph
graph = StateGraph(ProjectState)

# Add nodes (agents)
graph.add_node("intake", intake_parser)
graph.add_node("market", market_mapper)
graph.add_node("prd", product_doc_writer)
graph.add_node("wireframe", wireframer)
graph.add_node("prototype", prototyper)
graph.add_node("video", explainer_producer)
graph.add_node("campaign", campaign_composer)
graph.add_node("metrics", indication_okrs)

# Define edges (workflow)
graph.set_entry_point("intake")
graph.add_edge("intake", "market")
graph.add_edge("market", "prd")
graph.add_edge("prd", "wireframe")
graph.add_edge("wireframe", "prototype")
graph.add_edge("prototype", "video")
graph.add_edge("video", "campaign")
graph.add_edge("campaign", "metrics")
graph.add_edge("metrics", END)

# Compile
workflow = graph.compile()
```

---

## Agent Catalog

### 1. IntakeParser

**Purpose:** Validate and enrich user input from the intake form.

**Inputs:**
- `sector` (e.g., "healthtech", "fintech", "edtech")
- `idea` (free-form text description)
- `constraints` (optional: budget, timeline, tech stack)

**Outputs:**
- `validated_sector` (mapped to internal taxonomy)
- `goals` (extracted goals and objectives)
- `target_audience` (inferred user personas)
- `key_features` (list of must-have features)

**Implementation:**

```python
# agents/intake_parser.py
from dspy import ChainOfThought, Example
import dspy

class IntakeParser(dspy.Module):
    def __init__(self):
        super().__init__()
        self.parse = ChainOfThought("sector, idea -> goals, target_audience, key_features")
    
    def forward(self, sector: str, idea: str):
        # Few-shot examples for better prompting
        examples = [
            Example(
                sector="healthtech",
                idea="A telemedicine app for rural areas with limited internet",
                goals="Improve healthcare access, reduce travel time, work offline",
                target_audience="Rural patients, traveling doctors, community health workers",
                key_features="Offline mode, low-bandwidth video, appointment scheduling, EHR integration"
            )
        ]
        
        result = self.parse(sector=sector, idea=idea, examples=examples)
        return result

async def intake_parser(state: ProjectState) -> ProjectState:
    parser = IntakeParser()
    result = parser.forward(state["sector"], state["idea"])
    
    state["goals"] = result.goals
    state["target_audience"] = result.target_audience
    state["key_features"] = result.key_features
    
    return state
```

**Validation Rules:**
- Sector must match predefined list
- Idea must be 50-500 characters
- Extract at least 2 goals and 1 target audience

---

### 2. MarketMapper

**Purpose:** Research competitive landscape and retrieve relevant patterns from knowledge base.

**Inputs:**
- `sector`
- `goals`
- `key_features`

**Outputs:**
- `competitors` (list of similar products)
- `market_trends` (recent patterns in the sector)
- `risks` (common pitfalls to avoid)
- `opportunities` (whitespace in the market)

**Implementation:**

```python
# agents/market_mapper.py
from qdrant_client import QdrantClient
import openai

async def market_mapper(state: ProjectState) -> ProjectState:
    qdrant = QdrantClient(url="http://qdrant:6333")
    
    # Semantic search for similar products
    query_embedding = await openai.embeddings.create(
        model="text-embedding-3-small",
        input=f"{state['sector']}: {state['idea']}"
    )
    
    competitors = qdrant.search(
        collection_name="market_comps",
        query_vector=query_embedding.data[0].embedding,
        limit=5
    )
    
    # Structured market analysis
    analysis_prompt = f"""
    Sector: {state['sector']}
    Idea: {state['idea']}
    Similar products: {[c.payload for c in competitors]}
    
    Provide:
    1. Market positioning (2 sentences)
    2. Key trends (3 bullet points)
    3. Risks (3 bullet points)
    4. Opportunities (3 bullet points)
    """
    
    response = await openai.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": analysis_prompt}]
    )
    
    state["market_analysis"] = parse_analysis(response.choices[0].message.content)
    return state
```

**Data Sources:**
- Qdrant vector DB (preloaded with 1000+ product comparisons)
- Industry reports (cached summaries)
- Pattern library (common UX/features by sector)

---

### 3. ProductDocWriter

**Purpose:** Generate a comprehensive Product Requirements Document (PRD).

**Inputs:**
- `goals`
- `target_audience`
- `key_features`
- `market_analysis`

**Outputs:**
- `prd` (markdown document with user stories, acceptance criteria, tech stack)

**Implementation:**

```python
# agents/product_doc_writer.py
import dspy

class PRDGenerator(dspy.Module):
    def __init__(self):
        super().__init__()
        self.generate = dspy.ChainOfThought(
            "goals, audience, features, market -> prd"
        )
    
    def forward(self, **kwargs):
        prd_template = """
        # Product Requirements Document
        
        ## Overview
        {overview}
        
        ## Target Audience
        {audience}
        
        ## Core Features
        {features}
        
        ## User Stories
        {user_stories}
        
        ## Acceptance Criteria
        {acceptance_criteria}
        
        ## Technical Stack
        {tech_stack}
        
        ## Success Metrics
        {metrics}
        """
        
        result = self.generate(**kwargs)
        
        prd = prd_template.format(
            overview=result.overview,
            audience=result.audience,
            features=result.features,
            user_stories=result.user_stories,
            acceptance_criteria=result.acceptance_criteria,
            tech_stack=result.tech_stack,
            metrics=result.metrics
        )
        
        return prd

async def product_doc_writer(state: ProjectState) -> ProjectState:
    generator = PRDGenerator()
    
    prd = generator.forward(
        goals=state["goals"],
        audience=state["target_audience"],
        features=state["key_features"],
        market=state["market_analysis"]
    )
    
    state["prd"] = prd
    
    # Upload to S3
    prd_url = await upload_to_s3(
        content=prd,
        key=f"{state['project_id']}/docs/prd.md"
    )
    
    state["prd_url"] = prd_url
    return state
```

**Quality Checks:**
- PRD must be 800-2000 words
- Must include at least 5 user stories
- Must specify tech stack
- Must define 3+ success metrics

---

### 4. Wireframer

**Purpose:** Create low-fidelity wireframes (textual layouts + optional Figma integration).

**Inputs:**
- `prd`
- `key_features`

**Outputs:**
- `wireframe` (structured layout descriptions)

**Implementation:**

```python
# agents/wireframer.py
async def wireframer(state: ProjectState) -> ProjectState:
    prompt = f"""
    Based on this PRD, create wireframes for the key screens.
    
    PRD:
    {state['prd']}
    
    For each screen, provide:
    1. Screen name
    2. Layout description (header, nav, main content, footer)
    3. Key components (buttons, forms, lists, etc.)
    4. User flow connections
    
    Format as JSON.
    """
    
    response = await openai.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        response_format={"type": "json_object"}
    )
    
    wireframe = json.loads(response.choices[0].message.content)
    state["wireframe"] = wireframe
    
    # Optional: Generate Figma frames via API
    # figma_url = await create_figma_wireframes(wireframe)
    # state["figma_url"] = figma_url
    
    return state
```

**Output Example:**

```json
{
  "screens": [
    {
      "name": "Home",
      "layout": {
        "header": "Logo, navigation (About, Services, Contact)",
        "hero": "Headline, subheadline, CTA button",
        "features": "3-column grid with icons and descriptions",
        "footer": "Social links, copyright"
      },
      "components": ["Logo", "Nav", "Button", "FeatureCard"]
    },
    {
      "name": "Dashboard",
      "layout": {
        "sidebar": "User menu, navigation",
        "main": "Stats cards, recent activity table",
        "header": "Search bar, notifications"
      },
      "components": ["Sidebar", "StatsCard", "Table", "SearchBar"]
    }
  ]
}
```

---

### 5. Prototyper

**Purpose:** Generate a functional Svelte prototype from wireframes and PRD.

**Inputs:**
- `prd`
- `wireframe`

**Outputs:**
- `prototype_code` (Svelte components)
- `prototype_url` (deployed demo link)

**Implementation:**

```python
# agents/prototyper.py
async def prototyper(state: ProjectState) -> ProjectState:
    # Generate Svelte components for each screen
    components = []
    
    for screen in state["wireframe"]["screens"]:
        component_prompt = f"""
        Create a Svelte component for this screen:
        
        Name: {screen['name']}
        Layout: {screen['layout']}
        Components: {screen['components']}
        
        Requirements:
        - Use Tailwind CSS for styling
        - Make it responsive (mobile-first)
        - Include placeholder content
        - Add basic interactivity (button clicks, form validation)
        
        Output only the .svelte file code.
        """
        
        response = await openai.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": component_prompt}]
        )
        
        component_code = extract_code_block(response.choices[0].message.content)
        components.append({
            "name": screen["name"],
            "code": component_code
        })
    
    # Bundle into single-page app
    prototype_html = bundle_svelte_app(components)
    
    # Upload to S3
    prototype_url = await upload_to_s3(
        content=prototype_html,
        key=f"{state['project_id']}/prototypes/index.html",
        content_type="text/html"
    )
    
    state["prototype_code"] = components
    state["prototype_url"] = prototype_url
    
    return state
```

**Quality Checks:**
- All screens from wireframe must be generated
- HTML must be valid (validate with parser)
- Responsive design (test at 320px, 768px, 1024px viewports)

---

### 6. ExplainerProducer

**Purpose:** Generate a 60-90 second demo video with voiceover.

**Inputs:**
- `prototype_url`
- `prd`

**Outputs:**
- `demo_script` (voiceover text)
- `demo_video_url` (rendered MP4)

**Implementation:**

```python
# agents/explainer_producer.py
async def explainer_producer(state: ProjectState) -> ProjectState:
    # Step 1: Generate script
    script_prompt = f"""
    Create a 60-second demo script for this product:
    
    PRD Summary:
    {state['prd'][:500]}
    
    Structure:
    - Hook (5 seconds): Problem statement
    - Solution (20 seconds): How the product solves it
    - Features (25 seconds): Key features with benefits
    - CTA (10 seconds): Call to action
    
    Keep it conversational and engaging.
    """
    
    script = await openai.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": script_prompt}]
    )
    
    demo_script = script.choices[0].message.content
    state["demo_script"] = demo_script
    
    # Step 2: Generate voiceover (ElevenLabs, etc.)
    audio_url = await generate_voiceover(demo_script)
    
    # Step 3: Record prototype walkthrough
    # Use Playwright to record browser interactions
    video_frames = await record_prototype_walkthrough(state["prototype_url"])
    
    # Step 4: Render final video (ffmpeg)
    video_url = await render_video(
        frames=video_frames,
        audio=audio_url,
        output_key=f"{state['project_id']}/videos/demo.mp4"
    )
    
    state["demo_video_url"] = video_url
    return state
```

**Alternative (Simpler):** Generate slide deck + audio instead of video.

---

### 7. CampaignComposer

**Purpose:** Create a 3-email warmup campaign.

**Inputs:**
- `demo_video_url`
- `prototype_url`

**Outputs:**
- `campaign_emails` (list of email templates)

**Implementation:**

```python
# agents/campaign_composer.py
async def campaign_composer(state: ProjectState) -> ProjectState:
    emails = []
    
    # Email #1: Demo video
    email_1 = {
        "subject": "Your Custom Demo is Ready! 🎉",
        "body": f"""
        Hi there,
        
        Thanks for sharing your idea with us. We've created a custom demo 
        that shows how your vision could come to life.
        
        Watch the demo: {state['demo_video_url']}
        
        Let us know what you think!
        
        Best,
        The Team
        """,
        "send_delay_hours": 0
    }
    
    # Email #2: Prototype link
    email_2 = {
        "subject": "Try Your Interactive Prototype",
        "body": f"""
        Hi again,
        
        Want to see it in action? We've built a clickable prototype 
        that you can interact with.
        
        Try it here: {state['prototype_url']}
        
        Feel free to explore and share with your team.
        
        Cheers,
        The Team
        """,
        "send_delay_hours": 24
    }
    
    # Email #3: Schedule call
    email_3 = {
        "subject": "Let's Build This Together",
        "body": """
        Hi,
        
        If you're ready to move forward, let's chat!
        
        Book a 30-minute call: https://cal.com/youragency
        
        We'll discuss:
        - Timeline and budget
        - Technical approach
        - Next steps
        
        Looking forward to it!
        
        Best,
        The Team
        """,
        "send_delay_hours": 72
    }
    
    emails = [email_1, email_2, email_3]
    state["campaign_emails"] = emails
    
    return state
```

---

### 8. IndicationOKRs

**Purpose:** Log metrics, validate outputs, and close the feedback loop.

**Inputs:**
- All previous state

**Outputs:**
- `metrics` (performance data)

**Implementation:**

```python
# agents/indication_okrs.py
async def indication_okrs(state: ProjectState) -> ProjectState:
    metrics = {
        "project_id": state["project_id"],
        "pipeline_duration_seconds": time.time() - state["start_time"],
        "artifacts_generated": {
            "prd": bool(state.get("prd")),
            "wireframe": bool(state.get("wireframe")),
            "prototype": bool(state.get("prototype_url")),
            "demo_video": bool(state.get("demo_video_url")),
            "campaign": bool(state.get("campaign_emails"))
        },
        "validation_score": calculate_validation_score(state),
        "estimated_cost_usd": calculate_llm_cost(state),
        "errors": state.get("errors", [])
    }
    
    state["metrics"] = metrics
    
    # Log to database
    await db.table("workflow_runs").update({
        "status": "completed",
        "metadata": metrics
    }).eq("id", state["project_id"]).execute()
    
    # Log to monitoring (Sentry/DataDog)
    logger.info("Workflow completed", extra=metrics)
    
    return state

def calculate_validation_score(state: ProjectState) -> float:
    """Score output quality (0.0 - 1.0)"""
    checks = [
        state.get("prd") and len(state["prd"]) > 800,
        state.get("wireframe") and len(state["wireframe"]["screens"]) >= 2,
        state.get("prototype_url"),
        state.get("demo_video_url"),
        len(state.get("campaign_emails", [])) == 3
    ]
    return sum(checks) / len(checks)
```

---

## Prompt Engineering

### Best Practices

1. **Few-shot examples:** Always include 2-3 examples in critical prompts
2. **Structured outputs:** Use JSON mode when possible
3. **Validation:** Parse and validate LLM outputs before proceeding
4. **Fallbacks:** Have backup prompts if primary fails

### Example: Structured Output

```python
response = await openai.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": prompt}],
    response_format={"type": "json_object"},
    temperature=0.3  # Lower for more deterministic outputs
)

data = json.loads(response.choices[0].message.content)
validate_schema(data, expected_schema)
```

---

## DSPy Optimization

### Signature Tuning

```python
import dspy

# Define signature
class GeneratePRD(dspy.Signature):
    """Generate a Product Requirements Document from user input"""
    goals = dspy.InputField()
    audience = dspy.InputField()
    features = dspy.InputField()
    prd = dspy.OutputField(desc="Markdown PRD with user stories")

# Optimize with examples
class PRDModule(dspy.Module):
    def __init__(self):
        super().__init__()
        self.generate = dspy.ChainOfThought(GeneratePRD)
    
    def forward(self, goals, audience, features):
        return self.generate(goals=goals, audience=audience, features=features)

# Compile with training data
optimizer = dspy.BootstrapFewShot(metric=prd_quality_metric)
optimized_prd_module = optimizer.compile(
    PRDModule(),
    trainset=training_examples
)
```

---

## Human-in-the-Loop

### Review Points

1. **After IntakeParser:** Human reviews extracted goals/features
2. **After ProductDocWriter:** Human edits PRD before wireframing
3. **After Prototyper:** Human reviews prototype before demo generation

### Implementation

```python
# Add conditional edges for human review
def should_review(state: ProjectState) -> bool:
    return state.get("requires_review", False)

graph.add_conditional_edges(
    "prd",
    should_review,
    {
        True: "human_review",
        False: "wireframe"
    }
)
```

---

## Testing & Validation

### Unit Tests

```python
# tests/test_agents.py
import pytest
from agents.intake_parser import intake_parser

@pytest.mark.asyncio
async def test_intake_parser():
    state = {
        "sector": "healthtech",
        "idea": "Telemedicine for rural areas",
        "project_id": "test123"
    }
    
    result = await intake_parser(state)
    
    assert "goals" in result
    assert "target_audience" in result
    assert len(result["key_features"]) >= 2
```

### Integration Tests

```python
@pytest.mark.asyncio
async def test_full_pipeline():
    state = {
        "sector": "edtech",
        "idea": "AI tutor for high school math",
        "project_id": "integration_test"
    }
    
    result = await workflow.invoke(state)
    
    assert result["prototype_url"]
    assert result["demo_video_url"]
    assert len(result["campaign_emails"]) == 3
    assert result["metrics"]["validation_score"] > 0.8
```

---

## Next Steps

1. **Implement** each agent in isolation
2. **Test** with real user data
3. **Optimize** prompts using DSPy
4. **Deploy** to staging
5. **Monitor** and iterate

---

**Document Version:** 1.0  
**Last Review Date:** 2025-10-04  
**Next Review Date:** 2025-11-04

