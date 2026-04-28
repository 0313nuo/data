<h1>Get Started</h1>
<h2>1. Required Packages</h2>
Please ensure you are in the CEAVP directory. Install the required packages for our model using:

```Shell
pip install -r requirements.txt
```

<h2>2. Training</h2>
The training hyperparameters (epochs, batch size, learning rate etc.) can be adjusted according to available computational resources. For stable operation using the following parameters, verify that your GPU has **a minimum of 20GB of memory**.

```Shell
cd ..
python main.py --batch_size 8 --lr_encoder 5e-5 --lr_classifier 5e-4 --lr_regressor 5e-4
```

<h2>3. Evaluation</h2>
All training results and test evaluation results are in the subject_results directory.

