---
description: "Analyze economic data by cleaning, calculating metrics, identifying trends, creating charts, and generating professional insights with risk assessment"
name: "Economic Data Analysis"
argument-hint: "Provide economic dataset or describe analysis goal"
agent: "agent"
---

# Economic Data Analysis

Analyze the provided economic data and produce a concise, professional output.

## Objective

Perform a complete economic data analysis workflow:
1. Clean and validate the data
2. Calculate key metrics
3. Identify trends and comparisons
4. Generate real chart images as PNG files
5. Produce insights, risks, limitations, and a concise report

---

## 1. Data Preparation

Clean and validate the dataset before analysis.

### Tasks
- Check for missing values, duplicates, invalid entries, and obvious outliers
- Validate data types and value ranges
- Ensure time-based data is in the correct chronological order
- Document any assumptions, fixes, or transformations

### Rules
- Keep the original data unchanged
- Use a cleaned working version for analysis
- Flag suspicious values before removing or changing them
- If data is too incomplete or invalid, explain the issue and stop

---

## 2. Metrics and Calculations

Calculate relevant economic indicators based on the dataset.

### Examples
- Total
- Average / mean
- Minimum and maximum
- Absolute change
- Percentage change
- Year-over-year growth
- Compound annual growth rate (if appropriate)
- Shares by category or region

### Rules
- Use only the provided data unless the user explicitly asks for external context
- Show only useful calculations
- Do not invent missing values unless clearly stated and justified

---

## 3. Trends and Comparisons

Identify the main economic patterns.

### Tasks
- Detect upward, downward, stable, or volatile trends
- Compare years, sectors, regions, or categories
- Highlight anomalies, turning points, or unusually large changes
- Distinguish between facts and possible interpretations

---

## 4. Visualizations

Generate **real chart images as PNG files**, not just text descriptions or chart suggestions.

### Required charts
Create at least **2 actual charts**:
1. **Line chart** for trends over time
2. **Bar chart** for comparisons across years, sectors, regions, or categories

### Optional charts
Use additional charts if helpful:
- Scatter plot for relationships
- Pie chart or stacked bar chart for shares
- Histogram or box plot for distributions

### Chart requirements
- Generate actual PNG image files
- Use clear titles
- Label all axes
- Include units where relevant
- Add legends when needed
- Ensure readability and clarity
- Avoid misleading scales or clutter

### Important rule
Do **not** only describe recommended charts.  
You must actually create chart image files in `.png` format.

### Output for each chart
Provide:
- Chart title
- Brief explanation of what it shows
- File name or file path of the generated PNG image

### Execution requirement
If the environment supports code execution, use it to generate and save the charts as real PNG files.  
If real image generation is not possible in the current environment, clearly state that limitation instead of pretending the charts were created.

---

## 5. Insights and Interpretation

Summarize the key findings in plain language.

### Tasks
- Explain what the numbers show
- Highlight the most important economic signals
- Connect findings to plausible economic implications
- Distinguish clearly between observed results and possible causes

### Style
- Concise
- Evidence-based
- Professional
- Easy to understand

---

## 6. Risks and Limitations

Identify constraints that affect interpretation.

### Include
- Data quality issues
- Missing values or limited observations
- Assumptions made during cleaning or analysis
- External factors not captured in the dataset
- Any uncertainty in the conclusions

---

## 7. Professional Report

Produce a well-organized report suitable for coursework.

### Structure

#### Summary
- 2–3 sentence overview of the main findings

#### Data Preparation
- What was checked, cleaned, and assumed

#### Metrics and Analysis
- Key calculations and comparisons

#### Visualizations
- Embed or reference the generated chart PNG files
- Brief explanation of each

#### Insights
- What the data means

#### Risks and Limitations
- Caveats and uncertainties

#### Conclusion
- Bottom-line takeaway

---

## 8. Final Deliverables

The final output must include:

1. A concise written analysis
2. Key metrics and calculations
3. Trend and comparison analysis
4. Real generated chart images in `.png` format
5. File names or file paths for the generated charts
6. Insights, risks, and limitations
7. A short professional conclusion

---

## Output Rules

- Be concise and structured
- Use headings and bullet points where useful
- Support claims with data
- Do not invent data, charts, or results
- Do not claim causality unless supported
- If chart images are not actually created as PNG files, the task is incomplete

**Output Format:** A concise, professional, coursework-ready analysis with real PNG chart outputs.