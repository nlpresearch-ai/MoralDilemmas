# MoralDilemmas 

Dataset: ```data/dilemmas_with_detail_by_action.csv```

Croissant metadata: ```metadata/crossiant```

There are three use cases -- 1. evaluate model preferences on everyday dilemmas 2. evaluate model preference on dilemmas based on the given principle 3. evaluate the model's steerability by system prompt modulation on each given principle

We used gpt-4-turbo as demonstration below. We also provided our model responses on our dilemmas dataset run by five models, and some corresponding analysis datasets for replication purpose to avoid running them again, and you can search for the tag (replication purpose) to see the relevant commands.

## Setup the environment
```$ conda create --name <env> --file requirements.txt```

## Case 1: Evaluate model preferences on everyday dilemmas
1. Evaluate the gpt-4-turbo model on our dataset with dilemmas 
- dataset: ```data/dilemmas.csv```
- (own run) command: ```python eval/evaluate_model_on_dilemma.py -o eval/model_response_from_eval -m gpt-4-turbo -v gpt4 -p full```
- (for replication purpose: eval only first five dilemma):  ```python eval/evaluate_model_on_dilemma.py -o eval/model_response_from_eval -m gpt-4-turbo -v gpt4 -p only_first_five```
- output: ```eval/model_response_from_eval/gpt4.csv```

2. Combine model response from 1 to get the dilemmas details for each action

- (own run) command: ```python eval/combine_model_eval_with_dilemmas_action_separate.py -i eval/model_response_from_eval/gpt4.csv -dilemma data/dilemmas_with_detail_by_action.csv -o eval/model_response_from_eval -m gpt4```
- (for replication purpose: eval only first five dilemma) command: ```python eval/combine_model_eval_with_dilemmas_action_separate.py -i eval/model_response_from_eval/gpt4.csv -dilemma data/dilemmas_with_detail_by_action_test.csv -o eval/model_response_from_eval -m gpt4```


3. Analyze the model response from 1 with deilemmas details from 2 based on the five theories
- dataset with six model responses: ```eval/model_response_from_eval/all_models_eval_on_dilemmas_with_detail_by_action.csv```
- (Replication purpose: eval only first five dilemma) command: ```python analysis/analysis_for_model_responses_five_theories/analyze_model_resp_with_five_theories.py -i eval/model_response_from_eval/gpt4_eval_on_dilemmas_with_action_separate.csv -o analysis/analysis_for_model_responses_five_theories/model_resp.csv -m gpt4```
- (Replication purpose: provided all models responses for analysis) command: ```python analysis/analysis_for_model_responses_five_theories/analyze_model_resp_with_five_theories.py -i eval/model_response_from_eval/all_models_eval_on_dilemmas_with_detail_by_action.csv -o analysis/analysis_for_model_responses_five_theories/model_resp_all_models.csv -m gpt4 gpt35 llama2 llama3 mixtral_rerun claude```

4. Plot the graph with five theories
- (Replication purpose: eval only first five dilemma) command: ```python analysis/analysis_for_model_responses_five_theories/create_plot.py -i analysis/analysis_for_model_responses_five_theories/model_resp.csv -o analysis/analysis_for_model_responses_five_theories/graph```
- (Replication purpose: provided all models responses for analysis) command: ```python analysis/analysis_for_model_responses_five_theories/create_plot.py -i analysis/analysis_for_model_responses_five_theories/model_resp_all_models.csv -o analysis/analysis_for_model_responses_five_theories/graph```

## Case 2: Evaluate model preferences on everyday dilemmas based on the given principle

1. Get the relevant values for each principle from collected values by prompting the model 10 times
- dataset on values: ```data/values.csv```
- (own run) command: ```python eval_for_system_prompt_modulation/get_values_for_principle.py -i data/principle_openai.csv -o eval_for_system_prompt_modulation/values_and_system_prompt_for_principle -m gpt-4-0125-preview -p full```
- (Replication purpose: only first two principles) command: ```python eval_for_system_prompt_modulation/get_values_for_principle.py -i data/principle_openai.csv -o eval_for_system_prompt_modulation/values_and_system_prompt_for_principle -m gpt-4-0125-preview -p only_first_two_principles```


2. Calculate the value's relevance by the empirical probabiltiy from 1.
- (Replication purpose: only first two principles) command: ```python eval_for_system_prompt_modulation/calc_value_relevance.py -i eval_for_system_prompt_modulation/values_and_system_prompt_for_principle/principle_openai.csv -o eval_for_system_prompt_modulation/values_and_system_prompt_for_principle```

