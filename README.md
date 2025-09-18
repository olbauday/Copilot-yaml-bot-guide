# A Practical Guide to Writing Copilot Studio Adaptive Dialog YAML (Updated September 2024)

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
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.InitialIdea
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
      interruptionPolicy:
        allowInterruption: true
      variable: init:Global.UserName
      entity: StringPrebuiltEntity
      prompt: "What is your name?"
```

## Part 2: The Brains - Variables and Logic

### Rule 4: The Bot's Memory: Variables 🧠
Variables are how your bot remembers information. Choosing the right type is crucial.

| Variable Type | Scope | Use Case |
|---------------|-------|----------|
| Global.VariableName | Entire User Session | Long-term memory. Use this to pass information between different topics. |
| Topic.VariableName | Current Topic Only | Short-term memory for a single conversational task. It's forgotten once the topic ends. |
| Dialog.VariableName | Internal Processing | Temporary memory for a single turn or a looping dialog. Not meant for passing data between major topics. |

**CRITICAL: The `init:` Prefix Rule**
In Copilot Studio, **ALL variables used in Question actions MUST have the `init:` prefix**. This is mandatory and cannot be substituted with SetVariable initialization.

```yaml
# CORRECT ✅ - Using init: prefix in Questions (REQUIRED)
- kind: Question
  id: question_captureAgentIdea
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.AgentIdea
  prompt: "What's your agent idea?"
  entity: StringPrebuiltEntity

# WRONG ❌ - Missing init: prefix causes questions to be silently skipped
- kind: Question
  id: question_captureAgentIdea
  interruptionPolicy:
    allowInterruption: true
  variable: Global.AgentIdea          # Missing init: = SKIPPED QUESTION!
  prompt: "What's your agent idea?"
  entity: StringPrebuiltEntity

# ALSO WRONG ❌ - SetVariable cannot substitute for init: in Questions
- kind: SetVariable
  id: setVariable_initAgentIdea
  variable: Global.AgentIdea
  value: ""

- kind: Question
  id: question_captureAgentIdea
  interruptionPolicy:
    allowInterruption: true
  variable: Global.AgentIdea          # Still missing init: = STILL SKIPPED!
  prompt: "What's your agent idea?"
  entity: StringPrebuiltEntity
```

**Variable Rules:**
- **Always use `init:` prefix in Question actions** - This is mandatory for questions to work
- **Both Global and Topic scopes work** - `init:Global.Name` and `init:Topic.Name` are both valid
- **SetVariable cannot substitute for init:** - Even pre-initialized variables need the init: prefix in Questions
- **Use SetVariable for non-Question logic** - For calculations, concatenations, and conditional assignments

### Rule 5: Question Actions Structure - Complete Requirements
**CRITICAL**: Every Question action MUST have four required properties:

1. `interruptionPolicy:` with `allowInterruption: true` - Without this, questions are skipped
2. `variable:` with `init:` prefix - Where to store the response
3. `entity:` - What type of input to expect (lowercase!)
4. `prompt:` - What to ask the user

```yaml
# CORRECT ✅ - Complete Question template
- kind: Question
  id: question_getName
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.UserName      # Required: init: prefix
  entity: StringPrebuiltEntity        # Required: lowercase 'entity:'
  prompt: "What's your name?"         # Required: what to ask

# WRONG ❌ - Missing interruptionPolicy (question gets skipped)
- kind: Question
  id: question_getName
  variable: init:Global.UserName
  entity: StringPrebuiltEntity
  prompt: "What's your name?"

# WRONG ❌ - Wrong capitalization of entity
- kind: Question
  id: question_getName
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.UserName
  Entity: StringPrebuiltEntity        # Wrong! Should be lowercase 'entity:'
  prompt: "What's your name?"
```

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
value: =Concatenate("Hello, ", Global.UserName, "!")

# WRONG ❌ - & operator not supported in Copilot Studio
value: ="Hello, " & Global.UserName & "!"
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
prompt: "Hello, {Global.UserName}!"
```

**Multi-line Text**: For long prompts, use the pipe symbol (`|`) and a hyphen (`-`).

```yaml
prompt: |-
  Here is your summary:
  **Name:** {Global.AgentName}
  **Status:** {Global.AgentStatus}
```

## Part 3: Troubleshooting Common Errors (Updated with Current Knowledge)

### Error 1: Questions Being Silently Skipped (Most Common Issue)
**Root Cause**: Missing required properties in Question actions.

