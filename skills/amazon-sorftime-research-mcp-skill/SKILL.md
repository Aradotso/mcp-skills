---
name: amazon-sorftime-research-mcp-skill
description: Amazon product research, competitor analysis, category selection, keyword research, and review analysis using Sorftime MCP for cross-border e-commerce
triggers:
  - analyze Amazon listing
  - research Amazon product category
  - find Amazon keywords
  - analyze product reviews
  - competitor research Amazon
  - Amazon category selection
  - product research workflow
  - Amazon market research
---

# Amazon Sorftime Research MCP Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This skill enables AI agents to perform comprehensive Amazon product research, competitor analysis, category selection, keyword research, and review analysis using the Sorftime MCP service. It provides seven core skills for cross-border e-commerce decision-making: listing analysis, category selection, keyword research, review analysis, product research, Sif research, and Xiyou insight.

## Overview

The project integrates three MCP services (Sorftime, Sif, Xiyou) and provides Python-based analysis tools for:

- **Single Listing Analysis**: Deep-dive into competitor products with keyword, review, and trend analysis
- **Category Selection**: Market analysis of Top 100 products with five-dimension scoring
- **Keyword Research**: 8-dimension intelligent classification of 1500+ keywords with ad strategy guidance
- **Review Analysis**: 6-dimension pain point analysis with service risk warnings
- **Product Research**: LLM-driven deep research workflow from data collection to decision
- **Sif Research**: Market validation, competitor growth paths, keyword layout, and traffic diagnostics
- **Xiyou Insight**: 7 scenario workflows for ad monitoring, traffic gap, competitor analysis, new product launch, ad budget transparency, and keyword database building

## Installation

### Prerequisites

