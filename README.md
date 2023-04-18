# H2O LLM Studio

<h3 align="center">
    <p>Welcome to H2O LLM Studio, a framework and no-code GUI designed for<br />
    fine-tuning state-of-the-art large language models (LLMs).
</p>
</h3>


**With LLM Studio, you can**

- easily and effectively fine-tune LLMs **without the need for any coding experience**.
- use a **graphic user interface (GUI)** specially designed for large language models.
- finetune any LLM using a large variety of hyperparameters.
- use recent finetuning techniques such as [Low-Rank Adaptation (LoRA)](https://arxiv.org/abs/2106.09685) and 8-bit model training with a low memory footprint.
- use advanced evaluation metrics to judge generated answers by the model.
- track and compare your model performance visually. In addition, [Neptune](https://neptune.ai/) integration can be used.
- chat with your model and get instant feedback on your model performance.
- easily export your model to the [Hugging Face Hub](https://huggingface.co/) and share it with the community.


## Setup
LLM Studio requires a machine with Ubuntu 16.04+ and at least one recent Nvidia GPU with Nvidia drivers version >= 470.57.02. For langer models, we recommend at least 24GB of GPU memory.


To get started with LLM Studio, you'll need to install Python 3.10 if you don't have it on your machine already.
### System installs (Python 3.10)
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.10
sudo apt-get install python3.10-distutils
curl -sS https://bootstrap.pypa.io/get-pip.py | python3.10
```
### Create virtual environment (pipenv)
The following command will create a virtual environment using pipenv and will install the dependencies using pipenv:
```bash
make setup
```

## Run LLM Studio GUI

You can start LLM Studio using the following command:
```bash
make wave
```
This command will start the [H2O wave](https://github.com/h2oai/wave) server and app.
Navigate to http://localhost:10101/ (we recommend using Chrome) to access LLM Studio and start fine-tuning your models!

## Run LLM Studio with command line interface (CLI)
You can also use LLM Studio with the command line interface (CLI) and specify the configuration file that contains all the experiment parameters. 
To finetune using LLM Studio with CLI, activate the pipenv environment by running `make shell`, and then use the following command:
```bash
python train.py -C {path_to_config_file}
```

To run on multiple GPUs in DDP mode, run the following command:
```bash
bash distributed_train.sh {NR_OF_GPUS} -C {path_to_config_file}
```

By default, the framework will run on the first `k` GPUs. If you want to specify specific GPUs to run on, use the `CUDA_VISIBLE_DEVICES` environment variable before the command.

To start an interactive chat with your trained model, use the following command:
```bash
python prompt.py -e {experiment_name}
```
where `experiment_name` is the output folder of the experiment you want to chat with (see configuration).
The interactive chat will also work with model that were finetuned using the UI.

## Data Format
LLM studio expects a csv file with at least two columns, one being the instruct column, the other 
being the answer that the model should generate. You can also provide an extra validation dataframe using the same format or use an automatic train/validation split to evaluate the model performacen. 


During an experiment you can adapt the data representation with the following settings 

- **Prompt Column:** The column in the dataset containing the user prompt.
- **Answer Column:** The column in the dataset containing the expected output.
- **Text Prompt Start:** Text to be added before the user prompt.
- **Text Answer Separator:** The separator used between the prompt and the answer in the dataset.
- **Prepend Column Name:** Whether to add the column name to the text input from the left. As an example, if the prompt and answer columns are named "Question" and "Answer" enabling this option would add "Question: " before the question and "Answer: " before the answer. 
- **Add Eos Token To Answer:** Whether to add an explicit end-of-sequence token at the end of the answer.

### Example data:
We provide an example dataset (converted dataset from [OpenAssistant/oasst1](https://huggingface.co/datasets/OpenAssistant/oasst1))
that can be downloaded [here](https://www.kaggle.com/code/philippsinger/openassistant-conversations-dataset-oasst1?scriptVersionId=126228752). It is recommended to use `train_full.csv` for training.

## Training your model

With H2O LLM Studio, training your large language model is easy and intuitive. 
First, upload your dataset and then start training your model.

### Starting an experiment
H2O LLM Studio provides various parameters to set for a given experiment, with some of the most important being:

- **LLM Backbone**: This parameter determines the LLM architecture to use.
- **Mask Prompt Labels**: This option controls whether to mask the prompt labels during training and only train on the loss of the answer.
- **Hyperparameters** such as learning rate, batch size, and number of epochs determine the training process.
An overview of all parameters is given in the [parameter description](docs/parameters.md).
- **Evaluate Before Training** This option lets you evaluate the model before training, which can help you judge the quality of the LLM backbone before fine-tuning.

We provide several metric options for evaluating the performance of your model.
In addition to the BLEU score, we offer the GPT3.5 and GPT4 metrics that utilize the OpenAI API to determine whether 
the predicted answer is more favorable than the ground truth answer. 
To use these metrics, you can either export your OpenAI API key as an environment variable before starting LLM Studio,
or you can specify it in the Settings Menu within the UI.

### Monitoring the experiment
During the experiment, you can monitor the training progress and model performance in several ways:

- The **Charts** tab displays train/validation loss, metrics, and learning rate.
- The **Train Data Insights** tab shows you the first batch of the model to verify that the input data representation is correct.
- The **Validation Prediction Insights** tab displays model predictions for random/best/worst validation samples. This tab is available after the first validation run.
- **Logs** and **Config** show the logs and the configuration of the experiment.
- **Chat** tab lets you chat with your model and get instant feedback on its performance. This tab becomes available after the training is completed.

### Push to Huggingface 🤗
If you want to publish your model, you can export it with a single click to the [Hugging Face Hub](https://huggingface.co/)
and share it with the community. To be able to push your model to the Hub, you need to have an API token with write access.

### Compare experiments
In the **View Experiments** view, you can compare your experiments and see how different model parameters affect the model performance.
In addition, you can track your experiments with [Neptune](https://neptune.ai/) by enabling neptune logging when starting an experiment.

## Example: Run on OASST data via CLI

As an example, you can run an experiment on the OASST data via CLI.

First, get the data [here](https://www.kaggle.com/code/philippsinger/openassistant-conversations-dataset-oasst1?scriptVersionId=126228752) and place it into the `examples/data_oasst1` folder; or download it directly via API command:
```bash
kaggle kernels output philippsinger/openassistant-conversations-dataset-oasst1 -p examples/data_oasst1/
```

First, go into the interactive shell:
```bash
make shell
```

Then, you can run the experiment via:
```bash
python train.py -C examples/cfg_example_oasst1.py
```

After the experiment finishes, you can find all output artifacts in the `examples/output_oasst1` folder.
You can then use the `prompt.py` script to chat with your model:
```bash
python prompt.py -e examples/output_oasst1
```


## License
H2O LLM Studio is licensed under the Apache 2.0 license. Please see the [LICENSE](LICENSE) file for more information.