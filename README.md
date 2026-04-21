<h1>Get Started</h1>
<h2>1. Required Packages</h2>
请确保正在路径Lora-RAG下，Please install the required packages for our model via:

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
Load the image data by the following code. If encountering network problems, please download directly from [Kaggle](https://www.kaggle.com/datasets/raddar/chest-xrays-indiana-university).

```Shell
cd ../data
python download.py
```

The text data has been preprocessed and is ready for immediate use. The image paths and corresponding text data should be organized into separate JSON files for training and testing sets.

```Shell
python report2json.py --report_path ./report.csv --seed 42
```

<h2>3. Training</h2>
The training hyperparameters (epochs, batch size, gradient accumulation steps, LoRA rank, and alpha) can be adjusted according to available computational resources. For stable operation using the following parameters, verify that your GPU has **a minimum of 70GB memory**.

```Shell
cd ..
python train_rag.py --model_path microsoft/llava-med-v1.5-mistral-7b --json_file ./data/train_report.json --image_dir ./data/2/images/images_normalized --output_dir ./rag_weight --epochs 10 --batch_size 1 --gradient_accumulation_steps 64 --lora_r 64 --lora_alpha 128 --lr 2e-4
```

<h2>4. Evaluation</h2>
本文对训练好后的RAG模型进行测试评估，并对三种模型进行了对比，分别是基础模型、纯lora训练模型、加入rag训练模型。并在测试的过程中分别对三种模型加减rag推理过程进一步分析rag的作用影响。

<h3>(1) 下载纯lora训练模型</h3>

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

<h3>(2) 基础评估</h3>
没有RAG推理过程的llava-med的评估结果详见：

使用RAG推理的llavamed评估运行如下：
```Shell
cd ..
python base_llava.py \--json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--csv_output pretrained_with_rag.csv \--use_rag \--rag_top_k 3
```
没有RAG推理过程的纯Lora的评估结果详见：

使用RAG推理的纯Lora评估运行如下：
```Shell
python base_eval.py \--lora_path ./lora_final3 \--json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--csv_output lora_with_rag_results.csv \--use_rag \--rag_top_k 3
```

使用Rag权重但不添加Rag推理过程：
```Shell
python base_eval.py \--lora_path ./rag_weight/checkpoint-epoch-3 \--json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--csv_output with_rag_results.csv \--use_rag \--rag_top_k 3
```
使用Rag权重并添加Rag推理过程：
```Shell
python base_eval.py \--lora_path ./rag_weight/checkpoint-epoch-3 \--json_file ./data/test_report.json 
```

<h3>(3) 功能性评估 </h3>
每运行一个代码文件都会同时给到该模型有无RAG推理过程的结果对比。

LLava-med：
```Shell
python functional_llava.py \--json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--rag_top_k 3 \--output_prefix functional_test_pretrained
```
纯Lora权重：
```Shell
python functional_test.py \--lora_path ./rag_weigt/lora_final3 \--json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--rag_top_k 3 \--output_csv functional_test_rag.csv 
```

RAG权重：
```Shell
python functional_test.py \--lora_path ./rag_weigt/checkpoint-epoch-2 \--json_file ./data/test_report.json \--image_dir ./data/2/images/images_normalized \--rag_top_k 3 \--output_csv functional_test_rag.csv
```

<h3>(4) 使用LLM评估</h3>
本文使用的评估大模型是Qwen-Max

抽取样本（也可以全部测试，这里因为大模型免费额度问题只抽去50个样本）
```Shell
python extract_samples.py
```

下载Qwen使用条件
```Shell
Pip install dashscope
```

LLava-med：

使用基础llava-med权重但不使用RAG推理过程
```Shell
python llm_llava_no_raginfer.py
```
使用基础llava-med权重并添加RAG推理过程
```Shell
python llm_llava_with_raginfer.py
```

纯Lora权重：

使用纯Lora权重但不使用RAG推理过程
```Shell
python llm_lora_no_raginfer.py
```
使用纯Lora权重并添加RAG推理过程
```Shell
python llm_lora_with_raginfer.py
```

RAG权重：

使用Rag权重但不使用RAG推理过程
```Shell
python llm_rag_no_raginfer.py
```
使用Rag权重并添加RAG推理过程
```Shell
python llm_rag_with_raginfer.py
```