**The Complete Fix - Use This Exact Template:**
```yaml
# REQUIRED TEMPLATE ✅ - Use this for ALL Questions
- kind: Question
  id: question_DescriptiveName
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.VariableName    # or init:Topic.VariableName
  prompt: "Your question text here"
  entity: StringPrebuiltEntity
```

**Common Causes of Skipped Questions:**
- Missing `interruptionPolicy` block
- Missing `init:` prefix in variable
- Wrong capitalization of `entity` (must be lowercase)
- Using SetVariable instead of `init:` prefix

### Error 2: "Unknown element at path Triggers[0]" 
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
- `OnConversationStart` - For initial greetings (may not work reliably in all contexts)
- `OnRecognizedIntent` - When you have specific trigger phrases with `triggerQueries`

### Error 3: "Unknown element at path BeginDialog.Actions.Actions[X].Entity"
**Root Cause**: Incorrect capitalization or invalid entity type.

**The Most Common Cause - Wrong Capitalization:**
```yaml
# WRONG ❌ - Capital 'E' in Entity
- kind: Question
  id: question_getName
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.UserName
  Entity: StringPrebuiltEntity        # Should be lowercase!
  prompt: "What's your name?"

# CORRECT ✅ - Lowercase 'entity'
- kind: Question
  id: question_getName
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.UserName
  entity: StringPrebuiltEntity        # Correct lowercase
  prompt: "What's your name?"
```

### Error 4: BooleanPrebuiltEntity Issues (Updated Understanding)
**Previous Issue**: BooleanPrebuiltEntity was thought to be unreliable with natural language.

**Current Reality**: With correct syntax (init: prefix and interruptionPolicy), BooleanPrebuiltEntity works reliably for clear yes/no questions.

```yaml
# WORKS RELIABLY ✅ - With correct syntax
- kind: Question
  id: question_Confirmation
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.IsConfirmed
  prompt: "Do you want to continue? (Please answer Yes or No)"
  entity: BooleanPrebuiltEntity

# For more flexible responses, still prefer StringPrebuiltEntity with conditions
- kind: Question
  id: question_FlexibleConfirmation
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.ConfirmationText
  prompt: "Do you want to continue? Please let me know."
  entity: StringPrebuiltEntity

- kind: ConditionGroup
  conditions:
    - id: condition_IsPositive
      condition: =Or(Find("yes", Lower(Global.ConfirmationText)) > 0, Find("continue", Lower(Global.ConfirmationText)) > 0, Find("proceed", Lower(Global.ConfirmationText)) > 0)
```

### Error 5: "UnexpectedToken near line X" - YAML Formatting
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

### Error 6: Power Fx Display Issues
**Root Cause**: Complex Power Fx formulas in SendActivity not displaying properly.

**Solution**: Use `{VariableName}` syntax in activity text instead of complex formulas.

```yaml
# AVOID ❌ - Complex Power Fx in activities can display as raw formulas
activity: =Concatenate("Hello ", Global.UserName, ", your agent: ", Global.AgentName)

# PREFER ✅ - Variable substitution with curly braces
activity: "Hello {Global.UserName}, your agent: {Global.AgentName}"

# Or use SetVariable first for complex formulas
- kind: SetVariable
  id: setVariable_CreateMessage
  variable: Global.WelcomeMessage
  value: =Concatenate("Hello ", Global.UserName, ", your agent: ", Global.AgentName)

- kind: SendActivity
  id: sendActivity_ShowMessage
  activity: "{Global.WelcomeMessage}"
```

### Error 7: Conditional Logic Requirements
**CRITICAL**: Every condition in a `ConditionGroup` MUST have an `id` property.

```yaml
# CORRECT ✅
- kind: ConditionGroup
  id: if_IsNewIdea
  conditions:
    - id: condition_IsIdea              # Required id!
      condition: =Find("idea", Lower(Global.AgentStatus)) > 0
      actions:
        # ... actions if true
  defaultActions:
    # ... actions if false or no conditions match

# WRONG ❌ - Missing condition id
- kind: ConditionGroup
  id: if_IsNewIdea
  conditions:
    - condition: =Find("idea", Lower(Global.AgentStatus)) > 0  # Missing id!
      actions:
        # ... actions if true
```

## Part 4: Best Practices (Updated)

### Practice 1: Use the Complete Question Template
Always use this exact template for reliable Question actions:

```yaml
- kind: Question
  id: question_DescriptiveName
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.VariableName  # or init:Topic.VariableName
  prompt: "Clear, specific question text"
  entity: StringPrebuiltEntity        # Choose appropriate entity type
```

### Practice 2: Smart Entity Selection
Choose entity types based on real user behavior and question clarity:

```yaml
# For clear yes/no questions - BooleanPrebuiltEntity works well
- kind: Question
  id: question_SimpleConfirmation
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.IsConfirmed
  prompt: "Do you want to continue? (Please answer Yes or No)"
  entity: BooleanPrebuiltEntity

# For flexible responses - use StringPrebuiltEntity with smart parsing
- kind: Question
  id: question_FlexibleInput
  interruptionPolicy:
    allowInterruption: true
  variable: init:Global.UserResponse
  prompt: "Tell me about your requirements:"
  entity: StringPrebuiltEntity
```

### Practice 3: Simple and Predictable Flow Design
**The most common source of logical errors is trying to be too clever with conversation flow.** While features like GotoAction exist, they are unreliable for major jumps. The most stable and predictable structure is a simple "waterfall" that uses nested ConditionGroup blocks.

This often means you will have to repeat sections of your logic. While this makes the YAML file longer, it makes the bot's behavior predictable and easy to debug.

**Remember: A long, simple, predictable file is better than a short, clever, broken one.**

```yaml
# UNRELIABLE ❌ - Trying to jump to shared sections
- kind: ConditionGroup
  conditions:
    - id: condition_isPathA
      actions:
        # ... do Path A logic ...
        - kind: GotoAction
          actionId: shared_logic  # RISKY JUMP

# ROBUST ✅ - Duplicate logic within each path
- kind: ConditionGroup
  conditions:
    - id: condition_isPathA
      actions:
        # ... do Path A logic ...
        # --- DUPLICATED BUT RELIABLE LOGIC ---
        - kind: Question
          id: question_PathASpecific
          interruptionPolicy:
            allowInterruption: true
          variable: init:Global.PathAData
          prompt: "Path A specific question"
          entity: StringPrebuiltEntity
  defaultActions:
    # ... do Path B logic ...
    # --- DUPLICATED BUT RELIABLE LOGIC ---
    - kind: Question
      id: question_PathBSpecific
      interruptionPolicy:
        allowInterruption: true
      variable: init:Global.PathBData
      prompt: "Path B specific question"
      entity: StringPrebuiltEntity
```

### Practice 4: Incremental Testing Strategy
Build your bot step by step:
1. Start with basic welcome message and one question
2. Test each question individually before adding the next
3. Add conditional logic only after all questions work
4. Test each branch separately

This approach helps identify exactly where errors occur.

### Practice 5: Consistent Naming Convention
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

## Part 5: Supported vs Unsupported Features (Updated)

### ✅ Fully Supported and Reliable
- **SendActivity** - Send messages to users
- **Question** - Ask for user input (with correct syntax)
- **SetVariable** - Store and manipulate values
- **ConditionGroup** - If/else logic with proper structure
- **BeginDialog** - Call other topics
- **EndDialog** - End conversation cleanly
- **BooleanPrebuiltEntity** - Works reliably with correct syntax
- **StringPrebuiltEntity** - Most versatile for user input
- **NumberPrebuiltEntity** - Numeric input validation

### ⚠️ Use With Caution
- **GotoAction** - Only for simple loops back to recent questions, unreliable for complex flow control
- **AnswerQuestionWithAI** - Good for conversational content generation, NOT for structured data that needs parsing
- **Dialog scope variables** - Can cause recognition errors, prefer Global or Topic scope

### ❌ Unsupported or Unreliable
- **Complex GotoAction flows** - Causes silent failures and unpredictable behavior
- **`DoWhile` loops** - Not supported in Copilot Studio
- **Complex Power Fx table operations** - Often cause errors, use simple string concatenation instead
- **Questions without `init:` prefix** - Will be silently skipped
- **Questions without `interruptionPolicy`** - Will be silently skipped
- **Using SetVariable as substitute for `init:`** - Does not work

## Part 6: Working Examples

### Complete Minimal Bot
```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnUnknownIntent
  id: main
  actions:
    - kind: SendActivity
      id: sendActivity_Welcome
      activity: "Welcome to our bot!"

    - kind: Question
      id: question_GetName
      interruptionPolicy:
        allowInterruption: true
      variable: init:Global.UserName
      prompt: "What's your name?"
      entity: StringPrebuiltEntity

    - kind: Question
      id: question_GetAge
      interruptionPolicy:
        allowInterruption: true
      variable: init:Global.UserAge
      prompt: "How old are you?"
      entity: NumberPrebuiltEntity

    - kind: SendActivity
      id: sendActivity_Summary
      activity: "Nice to meet you, {Global.UserName}! You are {Global.UserAge} years old."

    - kind: EndDialog
      id: endDialog_Complete
```