3. Get the values conflicts and relevant dilemmas, and corresponding model responses based on 2. and dilemmas with model responses from case 1 step 2. (For dilemmas, we use our model responses on our dilemmas dataset run by five models.)

- (Replication purpose: only first two principles) command: ```python eval_for_system_prompt_modulation/get_values_conflicts_and_relevant_dilemmas_for_principle.py -i eval_for_system_prompt_modulation/values_and_system_prompt_for_principle/principle_openai_clean.csv -o eval_for_system_prompt_modulation/values_and_system_prompt_for_principle -m gpt4 -d eval/model_response_from_eval/all_models_eval_on_dilemmas_with_detail_by_action.csv```
- output: the model preference on the relevant dilemma:  column ```dilemma_combined_score_by_sup_opp_values_calculate_with_prob``` on ```eval_for_system_prompt_modulation/values_and_system_prompt_for_principle/principle_gpt4_eval_with_values_conflicts_and_dilemma.csv```.


## Case 3: Evaluate the model's steerability by system prompt modulation on each given principle
1. Generate system prompts
- (Replication purpose: only first two principles) command: ```python eval_for_system_prompt_modulation/generate_system_prompts_for_principle.py -i eval_for_system_prompt_modulation/values_and_system_prompt_for_principle/principle_gpt4_eval_with_values_conflicts_and_dilemma.csv -o eval_for_system_prompt_modulation/values_and_system_prompt_for_principle -m gpt-4-turbo```
- output: ```eval_for_system_prompt_modulation/values_and_system_prompt_for_principle/{model}_with_system_prompts.csv```
2. Evaluate models on relevant dilemmas for each principle by each set of system prompt
- command: ```python eval_for_system_prompt_modulation/evaluate_model_on_system_prompt.py -o eval_for_system_prompt_modulation/model_response_on_system_prompt/ -m gpt-4-turbo -i eval_for_system_prompt_modulation/values_and_system_prompt_for_principle/gpt-4-turbo_with_system_prompts.csv -d data/dilemmas_with_detail_by_action.csv```

<!-- - (replication purpose) command: ```python eval_for_system_prompt_modulation/evaluate_model_on_system_prompt.py -o eval_for_system_prompt_modulation/model_response_on_system_prompt/ -m gpt-4-turbo -i data/principle_openai_gpt-4_with_system_prompt.csv -d data/dilemmas_with_detail_by_action.csv``` -->
- output: ``eval_for_system_prompt_modulation/model_response_on_system_prompt/{model}_eval.csv``
3. Analyze the model responses
- command (for steering supporting value): ```python analysis/analysis_for_system_prompt_modulation/analyze_model_resp_on_system_prompt.py -p eval_for_system_prompt_modulation/model_response_on_system_prompt/gpt-4-turbo_eval.csv -d eval/model_response_from_eval/all_models_eval_on_dilemmas_with_detail_by_action.csv -o analysis/analysis_for_system_prompt_modulation -steer sup -c openai -m gpt4```

- command (for steering opposing value): ```python analysis/analysis_for_system_prompt_modulation/analyze_model_resp_on_system_prompt.py -p eval_for_system_prompt_modulation/model_response_on_system_prompt/gpt-4-turbo_eval.csv -d eval/model_response_from_eval/all_models_eval_on_dilemmas_with_detail_by_action.csv -o analysis/analysis_for_system_prompt_modulation -steer opp -c openai -m gpt4```
- output: ```{output_file_div}/principle_{company}_{model_name}_eval_with_system_prompt_system_prompt_sup.csv```
- output:  ```{output_file_div}/principle_{company}_{model_name}_eval_with_system_prompt_system_prompt_opp.csv```

4. Create plot from the analysis file from 3.
- command: ```python analysis/analysis_for_system_prompt_modulation/create_plot.py -sup analysis/analysis_for_system_prompt_modulation/principle_openai_gpt4_eval_with_system_prompt_system_prompt_sup_value.csv  -opp analysis/analysis_for_system_prompt_modulation/principle_openai_gpt4_eval_with_system_prompt_system_prompt_opp_value.csv -o analysis/analysis_for_system_prompt_modulation/ -p only_two_principles```
- (Replication purpose with provided complete run of all principles from 3) command: ```python analysis/analysis_for_system_prompt_modulation/create_plot.py -sup data/principle_openai_gpt4_eval_with_system_prompt_system_prompt_sup_value.csv -opp data/principle_openai_gpt4_eval_with_system_prompt_system_prompt_opp_value.csv -o analysis/analysis_for_system_prompt_modulation/ -p full```
