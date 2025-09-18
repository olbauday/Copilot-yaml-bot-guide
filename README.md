# A Practical Guide to Writing Copilot Studio Adaptive Dialog YAML (Updated)

These .yml files are the blueprint for your bot's conversations. They use YAML to define the structure of the conversation and Power Fx to handle the logic.

Think of it like building with LEGOs 🧱:

YAML is the set of instructions describing which bricks to use and how they stack on top of each other.

Power Fx is like the programmable motor inside a LEGO Technic piece that makes things happen based on rules and inputs.

## Part 1: The Fundamentals - YAML Syntax

### Rule 1: Indentation is Everything (The Golden Rule)
This is the most important rule in YAML and the source of most errors. Indentation defines the structure and relationships between elements.

- Always use spaces, never tabs. The standard is 2 spaces per level of indentation.
- Items at the same level of indentation are "siblings" (equal partners).
- An item indented under another is its "child" (a property of the parent).

```yaml
# CORRECT ✅
- kind: Question
  id: question_captureInitialIdea
  variable: Topic.InitialIdea
  entity: StringPrebuiltEntity
```

### Rule 2: The Core Building Blocks: Key-Value Pairs
Everything in YAML is a pair of a key and a value, separated by a colon and a space (key: value). The `kind` key is special, as it tells the platform what kind of component you are defining.

```yaml
kind: SendActivity
id: sendActivity_Welcome
activity: "Welcome to the bot!"
```

### Rule 3: Lists of Actions
Your bot's conversation is a list of actions that happen in order. In YAML, a list item starts with a hyphen and a space (`- `). The `actions:` key always introduces a list of these action blocks.

```yaml
# Correct structure for Copilot Studio
kind: AdaptiveDialog
beginDialog:
  kind: OnUnknownIntent
  id: main
  actions:
    # Action 1
    - kind: SendActivity
      id: sendActivity_Welcome
      activity: "Welcome!"

    # Action 2
    - kind: Question
      id: question_askName
      variable: Topic.UserName
      entity: StringPrebuiltEntity
      prompt: "What is your name?"
```

## Part 2: The Brains - Variables and Logic

### Rule 4: The Bot's Memory: Variables 🧠
Variables are how your bot remembers information. Choosing the right type is crucial.

| Variable Type | Scope | Use Case |
|---------------|-------|----------|
| Topic.VariableName | Current Topic Only | Short-term memory for a single conversational task. It's forgotten once the topic ends. |
| Global.VariableName | Entire User Session | Long-term memory. Use this to pass information between different topics. |
| Dialog.VariableName | Internal Processing | Temporary memory for a single turn or a looping dialog. Not meant for passing data between major topics. |

**IMPORTANT: Variable Initialization**
Copilot Studio requires you to initialize ALL variables before using them in Question actions. **The `init:` prefix is NOT supported and will cause errors.**

```yaml
# CORRECT ✅ - Initialize first with SetVariable, then use in Question
- kind: SetVariable
  id: setVariable_initAgentIdea
  variable: Topic.AgentIdea
  value: ""

- kind: Question
  id: question_captureAgentIdea
  variable: Topic.AgentIdea       # No init: prefix!
  prompt: "What's your agent idea?"
  entity: StringPrebuiltEntity

# WRONG ❌ - Using init: prefix causes "InvalidPropertyPath" errors
- kind: Question
  id: question_captureAgentIdea
  variable: init:Topic.AgentIdea  # This will break!
  prompt: "What's your agent idea?"
  entity: StringPrebuiltEntity
```

**Variable Rules:**
- **Never use `init:` prefix** - This causes InvalidPropertyPath errors
- **Always initialize with SetVariable first** - Create the variable with an empty value
- **Then use the variable name directly** - No prefixes needed in Questions or other actions

### Rule 5: Question Actions Structure
**CRITICAL**: Every Question action MUST have three required properties:

1. `variable:` - Where to store the response (NOT `property:`)
2. `entity:` - What type of input to expect  
3. `prompt:` - What to ask the user

```yaml
# CORRECT ✅
- kind: Question
  id: question_getName
  variable: Topic.UserName        # Required: where to store response
  entity: StringPrebuiltEntity    # Required: input type
  prompt: "What's your name?"     # Required: what to ask

# WRONG ❌ - Missing entity or using wrong capitalization
- kind: Question
  id: question_getName
  variable: Topic.UserName
  Entity: StringPrebuiltEntity    # Wrong! Should be lowercase 'entity:'
  prompt: "What's your name?"
```

