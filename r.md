<h1>Get Started</h1>
<h2>1. Required Packages</h2>
Please ensure you are in the Lora-RAG directory. Install the required packages for our model using:

```Shell
pip install -r requirements.txt
```

<h2>2. LLaVA-Med Deployment</h2>

```Shell
cd LLaVA-Med
pip install --upgrade pip
pip install --user -e .
```

<h2>3. Dataset</h2>
<h3>(1) Description</h3>
To achieve the research objectives, this study utilizes the IU X-Ray Dataset. Collected retrospectively between 2011 and 2018 by researchers at Indiana University Health from two large hospital systems within Indiana's patient care network, this dataset was specifically constructed for chest X-ray image understanding and report generation tasks.

<h3>(2) Download and Transform</h3>
Load the image data using the following code. If you encounter network issues, please download directly from [Kaggle](https://www.kaggle.com/datasets/raddar/chest-xrays-indiana-university).

```Shell
cd ../data
python download.py
```

The text data has been preprocessed and is ready for immediate use. The image paths and corresponding text data should be organized into separate JSON files for the training and testing sets.

```Shell
python report2json.py --report_path ./report.csv --seed 42
```

<h2>3. Training</h2>
The training hyperparameters (epochs, batch size, gradient accumulation steps, LoRA rank, and alpha) can be adjusted according to available computational resources. For stable operation using the following parameters, verify that your GPU has **a minimum of 20GB of memory**.

```Shell
cd ..
python train_rag.py --model_path microsoft/llava-med-v1.5-mistral-7b --json_file ./data/train_report.json --image_dir ./data/2/images/images_normalized --output_dir ./rag_weight --epochs 10 --batch_size 1 --gradient_accumulation_steps 64 --lora_r 64 --lora_alpha 128 --lr 2e-4
```

<h2>4. Evaluation</h2>
This paper evaluates the trained RAG model and compares three variants: the baseline model, the pure LORA-trained model, and the model trained with RAG. During testing, the RAG inference process was added to or removed from each of the three models to further analyze the impact of RAG.

<h3>(1) Download the pure LORA-trained model</h3>

```Shell
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
git lfs install
```

```Shell
cd ..
git lfs clone https://github.com/H1963977384/LoRA_Weight.git
cd LoRA_Weight
unzip lora_final4.zip
```

<h3>(2) Baseline Evaluation</h3>
For evaluation results of llava-med without RAG inference, see: https://github.com/H1963977384/A-Fine-Tuned-LLaVA-Med-Framework-for-Automatic-Chest-X-ray-Report-Generation.git 

The evaluation run of llava-med using RAG inference is as follows:
```Shell
cd ..
python base_llava.py \- -json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--csv_output pretrained_with_rag.csv \--use_rag \--rag_top_k 3
```
For evaluation results of pure LoRA without RAG reasoning, see: https://github.com/H1963977384/A-Fine-Tuned-LLaVA-Med-Framework-for-Automatic-Chest-X-ray-Report-Generation.git 

The evaluation run for pure LoRA using RAG reasoning is as follows:
```Shell
python base_eval.py \--lora_path ./lora_final3 \--json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--csv_output lora_with_rag_results.csv \--use_rag \--rag_top_k 3
```

Using RAG weights without adding the RAG reasoning process:
```Shell
python base_eval.py \--lora_path ./rag_weight/checkpoint-epoch-3 \--json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--csv_output with_rag_results.csv \- -use_rag --rag_top_k 3
```
Using RAG weights and including the RAG inference process:

```Shell
python base_eval.py --lora_path ./rag_weight/checkpoint-epoch-3 --json_file ./data/test_report.json 
```

<h3>(3) Functional Evaluation </h3>
Each time a script is run, it provides a comparison of the model's results with and without the RAG inference process.

LLava-med:
```Shell
python functional_llava.py --json_file ./data/test_report.json --image_dir ./data/2/images/images_normalized --rag_top_k 3 --output_prefix functional_test_pretrained
```
Pure LORA Weights:
```Shell
python functional_test.py \--lora_path ./rag_weigt/lora_final3 \--json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--rag_top_k 3 \--output_csv functional_test_rag.csv 
```

RAG weights:
```Shell
python functional_test.py --lora_path ./rag_weights/checkpoint-epoch-2 --json_file ./data/test_report.json --image_dir ./data/2/images/images_normalized --rag_top_k 3 --output_csv functional_test_rag.csv
```

<h3>(4) Evaluating with an LLM</h3>
The large model used for evaluation in this article is Qwen-Max

Extracting samples (you can also test all of them; here, due to the large model’s free quota, only 50 samples were extracted)
```Shell
python extract_samples.py
```

Prerequisites for downloading Qwen
```Shell
pip install dashscope
```

LLava-med:

Use the base LLava-med weights without the RAG inference process
```Shell
python llm_llava_no_raginfer.py
```
Use the base LLava-med weights with the RAG inference process
```Shell
python llm_llava_with_raginfer.py
```

Pure Lora Weights:

Use pure Lora weights but do not use the RAG inference process
```Shell
python llm_lora_no_raginfer.py
```
Use pure Lora weights and add the RAG inference process
```Shell
python llm_lora_with_raginfer.py
```

RAG weights:

Use RAG weights but do not use the RAG inference process
```Shell
python llm_rag_no_raginfer.py
```
Use RAG weights and add the RAG inference process
```Shell
python llm_rag_with_raginfer.py
```


