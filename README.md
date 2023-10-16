# MULTITuDE: Large-Scale Multilingual Machine-Generated Text Detection Benchmark
Source code for replication of the experiments in the paper accepted to the EMNLP 2023 conference.

## Cite
If you use the data, code, or the information in this repository, cite the following paper, please.
```bibtex
@inproceedings{
  macko2023multitude,
  title={{MULTIT}u{DE}: Large-Scale Multilingual Machine-Generated Text Detection Benchmark},
  author={Dominik Macko and Robert Moro and Adaku Uchendu and Jason S Lucas and Michiharu Yamashita and Matúš Pikuliak and Ivan Srba and Thai Le and Dongwon Lee and Jakub Simko and Maria Bielikova},
  booktitle={The 2023 Conference on Empirical Methods in Natural Language Processing},
  year={2023}
}
```

## Install Dependencies
The provided Google Colab notebooks are selfcontained; however, the other python scripts have some dependencies to be installed.

(A more stable option:) Create a conda environment using the provided .yaml file.
```
conda env create --file=multitude.yaml
conda activate multitude
```
Or at least, install the following python (>=3.8) modules/packages:
```
pip install transformers accelerate bitsandbytes peft langcodes[data] nvidia-ml-py3 openai backoff
```

## Source Code Structure
The whole experiments can be run by executing scripts according to their ordinal number in the name.
| # | Description |
| :-: | :-: |
| 01 | Google Colab notebook for dataset creation. The human texts are subsampled from [MassiveSumm](https://github.com/danielvarab/massive-summ), where also author version has been requested. |
| 02 | A script to generate texts from various large language models (HuggingFace pretrained models as well as OpenAI API accessible models). |
| 03 | A script for fine-tuning of the detectors. |
| 04 | A script for inference of fine-tuned detection models to obtain predictions. |
| 05 | A script for predictions based on statistical entropy detection method with hyperparameter tuning of Random Forest Classifier. |
| 06 | Google Colab notebook for results analysis. |

### Running the Experiments
Some of the python scripts use global variables to set various directory paths at the top of the files. Ensure the directories exist, by default run:
```
mkdir -p cache
mkdir -p offload_folder
mkdir -p results
```

The MULTITuDE dataset can be downloaded from [ToDo]. If downloaded the dataset, continue with the step 4. For just analysis of the results, proceed directly to the last step.

1. For dataset reproduction, run the first part of the provided [notebook](01_dataset_creation.ipynb). This downloads the human-text data, preprocesses them, and subsamples them into to file "MassiveSumm_selected.csv".
2. For generation of machine texts for titles (headlines) available for human texts, run the script as provided in the example below, which uses GPT-3 model for text generation.
   ```
   python 02_generate_text.py <llm> <dataset>
   ```
   For experiments in the paper, the dataset is the mentioned "MassiveSumm_selected.csv" file and the `llm` argument values are the following:
   - text-davinci-003
   - gpt-3.5-turbo
   - gpt-4
   - alpaca-lora-30b
   - vicuna-13b
   - llama-65b
   - opt-66b
   - opt-iml-max-1.3b
   
   Example:
   ```
   python 02_generate_text.py "text-davinci-003" "MassiveSumm_selected.csv"
   ```
3. After generation of machine texts from all the LLMs as in the paper, run the second part of the provided [notebook](01_dataset_creation.ipynb). This preprocesses generated texts, combines them with the human texts, and removes the duplicates. These precesses result in the "multitude.csv" dataset file generated (required for further experiments).
4. Fine-tune the pre-trained LLMs for machine-generated text detection task using train split of the dataset. Run the script as provided in the example below.
   ```
   python 03_finetune_detector.py <detector-base-model> <train-language> <train-llm>
   ```
   For experiments in the paper, `detector-base-model` argument values are the following:
   - bert-base-multilingual-cased
   - roberta-large-openai-detector
   - google/electra-large-discriminator
   - gpt2-medium
   - xlm-roberta-large
   - microsoft/mdeberta-v3-base
   - ai-forever/mGPT
   
   `train-language` argument values are the following: "en", "es", "ru", "en3", "all".
   `train-llm` argument values are the same as `llm` values in the step 2. In addition, there is the "all" value for usage of data generated by all mentioned LLMs.
   
   Example that uses mDeBERTa base model and Spanish data generated by GPT-3 model along with Spanish human texts:
   ```
   python 03_finetune_detector.py "microsoft/mdeberta-v3-base" "es" "text-davinci-003"
   ```
5. Generate predictions on the whole test split of the dataset. Run the script as provided in the example below (argument values are the same as in the previous step).
   ```
   python 04_test_detector.py <detector-base-model> <train-language> <train-llm>
   ```

   Example that uses mDeBERTa base model fine-tuned on English big subset generated by all text-generation LLMs (+ human texts).
   ```
   python 04_test_detector.py "microsoft/mdeberta-v3-base" "en3" "all"
   ```
6. Generate predictions on the whole test split of the dataset for black-box and statistical methods. We do not distribute the code for those, the predictions are available in the results directory. For our hyperparameter-optimized Random Forest predictions using entropy and mGPT base model, run the following code:
   ```
   python 05_entropy_rf_tuned.py
   ```  
7. For evaluation of the results, run the provided results analysis [notebook](06_results_analysis.ipynb), which provides various insights and also follows research questions in the paper.