**IMPORTANT: Entity Property Capitalization**
The `entity:` property must be lowercase! Using `Entity:` (capital E) will cause "Unknown element" errors.

**Common Entity Types:**
- `StringPrebuiltEntity` - For text responses (most common)
- `BooleanPrebuiltEntity` - For yes/no responses (returns true/false)
- `NumberPrebuiltEntity` - For numeric input
- `DateTimePrebuiltEntity` - For dates and times
- `EmailPrebuiltEntity` - For email addresses
- `URLPrebuiltEntity` - For web URLs
- `PhoneNumberPrebuiltEntity` - For phone numbers

**Complete List of Valid Entities from Copilot Studio:**
```
StringPrebuiltEntity, BooleanPrebuiltEntity, NumberPrebuiltEntity, 
DateTimePrebuiltEntity, EmailPrebuiltEntity, URLPrebuiltEntity,
PhoneNumberPrebuiltEntity, PersonNamePrebuiltEntity, 
OrganizationPrebuiltEntity, CityPrebuiltEntity, 
CountryOrRegionPrebuiltEntity, StatePrebuiltEntity,
StreetAddressPrebuiltEntity, ZipCodePrebuiltEntity,
MoneyPrebuiltEntity, PercentagePrebuiltEntity, 
TemperaturePrebuiltEntity, WeightPrebuiltEntity,
SpeedPrebuiltEntity, AgePrebuiltEntity, ColorPrebuiltEntity,
ContinentPrebuiltEntity, DatePrebuiltEntity,
DateTimeNoTimeZonePrebuiltEntity, DurationPrebuiltEntity,
EventPrebuiltEntity, FilePrebuiltEntity, LanguagePrebuiltEntity,
OrdinalPrebuiltEntity, PointOfInterestPrebuiltEntity
```

### Rule 6: The Logic Engine: Power Fx Formulas
Whenever you see `value: =`, the text that follows is a Power Fx formula.

**Text Concatenation**: Use `Concatenate()` function, NOT the `&` operator.

```yaml
# CORRECT ✅
value: =Concatenate("Hello, ", Topic.UserName, "!")

# WRONG ❌ - & operator not supported in Copilot Studio
value: ="Hello, " & Topic.UserName & "!"
```

**Line Breaks**: Use `Char(10)` for line breaks in Power Fx formulas.

```yaml
# CORRECT ✅
value: =Concatenate("Line 1", Char(10), "Line 2")

# WRONG ❌
value: ="Line 1\nLine 2"
```

**Displaying Variables in Messages**: Use curly braces (`{}`) in prompt and SendActivity blocks.

```yaml
prompt: "Hello, {Topic.UserName}!"
```

**Multi-line Text**: For long prompts, use the pipe symbol (`|`) and a hyphen (`-`).

```yaml
prompt: |-
  Here is your summary:
  **Name:** {Topic.AgentName}
  **Status:** {Topic.AgentStatus}
```

## Part 3: Troubleshooting Common Errors (Updated)

### Error 1: "Unknown element at path Triggers[0]" 
**Root Cause**: Using incorrect dialog structure.

**The Correct Structure for Copilot Studio:**
```yaml
# CORRECT ✅ - Using beginDialog structure (what actually works)
kind: AdaptiveDialog
beginDialog:
  kind: OnUnknownIntent
  id: main
  actions:
    # ... your actions here
```

**Valid Trigger Kinds:**
- `OnUnknownIntent` - Most reliable for general-purpose bots that respond to any user input
- `OnRecognizedIntent` - When you have specific trigger phrases with `triggerQueries`
- `OnConversationStart` - For initial greetings (may not work reliably in all contexts)

**Working Example with OnRecognizedIntent:**
```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnRecognizedIntent
  id: main
  intent:
    displayName: Agent Creator
    triggerQueries:
      - Create an agent
      - New agent
      - I want to create an agent
  actions:
    - kind: SendActivity
      id: sendActivity_Welcome
      activity: "Welcome to the Agent Creator!"
```

### Error 2: "Unknown element at path BeginDialog.Actions.Actions[X].Entity"
**Root Cause**: Incorrect capitalization or invalid entity type.