### Conditional Logic Example
```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnUnknownIntent
  id: main
  actions:
    - kind: Question
      id: question_GetPreference
      interruptionPolicy:
        allowInterruption: true
      variable: init:Global.UserPreference
      prompt: "Do you prefer cats or dogs?"
      entity: StringPrebuiltEntity

    - kind: ConditionGroup
      id: if_LikesCats
      conditions:
        - id: condition_CatLover
          condition: =Find("cat", Lower(Global.UserPreference)) > 0
          actions:
            - kind: SendActivity
              id: sendActivity_CatResponse
              activity: "Great! Cats are wonderful companions! 🐱"
        - id: condition_DogLover
          condition: =Find("dog", Lower(Global.UserPreference)) > 0
          actions:
            - kind: SendActivity
              id: sendActivity_DogResponse
              activity: "Awesome! Dogs are amazing friends! 🐕"
      defaultActions:
        - kind: SendActivity
          id: sendActivity_NeutralResponse
          activity: "Both cats and dogs make great pets!"

    - kind: EndDialog
      id: endDialog_Complete
```

### Data Collection Example
```yaml
kind: AdaptiveDialog
beginDialog:
  kind: OnUnknownIntent
  id: main
  actions:
    - kind: Question
      id: question_GetName
      interruptionPolicy:
        allowInterruption: true
      variable: init:Global.CustomerName
      prompt: "What's your name?"
      entity: StringPrebuiltEntity

    - kind: Question
      id: question_GetEmail
      interruptionPolicy:
        allowInterruption: true
      variable: init:Global.CustomerEmail
      prompt: "What's your email address?"
      entity: EmailPrebuiltEntity

    - kind: Question
      id: question_GetFeedback
      interruptionPolicy:
        allowInterruption: true
      variable: init:Global.CustomerFeedback
      prompt: "Please share your feedback about our service:"
      entity: StringPrebuiltEntity

    # Build summary
    - kind: SetVariable
      id: setVariable_CreateSummary
      variable: Global.CustomerSummary
      value: =Concatenate("Customer: ", Global.CustomerName, Char(10), "Email: ", Global.CustomerEmail, Char(10), "Feedback: ", Global.CustomerFeedback)

    - kind: SendActivity
      id: sendActivity_ShowSummary
      activity: |-
        Thank you for your feedback! Here's what we received:

        {Global.CustomerSummary}

    - kind: EndDialog
      id: endDialog_Complete
```

## Part 7: Critical Reminders for Success

1. **ALWAYS use the complete Question template** - Missing any required property causes silent failures
2. **NEVER skip the `init:` prefix** - Questions without it will be skipped entirely  
3. **ALWAYS include `interruptionPolicy`** - Questions without it will be skipped entirely
4. **Use lowercase `entity:`** - Capital E causes "Unknown element" errors
5. **Test incrementally** - Add one question at a time and test before continuing
6. **Keep flows simple and predictable** - Duplicate logic rather than using complex jumps
7. **Use `{VariableName}` in activities** - Avoid complex Power Fx formulas in display text
8. **Every condition needs an `id`** - Missing condition IDs cause errors

## Part 8: Debugging Checklist

When your bot isn't working:

**Questions being skipped?**
- ✅ Check for `interruptionPolicy` block
- ✅ Check for `init:` prefix in variable
- ✅ Check `entity:` is lowercase
- ✅ Check all required properties are present

**Getting "Unknown element" errors?**
- ✅ Check YAML indentation (exactly 2 spaces per level)
- ✅ Check entity names are spelled correctly
- ✅ Check condition IDs are present
- ✅ Check for proper quotes around strings with special characters

**Conditional logic not working?**
- ✅ Check every condition has an `id` property
- ✅ Check Power Fx syntax in conditions
- ✅ Check `defaultActions` is present for fallback behavior
- ✅ Test each condition branch separately

**Variables not displaying?**
- ✅ Use `{VariableName}` syntax in activity text
- ✅ Check variable names match exactly (case sensitive)
- ✅ Avoid complex Power Fx formulas in SendActivity

**Bot behavior unpredictable?**
- ✅ Avoid complex GotoAction flows
- ✅ Use simple waterfall structure with nested conditions
- ✅ Test each section independently
- ✅ Keep logic within single ConditionGroup blocks when possible

This guide reflects the current, tested, and verified syntax for Copilot Studio as of September 2024. Following these patterns will result in reliable, predictable bot behavior.
