# Hands-on Lab: build your first agents using CrewAI and watsonx.ai

In this lab, you are going to learn how to build AI agent teams that work together to tackle complex tasks.

# What is CrewAI?

**CrewAI is a cutting-edge framework for orchestrating autonomous AI agents.**

CrewAI enables you to create AI teams where each agent has specific roles, tools, and goals, working together to accomplish complex tasks.

Think of it as assembling your dream team - each member (agent) brings unique skills and expertise, collaborating seamlessly to achieve your objectives.

## How CrewAI Works

Just like a company has departments (Sales, Engineering, Marketing) working together under leadership to achieve business goals, CrewAI helps you create an organization of AI agents with specialized roles collaborating to accomplish complex tasks.

![CrewAI Framework Overview](https://mintlify.s3.us-west-1.amazonaws.com/crewai/crewAI-mindmap.png)

CrewAI Framework Overview

| Component | Description | Key Features |
| --- | --- | --- |
| **Crew** | The top-level organization | • Manages AI agent teams  <br>• Oversees workflows  <br>• Ensures collaboration  <br>• Delivers outcomes |
| **AI Agents** | Specialized team members | • Have specific roles (researcher, writer)  <br>• Use designated tools  <br>• Can delegate tasks  <br>• Make autonomous decisions |
| **Process** | Workflow management system | • Defines collaboration patterns  <br>• Controls task assignments  <br>• Manages interactions  <br>• Ensures efficient execution |
| **Tasks** | Individual assignments | • Have clear objectives  <br>• Use specific tools  <br>• Feed into larger process  <br>• Produce actionable results |

### How It All Works Together

1.  The **Crew** organizes the overall operation
2.  **AI Agents** work on their specialized tasks
3.  The **Process** ensures smooth collaboration
4.  **Tasks** get completed to achieve the goal

## Key Features

## Role-Based Agents

Create specialized agents with defined roles, expertise, and goals - from researchers to analysts to writers

## Flexible Tools

Equip agents with custom tools and APIs to interact with external services and data sources

## Intelligent Collaboration

Agents work together, sharing insights and coordinating tasks to achieve complex objectives

## Task Management

Define sequential or parallel workflows, with agents automatically handling task dependencies

## Why Choose CrewAI?

*   🧠 **Autonomous Operation**: Agents make intelligent decisions based on their roles and available tools
*   📝 **Natural Interaction**: Agents communicate and collaborate like human team members
*   🛠️ **Extensible Design**: Easy to add new tools, roles, and capabilities
*   🚀 **Production Ready**: Built for reliability and scalability in real-world applications

# Quickstart

Build your first AI agent with CrewAI in under 5 minutes.

## Build your first CrewAI Agent

Let’s create a simple crew that will help us `research` and `report` on the `latest AI developments` for a given topic or subject.

Before we proceed, make sure you have `crewai` and `crewai-tools` installed. If you haven’t installed them yet, you can do so by running the following command (more information in the [installation guide](https://docs.crewai.com/installation)):

```
pip install crewai crewai-tools
```

Follow the steps below to get crewing! 🚣‍♂️

1. Create your crew

Create a new crew project by running the following command in your terminal. This will create a new directory called `latest-ai-development` with the basic structure for your crew.

```shell
crewai create crew latest-ai-development
```

   - Select `watsonx` as provider.
   - Select `watsonx/mistralai/mistral-large` as the LLM to use.
   - If in a workshop, you can skip the `WATSONX URL`, `WATSONX API Key` and `WATSONX Project Id` prompts and copy the credentials provided by your instructor in the `.env` file. Otherwise, provide those values.  

2. Modify your \`agents.yaml\` file

You can also modify the agents as needed to fit your use case or copy and paste as is to your project. Any variable interpolated in your `agents.yaml` and `tasks.yaml` files like `{topic}` will be replaced by the value of the variable in the `main.py` file.

```yaml
# src/latest_ai_development/config/agents.yaml
researcher:
  role: >
    {topic} Senior Data Researcher
  goal: >
    Uncover cutting-edge developments in {topic}
  backstory: >
    You're a seasoned researcher with a knack for uncovering the latest
    developments in {topic}. Known for your ability to find the most relevant
    information and present it in a clear and concise manner.

reporting_analyst:
  role: >
    {topic} Reporting Analyst
  goal: >
    Create detailed reports based on {topic} data analysis and research findings
  backstory: >
    You're a meticulous analyst with a keen eye for detail. You're known for
    your ability to turn complex data into clear and concise reports, making
    it easy for others to understand and act on the information you provide.
```

3. Modify your \`tasks.yaml\` file

```yaml
# src/latest_ai_development/config/tasks.yaml
research_task:
  description: >
    Conduct a thorough research about {topic}
    Make sure you find any interesting and relevant information given
    the current year is 2024.
  expected_output: >
    A list with 10 bullet points of the most relevant information about {topic}
  agent: researcher

reporting_task:
  description: >
    Review the context you got and expand each topic into a full section for a report.
    Make sure the report is detailed and contains any and all relevant information.
  expected_output: >
    A fully fledge reports with the mains topics, each with a full section of information.
    Formatted as markdown without '```'
  agent: reporting_analyst
  output_file: report.md
```

4. Modify your \`crew.py\` file

```python
# src/latest_ai_development/crew.py
from crewai import Agent, Crew, Process, Task
from crewai.project import CrewBase, agent, crew, task
from crewai_tools import SerperDevTool

@CrewBase
class LatestAiDevelopment():
  """LatestAiDevelopment crew"""

  @agent
  def researcher(self) -> Agent:
    return Agent(
      config=self.agents_config['researcher'],
      verbose=True,
      tools=[SerperDevTool()]
    )

  @agent
  def reporting_analyst(self) -> Agent:
    return Agent(
      config=self.agents_config['reporting_analyst'],
      verbose=True
    )

  @task
  def research_task(self) -> Task:
    return Task(
      config=self.tasks_config['research_task'],
    )

  @task
  def reporting_task(self) -> Task:
    return Task(
      config=self.tasks_config['reporting_task'],
      output_file='output/report.md' # This is the file that will be contain the final report.
    )

  @crew
  def crew(self) -> Crew:
    """Creates the LatestAiDevelopment crew"""
    return Crew(
      agents=self.agents, # Automatically created by the @agent decorator
      tasks=self.tasks, # Automatically created by the @task decorator
      process=Process.sequential,
      verbose=True,
    )
```

5. \[Optional\] Add before and after crew functions


```python
# src/latest_ai_development/crew.py
from crewai import Agent, Crew, Process, Task
from crewai.project import CrewBase, agent, crew, task, before_kickoff, after_kickoff
from crewai_tools import SerperDevTool

@CrewBase
class LatestAiDevelopmentCrew():
  """LatestAiDevelopment crew"""

  @before_kickoff
  def before_kickoff_function(self, inputs):
    print(f"Before kickoff function with inputs: {inputs}")
    return inputs # You can return the inputs or modify them as needed

  @after_kickoff
  def after_kickoff_function(self, result):
    print(f"After kickoff function with result: {result}")
    return result # You can return the result or modify it as needed

  # ... remaining code
```

6. Feel free to pass custom inputs to your crew

For example, you can pass the `topic` input to your crew to customize the research and reporting.

```python
#!/usr/bin/env python
# src/latest_ai_development/main.py
import sys
from latest_ai_development.crew import LatestAiDevelopmentCrew

def run():
  """
  Run the crew.
  """
  inputs = {
    'topic': 'AI Agents'
  }
  LatestAiDevelopmentCrew().crew().kickoff(inputs=inputs)
```

7. Set your environment variables

Before running your crew, make sure you have the following keys set as environment variables in your `.env` file:

*   A [watsonx.ai API Key](https://cloud.ibm.com/iam/apikeys): `WATSONX_APIKEY=...`
*   A **watsonx.ai Project ID**: `WATSONX_PROJECT_ID=...`
*   A **watsonx.ai URL** e.g.: `WATSONX_URL=https://us-south.ml.cloud.ibm.com`
*   A [Serper.dev](https://serper.dev/) API key: `SERPER_API_KEY=YOUR_KEY_HERE`
*   **Note:** if you are running this lab as part of a workshop, use the credentials provided by your instructor.

1. Lock and install the dependencies

Lock the dependencies and install them by using the CLI command but first, navigate to your project directory:

```shell
cd latest_ai_development
crewai install
```

9. Run your crew

To run your crew, execute the following command in the root of your project:

```bash
crewai run
```

10

View your final report

You should see the output in the console and the `report.md` file should be created in the root of your project with the final report.

Here’s an example of what the report should look like:

```markdown
# Comprehensive Report on the Rise and Impact of AI Agents in 2024

## 1. Introduction to AI Agents
In 2024, Artificial Intelligence (AI) agents are at the forefront of innovation across various industries. As intelligent systems that can perform tasks typically requiring human cognition, AI agents are paving the way for significant advancements in operational efficiency, decision-making, and overall productivity within sectors like Human Resources (HR) and Finance. This report aims to detail the rise of AI agents, their frameworks, applications, and potential implications on the workforce.

## 2. Benefits of AI Agents
AI agents bring numerous advantages that are transforming traditional work environments. Key benefits include:

- **Task Automation**: AI agents can carry out repetitive tasks such as data entry, scheduling, and payroll processing without human intervention, greatly reducing the time and resources spent on these activities.
- **Improved Efficiency**: By quickly processing large datasets and performing analyses that would take humans significantly longer, AI agents enhance operational efficiency. This allows teams to focus on strategic tasks that require higher-level thinking.
- **Enhanced Decision-Making**: AI agents can analyze trends and patterns in data, provide insights, and even suggest actions, helping stakeholders make informed decisions based on factual data rather than intuition alone.

## 3. Popular AI Agent Frameworks
Several frameworks have emerged to facilitate the development of AI agents, each with its own unique features and capabilities. Some of the most popular frameworks include:

- **Autogen**: A framework designed to streamline the development of AI agents through automation of code generation.
- **Semantic Kernel**: Focuses on natural language processing and understanding, enabling agents to comprehend user intentions better.
- **Promptflow**: Provides tools for developers to create conversational agents that can navigate complex interactions seamlessly.
- **Langchain**: Specializes in leveraging various APIs to ensure agents can access and utilize external data effectively.
- **CrewAI**: Aimed at collaborative environments, CrewAI strengthens teamwork by facilitating communication through AI-driven insights.
- **MemGPT**: Combines memory-optimized architectures with generative capabilities, allowing for more personalized interactions with users.

These frameworks empower developers to build versatile and intelligent agents that can engage users, perform advanced analytics, and execute various tasks aligned with organizational goals.

## 4. AI Agents in Human Resources
AI agents are revolutionizing HR practices by automating and optimizing key functions:

- **Recruiting**: AI agents can screen resumes, schedule interviews, and even conduct initial assessments, thus accelerating the hiring process while minimizing biases.
- **Succession Planning**: AI systems analyze employee performance data and potential, helping organizations identify future leaders and plan appropriate training.
- **Employee Engagement**: Chatbots powered by AI can facilitate feedback loops between employees and management, promoting an open culture and addressing concerns promptly.

As AI continues to evolve, HR departments leveraging these agents can realize substantial improvements in both efficiency and employee satisfaction.

## 5. AI Agents in Finance
The finance sector is seeing extensive integration of AI agents that enhance financial practices:

- **Expense Tracking**: Automated systems manage and monitor expenses, flagging anomalies and offering recommendations based on spending patterns.
- **Risk Assessment**: AI models assess credit risk and uncover potential fraud by analyzing transaction data and behavioral patterns.
- **Investment Decisions**: AI agents provide stock predictions and analytics based on historical data and current market conditions, empowering investors with informative insights.

The incorporation of AI agents into finance is fostering a more responsive and risk-aware financial landscape.

## 6. Market Trends and Investments
The growth of AI agents has attracted significant investment, especially amidst the rising popularity of chatbots and generative AI technologies. Companies and entrepreneurs are eager to explore the potential of these systems, recognizing their ability to streamline operations and improve customer engagement.

Conversely, corporations like Microsoft are taking strides to integrate AI agents into their product offerings, with enhancements to their Copilot 365 applications. This strategic move emphasizes the importance of AI literacy in the modern workplace and indicates the stabilizing of AI agents as essential business tools.

## 7. Future Predictions and Implications
Experts predict that AI agents will transform essential aspects of work life. As we look toward the future, several anticipated changes include:

- Enhanced integration of AI agents across all business functions, creating interconnected systems that leverage data from various departmental silos for comprehensive decision-making.
- Continued advancement of AI technologies, resulting in smarter, more adaptable agents capable of learning and evolving from user interactions.
- Increased regulatory scrutiny to ensure ethical use, especially concerning data privacy and employee surveillance as AI agents become more prevalent.

To stay competitive and harness the full potential of AI agents, organizations must remain vigilant about latest developments in AI technology and consider continuous learning and adaptation in their strategic planning.

## 8. Conclusion
The emergence of AI agents is undeniably reshaping the workplace landscape in 2024. With their ability to automate tasks, enhance efficiency, and improve decision-making, AI agents are critical in driving operational success. Organizations must embrace and adapt to AI developments to thrive in an increasingly digital business environment.
```

### Note on Consistency in Naming

The names you use in your YAML files (`agents.yaml` and `tasks.yaml`) should match the method names in your Python code. For example, you can reference the agent for specific tasks from `tasks.yaml` file. This naming consistency allows CrewAI to automatically link your configurations with your code; otherwise, your task won’t recognize the reference properly.

#### Example References

Note how we use the same name for the agent in the `agents.yaml` (`email_summarizer`) file as the method name in the `crew.py` (`email_summarizer`) file.

```yaml
email_summarizer:
    role: >
      Email Summarizer
    goal: >
      Summarize emails into a concise and clear summary
    backstory: >
      You will create a 5 bullet point summary of the report
    llm: mixtal_llm
```

Note how we use the same name for the agent in the `tasks.yaml` (`email_summarizer_task`) file as the method name in the `crew.py` (`email_summarizer_task`) file.

```yaml
email_summarizer_task:
    description: >
      Summarize the email into a 5 bullet point summary
    expected_output: >
      A 5 bullet point summary of the email
    agent: email_summarizer
    context:
      - reporting_task
      - research_task
```

Use the annotations to properly reference the agent and task in the `crew.py` file.

### Annotations include:

*   `@agent`
*   `@task`
*   `@crew`
*   `@tool`
*   `@before_kickoff`
*   `@after_kickoff`
*   `@callback`
*   `@output_json`
*   `@output_pydantic`
*   `@cache_handler`

```python
# ...
@agent
def email_summarizer(self) -> Agent:
    return Agent(
        config=self.agents_config["email_summarizer"],
    )

@task
def email_summarizer_task(self) -> Task:
    return Task(
        config=self.tasks_config["email_summarizer_task"],
    )
# ...
```

In addition to the [sequential process](../how-to/sequential-process), you can use the [hierarchical process](../how-to/hierarchical-process), which automatically assigns a manager to the defined crew to properly coordinate the planning and execution of tasks through delegation and validation of results. You can learn more about the core concepts [here](/concepts).