**The Most Common Cause - Wrong Capitalization:**
```yaml
# WRONG ❌ - Capital 'E' in Entity
- kind: Question
  id: question_getName
  variable: Topic.UserName
  Entity: StringPrebuiltEntity    # Should be lowercase!
  prompt: "What's your name?"

# CORRECT ✅ - Lowercase 'entity'
- kind: Question
  id: question_getName
  variable: Topic.UserName
  entity: StringPrebuiltEntity    # Correct lowercase
  prompt: "What's your name?"
```

**Valid Entity Types:**
Make sure you're using exactly one of these supported entity types:
- `StringPrebuiltEntity`, `BooleanPrebuiltEntity`, `NumberPrebuiltEntity`
- `EmailPrebuiltEntity`, `URLPrebuiltEntity`, `PhoneNumberPrebuiltEntity`
- `DateTimePrebuiltEntity`, `MoneyPrebuiltEntity`, `PercentagePrebuiltEntity`
- And many others (see Rule 5 for complete list)

### Error 3: "SystemError" or Re-prompting Loops with BooleanPrebuiltEntity
**Root Cause**: The BooleanPrebuiltEntity is extremely brittle. It expects a clear "Yes" or "No" response. Natural language from the user, such as "yes, please continue" or "it will be connected to a sharepoint list", will fail validation, causing the bot to get stuck and repeat the question.

**CRITICAL**: Avoid BooleanPrebuiltEntity for most conversational flows. It is almost always better to use a StringPrebuiltEntity and a ConditionGroup to check for positive keywords. This makes your bot far more robust.

```yaml
# AVOID ❌ - Fails with natural language, causes re-prompting loops
- kind: Question
  variable: Topic.Confirmation
  entity: BooleanPrebuiltEntity
  prompt: "Does this agent use data sources?"

# PREFER ✅ - Handles flexible user input
- kind: Question
  variable: Topic.ConfirmationText
  entity: StringPrebuiltEntity
  prompt: "Does this agent use any data sources?"

- kind: ConditionGroup
  conditions:
    - id: condition_IsPositive
      condition: =Or(Find("yes", Lower(Topic.ConfirmationText)) > 0, Find("sharepoint", Lower(Topic.ConfirmationText)) > 0)
      actions:
        # ... handle the "yes" case ...
```

### Error 4: "UnexpectedToken near line X" - YAML Formatting
**Root Cause**: YAML formatting issues - missing quotes, wrong indentation, or malformed strings.

**Common Fixes:**
- Ensure all string values with special characters are quoted
- Check indentation is exactly 2 spaces per level  
- Verify all list items start with `- ` (hyphen + space)
- Use `|` or `|-` for multi-line strings instead of complex concatenation

```yaml
# WRONG ❌ - Unquoted strings with special characters
activity: Welcome! Let's create your agent: it'll be great

# CORRECT ✅ - Quoted strings
activity: "Welcome! Let's create your agent: it'll be great"

# WRONG ❌ - Bad indentation
actions:
- kind: SendActivity
  id: test

# CORRECT ✅ - Proper 2-space indentation
actions:
  - kind: SendActivity
    id: test
```

### Error 8: "Response variable is missing" on a Question
**Root Cause**: ConditionGroup blocks without explicit defaultActions can silently fail when conditions aren't met.

**Solution**: Always provide explicit paths for both true and false cases:

```yaml
# RISKY ❌ - Missing defaultActions can cause silent failures
- kind: ConditionGroup
  id: if_UserWantsFeature
  conditions:
    - id: condition_CheckFeature
      condition: =Topic.UserInput = "yes"
      actions:
        # ... handle yes case ...
  # No defaultActions! What happens if condition is false?

# ROBUST ✅ - Explicit handling for all cases
- kind: ConditionGroup
  id: if_UserWantsFeature
  conditions:
    - id: condition_CheckFeature
      condition: =Topic.UserInput = "yes"
      actions:
        # ... handle yes case ...
  defaultActions:
    # ... explicitly handle no/other cases ...
```
**Root Cause**: Using unsupported variable scopes or mixing scopes incorrectly.

