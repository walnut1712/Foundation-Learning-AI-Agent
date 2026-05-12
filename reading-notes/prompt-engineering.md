# Prompt Engineering: Zero-Shot, Few-Shot, CoT, ReAct

**Date**: May 12, 2026  
**Purpose**: Comprehensive guide to modern prompt engineering techniques  
**Scope**: Definitions, principles, and practical experiments for zero-shot, few-shot, Chain of Thought (CoT), and ReAct prompting

---

## Executive Summary

Prompt engineering is the art of structuring queries to LLMs for optimal output. The four core techniques represent an evolution from simple instruction-following to sophisticated reasoning:

1. **Zero-Shot**: Ask directly without examples (requires strong model capability)
2. **Few-Shot**: Provide examples before asking (teaches by example)
3. **Chain of Thought**: Demand step-by-step reasoning (improves complex reasoning)
4. **ReAct**: Interleave reasoning with actions (enables tool use and real-world interaction)

**Key Principle**: Better prompts → better outputs. The goal isn't to trick the model; it's to clarify what you want and how you want it reasoned through.

---

## Part 1: Zero-Shot Prompting

### Definition

**Zero-Shot**: Asking the model to complete a task without providing examples. The model relies entirely on its pretraining and your instruction.

**Core Concept**:

- No demonstrations
- Only task description and context
- Model must generalize from learned patterns
- Success depends on task clarity and model capability

### When Zero-Shot Works Best

✅ **Good for**:

- Well-defined tasks (classification, summarization, Q&A)
- Tasks the model has seen often during training
- Simple instructions where capability is obvious
- Quick iterations when you don't have examples

❌ **Poor for**:

- Unusual or novel tasks
- Tasks requiring specific formatting
- Complex reasoning that needs scaffolding
- Domain-specific conventions the model might misinterpret

### Key Principles

1. **Clarity is paramount**: "Summarize this text" vs "Extract the main argument as a 1-sentence claim"
2. **Role-setting helps**: "You are a data analyst..." provides context
3. **Output format specification**: "Format as JSON" or "List as bullet points" guides output
4. **Constraint specification**: "Use only 50 words" provides boundaries

### Best Practices

```
Structure: [Role] + [Task] + [Context] + [Output Format] + [Constraints]

Example pattern:
"You are a technical writer. Summarize the following code changes for a pull request description.
Keep it under 100 words and use simple language for non-expert readers.

Code changes:
[code here]"
```

### Experiment 1: Zero-Shot Classification with Role-Setting

**Objective**: Classify customer feedback sentiment without providing examples

**Hypothesis**: Adding role context ("You are a sentiment analyst") improves classification accuracy vs. plain instruction

**Setup**:

**Version A (Plain Instruction)**:

```
Classify the following as Positive, Negative, or Neutral:
"The product works well but the documentation was confusing and took me hours to figure out"
```

**Version B (Role-Based)**:

```
You are a sentiment analyst expert at identifying nuanced customer feedback.
Classify as Positive, Negative, or Neutral, noting if sentiment is mixed.
Explain your reasoning in 1 sentence.

Feedback: "The product works well but the documentation was confusing and took me hours to figure out"
```

**Expected Output A**: Likely "Negative" (focuses on final impact)

**Expected Output B**:

```
Classification: Mixed (Positive product, Negative support)
Reasoning: Product quality is praised but offset by frustration with documentation.
```

**Learning**: Role-setting encourages nuanced analysis. Version B captures complexity Version A misses.

**Variations to Test**:

- Add criteria: "Consider: usability, support, value, documentation"
- Add calibration: "If the main complaint is easily fixable, consider it Neutral rather than Negative"

---

### Experiment 2: Zero-Shot Q&A with Format Specification

**Objective**: Extract structured information without examples

**Hypothesis**: Explicit format specification improves consistency vs. vague instruction

**Setup**:

**Version A (Vague)**:

```
Answer: What are the main ingredients in chocolate cake?
```

**Version B (Structured)**:

```
Answer the following using this format:
- Ingredient name: quantity

Question: What are the main ingredients in chocolate cake?
```

**Expected Output A**: Unstructured prose or inconsistent list format

**Expected Output B**:

```
- Flour: 2 cups
- Cocoa powder: 3/4 cup
- Sugar: 1.75 cups
- Eggs: 3 large
- Butter: 1/2 cup
- Milk: 1 cup
```

