# MoralDilemmas 

Dataset: ```data/dilemmas_with_detail_by_action.csv```

Croissant metadata: ```metadata/crossiant```

There are three use cases -- 1. evaluate model preferences on everyday dilemmas 2. evaluate model preference on dilemmas based on the given principle 3. evaluate the model's steerability by system prompt modulation on each given principle

We used ``gpt-4-turbo`` as demonstrated below. We also provided our model responses on our dilemmas dataset run by five models, and some corresponding analysis datasets for replication purposes to avoid running them again through the flag (``replication_purpose``).

## Setup the environment
```
$ conda create --name <env> --file requirements.txt
```

## Case 1: Evaluate model preferences on everyday dilemmas
1. Evaluate the gpt-4-turbo model on our dilemma dataset. 
- You can set the output file div be `eval/model_response_from_eval`. Here, we implemented the OpenAI client code, you can set the model name e.g. `gpt-4-turbo`. We also have a replication purpose flag to allow you to run the evaluation on the first five lines only. 

```
python eval/evaluate_model_on_dilemma.py \
--output_jsonl_file_div <output_jsonl_file_div> \ 
--model <gpt-4-turbo> \
--model_name_for_variable <own_created_name_e.g.gpt4> \
--replication_purpose <full/only_first_five> \
--api_key <optional_you_can_set_as_.env_or_global>
```

2. Combine model response from 1 to get the details of the dilemma for each action. 
- You can use the model response file generated from step 1 on the output_jsonl_file_div `eval/model_response_from_eval`. The MoralDilemmas dataset can be found in `data/dilemmas_with_detail_by_action.csv`. If you chose to do `only_first_five` in step 1, you will need the same subset of MoralDilemma to prevent error (can be found in `data/dilemmas_with_detail_by_action_test.csv`).
- You can set this combined model response file on `eval/model_response_from_eval` for reference. The data of `<model_name>_eval_on_dilemmas_with_action_separate.csv` will be created.

```
python eval/combine_model_eval_with_dilemmas_action_separate.py \
--input_model_resp_file <step_1_output_jsonl_file> \
--input_dilemma_file <MoralDilemma_dataset> \
--output_analysis_file <output_analysis_file_div> \
--model <model_name_for_variable>
```

3. Analyze the combined model responses file with dilemmas details from step 2 based on the five theories.
- For input, you can use the output file from step 2, which can be found in ``eval/model_response_from_eval/<model_name>_eval_on_dilemmas_with_action_separate.csv``. We also provided the evaluated MoralDilemma dataset with six model responses in ```eval/model_response_from_eval/all_models_eval_on_dilemmas_with_detail_by_action.csv```
- For models, input the model names used in fields from previous steps e.g. gpt4 to analyze. For our provided MoralDilemma dataset with six models responses, you could have ``gpt4 gpt35 llama2 llama3 mixtral_rerun claude``

```
python analysis/analysis_for_model_responses_five_theories/analyze_model_resp_with_five_theories.py \
--input_model_resp_file <output_from_step_2> \
--output_analysis_file <e.g.analysis/analysis_for_model_responses_five_theories/model_resp.csv> \
--models <model_name(s)>
```

4. Plot the graphs from the analyzed files with five theory dimensions.
- You can choose the analyzed files by step 3 in the folder ``analysis/analysis_for_model_responses_five_theories/``

```
python analysis/analysis_for_model_responses_five_theories/create_plot.py \
--input_analysis_file <output_path_in_step_3/model_resp.csv> \
--output_graph_div <path_for_graphs>
```

## Case 2: Evaluate model preferences on everyday dilemmas based on the given principle