**Solutions:**
```yaml
# AVOID ❌ - Dialog scope often causes "isn't recognized" errors
variable: Dialog.SourceName

# PREFER ✅ - Topic scope for most use cases
variable: Topic.SourceName

# USE SPARINGLY ✅ - Global only when you need persistence across topics
variable: Global.UserPreferences
```

**Scope Guidelines:**
- **`Topic.`** - Use for 95% of variables (current conversation)
- **`Global.`** - Only when you need data to persist across different topics
- **`Dialog.`** - Avoid - often causes recognition errors
**Root Cause**: Missing `entity` property on Question actions.

**Solution**: Always include the `entity` property to define what type of input you expect.

```yaml
# CORRECT ✅
- kind: Question
  id: question_GetInput
  variable: Topic.UserInput
  entity: StringPrebuiltEntity  # This is required!
  prompt: "Please enter your response:"
```

### Error 2: "Unknown element at path Triggers[0]" 
**Root Cause**: Using incorrect dialog structure or wrong trigger kinds.

**Two Common Causes:**

**A) Wrong Root Structure:**
```yaml
# WRONG ❌ - Using beginDialog (from older versions)
kind: AdaptiveDialog
beginDialog:
  kind: OnConversationStart
  id: main

# CORRECT ✅ - Using triggers array
kind: AdaptiveDialog
triggers:
  - kind: OnConversationStart
    id: main
    actions:
      # ... your actions here
```

**B) Wrong Trigger Kind:**
- `OnConversationStart`: Use this only for the main topic that kicks off a conversation with the user
- `OnUnknownIntent`: Use this for fallback/default responses  
- `OnDialogStart`: Use this only for a sub-topic called from another topic via BeginDialog

**Valid Trigger Kinds in Copilot Studio:**
- `OnConversationStart` - When conversation begins
- `OnUnknownIntent` - When user input doesn't match any intent
- `OnActivity` - For specific activity types
- `OnError` - Error handling
- `OnSignIn` - Authentication events
- `OnSystemIntent` - System-generated intents
- `OnEscalate` - Escalation to human agents

**Complete Correct Structure:**
```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnUnknownIntent
  id: main
  actions:
    - kind: SendActivity
      id: sendActivity_Welcome
      activity: "Hello! Welcome to our bot."
    
    - kind: Question
      id: question_GetName
      variable: Topic.UserName
      entity: StringPrebuiltEntity
      prompt: "What's your name?"
```

### Error 3: Conditional Logic Requirements
**CRITICAL**: Every condition in a `ConditionGroup` MUST have an `id` property.

```yaml
# CORRECT ✅
- kind: ConditionGroup
  id: if_IsNewIdea
  conditions:
    - id: condition_IsIdea              # Required id!
      condition: =Topic.AgentStatus = "This is just an idea"
      actions:
        # ... actions if true
  defaultActions:
    # ... actions if false or no conditions match

# WRONG ❌ - Missing condition id
- kind: ConditionGroup
  id: if_IsNewIdea
  conditions:
    - condition: =Topic.AgentStatus = "This is just an idea"  # Missing id!
      actions:
        # ... actions if true
```

### Error 4: Power Fx Function Compatibility
**Not all Power Fx functions work in Copilot Studio**. Here are the key differences:

**Text Operations:**
```yaml
# CORRECT ✅ - Supported functions
=Concatenate(text1, text2, text3)
=IsBlank(variable)
=Char(10)  # for line breaks

# WRONG ❌ - Not supported in Copilot Studio
=text1 & text2 & text3    # Use Concatenate() instead
=IsEmpty(variable)         # Use IsBlank() instead
="\n"                      # Use Char(10) instead
```

**Table Operations:**
```yaml
# AVOID ❌ - Complex table operations often cause issues
=Patch(Global.Sources, Defaults(Global.Sources), {Name: value})
=Concat(Global.Sources, "**- Name:** " & ThisRecord.Name)

# PREFER ✅ - Simple string concatenation
=Concatenate(Global.ExistingData, "New item: ", Topic.NewValue, Char(10))
```

## Part 4: Best Practices (New Section)

### Practice 1: Always Initialize Variables
Initialize ALL variables before using them in Question actions:

```yaml
# Initialize all variables at the start
- kind: SetVariable
  id: setVariable_initName
  variable: Topic.UserName
  value: ""

- kind: SetVariable
  id: setVariable_initEmail
  variable: Topic.UserEmail
  value: ""

# Then use them in questions
- kind: Question
  id: question_getName
  variable: Topic.UserName
  entity: StringPrebuiltEntity
  prompt: "What's your name?"
```