- Claude Code CLI or similar AI coding agent
- Python 3.8+
- Bash shell environment
- Sorftime API Key from [sorftime.com/zh-cn/mcp](https://sorftime.com/zh-cn/mcp)

### Setup MCP Configuration

Create or update `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "sorftime": {
      "type": "streamableHttp",
      "url": "https://mcp.sorftime.com?key=${SORFTIME_API_KEY}",
      "name": "Sorftime MCP"
    },
    "sif": {
      "type": "streamableHttp",
      "url": "https://mcp.sif.com?key=${SIF_API_KEY}",
      "name": "Sif MCP"
    },
    "xiyou": {
      "type": "streamableHttp",
      "url": "https://mcp.xiyou.com?key=${XIYOU_API_KEY}",
      "name": "Xiyou MCP"
    }
  }
}
```

Set environment variables:

```bash
export SORFTIME_API_KEY="your_sorftime_api_key"
export SIF_API_KEY="your_sif_api_key"
export XIYOU_API_KEY="your_xiyou_api_key"
```

### Install Python Dependencies

```bash
pip install -r requirements.txt
```

## Core Skills and Commands

### 1. Listing Analysis (`amazon-analyse`)

Analyze a single ASIN with comprehensive data including product details, reviews, keywords, and trends.

```bash
# Analyze US marketplace listing
/amazon-analyse B07PWTJ4H1 US

# Analyze German marketplace listing
/amazon-analyse B08N5WRWNW DE
```

**Python Implementation Pattern**:

```python
import json
import requests
from datetime import datetime

def analyze_listing(asin: str, site: str, mcp_url: str):
    """
    Comprehensive listing analysis using Sorftime MCP
    """
    # 1. Get product details
    product_detail = requests.post(
        mcp_url,
        json={
            "method": "product_detail",
            "params": {"asin": asin, "site": site}
        }
    ).json()
    
    # 2. Get traffic keywords (top 100)
    traffic_keywords = requests.post(
        mcp_url,
        json={
            "method": "product_traffic_terms",
            "params": {"asin": asin, "site": site, "limit": 100}
        }
    ).json()
    
    # 3. Get product reviews (last 100)
    reviews = requests.post(
        mcp_url,
        json={
            "method": "product_reviews",
            "params": {"asin": asin, "site": site, "limit": 100}
        }
    ).json()
    
    # 4. Get product trends (90 days)
    trends = requests.post(
        mcp_url,
        json={
            "method": "product_trend",
            "params": {
                "asin": asin,
                "site": site,
                "days": 90
            }
        }
    ).json()
    
    # 5. Get competitor keyword layout
    competitor_keywords = requests.post(
        mcp_url,
        json={
            "method": "competitor_product_keywords",
            "params": {"asin": asin, "site": site}
        }
    ).json()
    
    # Generate report
    report_data = {
        "asin": asin,
        "site": site,
        "product": product_detail,
        "keywords": traffic_keywords,
        "reviews": reviews,
        "trends": trends,
        "competitor_keywords": competitor_keywords,
        "analysis_date": datetime.now().isoformat()
    }
    
    # Save to reports directory
    report_path = f"reports/analysis_{asin}_{site}_{datetime.now().strftime('%Y%m%d')}.md"
    with open(report_path, 'w') as f:
        f.write(generate_markdown_report(report_data))
    
    return report_data
```

**Output**: `reports/analysis_{ASIN}_{SITE}_{DATE}.md`

### 2. Category Selection (`category-selection`)

Analyze category market with Top 100 products and five-dimension scoring model.

```bash
# Analyze US Sofas & Couches category
/category-selection "Sofas & Couches" US

# Analyze specific number of products
/category-selection "Wireless Earbuds" US --limit 20
```

**Python Implementation Pattern**:

```python
def analyze_category(category_name: str, site: str, limit: int = 100, mcp_url: str):
    """
    Category selection analysis with five-dimension scoring
    """
    # 1. Search for category nodeId
    category_search = requests.post(
        mcp_url,
        json={
            "method": "category_name_search",
            "params": {"query": category_name, "site": site}
        }
    ).json()
    
    node_id = category_search[0]['nodeId']
    
    # 2. Get category report (Top products)
    category_report = requests.post(
        mcp_url,
        json={
            "method": "category_report",
            "params": {
                "nodeId": node_id,
                "site": site,
                "limit": limit
            }
        }
    ).json()
    
    # 3. Get category trends
    category_trends = requests.post(
        mcp_url,
        json={
            "method": "category_trend",
            "params": {"nodeId": node_id, "site": site}
        }
    ).json()
    
    # 4. Calculate five-dimension scores
    scores = calculate_five_dimension_scores(category_report, category_trends)
    
    # 5. Generate multi-format reports
    report_dir = f"category-reports/{category_name}_{site}_{datetime.now().strftime('%Y%m%d')}"
    os.makedirs(report_dir, exist_ok=True)
    
    # Save Markdown report
    with open(f"{report_dir}/report.md", 'w') as f:
        f.write(generate_category_markdown(category_report, scores))
    
    # Save Excel with charts
    generate_excel_report(f"{report_dir}/category_report.xlsx", category_report, scores)
    
    # Save HTML dashboard
    generate_html_dashboard(f"{report_dir}/dashboard.html", category_report, scores)
    
    return scores

def calculate_five_dimension_scores(report_data, trend_data):
    """
    Five-dimension scoring model (100 points total)
    """
    return {
        "market_scale": calculate_market_scale_score(report_data),  # 30 points
        "growth_potential": calculate_growth_score(trend_data),      # 20 points
        "competition_level": calculate_competition_score(report_data), # 20 points
        "entry_barrier": calculate_barrier_score(report_data),       # 15 points
        "profit_margin": calculate_profit_score(report_data)         # 15 points
    }
```

**Five-Dimension Scoring Model**:

| Dimension | Points | Criteria |
|-----------|--------|----------|
| Market Scale | 30 | Total sales volume, search volume, top seller performance |
| Growth Potential | 20 | YoY growth rate, trend momentum, seasonality |
| Competition Level | 20 | Number of sellers, review count distribution, brand concentration |
| Entry Barrier | 15 | Average review count, established brand presence, capital requirements |
| Profit Margin | 15 | Price range, cost structure, margin opportunity |

**Output**: 
- `category-reports/{CATEGORY}_{SITE}_{DATE}/report.md`
- `category-reports/{CATEGORY}_{SITE}_{DATE}/dashboard.html`
- `category-reports/{CATEGORY}_{SITE}_{DATE}/category_report.xlsx`

### 3. Keyword Research (`keyword-research`)

Deep keyword research with 8-dimension intelligent classification for 1500+ keywords.

```bash
# Research keywords for an ASIN
/keyword-research B0D9ZTW7PS US
```

**Python Implementation Pattern**:

```python
def research_keywords(asin: str, site: str, mcp_url: str):
    """
    8-dimension keyword classification with ad strategy guidance
    """
    # 1. Get traffic keywords
    traffic_keywords = requests.post(
        mcp_url,
        json={
            "method": "product_traffic_terms",
            "params": {"asin": asin, "site": site, "limit": 100}
        }
    ).json()
    
    # 2. Expand related keywords for each traffic term
    all_keywords = []
    for kw in traffic_keywords[:10]:  # Top 10 keywords
        related = requests.post(
            mcp_url,
            json={
                "method": "keyword_related_words",
                "params": {"keyword": kw['keyword'], "site": site}
            }
        ).json()
        all_keywords.extend(related)
    
    # 3. Get keyword details (search volume, CPC)
    for kw in all_keywords:
        detail = requests.post(
            mcp_url,
            json={
                "method": "keyword_detail",
                "params": {"keyword": kw['keyword'], "site": site}
            }
        ).json()
        kw.update(detail)
    
    # 4. Classify keywords into 8 dimensions
    classified = classify_keywords_8_dimensions(all_keywords)
    
    # 5. Generate reports
    report_dir = f"keyword-reports/{asin}_{site}_{datetime.now().strftime('%Y%m%d')}"
    os.makedirs(report_dir, exist_ok=True)
    
    # Save CSV with all keywords
    save_keywords_csv(f"{report_dir}/keywords.csv", classified)
    
    # Save negative keywords list
    with open(f"{report_dir}/negative_words.txt", 'w') as f:
        f.write('\n'.join([kw['keyword'] for kw in classified['NEGATIVE']]))
    
    # Save category-specific CSVs
    for category, keywords in classified.items():
        save_keywords_csv(f"{report_dir}/keywords_{category.lower()}.csv", keywords)
    
    # Generate dashboard
    generate_keyword_dashboard(f"{report_dir}/dashboard.html", classified)
    
    return classified

def classify_keywords_8_dimensions(keywords: list) -> dict:
    """
    8-dimension intelligent classification
    """
    categories = {
        "NEGATIVE": [],    # Negative/sensitive words
        "BRAND": [],       # Brand names
        "MATERIAL": [],    # Material descriptors
        "SCENARIO": [],    # Use case/scene words
        "ATTRIBUTE": [],   # Attribute modifiers
        "FUNCTION": [],    # Functional words
        "CORE": [],        # Core product words
        "OTHER": []        # Uncategorized
    }
    
    for kw in keywords:
        category = classify_single_keyword(kw)
        categories[category].append(kw)
    
    return categories
```

**8-Dimension Classification**:

| Dimension | Use Case | Ad Strategy |
|-----------|----------|-------------|
| NEGATIVE | Irrelevant/sensitive terms | Direct negation in campaigns |
| BRAND | Competitor brand names | Competitor targeting or negation |
| MATERIAL | Material descriptors (e.g., "stainless steel") | Exact match campaigns |
| SCENARIO | Use case keywords (e.g., "outdoor camping") | Scene-based ad groups |
| ATTRIBUTE | Attribute modifiers (e.g., "waterproof") | Long-tail exact match |
| FUNCTION | Functional keywords (e.g., "fast charging") | Broad match for discovery |
| CORE | Core product terms (e.g., "bluetooth speaker") | High bid, top-of-search placement |
| OTHER | Miscellaneous | Supplementary targeting |

**Output**:
- `keyword-reports/{ASIN}_{SITE}_{DATE}/report.md`
- `keyword-reports/{ASIN}_{SITE}_{DATE}/keywords.csv`
- `keyword-reports/{ASIN}_{SITE}_{DATE}/negative_words.txt`
- `keyword-reports/{ASIN}_{SITE}_{DATE}/keywords_{category}.csv`
- `keyword-reports/{ASIN}_{SITE}_{DATE}/dashboard.html`

### 4. Review Analysis (`review-analysis`)

6-dimension pain point analysis with service risk warnings.

```bash
# Analyze product reviews
/review-analysis B0DZCBYCNY US
```

**Python Implementation Pattern**:

```python
def analyze_reviews(asin: str, site: str, mcp_url: str):
    """
    6-dimension pain point analysis with service risk assessment
    """
    # 1. Get product reviews (negative reviews priority)
    reviews = requests.post(
        mcp_url,
        json={
            "method": "product_reviews",
            "params": {
                "asin": asin,
                "site": site,
                "limit": 100,
                "rating_filter": "1,2,3"  # Low ratings only
            }
        }
    ).json()
    
    # 2. Analyze pain points in 6 dimensions
    pain_points = analyze_six_dimension_pain_points(reviews)
    
    # 3. Calculate service risk metrics
    service_risks = calculate_service_risks(reviews)
    
    # 4. Generate improvement suggestions
    suggestions = generate_improvement_suggestions(pain_points, service_risks)
    
    # 5. Save reports
    report_dir = f"review-analysis-reports/{asin}_{site}_{datetime.now().strftime('%Y%m%d')}"
    os.makedirs(f"{report_dir}/data", exist_ok=True)
    
    # Save raw SSE response
    with open(f"{report_dir}/data/raw_reviews_sse.txt", 'w') as f:
        f.write(json.dumps(reviews, indent=2))
    
    # Save structured analysis
    analysis_data = {
        "pain_points": pain_points,
        "service_risks": service_risks,
        "suggestions": suggestions
    }
    with open(f"{report_dir}/data/negative_reviews_analysis.json", 'w') as f:
        f.write(json.dumps(analysis_data, indent=2, ensure_ascii=False))
    
    # Generate Markdown report
    with open(f"{report_dir}/report.md", 'w') as f:
        f.write(generate_review_analysis_report(analysis_data))
    
    return analysis_data

def analyze_six_dimension_pain_points(reviews: list) -> dict:
    """
    6-dimension pain point framework
    """
    dimensions = {
        "electronic_failure": [],      # Battery, charging, connectivity issues
        "structural_issues": [],       # Parts broken, seal failure, connector breaks
        "design_defects": [],          # UX issues, complexity, missing features
        "appearance_material": [],     # Odor, allergies, color mismatch, scratches
        "description_mismatch": [],    # Feature expectation gaps, size/color differences
        "service_logistics": []        # Used/defective items, missing parts, difficult returns
    }
    
    for review in reviews:
        categorized_issues = categorize_review_issues(review)
        for dimension, issues in categorized_issues.items():
            dimensions[dimension].extend(issues)
    
    return dimensions

def calculate_service_risks(reviews: list) -> dict:
    """
    Service risk assessment with threshold warnings
    """
    total_reviews = len(reviews)
    
    risk_metrics = {
        "used_defective_items": 0,      # Warning: >2%, Danger: >5%
        "missing_parts": 0,             # Warning: >1%, Danger: >3%
        "return_difficulties": 0,       # Warning: >5%, Danger: >10%
        "customer_service_issues": 0    # Warning: >3%, Danger: >7%
    }
    
    for review in reviews:
        if "used" in review['text'].lower() or "defective" in review['text'].lower():
            risk_metrics["used_defective_items"] += 1
        if "missing" in review['text'].lower():
            risk_metrics["missing_parts"] += 1
        if "return" in review['text'].lower() and "difficult" in review['text'].lower():
            risk_metrics["return_difficulties"] += 1
        if "customer service" in review['text'].lower() and review['rating'] <= 2:
            risk_metrics["customer_service_issues"] += 1
    
    # Calculate percentages and risk levels
    risk_assessment = {}
    for metric, count in risk_metrics.items():
        percentage = (count / total_reviews) * 100
        risk_level = determine_risk_level(metric, percentage)
        risk_assessment[metric] = {
            "count": count,
            "percentage": percentage,
            "risk_level": risk_level
        }
    
    return risk_assessment
```

**6-Dimension Pain Point Framework**:

| Dimension | Issues Identified | Severity Assessment |
|-----------|-------------------|---------------------|
| Electronic Failure | Battery, charging, connectivity, feature malfunction | High/Medium/Low |
| Structural Issues | Parts broken, seal failure, connector breaks | High/Medium/Low |
| Design Defects | Software UX, complexity, missing features | High/Medium/Low |
| Appearance/Material | Odor, allergies, color mismatch, scratches | High/Medium/Low |
| Description Mismatch | Feature gaps, size/color differences | High/Medium/Low |
| Service/Logistics | Used/defective items, missing parts, return difficulties | High/Medium/Low |

**Service Risk Thresholds**:

| Issue Type | Warning | Danger |
|------------|---------|--------|
| Used/Defective Items | >2% | >5% |
| Missing Parts | >1% | >3% |
| Return Difficulties | >5% | >10% |
| Customer Service Issues | >3% | >7% |

**Output**:
- `review-analysis-reports/{ASIN}_{SITE}_{DATE}/report.md`
- `review-analysis-reports/{ASIN}_{SITE}_{DATE}/data/raw_reviews_sse.txt`
- `review-analysis-reports/{ASIN}_{SITE}_{DATE}/data/negative_reviews_analysis.json`

### 5. Product Research (`product-research`)

LLM-driven deep research workflow from data collection to decision.

```bash
# Research bluetooth speaker market
/product-research "bluetooth speaker" US

# Research laptop backpack market
/product-research "laptop backpack" GB
```

**Python Implementation Pattern**:

```python
def product_research_workflow(keyword: str, site: str, mcp_url: str):
    """
    LLM-driven product research workflow
    Step 1: Information Collection
    Step 2: Data Acquisition
    Step 3: Attribute Tagging
    Step 4: Cross Analysis
    Step 5: Competitor & VOC
    Step 6: Evaluation & Decision
    Step 7: Report Generation
    """
    # Step 1: Search products
    products = requests.post(
        mcp_url,
        json={
            "method": "product_search",
            "params": {
                "keyword": keyword,
                "site": site,
                "limit": 50
            }
        }
    ).json()
    
    # Step 2: Collect detailed data for top products
    product_data = []
    for product in products[:20]:
        detail = get_product_complete_data(product['asin'], site, mcp_url)
        product_data.append(detail)
    
    # Step 3-6: LLM analysis (pseudocode - actual LLM calls)
    analysis_results = {
        "attribute_analysis": llm_attribute_tagging(product_data),
        "cross_analysis": llm_cross_analysis(product_data),
        "competitor_analysis": llm_competitor_analysis(product_data),
        "voc_analysis": llm_voc_analysis(product_data),
        "market_decision": llm_market_decision(product_data)
    }
    
    # Step 7: Generate reports
    report_dir = f"product-research-reports/{keyword.replace(' ', '_')}_{site}_{datetime.now().strftime('%Y%m%d')}"
    os.makedirs(report_dir, exist_ok=True)
    
    # Save structured data
    with open(f"{report_dir}/data.json", 'w') as f:
        f.write(json.dumps({
            "products": product_data,
            "analysis": analysis_results
        }, indent=2, ensure_ascii=False))
    
    # Generate Markdown report (LLM-written)
    report_markdown = llm_generate_report(product_data, analysis_results)
    with open(f"{report_dir}/report.md", 'w') as f:
        f.write(report_markdown)
    
    # Generate visualization dashboard
    generate_research_dashboard(f"{report_dir}/dashboard.html", product_data, analysis_results)
    
    return analysis_results

def get_product_complete_data(asin: str, site: str, mcp_url: str) -> dict:
    """
    Collect all data points for a product
    """
    return {
        "detail": get_product_detail(asin, site, mcp_url),
        "trends": get_product_trends(asin, site, mcp_url),
        "keywords": get_product_keywords(asin, site, mcp_url),
        "reviews": get_product_reviews(asin, site, mcp_url),
        "tiktok_videos": get_tiktok_videos(asin, site, mcp_url),
        "supply_chain": get_1688_suppliers(asin, mcp_url)
    }
```

**Research Workflow Phases**:

1. **Information Collection**: Search and identify candidate products
2. **Data Acquisition**: Collect comprehensive data (details, trends, keywords, reviews)
3. **Attribute Tagging**: LLM extracts features, materials, use cases
4. **Cross Analysis**: Compare products across dimensions
5. **Competitor & VOC**: Analyze competitor strategies and customer voice
6. **Evaluation & Decision**: LLM provides market entry recommendation
7. **Report Generation**: Markdown report + structured data + dashboard

**Output**:
- `product-research-reports/{KEYWORD}_{SITE}_{DATE}/report.md`
- `product-research-reports/{KEYWORD}_{SITE}_{DATE}/data.json`
- `product-research-reports/{KEYWORD}_{SITE}_{DATE}/dashboard.html`

### 6. Sif Amazon Research (`sif-amazon-research`)

Market validation, competitor analysis, traffic diagnostics using Sif MCP.

```bash
# Interactive research workflow
/sif-amazon-research
```

**Sif Research Decision Types**:

| Decision Type | Purpose | Key Questions |
|---------------|---------|---------------|
| Market Entry | Should we enter this market? | Is there demand? Can we compete? |
| Product Launch | Should we launch this product? | Is validation complete? Ready to scale? |
| Growth Strategy | Should we expand or optimize? | Double down or pause? |
| Root Cause Analysis | Why did performance change? | Traffic drop? Ad issue? Competition? |

**Python Implementation Pattern**:

```python
def sif_research_workflow(decision_type: str, params: dict, sif_mcp_url: str):
    """
    Sif MCP research with evidence-based decision framework
    """
    # Step 1: Define decision context
    context = {
        "decision_type": decision_type,
        "candidate_asin": params.get("asin"),
        "keywords": params.get("keywords", []),
        "site": params.get("site", "US")
    }
    
    # Step 2: Collect Sif evidence
    evidence = collect_sif_evidence(context, sif_mcp_url)
    
    # Step 3: Cross-validate evidence
    validated_evidence = cross_validate_evidence(evidence)
    
    # Step 4: Generate business decision
    decision = generate_business_decision(validated_evidence, context)
    
    # Step 5: Output report
    report_dir = f"sif-research-reports/{decision_type}_{datetime.now().strftime('%Y%m%d%H%M%S')}"
    os.makedirs(report_dir, exist_ok=True)
    
    with open(f"{report_dir}/decision_report.md", 'w') as f:
        f.write(format_sif_decision_report(decision, validated_evidence))
    
    return decision

def collect_sif_evidence(context: dict, sif_mcp_url: str) -> dict:
    """
    Collect evidence from Sif MCP service
    """
    evidence = {}
    
    # Market demand evidence
    if context.get("keywords"):
        evidence["market_demand"] = requests.post(
            sif_mcp_url,
            json={
                "method": "keyword_market_research",
                "params": {"keywords": context["keywords"], "site": context["site"]}
            }
        ).json()
    
    # Competition evidence
    if context.get("candidate_asin"):
        evidence["competition"] = requests.post(
            sif_mcp_url,
            json={
                "method": "competitor_landscape",
                "params": {"asin": context["candidate_asin"], "site": context["site"]}
            }
        ).json()
    
    # Traffic structure evidence
    if context.get("candidate_asin"):
        evidence["traffic_structure"] = requests.post(
            sif_mcp_url,
            json={
                "method": "traffic_source_analysis",
                "params": {"asin": context["candidate_asin"], "site": context["site"]}
            }
        ).json()
    
    # Sales trends evidence
    if context.get("candidate_asin"):
        evidence["sales_trends"] = requests.post(
            sif_mcp_url,
            json={
                "method": "sales_trend_analysis",
                "params": {"asin": context["candidate_asin"], "site": context["site"]}
            }
        ).json()
    
    # Ad performance evidence
    if context.get("candidate_asin"):
        evidence["ad_performance"] = requests.post(
            sif_mcp_url,
            json={
                "method": "ad_performance_analysis",
                "params": {"asin": context["candidate_asin"], "site": context["site"]}
            }
        ).json()
    
    return evidence

def generate_business_decision(evidence: dict, context: dict) -> dict:
    """
    Generate business decision with confidence level
    """
    return {
        "decision": "ENTER" | "DO_NOT_ENTER" | "LAUNCH" | "DO_NOT_LAUNCH" | "EXPAND" | "OPTIMIZE" | "STOP",
        "confidence": 0.85,  # 0.0 to 1.0
        "evidence_chain": [
            {"finding": "Market demand validated", "confidence": 0.9, "source": "keyword_market_research"},
            {"finding": "Competition moderate", "confidence": 0.8, "source": "competitor_landscape"}
        ],
        "risks": [
            {"risk": "Seasonal demand pattern", "severity": "MEDIUM", "mitigation": "Plan inventory accordingly"}
        ],
        "missing_data": [
            {"data": "Supplier MOQ requirements", "impact": "MEDIUM", "action": "Contact 1688 suppliers"}
        ],
        "next_actions": [
            {"action": "Validate supplier pricing", "priority": "HIGH", "deadline": "3 days"},
            {"action": "Test ad creative variants", "priority": "MEDIUM", "deadline": "1 week"}
        ]
    }
```

### 7. Xiyou Insight (`xiyou-insight`)

7-scenario analysis workflows for ad monitoring, traffic gaps, and competitor intelligence.

```bash
# Ad monitoring workflow
/xiyou-insight --scenario ad_monitoring --asin B07PWTJ4H1 --site US --keyword "bluetooth speaker"

# Traffic gap analysis
/xiyou-insight --scenario traffic_gap --own_asin B07PWTJ4H1 --competitor_asins B08XYZ,B09ABC --site US

# Competitor analysis
/xiyou-insight --scenario competitor_analysis --asin B08N5WRWNW --site US

# New product launch tracking
/xiyou-insight --scenario new_product --asin B0D9ZTW7PS --site US

# Ad budget transparency
/xiyou-insight --scenario ad_budget --asin B07PWTJ4H1 --site US

# Keyword database building
/xiyou-insight --scenario keyword_database --site US --competitor_asins B07ABC,B08XYZ --core_keywords "wireless earbuds,bluetooth headphones"
```

**7 Xiyou Scenarios**:

| Scenario | Purpose | Key Tools |
|----------|---------|-----------|
| Ad Monitoring | Hourly rank tracking, ad placement effectiveness | `get_asin_keyword_rank_hourly`, `get_asin_ad_change_trends` |
| Traffic Gap | Find keywords where competitors rank but you don't | `get_asin_keywords`, cross-ASIN comparison |
| Competitor Analysis | Deconstruct competitor traffic and ad strategy | `get_asin_traffic`, `get_asin_keywords`, `get_asin_ad_change_trends` |
| New Product Launch | Track traffic stage transitions and keyword momentum | `get_a