1. Get the relevant values for each principle from collected values by prompting the model 10 times.
- To provide a value list, we used 301 values with the five theories dimensions data as default: ```data/values.csv```
- For input principle file, you can select the principle files from folder ``data``. If you want to continue to analyze with the OpenAI model, you can use the OpenAI ModelSpec: ``data/principle_openai.csv`
- For model, we use ``gpt-4-0125-preview`` to help us to get relevant values from our value list. You can also use other models.
- For replication purpose, you can choose to run ```only_first_two_principles``` for debug.
```
python eval_for_system_prompt_modulation/get_values_for_principle.py \
--input_principle_file <data/principle_company.csv> \
--output_jsonl_file_div <e.g._eval_for_system_prompt_modulation/values_and_system_prompt_for_principle> \
--model <openai_model_e.g._gpt-4-0125-preview> \
--replication_purpose <full_/or/_only_first_two_principles>\
--api_key <optional_you_can_set_as_.env_or_global>
```

2. Calculate the value's relevance by the empirical probabilities from the 10 model responses in step 1.
- For input, it is the output file from step 1. If you follow the previous step, it could be ``eval_for_system_prompt_modulation/values_and_system_prompt_for_principle/principle_openai.csv``
```
python eval_for_system_prompt_modulation/calc_value_relevance.py \
--input_system_prompt_file <output_from_step_1> \
--output_jsonl_file_div <e.g._eval_for_system_prompt_modulation/values_and_system_prompt_for_principle>
```

3. Get the values conflicts and relevant dilemmas for each principle. 
- For input principle file, we use output file from Step 2 to get the values per principle. You can find from ``eval_for_system_prompt_modulation/values_and_system_prompt_for_principle/<model_name>_clean.csv``
- For input dilemma file, we use output file from Step 2 of Case 1 to get the moral responses on MoralDilemmas. Here, we provided the six model responses on our MoralDilemmas dataset: ```eval/model_response_from_eval/all_models_eval_on_dilemmas_with_detail_by_action.csv```.
```
python eval_for_system_prompt_modulation/get_values_conflicts_and_relevant_dilemmas_for_principle.py \
--input_principle_file <output_from_step_2_e.g._principle_company_clean.csv> \
--input_dilemma_file <output_from_case_1_step_2> \
--output_jsonl_file_div <eval_for_system_prompt_modulation/values_and_system_prompt_for_principle> \
--model <model_name> \
```

## Case 3: Evaluate the model's steerability by system prompt modulation on each given principle
1. Generate system prompts for each principle -- one for steering to supporting values and one for steering to opposing value for each principle
- For input, it is from the output by case 2 step 3: ``<output_path>/principle_gpt4_eval_with_values_conflicts_and_dilemma.csv``
- 
```
python eval_for_system_prompt_modulation generate_system_prompts_for_principle.py \ 
--input_system_prompt_file <output_from_step_3_in_case_2> \
--output_jsonl_file_div <e.g._eval_for_system_prompt_modulation/values_and_system_prompt_for_principle> \
--model gpt-4-turbo \
--api_key <optional_you_can_set_as_.env_or_global>
```
2. Evaluate models on relevant dilemmas for each principle by each set of system prompts.
- For input system prompt file, it is the output file from step 1: ``<output_path>/<model>_with_system_prompts.csv``
- For dilemma file, it is our MoralDilemmas data
- For model, input the model we want to use for evaluation: e.g. ``gpt-4-turbo``
```
python eval_for_system_prompt_modulation/evaluate_model_on_system_prompt.py \
--input_system_prompt_file <output_from_step_1> \
--input_dilemma_file <data/dilemmas_with_detail_by_action.csv> \
--output_jsonl_file_div <e.g._eval_for_system_prompt_modulation/model_response_on_system_prompt/> \
--model gpt-4-turbo \
--api_key <optional_you_can_set_as_.env_or_global>
```
- output: ``eval_for_system_prompt_modulation/model_response_on_system_prompt/{model}_eval.csv``

3. Analyze the model responses from step 2 to evaluate the steerability of the model.
- For input principle file, it is the output from step 2. You could find it on ``eval_for_system_prompt_modulation/model_response_on_system_prompt/{model}_eval.csv``
- For input dilemma file, we provided the six model responses on our MoralDilemma dataset: ``eval/model_response_from_eval/all_models_eval_on_dilemmas_with_detail_by_action.csv``. If you followed the whole pipeline, you could also find the file by case 1 step 1 from ``eval/model_response_from_eval/<model_name>_eval_on_dilemmas_with_detail_by_action.csv``
- For steer, the script will analyze model responses for one set of system prompts per time. Steer == ``sup`` means that it analyzes the model responses when having system prompts steering to supporting values.
```
python analysis/analysis_for_system_prompt_modulation/analyze_model_resp_on_system_prompt.py \
--input_principle_file <output_from_step_2> \
--input_dilemma_file <output_from_case_1_step_2_or_provided_file> \
--output_div <e.g.analysis/analysis_for_system_prompt_modulation> \
--steer <sup/opp> \
--company_name <use_for_create_output_file_name_e.g._openai> \
--model <the_name_used_for_field_and_also_for_output_file_name>
```

<!-- - output: ```{output_file_div}/principle_{company}_{model_name}_eval_with_system_prompt_system_prompt_sup.csv```
- output:  ```{output_file_div}/principle_{company}_{model_name}_eval_with_system_prompt_system_prompt_opp.csv``` -->

4. Create a plot from the analysis files from step 3. 
- We used the two output analysis files from step 3 -- one is from steering to supporting value; another one is from steering to opposing value.
- For sup and opp, you can find the output files ``principle_<companyname>_<modelname>_eval_with_system_prompt_system_prompt_<sup/opp>_value.csv`` from step 3 on: ``analysis/analysis_for_system_prompt_modulation/``. We also provided the complete files on: ``data/``
- For replication_purpose, if you followed the pipeline and chose to evaluate only two principles in case 2 step 1, you can use `only_two_principles` here to have a better illustration of graphs. If you use our provided complete files, you could go for `full`.
```
python analysis/analysis_for_system_prompt_modulation/create_plot.py \ 
--input_support_value_file <path/...system_prompt_sup_value.csv> \ 
--input_oppose_value_file <path/...system_prompt_sup_value.csv> \ 
--output_graph_div <analysis/analysis_for_system_prompt_modulation/> \
--replication_purpose <only_two_principles/or/full>
```