**Learning**: Format specification dramatically improves consistency. Useful for downstream parsing.

**Variations to Test**:

- Nested structure: "Format as JSON with {ingredient: {name, quantity, unit}}"
- Conditional format: "List ingredients as Markdown table with columns: ingredient | amount | substitute_available (yes/no)"

---

### Experiment 3: Zero-Shot Reasoning Limits

**Objective**: Identify when zero-shot fails and needs other techniques

**Hypothesis**: Complex multi-step reasoning fails in zero-shot; simple reasoning succeeds

**Setup**:

**Simple Task (Expect Success)**:

```
What's 17 + 28?
```

**Complex Task (Expect Struggle)**:

```
A store sells apples for $1.50 each. You have a coupon for 20% off the total.
You buy 7 apples. How much do you pay?
```

**Expected Result for Simple**: ✅ Correct (45)

**Expected Result for Complex**: ⚠️ Likely wrong or right by accident. Model may make arithmetic errors or skip steps.

**Follow-up with CoT** (to be tested in CoT section):

```
A store sells apples for $1.50 each. You have a coupon for 20% off the total.
You buy 7 apples. Show your work step by step:
1. Calculate base cost
2. Calculate discount
3. Calculate final price
```

**Learning**: Zero-shot works for tasks with clear single-step logic. Multi-step math or reasoning needs scaffolding.

---

## Part 2: Few-Shot Prompting

### Definition

**Few-Shot**: Providing examples (typically 2-5) of the desired task before asking the model to perform it on new inputs.

**Core Concept**:

- 1-3 examples per demonstration
- Examples show both input and desired output
- Model learns pattern from examples (in-context learning)
- No model retraining; learning happens at inference time
- More examples ≠ always better (often 2-3 optimal; more adds noise)

### When Few-Shot Works Best

✅ **Good for**:

- Unusual tasks or specific formatting
- Domain-specific conventions
- Rare or edge cases you want to handle consistently
- When zero-shot fails or produces inconsistent output

❌ **Poor for**:

- Tasks with many possible formats (too many examples needed)
- Tasks the model already knows well (zero-shot sufficient)
- When you don't have clean examples
- Very large example sets (token budget, noise)

### Key Principles

1. **Quality over quantity**: 2 perfect examples > 5 mediocre ones
2. **Diversity**: Examples should cover the range of expected inputs
3. **Order matters**: Put best/clearest examples first
4. **Consistency**: Format examples identically to how you want output
5. **Explicit labels**: Make the output explicit ("Output: ...") not implied

### Best Practices

```
Structure:
[Task Description]
[Example 1: Input + Output]
[Example 2: Input + Output]
[Example 3: Input + Output]
[New Input: ? → Now your turn]

Example pattern:
"Classify the following feedback as Feature Request, Bug Report, or Other.

Example 1:
Input: 'The app crashes when I upload images larger than 5MB'
Output: Bug Report

Example 2:
Input: 'Would love to see dark mode in the app'
Output: Feature Request

Example 3:
Input: 'Love how responsive the UI is!'
Output: Other

Now classify:
Input: 'The export function doesn't work with Chinese characters'
Output: ?"
```

### Experiment 1: Few-Shot Task Definition (Domain-Specific)

**Objective**: Teach the model to identify and correct grammatical errors in a specific, inconsistent way

**Hypothesis**: Few-shot examples enable consistent, custom error correction behavior vs. generic grammar correction

**Setup**:

**Version A (Zero-Shot Generic Grammar)**:

```
Correct this sentence:
"The team have completed their work"
```

**Version B (Few-Shot Custom Pattern)**:

```
Correct grammar while preserving informal British English style:

Example 1:
Input: "The team have finished the project yesterday"
Output: "The team have finished the project"
(Removed: "yesterday" - unnecessary when talking about recent past)

Example 2:
Input: "We was going to the meeting"
Output: "We were going to the meeting"
(Fixed: "was" → "were" for plural subject)

Example 3:
Input: "She don't like coffee"
Output: "She doesn't like coffee"
(Fixed: subject-verb agreement)

Now correct this:
Input: "The team have completed their work"
Output: ?
```

**Expected Output A**: "The team has completed their work" (Applies American grammar rules)

