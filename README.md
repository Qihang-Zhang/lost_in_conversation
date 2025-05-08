# LLMs Get Lost in Multi-Turn Conversation

<p align="center">
  <img height="500" src="https://github.com/microsoft/lost_in_conversation/blob/main/images/Lost_in_Conv_Github_Teaser.png?raw=true">
</p>

Lost in Conversation is a code repository to facilitate benchmarking LLMs on multi-turn task completion and the reproduction of experiments included in the accompanying paper: "LLMs Get Lost in Multi-Turn Conversation". Simulating conversations with the provided code can reproduce findings from the paper, such as results in the below Table:

<p align="center">
  <img height="500" src="https://github.com/microsoft/lost_in_conversation/blob/main/images/Lost_in_Conv_Main_Table.png?raw=true">
</p>


## Repository Contents

- The `run_experiments.py` file which can be used to run experiments to validate findings of the paper.
- Simulator-related code (`simulator_*.py), that allows to simulation single-turn and multi-turn conversations.
- Task-related code (`tasks/`) that defines all the task-specific logic to run and evaluate simulations for the seven tasks included in our experiments (code, database, actions, math, data2text, summary, translation).
- Web-based viewer to inspect simulated conversations (`app_conv_viewer.py`)
- Prompt-related content (`prompts/`) which contains all the prompts used (1) either during simulation, (2) or during the sharding process (see paper Appendix on Sharding Process).
- Instruction data (`data/sharded_instructions_600.json`) which contains the 600 sharded instructions used to simulate conversations.

## Getting Started

### 1. Simulating Conversations

You can simply run the following command:
```sh
python run_experiments.py
```

Which will simulate conversations with all the default parameters (run with `--help` to find out what they are).
This will require to set the environment variable `OPENAI_API_KEY` or `AZURE_OPENAI_API_KEY` + `AZURE_OPENAI_ENDPOINT`.

<i><b>!! Be aware:</b> running this command will run LLM-based API calls, which will cost you real money.</i>

For the paper's experiments, we included models from other organizations, which require integration beyond OpenAI's API. If you would like the additional API integration files, you can contact us.

### 2. Viewing Simulated Conversations

Use the streamlit app to view conversations:
```sh
streamlit run app_conv_viewer.py
```

### 3. Adding a New Task

You can look at the `task_base.py` file for a sense of the required functions.
Then create a folder in `tasks/<new_task>` with your task logic files, and add an entry into `tasks.py` with the new task class. That's it!


### 4. Setup Details

- Simulation of the `code` task does not work on Windows (due to issue with eval code working only on Unix)
- Simulation of the `database` task requires downloading test databases (<5 Gb) and copying the files to `data/spider/databases/`. See instructions in `data/spider/README.md`

## Dataset Contents

We release sharded instructions required to run simulation. You can find the file at `data/sharded_instructions_600.json`. Each sample has the following schema:

```
{
    "task_id": "sharded-{original-task-id}",
    "task": "code|database|actions|math|data2text|summary|translation",
    "shards: [{"shard_id": 1, "shard_text": "[...]"}, {"shard_id": 2, "shard_text": "[...]"}, ...],
    "<task_specific_keys>": <task_specific_values>
}
```

The task-specific keys and values depend on the task, and are used for bookkeeping and evaluation (e.g., the reference answer, tests, etc.).

## Dataset Creation & Processing

The data was created through automatic generation followed by manual curation by the authors of the work, between January and April 2025. The exact process is described in the “Sharding Process” Appendix section of the paper. 

Creating datasets involved transforming existing data (fully-specified single-turn instructions) from seven datasets, as listed in the paper. All datasets correspond to datasets released to evaluate LLM performance on generation tasks.

## Intended Use

Lost in Conversation is best suited to simulate single-turn and multi-turn conversations for the tasks we’ve set up. Six tasks are included with our release, all analytical generation tasks: (1) Python programming, (2) Database query generation, (3) API function calling, (4) elementary math problems, (5) data2text tasks, (6) summarization. 

Lost in Conversation is being shared with the research community to facilitate reproduction of our results and foster further research in this area. 

Lost in Conversation is intended to be used by domain experts who are independently capable of evaluating the quality of outputs before acting on them. 

## Out-of-scope Use

Lost in Conversation is not intended to simulate realistic interaction between humans and LLMs, and should not be used to replace human studies or human annotation. Lost in Conversation simulations are a computational tool to study LLMs and their behavior in multi-turn conversation. The code and findings cannot be used to make claims about humans or real-users of the systems. 

We do not recommend using Lost in Conversation in commercial or real-world applications without further testing and development. It is being released for research purposes. 

Lost in Conversation was not designed or evaluated for all possible downstream purposes. Developers should consider its inherent limitations as they select use cases, and evaluate and mitigate for accuracy, safety, and fairness concerns specific to each intended downstream use. 

Lost in Conversation should not be used in highly regulated domains where inaccurate outputs could suggest actions that lead to injury or negatively impact an individual's legal, financial, or life opportunities. 

We do not recommend using Lost in Conversation in the context of high-risk decision making (e.g. in law enforcement, legal, finance, or healthcare). 


## Limitations

Using the Lost in Conversation code requires the use of an LLM. We do not provide access to an LLM (whether locally-hosted or API-based), and users of the code must integrate the code with their own LLM provider (e.g., OpenAI API key). 

Lost in Conversation was developed for research and experimental purposes. Further testing and validation are needed before considering its application in commercial or real-world scenarios. 

Lost in Conversation was designed and tested using the English language. Performance in other languages may vary and should be assessed by someone who is both an expert in the expected outputs and a native speaker of that language. 

Outputs generated by AI may include factual errors, fabrication, or speculation. Users are responsible for assessing the accuracy of generated content. All decisions leveraging outputs of the system should be made with human oversight and not be based solely on system outputs. 

The conversations simulated using Lost in Conversation are not intended to represent natural human-AI conversations. The simulations should not be used in place of user studies to draw conclusions about human-behavior. The code is intended primarily to evaluate limitations of LLMs, and not human users. 

Lost in Conversation included experiments with a total of 15 models, including both open-weights and API-based models. See the paper for the detailed list, including how each model was accessed to understand the capabilities and limitations of this model. Though we expect experiments to be consistent in other LLMs (as we observed it in all 15 models we tested), experiments should be conducted to confirm such claims. 

There has not been a systematic effort to ensure that systems using Lost in Conversation are protected from security vulnerabilities such as indirect prompt injection attacks. Any systems using it should take proactive measures to harden their systems as appropriate. 

## Best Practices

Better performance can be achieved by experimenting at small-scale and then proceeding with larger runs. The code has parameters for parallelization (such as num_workers in the run_experiments.py file). These require experimentation, and should be based on the level of access the user has for a given LLM access, to avoid overloading providers and respecting rate limits. 

We have manually validated the samples used during the simulation conducted for the paper. However, future use might involve instructions that are not manually validated which could lead to degradation of the simulation environment. We encourage users of Lost in Conversation to validate the instructions they run simulations on manually, or exercise caution in interpreting results on unverified samples. The thinking is, once a sample is manually validated, degradation in performance from an LLM during simulation should be a reflection of model behavior. With uninspected instructions, degradation in performance could also be due to an invalid simulation (infeasible sample). 

We strongly encourage users to use LLMs/MLLMs that support robust Responsible AI mitigations, such as Azure Open AI (AOAI) services. Such services continually update their safety and RAI mitigations with the latest industry standards for responsible use. For more on AOAI’s best practices when employing foundations models for scripts and applications: 

[Blog post on responsible AI features in AOAI that were presented at Ignite 2023](https://techcommunity.microsoft.com/t5/ai-azure-ai-services-blog/announcing-new-ai-safety-amp-responsible-ai-features-in-azure/ba-p/3983686) 

[Overview of Responsible AI practices for Azure OpenAI models] (https://learn.microsoft.com/en-us/legal/cognitive-services/openai/overview) 

[Azure OpenAI Transparency Note](https://learn.microsoft.com/en-us/legal/cognitive-services/openai/transparency-note) 

[OpenAI’s Usage policies](https://openai.com/policies/usage-policies) 

[Azure OpenAI’s Code of Conduct](https://learn.microsoft.com/en-us/legal/cognitive-services/openai/code-of-conduct) 

Users are responsible for sourcing their datasets legally and ethically. This could include securing appropriate copy rights, ensuring consent for use of audio/images, and/or the anonymization of data prior to use in research.    

Users are reminded to be mindful of data privacy concerns and are encouraged to review the privacy policies associated with any models and data storage solutions interfacing with Lost in Conversation.  

It is the user’s responsibility to ensure that the use of Lost in Conversation complies with relevant data protection regulations and organizational guidelines. 


## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.

## Cite the work

If you make use of the code, data, or findings, please cite our paper:
```
@inproceedings{laban2025lost_in_conv,
  title={LLMs Get Lost In Multi-Turn Conversation},
  author={Laban, Philippe and Hayashi, Hiroaki and Zhou, Yingbo and Neville, Jennifer},
}
```

## Contact

We welcome feedback and collaboration from our audience. If you have suggestions, questions, or observe unexpected/offensive behavior in our technology, please contact us at plaban@microsoft.com.  

If the team receives reports of undesired behavior or identifies issues independently, we will update this repository with appropriate mitigations.