### Practice 2: Smart Entity Selection
Choose entity types based on real user behavior, not ideal responses:

```yaml
# AVOID ❌ - Rigid entity types that break with natural language
- kind: Question
  variable: Topic.Confirmation
  prompt: "Is this correct? (Yes/No)"
  entity: BooleanPrebuiltEntity  # Breaks if user elaborates

# PREFER ✅ - Flexible entities with smart parsing
- kind: Question
  variable: Topic.ConfirmationText
  prompt: "Is this correct? Please confirm or let me know what to change."
  entity: StringPrebuiltEntity

- kind: ConditionGroup
  conditions:
    - id: condition_IsPositive
      condition: =Or(Find("yes", Lower(Topic.ConfirmationText)) > 0, Find("correct", Lower(Topic.ConfirmationText)) > 0, Find("good", Lower(Topic.ConfirmationText)) > 0)
```

### Practice 3: Simple Prompt Design
Avoid complex dynamic prompts that can cause parsing errors:

```yaml
# AVOID ❌ - Complex dynamic prompts
prompt: =Concatenate("What is the URL for '", Dialog.SourceName, "'? Please include the full path.")

# PREFER ✅ - Simple static prompts
prompt: "What is the URL for this data source?"
```

### Practice 6: Incremental Testing Strategy
**The most common source of logical errors is trying to be too clever with conversation flow.** While features like GotoAction exist, they are unreliable for major jumps. The most stable and predictable structure is a simple "waterfall" that uses nested ConditionGroup blocks.

This often means you will have to repeat sections of your logic. While this makes the YAML file longer, it makes the bot's behavior predictable and easy to debug.

**Remember: A long, simple, predictable file is better than a short, clever, broken one.**

```yaml
# UNRELIABLE ❌ - Trying to jump to a shared section
actions:
  - kind: ConditionGroup
    conditions:
      - id: condition_isPathA
        actions:
          # ... do Path A logic ...
          - kind: GotoAction
            actionId: data_source_logic  # RISKY JUMP
    defaultActions:
      # ... do Path B logic ...
      - kind: GotoAction
        actionId: data_source_logic      # RISKY JUMP

# Shared logic that is hard to jump to reliably
- kind: Question
  id: data_source_logic
  # ...

# ROBUST ✅ - Repeating the logic inside each path
actions:
  - kind: ConditionGroup
    conditions:
      - id: condition_isPathA
        actions:
          # ... do Path A logic ...
          # --- COPIED DATA SOURCE LOGIC ---
          - kind: Question
            id: data_source_logic_PathA
            # ... (duplicated but reliable)
    defaultActions:
      # ... do Path B logic ...
      # --- COPIED DATA SOURCE LOGIC ---
      - kind: Question
        id: data_source_logic_PathB
        # ... (duplicated but reliable)
```

**Key Principles:**
- **Simple and repetitive is better than complex and clever**
- **Nest logic within ConditionGroups rather than jumping between sections**
- **Duplicate code blocks rather than trying to share them via GotoAction**
- **Keep related logic together in the same conditional block**
Avoid complex table operations. Use simple string concatenation for lists:

```yaml
# Instead of complex table operations, use simple strings
- kind: SetVariable
  id: setVariable_addToList
  variable: Global.ItemList
  value: =Concatenate(Global.ItemList, "- ", Topic.NewItem, Char(10))
```

### Practice 3: Consistent Naming Convention
Use descriptive, consistent IDs:

```yaml
# Good naming patterns
- kind: Question
  id: question_GetUserName          # Clear purpose
  
- kind: SetVariable
  id: setVariable_InitUserData      # Clear action

- kind: ConditionGroup
  id: if_UserIsRegistered           # Clear condition
  conditions:
    - id: condition_IsRegistered    # Clear condition name
```

### Practice 4: Test Incrementally
Build your bot step by step:
1. Start with basic welcome message
2. Add one question at a time
3. Test each addition before continuing
4. Add conditional logic last

This approach helps identify exactly where errors occur.

## Part 6: Supported vs Unsupported Features

### AnswerQuestionWithAI Action ⚠️ LIMITED USE CASES
This action uses AI to generate responses. **It is best used for creative or conversational tasks, NOT for reliable data extraction.**