**Expected Output B**: "The team have completed their work" (Preserves British English; recognizes it's correct)

**Learning**: Few-shot examples define custom rules. Without them, the model applies generic patterns.

**Why This Matters**: Domain-specific language rules, company conventions, or style guides can be taught through examples rather than long instructions.

---

### Experiment 2: Few-Shot Format Teaching

**Objective**: Teach complex output format through examples alone

**Hypothesis**: 3 examples teach format better than written format specification

**Setup**:

**Version A (Written Specification)**:

```
Convert the following to JSON format with fields: name (string), age (integer),
email (string), is_active (boolean), tags (array of strings).

Input: "John Smith, 34 years old, john@example.com, account active, has tags: developer, python"
Output: ?
```

**Version B (Few-Shot Examples)**:

```
Convert to JSON format:

Example 1:
Input: "Alice Johnson, 28 years old, alice@work.com, account active, tags: manager, hiring"
Output: {"name": "Alice Johnson", "age": 28, "email": "alice@work.com", "is_active": true, "tags": ["manager", "hiring"]}

Example 2:
Input: "Bob Lee, 45 years old, bob@company.org, account inactive, tags: retired, advisor"
Output: {"name": "Bob Lee", "age": 45, "email": "bob@company.org", "is_active": false, "tags": ["retired", "advisor"]}

Example 3:
Input: "Carol White, 31 years old, carol@domain.net, account active, tags: engineer, python, leadership"
Output: {"name": "Carol White", "age": 31, "email": "carol@domain.net", "is_active": true, "tags": ["engineer", "python", "leadership"]}

Now convert:
Input: "John Smith, 34 years old, john@example.com, account active, has tags: developer, python"
Output: ?
```

**Expected Output A**: May have format inconsistencies (quotes, spacing, field order varies)

**Expected Output B**: Consistent format matching the examples

**Learning**: Examples teach format more reliably than written specifications. Useful for data pipelines where consistency matters.

**Variations to Test**:

- Add edge case example: inactive user with no tags
- Add error example: "Example of what NOT to do" (though models sometimes struggle with negative examples)

---

### Experiment 3: Few-Shot Diversity and Edge Cases

**Objective**: Test whether diverse examples improve performance on varied inputs

**Hypothesis**: Examples covering edge cases improve handling of unusual inputs

**Setup**:

**Version A (Limited Examples - Happy Path Only)**:

```
Extract company name from business descriptions:

Example 1:
Input: "Apple makes iPhones and computers"
Output: Apple

Example 2:
Input: "Tesla builds electric vehicles"
Output: Tesla

Now extract from:
Input: "Johnson & Johnson: healthcare and consumer products"
Output: ?
```

**Version B (Diverse Examples - Including Edge Cases)**:

```
Extract company name from business descriptions:

Example 1:
Input: "Apple makes iPhones and computers"
Output: Apple

Example 2:
Input: "Tesla builds electric vehicles"
Output: Tesla

Example 3:
Input: "Johnson & Johnson: healthcare and consumer products"
Output: Johnson & Johnson

Example 4:
Input: "The 3M Company manufactures adhesives and industrial products"
Output: 3M

Now extract from:
Input: "AT&T provides telecommunications services"
Output: ?
```

**Expected Output A**: May return "Johnson" (stops at ampersand)

**Expected Output B**: Returns "Johnson & Johnson" or "AT&T" (learned the pattern from examples)

**Learning**: Diverse examples including edge cases (ampersands, numbers, special characters) teach robustness. Few-shot learns from what you show it.

---

## Part 3: Chain of Thought (CoT) Prompting

### Definition

**Chain of Thought**: Explicitly asking the model to show its reasoning step-by-step before providing the final answer.

**Core Concept**:

- Model verbalizes intermediate reasoning
- Makes thinking process visible and checkable
- Often improves answer quality on complex tasks
- Especially powerful for: math, logic, multi-step reasoning
- Can be combined with few-shot (few-shot CoT)

### Mechanism: Why It Works

```
Without CoT:
Problem → [Black box] → Answer (often wrong)

With CoT:
Problem → Step 1: Define variables → Step 2: Set up equation →
  Step 3: Solve → Step 4: Check answer → Answer (usually right)

Benefit: Each step is inspectable and correctible
```

**Research Finding**: Wei et al. (2022) showed CoT dramatically improves performance on:

- Math word problems: 17.7% → 78.7% (77-point improvement!)
- Commonsense reasoning: 60% → 88%
- Symbolic manipulation: 34% → 90%

### When CoT Works Best

✅ **Good for**:

- Math and arithmetic
- Logic puzzles and reasoning
- Multi-step processes (recipes, procedures)
- Complex analysis or explanation
- Catching model's own errors (visible reasoning can be wrong-checked)

❌ **Poor for**:

- Simple classification or lookup
- Tasks requiring single correct word
- Cases where steps are obvious/repetitive
- Very complex tasks (may lead to error accumulation)

### Key Principles

1. **"Let's think step by step"**: Even this phrase alone improves performance
2. **Explicit numbering**: Makes steps clear and parseable
3. **Intermediate verification**: "Does this make sense?" at key points
4. **Show all work**: Even obvious steps help prevent errors
5. **One thought per step**: Avoid combining multiple concepts in one line

### Best Practices

```
Structure:
[Problem]
Let's think through this step by step:
1. [First step - what are we dealing with?]
2. [Next step - what's the approach?]
3. [Execution - work through it]
4. [Verification - does it check out?]
5. [Final answer]

Example pattern:
"A recipe calls for 3 cups of flour for 12 cookies.
How much flour for 30 cookies? Think step by step:
1. Establish the ratio: ...
2. Set up the proportion: ...
3. Calculate: ...
4. Verify: ...
5. Answer: ..."
```

### Experiment 1: CoT vs. Zero-Shot Math Problem

**Objective**: Compare zero-shot vs. CoT on arithmetic reasoning

**Hypothesis**: CoT dramatically improves accuracy on multi-step math problems

**Setup**:

**Version A (Zero-Shot)**:

```
Question: There are 15 red balls and 25 blue balls.
If I remove 1/3 of the red balls and 1/5 of the blue balls,
how many balls remain?

Answer: ?
```

**Version B (CoT)**:

```
Question: There are 15 red balls and 25 blue balls.
If I remove 1/3 of the red balls and 1/5 of the blue balls,
how many balls remain?

Let's work through this step by step:
1. Red balls remaining: Start with 15, remove 1/3 →
2. Blue balls remaining: Start with 25, remove 1/5 →
3. Total remaining: Red + Blue =

Answer: ?
```

**Expected Output A**: May be wrong (arithmetic errors) or vague (no showing of work)

**Expected Output B**:

```
1. Red balls remaining: 15 - (15 × 1/3) = 15 - 5 = 10 red balls
2. Blue balls remaining: 25 - (25 × 1/5) = 25 - 5 = 20 blue balls
3. Total remaining: 10 + 20 = 30 balls

Answer: 30 balls remain
```

**Learning**: CoT exposes reasoning and catches errors. Intermediate steps make arithmetic checkable.

**Bonus**: If you get the wrong answer in CoT version, you can see where the model went wrong and correct it.

---

### Experiment 2: Few-Shot CoT (Demonstrating the Pattern)

**Objective**: Combine few-shot examples with CoT format

**Hypothesis**: Few-shot CoT examples teach both the format and the reasoning style

**Setup**:

**Version A (Few-Shot without CoT format)**:

```
Extract the name and infer the role from descriptions:

Example 1:
Input: "Sarah works with computer systems"
Output: Name: Sarah, Role: IT Professional

Example 2:
Input: "Tom builds structures and designs buildings"
Output: Name: Tom, Role: Architect

Now infer:
Input: "Jessica manages teams and creates schedules"
Output: ?
```

**Version B (Few-Shot CoT - Model sees reasoning)**:

```
Extract the name and infer the role from descriptions. Show your reasoning:

Example 1:
Input: "Sarah works with computer systems"
Reasoning: "works with computer systems" → IT/Technology role
Output: Name: Sarah, Role: IT Professional

Example 2:
Input: "Tom builds structures and designs buildings"
Reasoning: "builds structures" + "designs buildings" → Architecture/Engineering
Output: Name: Tom, Role: Architect

Now infer (show your reasoning):
Input: "Jessica manages teams and creates schedules"
Reasoning: ?
Output: ?
```

**Expected Output A**: Likely correct but reasoning opaque

**Expected Output B**:

```
Reasoning: "manages teams" + "creates schedules" → Leadership/Management/Project role
Output: Name: Jessica, Role: Project Manager or Team Lead
```

**Learning**: Few-shot CoT teaches both the answer format AND how to reason through similar problems. Powerful for complex classification.

---

### Experiment 3: CoT Applied to Logic Puzzles

**Objective**: Test CoT on non-mathematical reasoning

**Hypothesis**: CoT improves reasoning on logic puzzles despite no arithmetic

**Setup**:

**Version A (Zero-Shot)**:

```
Alice, Bob, and Carol went to a restaurant.
Alice ordered pasta. Bob ordered pizza.
Carol ordered neither pasta nor pizza.
Who ordered soup?

Answer: ?
```

**Version B (CoT)**:

```
Alice, Bob, and Carol went to a restaurant.
Alice ordered pasta. Bob ordered pizza.
Carol ordered neither pasta nor pizza.
Who ordered soup?

Let's think through what we know:
1. What did Alice order?
2. What did Bob order?
3. What did Carol NOT order?
4. What options remain for Carol?
5. Can we determine who ordered soup?

Answer: ?
```

**Expected Output A**: Likely says "Carol" (incorrect assumption)

**Expected Output B**:

```
1. Alice ordered pasta
2. Bob ordered pizza
3. Carol ordered neither pasta nor pizza
4. Carol could have ordered: soup, salad, or other items
5. We know Carol didn't order pasta or pizza, but we don't know if she ordered soup or something else
   The other people (Alice = pasta, Bob = pizza) also didn't order soup

Answer: We cannot determine who ordered soup from the given information
```

**Learning**: CoT reveals incomplete reasoning. Model is forced to face gaps in information. Improves from "guess" to "acknowledge uncertainty."

---

## Part 4: ReAct (Reasoning + Acting)

### Definition

**ReAct**: Interleaving model reasoning with actions/observations. The model thinks, decides to act, gets feedback, then reasons again.

**Core Concept**:

- Reason → Act → Observe → Reason → Act → ... (loop)
- Acts are tool calls: search, calculate, access data, etc.
- Observations feed back into reasoning
- Models become "agentic" — they plan and execute
- Enables interaction with external tools and data

### Mechanism: The ReAct Loop

```
[Thought]
"I need to find the current price of Tesla stock to answer this"

[Action]
search("Tesla stock price 2026")

[Observation]
"Tesla stock: $187.43"

[Thought]
"Good, now I know the price. Let me calculate if it's up from last year"

[Action]
search("Tesla stock price 2025")

[Observation]
"Tesla stock 2025: $165.20"

[Thought]
"Now I can calculate the percentage increase"

[Action]
calculate("(187.43 - 165.20) / 165.20 * 100")

[Observation]
"13.4% increase"

[Final Answer]
"Tesla stock is up 13.4% year-over-year"
```

### When ReAct Works Best

✅ **Good for**:

- Real-world questions needing current data
- Multi-step tasks requiring tool use
- Complex problems where you need to verify assumptions
- Tasks combining reasoning with execution
- Building AI agents/assistants
- Debugging (reason about problem, look at code, reason again, check output, etc.)

❌ **Poor for**:

- Simple factual questions from training data
- Tasks without external tools available
- One-shot problems (no iteration needed)
- When speed matters (actions add latency)
- When tools are unreliable (garbage feedback loops)

### Key Principles

1. **Clear thought format**: Model must articulate what it's thinking
2. **Discrete actions**: Each action is a specific call (search, calculator, code executor, etc.)
3. **Rich feedback**: Observations must be detailed enough to inform next thinking
4. **Error handling**: If action fails, model must reason about failure and try differently
5. **Termination condition**: Model must know when to stop and give final answer

### Best Practices

```
Structure:
[Question]
You have access to tools: [Tool 1], [Tool 2], [Tool 3]

[Model outputs:
Thought: ...
Action: [Tool] with [parameters]
]

[System returns:
Observation: [Tool output]
]

[Model outputs:
Thought: ...
Action: [Tool] or [Final Answer]
]

Example pattern:
"Question: What was the GDP of Japan in 2025?
You have access to: search_web, lookup_data

Thought: I need to find Japan's 2025 GDP
Action: search_web(query='Japan GDP 2025')

Observation: Japan's GDP in 2025 was approximately $4.23 trillion

Thought: I have the answer
Final Answer: Japan's GDP in 2025 was $4.23 trillion"
```

### Experiment 1: ReAct for Multi-Step Problem Solving

**Objective**: Demonstrate ReAct solving a problem that requires multiple information sources

**Hypothesis**: ReAct can break down complex questions and gather needed information through sequential actions

**Setup**:

**Version A (Zero-Shot - No Tools)**:

```
Question: Is it more profitable to invest $10,000 in Tesla or Apple stock right now?
Provide a recommendation.

Answer: ?
```

**Version B (ReAct with Tools)**:

```
Question: Is it more profitable to invest $10,000 in Tesla or Apple stock right now?

Available tools:
- get_stock_price(ticker, date)
- get_company_pe_ratio(ticker)
- get_market_sentiment(ticker)

Let me work through this:

Thought: To make a recommendation, I need current prices, valuations, and sentiment for both stocks

Action: get_stock_price("TSLA", "2026-05-12")
Observation: Tesla $187.43

Action: get_stock_price("AAPL", "2026-05-12")
Observation: Apple $198.76

Action: get_company_pe_ratio("TSLA")
Observation: Tesla P/E: 42.3

Action: get_company_pe_ratio("AAPL")
Observation: Apple P/E: 28.1

Action: get_market_sentiment("TSLA")
Observation: Mixed sentiment, growth concerns

Action: get_market_sentiment("AAPL")
Observation: Positive sentiment, stable growth

Thought: Now I have data to make a recommendation. Let me analyze:
- Tesla: Higher valuation (42.3 P/E), mixed sentiment, volatile
- Apple: More reasonable valuation (28.1 P/E), positive sentiment, stable

Final Answer: Based on the data, Apple appears more suitable for a conservative $10,000 investment due to:
1. Lower valuation (P/E 28.1 vs 42.3)
2. Positive market sentiment vs. Tesla's mixed sentiment
3. More stable growth profile
However, if you have high risk tolerance, Tesla offers potential for higher returns.
```

**Expected Output A**: Vague or based on general knowledge (no current data)

**Expected Output B**: Specific, data-driven, with clear reasoning steps

**Learning**: ReAct enables the model to gather specific information and base reasoning on real data rather than training data knowledge.

---

### Experiment 2: ReAct with Error Recovery

**Objective**: Test how ReAct handles tool failures and recovers

**Hypothesis**: ReAct can identify when a tool fails and try alternative approaches

**Setup**:

**Scenario**:

```
Question: What's the current exchange rate of USD to JPY?

Available tools:
- currency_api(from, to) - sometimes fails if rate not cached
- search_current_rates(currency_pair)
- fallback_historical_rate(from, to)

Expected ReAct flow:

Thought: I need current USD to JPY rate

Action: currency_api("USD", "JPY")
Observation: [ERROR: Rate data unavailable]

Thought: The API failed. Let me try the search tool

Action: search_current_rates("USD-JPY")
Observation: USD to JPY: 149.23 (as of May 2026)

Thought: I have the answer
Final Answer: The current USD to JPY exchange rate is approximately 149.23
```

**Learning**: ReAct handles tool failures gracefully by trying alternatives. This is crucial for real-world agent systems.

---

### Experiment 3: ReAct for Debugging

**Objective**: Demonstrate ReAct solving a code problem through iterative reasoning and action

**Hypothesis**: ReAct with code execution enables effective debugging

**Setup**:

**Scenario**:

````
Question: Why does this Python code fail? Fix it.

Code:
```python
def calculate_average(numbers):
    total = sum(numbers)
    return total / len(numbers)

result = calculate_average([])
print(result)
````

Available tools:

- execute_code(code)
- search_documentation(topic)

ReAct flow:

Thought: I should run the code to see what error occurs

Action: execute_code("""
def calculate_average(numbers):
total = sum(numbers)
return total / len(numbers)

result = calculate_average([])
print(result)
""")

Observation: [ERROR: ZeroDivisionError: division by zero]

Thought: The error is clear: dividing by 0 when the list is empty. I need to handle the empty list case.

Action: execute_code("""
def calculate_average(numbers):
if len(numbers) == 0:
return 0 # or raise an error, depending on intent
total = sum(numbers)
return total / len(numbers)

result = calculate_average([])
print(result)
""")

Observation: Output: 0

Thought: This works. Let me verify with a non-empty list too.

Action: execute_code("""
def calculate_average(numbers):
if len(numbers) == 0:
return 0
total = sum(numbers)
return total / len(numbers)

print(calculate_average([])) # Test empty: 0
print(calculate_average([1,2,3])) # Test normal: 2.0
""")

Observation: Output: 0 \n 2.0

Final Answer: The issue was division by zero. Fixed by checking if the list is empty before dividing.
Updated code handles both empty and non-empty cases.

```

**Learning**: ReAct enables interactive debugging—think, execute, observe error, think again, fix. More effective than static code review.

---

## Part 5: Comparative Analysis

### Decision Matrix: Which Technique to Use?

| Task Type | Zero-Shot | Few-Shot | CoT | ReAct |
|-----------|-----------|----------|-----|-------|
| Simple classification | ✅ Fast | ⚠️ Overkill | ❌ Verbose | ❌ Overkill |
| Novel domain task | ❌ Likely fails | ✅ Teach by example | ⚠️ Helps with reasoning | ⚠️ If tools available |
| Math problem | ❌ Often wrong | ⚠️ Some help | ✅ Excellent | ✅ Can verify |
| Logic puzzle | ❌ Guesses | ⚠️ Better | ✅ Shows reasoning | ✅ Verify with actions |
| Current information | ❌ Stale data | ❌ Stale data | ❌ Stale data | ✅ Fetch live data |
| Format specification | ❌ Generic | ✅ Teach format | ⚠️ May work | ⚠️ If tools format |
| Multi-step process | ⚠️ Vague | ⚠️ Examples help | ✅ Show all steps | ✅ Execute steps |

### Combination Patterns

**CoT + Few-Shot**: Provide examples WITH reasoning
- Most powerful for complex domain tasks
- Examples teach both answer format and reasoning style

**ReAct + CoT**: Let model reason, then act, then reason about results
- Enables sophisticated agent behavior
- Real-world problem solving

**Few-Shot + Tool Use**: Teach with examples, then let model use tools
- Combines pattern teaching with information gathering

---

## Part 6: Implementation Checklist

### Applying These Techniques

**For Zero-Shot**:
- [ ] Task is well-defined and model likely has seen similar patterns
- [ ] Clear instruction with role, context, format spec
- [ ] Constraints and output format specified
- [ ] Test with 2-3 examples to validate approach

**For Few-Shot**:
- [ ] Zero-shot is insufficient or inconsistent
- [ ] You have 2-3 clean examples of input + output
- [ ] Examples cover the range of expected inputs
- [ ] Examples are formatted exactly as desired output
- [ ] More than 5 examples added (increases noise)

**For CoT**:
- [ ] Task involves reasoning, math, or logic
- [ ] You want to see the model's thinking process
- [ ] Use phrases like "Let's think step by step" or "Show your work"
- [ ] Each step is numbered and explicit
- [ ] Model is failing at simple level, CoT forces deeper processing

**For ReAct**:
- [ ] Task requires current information or tool use
- [ ] You have tools/APIs available to call
- [ ] Problem is multi-step, benefit from iterative approach
- [ ] You want to handle tool failures gracefully
- [ ] Latency acceptable (actions add round-trip time)

---

## Part 7: Practical Examples by Domain

### Software Development

**Task**: Debug a failing test

**Recommended Technique**: ReAct + CoT
```

Question: Why is this test failing?

Thought: Let me first see what the test does and what error it produces

Action: execute_test("test_user_creation.py")
Observation: [FAILED] AssertionError: Expected user.id to be UUID, got None

Thought: The user ID is None. Let me look at the User creation code

Action: read_code("models/user.py", "def **init**")
Observation: [User model initialization code - no UUID assignment]

Thought: The issue is that the UUID isn't being assigned on creation. Let me check the factory

Action: read_code("tests/factories.py", "UserFactory")
Observation: [Factory code - also not assigning UUID]

Final Answer: The UserFactory doesn't assign a UUID. Need to update factory or User.**init** to generate UUID on creation.

```

### Data Analysis

**Task**: Summarize dataset trends

**Recommended Technique**: ReAct + Few-Shot
```

You have access to: query_database, calculate_statistics, plot_data

Few-Shot Example:
"Here are 10 records showing user signup growth.
The trend shows 15% month-over-month growth."

Action: query_database("SELECT DATE_TRUNC('month', created_at), COUNT(\*) FROM users GROUP BY 1 ORDER BY 1")
Action: calculate_statistics(results, metric='growth_rate')
Action: plot_data(results, type='trend')

Final Answer: Monthly user signups showing steady 12-18% growth, accelerating in recent quarters

```

### Customer Service

**Task**: Answer complex customer question with product/context knowledge

**Recommended Technique**: Few-Shot CoT + ReAct
```

Few-Shot Example: Shows how to think through common issues
CoT: "Let me think about what the customer is experiencing..."
ReAct: Search knowledge base, check customer account, verify product status

This combines:

- Pattern matching from few-shot
- Transparent reasoning from CoT
- Real information from ReAct

```

### Content Creation

**Task**: Write product description following brand guidelines

**Recommended Technique**: Few-Shot
```

Here are 3 example product descriptions in our brand voice:

1. [Example emphasizing sustainability]
2. [Example emphasizing innovation]
3. [Example emphasizing value]

Now write one for: [New Product]

```

---

## Part 8: Common Pitfalls and Fixes

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| **Too vague prompt** | Output misses what you wanted | Add role, context, format spec (zero-shot principles) |
| **Examples not diverse** | Fails on edge cases | Add more varied examples or CoT to handle exceptions |
| **CoT too verbose** | Token waste, slower | Use CoT only when it matters (reasoning tasks) |
| **CoT wrong format** | Model doesn't verbalize thinking | Use clear "Thought: ... Action: ..." separators |
| **Tool calls fail silently** | Model accepts bad data | Add error handling and verification steps in ReAct |
| **Too many few-shot examples** | Performance degrades | Reduce to 2-3 highest-quality examples |
| **ReAct infinite loop** | Agent never concludes | Set max action limit or require explicit "Final Answer" |
| **Mixing techniques badly** | Confusing to model | Pick one or deliberately combine compatible approaches |

---

## Part 9: Key Metrics and Validation

### How to Know If Your Prompt Is Working

**Metric 1: Consistency**
- Run same prompt 10 times (or retrieve cached results)
- Check: Do outputs have consistent format? Structure? Quality?
- Target: >90% consistency for production use

**Metric 2: Correctness**
- For tasks with clear right answer: What % of outputs are correct?
- For fuzzy tasks: Do outputs have correct reasoning?
- Target: >95% for production, >80% for discovery

**Metric 3: Efficiency**
- Tokens used per task
- Latency (especially for ReAct with multiple actions)
- Cost if using paid API
- Target: <1000 tokens for most tasks

**Metric 4: Signal Quality**
- For ReAct: Do observations inform next reasoning?
- For CoT: Does reasoning match actual thinking?
- For Few-Shot: Do examples teach consistent patterns?
- Target: Reasoning is coherent, builds on evidence

---

## Conclusion: The Hierarchy of Techniques

```

Complexity/Power/Cost →

Zero-Shot (simple, fast, cheap)
↓
Few-Shot (teach by example)
↓
Chain of Thought (expose reasoning)
↓
ReAct (add action and feedback loops)
↓
Multi-Agent ReAct (multiple reasoning agents)

```

**Start with the simplest that works**. Add complexity only when needed:
- Zero-shot fails? → Try few-shot
- Few-shot inconsistent? → Add CoT
- Need current data? → Use ReAct
- Still struggling? → Combine approaches or add tools

**The art is knowing which lever to pull at which moment.**

---

## References

- **Prompt Engineering Guide**: https://www.promptingguide.ai
- **Chain of Thought**: Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (2022)
- **ReAct**: Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2023)
- **Few-Shot Learning**: Brown et al., "Language Models are Few-Shot Learners" (2020)
- **OpenAI Cookbook**: https://github.com/openai/openai-cookbook
- **Anthropic Prompt Engineering**: https://docs.anthropic.com/prompt-engineering/overview

---

## Appendix: Quick Reference Card

### When to Use Each

| Situation | Use |
|-----------|-----|
| "Model doesn't understand the task" | Few-Shot or Zero-Shot with role-setting |
| "Model gets wrong answer on math" | CoT ("Show your work") |
| "I need current/external information" | ReAct with search/database tools |
| "Output format is wrong/inconsistent" | Few-Shot examples in exact format |
| "Model reasoning is wrong" | CoT to expose thinking |
| "Task is novel/unusual" | Few-Shot examples of what you want |
| "Simple classification task" | Zero-Shot |
| "Debugging or tool use needed" | ReAct |

**Remember**: Good prompts ≠ tricking the model. Good prompts = clearly explaining what you want and how to think about it.
```
