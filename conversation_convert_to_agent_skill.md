Converting an ongoing chat or a specific project history into a structured .md file for an agent skill is a great way to "lock in" logic and behavior. To get a high-quality output, you need a meta-prompt that instructs the AI to analyze the context, extract the core objectives, and format them into a technical specification.
You can use the following prompt to transform your current session:

### The Transformation Prompt

> "Analyze our entire chat history and the current trajectory of this conversation. Extract the core logic, instructions, and specialized knowledge we have developed. Using this data, generate a comprehensive **Agent Skill Specification** in Markdown format (.md).
> Please include the following sections:
>  1. **Skill Name & Description:** A concise summary of what this agent does.
>  2. **System Role/Persona:** The specific tone and identity the agent should adopt.
>  3. **Core Constraints & Rules:** A list of 'Always' and 'Never' behaviors based on our discussion.
>  4. **Step-by-Step Logic/Workflows:** The methodology used to solve the tasks we've been working on.
>  5. **Knowledge Context:** Key facts or technical details established in this chat that the agent must remember.
>  6. **Example Inputs/Outputs:** Representative samples of how the agent should respond.
> Format the output as a clean, copy-pasteable Markdown block."


### Why this works

 * **Persona Extraction:** It identifies the "vibe" of the chat so the new agent doesn't just provide data, but provides it in the style you've grown accustomed to.
 * **Constraint Mapping:** It looks for times you've corrected the AI or asked for specific formatting (e.g., "don't use LaTeX for simple units") and bakes those into the permanent rules.
 * **Modular Design:** By requesting a .md format, it creates a file that is easily digestible by most LLM "System Prompt" or "Instructions" fields in various agent builders.

### Tips for Implementation

 * **The "Last Call" Check:** Before running the prompt above, give one final instruction like, "Before we turn this into a skill, remember that I want all outputs to be [specific requirement]." This ensures that detail is included in the final file.
 * **Manual Refinement:** Once the AI generates the Markdown, review the **Core Constraints** section. This is usually where you'll want to add your most "opinionated" rules to ensure the agent doesn't drift during future use.