**CRITICAL WARNING**: Do not use this action to generate structured data that you intend to parse into variables. The AI's output format is not guaranteed, even with very strict prompts. It will often include conversational text that breaks parsing logic (e.g., using Split()).

**GOOD USE CASES ✅:**
- Summarizing a topic in a friendly paragraph
- Brainstorming ideas for a user  
- Rephrasing text into a different tone

**BAD USE CASES ❌:**
- Trying to make it output JSON
- Expecting a strict KEY: VALUE format to populate variables
- Any structured data that needs to be parsed

```yaml
# GOOD ✅ - Conversational content generation
- kind: AnswerQuestionWithAI
  id: ai_SummarizeIdea
  variable: Topic.FriendlySummary
  userInput: =Topic.UserIdea
  question: "Summarize this idea in a friendly, encouraging paragraph"

# BAD ❌ - Trying to generate structured data
- kind: AnswerQuestionWithAI
  id: ai_GenerateStructured
  variable: Topic.ParsedData
  userInput: =Topic.UserIdea
  question: "Generate this in format: NAME: [name]\nSUMMARY: [summary]"
  # The AI will often add extra text that breaks parsing!
```

**Alternative**: Instead of using AI to generate structured summaries, ask the user specific questions and build the summary yourself using Concatenate().

### GotoAction for Flow Control ⚠️ UNRELIABLE
While GotoAction exists to create loops or jump between actions, **it is unreliable and not recommended for complex flows.** In testing, it causes silent failures and unexpected crashes. The property name (actionId vs. targetActionId) is also inconsistent.

**Limited Acceptable Use**: Simple, self-contained loops where you jump back to a recent question.

**Better Approach**: Prefer nesting logic inside ConditionGroup blocks, even if it means repeating code.

```yaml
# USE WITH CAUTION ✅ - Simple loop back to recent step
- kind: Question
  id: question_GetSourceName
  # ... ask for source name ...
  
- kind: Question
  id: question_AddAnotherSource
  # ... ask to add another ...
  
- kind: ConditionGroup
  conditions:
    - id: condition_WantsToAddAnother
      condition: =Topic.AddAnotherSourceResponse = true
      actions:
        # This jump backwards to a recent step is usually safe
        - kind: GotoAction
          actionId: question_GetSourceName

# AVOID ❌ - Jumping across major logical blocks
- kind: ConditionGroup
  conditions:
    - id: condition_isPathA
      actions:
        # ... logic for Path A ...
        - kind: GotoAction  # This is UNRELIABLE!
          actionId: some_action_far_down_the_file
```

### BooleanPrebuiltEntity for Yes/No Questions
For true/false responses, use BooleanPrebuiltEntity:

```yaml
- kind: Question
  id: question_confirmDraft
  variable: Topic.IdeaApproved
  prompt: "Does this look right to you? (Please answer with Yes or No)"
  entity: BooleanPrebuiltEntity  # Returns true/false
```

### EndDialog Action
Use EndDialog to cleanly end the conversation or topic:

```yaml
- kind: EndDialog
  id: endDialog_thisTopic
```

### ✅ Supported Action Kinds
- `SendActivity` - Send messages
- `Question` - Ask for user input
- `SetVariable` - Store values
- `ConditionGroup` - If/else logic
- `BeginDialog` - Call other topics
- `EndDialog` - End conversation

### ❌ Commonly Attempted But Unsupported
- `DoWhile` loops - Not supported
- `BreakLoop` - Only works in supported loop contexts  
- `IfCondition` - Use `ConditionGroup` instead
- **`init:` prefix in variable names** - Causes InvalidPropertyPath errors
- **Complex GotoAction flows** - Unreliable for major jumps, causes silent failures
- **AnswerQuestionWithAI for structured data** - Output format not guaranteed, breaks parsing
- Complex Power Fx table operations
- `Dialog.` scope variables - Often cause recognition errors

### ⚠️ Use With Extreme Caution
- **GotoAction** - Only for simple loops back to recent questions
- **BooleanPrebuiltEntity** - Extremely brittle with natural language input
- **AnswerQuestionWithAI** - Only for conversational content, not data extraction

Remember: Copilot Studio is more restrictive than Power Platform flows, so simpler approaches often work better!
