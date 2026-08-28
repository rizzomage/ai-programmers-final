# PROJECT OVERVIEW
This application will consume business policies, regulatory data, and requirements and produce structured, executable decision assets that can be leveraged as either direct integration or invocable webservices by other applications.

In the business world today, many companies experience multifold issues with proper maintenance and understanding of automated decisions made within core application software. While much improvement has been made through creating individual decision assets (typically using Decision Model & Notation (DMN) as a standard) separate from direct application code, the processes around creating those decision assets remains, in many instances, very manual and time-consuming with many humans and handoffs involved in the process. I propose that many of these existing manual processes can instead leverage an LLM in asset creation, then flowing through existing governance workflows to ensure completion and correctness.

This will likely leverage the use of multiple AI agents.

# Use Cases & Workflow
This application would impact the following user roles within a decision asset creation process flow:
* Decision modelers and analysts - These users manually create decision assets today and their role would shift away from direct creation and more toward validation of LLM output.
* Schema management team - Part of the creation of decision assets is in maintenance of application schemas these assets leverage. Because often there are new fields that must be added to existing schemas, an LLM agent can identify these requirements through the asset creation process and suggest the necessary updates (only applying changes with human approval).
* Engineering - Finally, decision assets must typically be 'wrapped' in a microservice in order to be truly executable. Today this is managed via manual wrapper coding done by engineering teams, where one or more LLM agents could generate the necessary execution wrapper, perform tests against it, and (with human review and approval) deploy into a live test environment.

# AI Features
The following AI-centered components would be used in this project:
* Prompt engineering - the core of the project. Prompts will be leveraged in consuming a given business policy or document, identifying the key requirements from it, and producing structured outputs. Prompts such as "Identify inputs and outputs from these requirements" and "For inputs and outputs which do not currently exist in the schema, suggest 
* Structured outputs - Schemas are managed via JSON and decision assets are created using DMN. As well, test cases for unit testing decision assets will need to be in JSON format. Because of this, structured outputs will be necessary for several LLM steps.
* RAG/Vector databases - This would likely be used for ingesting supplementary documentation - eg. definition of business terms, field naming guidance, etc. - and leveraging this to better inform LLM outputs, particularly during consumption of business policy and documentation.
* Evaluation - HITL pauses injected between agents to review output before it moves to the next step in the process - eg. structured decision output, suggested schema updates, execution wrapper, etc.

# Technical Approach
The application will be constructed of a front-end UI that is fed information for display from a multi-agent process. 

The front-end will track progress through steps for each policy and/or document transformation into one or more decision assets. This will include mandatory pauses for HITL output evaluation - with automated notification to the user(s) necessary to provide review and approval - and the ability for human modification of a step's output before approving it as input to the next step, as well as metrics tracking (eg. how often outputs are tweaked before approval, etc.). As this is an internal company platform, it will leverage single sign-on authentication, with different security access levels governing who can edit/approve at each step pause.

The back-end stepped process will run as follows. Geared steps are expected to leverage AI, while scripted steps should not require an LLM.

Main flow:
![alt text](https://github.com/rizzomage/ai-programmers-final/blob/main/images/project-flow-1.png)

Sub-process - Schema updates:
![alt text](https://github.com/rizzomage/ai-programmers-final/blob/main/images/project-flow-2.png)

Sub-process - Execution wrapper:
![alt text](https://github.com/rizzomage/ai-programmers-final/blob/main/images/project-flow-3.png)

I expect this to primarily leverage LLM APIs - whichever is the most cost-effective - and vector storage, as well as some manner of rules file particularly to best define how to structure the code for an execution wrapper.

# Example Prompts & Expected Outputs
* Sample prompt #1 - "Analyze the provided policy document and identify business decision points contained within. For each decision point, generate a structured decision model using Decision Model & Notation (DMN) format."
* Sample prompt #2 - "For each decision model, extract the input data fields required and output data fields produced. List these in the specified format ("{fieldFormat}")."
* Sample prompt #3 - "For each data field, analyze whether or not it is likely to exist in the base application schema provided. Provide suggested schema element matches, with their fully realized JSON path, in a list for each data field. For fields which do not appear to have a valid match in the existing schema, collect these fields in an 'Unmatched' list." (Note: a separate prompt, with structured formatting, will be used to then suggest schema updates for Unmatched fields.)

Expected output from sample prompt #1:
* The output would be twofold - a raw list of decision points and a structured DMN model for each.
* DMN typically follows an XML/XSD format and documentation describing this format structure would be chunked into a vector database to be leveraged by the step and inform the prompt.

Expected output from sample prompt #2:
* A pydantic-formatted list of field data Name (str) and data type (str) for each.

Expected output from sample prompt #3:
* A list of fields by Name followed by a sub-list of potential existing schema matches with their fully realized JSON paths included.

# Evaluation Strategy
As above, primarily this will leverage HITL evaluation to ensure completeness and correctness of the assets and suggestions generated prior to being committed. As well, I believe the possibility exists to track the efficacy of the generation steps - how often did a human need to correct the output. These metrics would be stored and available for view via the front-end UI.

If output tweaks by a HITL hit a certain frequency threshold, prompts and supplementary documentation (eg. wrapper code rules) should be thoroughly evaluated and experimentation run for best verbiage to meet efficacy KPIs.

# Observability Plan
I believe we would need to at least track performance - eg. how long does each step take, how many tokens are used, etc. - as well as hard errors - eg. service unavailable, token usage limit exceeded, etc. We would also want to track frequencies on how often new policy document evaluations are triggered by a user and how many are aborted (left unapproved/incomplete or force-halted by a user). 
