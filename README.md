# Evaluating the Robustness of Cloud Auto-AI Models against Adversarial Attacks

**Collaborators: Aashka Trivedi and Anya Trivedi**

This project aims to compare the performance of off-the-shelf Cloud Auto-AI services when exposed to adversarial examples. Various cloud providers such as IBM cloud or Google Cloud Platorm (GCP)house powerful ”Auto-AI” platforms that allow users to build world-class Machine Learning models without much ML expertise. In particular, we aim to see how robust these Auto-AI models are towards adversarial examples in the Image Classification domain, and perform a comparative study between the two cloud platforms’ models.

We aim to train Auto-AI models from the two cloud platforms to perform image classification on the MNIST-10 dataset (a hand-written digit recognition dataset with 10 classes). We then generate adversaries for the models for each dataset using the AdverTorch framework. We measure the percentage of mis-classified examples to gauge the robustness of the model.

## IBM Cloud

### Implementation Details

IBM Cloud's Auto-AI takes in CSV files as inputs, not images. We thus use the MNIST training and test data provided by PyTorch, which we save in csv format. This is done in the later half of the `code/AdversarialExampleGeneration.ipynb` notebook. The Training and Testing CSV files are `mnist_train_pytorch.csv` and `mnist_test_pytorch.csv` respectively.

The best pipeline's notebook and the experiment notebook are saved in `code/ibmcloud_autoai/MNIST_p12.ipynb` and `code/ibmcloud_autoai/MNIST_experiment.ipynb` respectively.

### Observations

1. AutoAI picks the top 3 algorithms (LGBM, XGB, Gradient Boosting), and builds 4 pipelines (with different hyperparameters) with each algorithm
2. The best model is selected as Pipeline 12, which is an Gradient Boosting Classifier with "HPO-1" and "HPO-2" Optimization with Feature Extraction, which has an accuracy score of 0.919 on the validation set, and 0.996 on the test set.

## Google Cloud Platform

### Implementation Details

GCP's AutoML Tables takes in CSV files as inputs, for images AutoMl's Vision can be used. In order for consistency, we use the MNIST training and test data from `mnist_train_pytorch.csv` and `mnist_test_pytorch.csv` respectively. The same data is used for training in IBMCloud.

All work related to both training the model, as well as testing (on MNIST testing data as well as adversarial examples) is done `code/gcp_automl/gcp_adversarial.ipynb`.

### Observations

1. Table trains the model based on the training data, on a Jupyter notebook housed in AI Platform using Cloud Storage Buckets, all linked to the same project. Model details are not provided by GCp, and performance is meausred using Log Loss

## Adversarial Example Generation

To generate the adversarial examples, we follow [PyTorch's Tutorial](https://pytorch.org/tutorials/beginner/fgsm_tutorial.html) that uses FGSM to generate adversarial examples. We use epsilon values of 0.10, 0.15, 0.20, 0.25 and 0.30 to conduct the attacks, whose code is in `code/AdversarialExampleGeneration.ipynb`.

The goal of this project is to test the adversarial robustness of AutoAi models of different cloud services. Because these models tend to not match PyTorch model configurations, we use a LeNet Model trained on MNIST to generate adversarial examples. We use the *transferrability property* that examples that are adversarial on one model will be adversarial on another. We use the LeNet checkpoint trained by PyTorch, which is stored in `models/lenet_mnist_model.pth` for the same.

The generated adversarial examples have been stored in csv format as `adv_examples_15.csv` (for epsilon=0.15) and `adv_examples_30.csv` (for epsilon=0.30). It is observed that for higher epsiolons, the parent LeNet model has poorer accuracy, but are also more apparently distorted to the human eye.

## Adversarial Robustness

The AutoAI model generated by IBMCloud's AutoAI has a 99.6% accuracy on non-adversarial examples (i.e., the test set with epsilon=0). However, this model is *not at all* robust to adversarial targetting. Using the FGSM algorithm, the model has an accuracy of only 10.49% when epsiolon is 0.15, and an accuracy of only 8.90% when epsilon is 0.30. The actual labels and the predicted labels are stored in `data_compressed/ibm_predictions/adv15_predictions.csv` and `data_compressed/ibm_predictions/adv30_predictions.csv`. The robustness of the IBM generated AutoAi model has been analysed in `code/ibmcloud_autoai/AutoAIAdversarialRobustness.ipynb`.

The AutoAI model generated by GCP's AutoML Tables has a 98.6% accuracy on non-adversarial examples (i.e., the test set with epsilon=0). However, this model is *not at all* robust to adversarial targetting. Using the FGSM algorithm, the model has an accuracy of only 18.07% when epsiolon is 0.15, and an accuracy of only 10.81% when epsilon is 0.30. The actual labels and the predicted labels are stored in `data_compressed/gcp_scored/adv_15_scored_pred.csv` and `data_compressed/gcp_scored/adv_15_scored_pred.csv`. The robustness of the GCP generated AutoML model has been analysed in `code/gcp_automl/gcp_adversarial.ipynb`.

## Defensive Distillation

To test a pipeline to defensively distil the cloud AutoAI models, we distil them into a simple LeNet based architecture, in `code/DefensiveDistilation.ipynb`. We see that a model distilled using the IBM AutoAI model as a teacher performs better than the original model under adversarial attacks, but is not as robust as the same student architecture distilled using the GCP AutoML model as a teacher. However, the performance still degrades under adversarial attack, so it may not be useful to use only defensive distillation as a means of defense.

## Major Findings

1. Training AutoAI models on the cloud is extremely easy and intuitive, and results in very high performing models.
2. AutoAI models are not robust against adversarial attacks, performing no better than random guessing when presented with images having large perturbations. Additionally, better performance on the test set does not mean more robustness to adversarial attacks.
3. Defensive distillation helps improve robustness to an extent, but cannot be use as a measure of defense against adversarial attacks on its own.

## Code Organisation

```
cloud-auto-ai-adversarial-robustness
 ┣ code
 ┃ ┣ gcp_automl
 ┃ ┃ ┗ gcp_adversarial.ipynb
 ┃ ┣ ibmcloud_autoai
 ┃ ┃ ┣ AutoAIAdversarialRobustness.ipynb
 ┃ ┃ ┣ MNIST-Experiment.ipynb
 ┃ ┃ ┗ MNIST-P12.ipynb
 ┃ ┣ AdversarialExampleGeneration.ipynb
 ┃ ┗ DefensiveDistillation.ipynb
 ┣ data_compressed
 ┃ ┣ distillation
 ┃ ┃ ┣ gcp_distil_test.csv.zip
 ┃ ┃ ┣ gcp_distil_train.csv.zip
 ┃ ┃ ┣ scoring_test.csv.zip
 ┃ ┃ ┗ scoring_train.csv.zip
 ┃ ┣ gcp_scored
 ┃ ┃ ┣ README.md
 ┃ ┃ ┣ adv_15_scored_pred.csv.zip
 ┃ ┃ ┣ adv_30_scored_pred.csv.zip
 ┃ ┃ ┣ mnist_test_scored_pred.csv
 ┃ ┃ ┗ mnist_test_scored_pred.csv.zip
 ┃ ┣ ibm_predictions
 ┃ ┃ ┣ adv15_predictions.csv.zip
 ┃ ┃ ┗ adv30_predictions.csv.zip
 ┃ ┣ adv_examples_10.csv.zip
 ┃ ┣ adv_examples_15.csv.zip
 ┃ ┣ adv_examples_15_gcp.csv.zip
 ┃ ┣ adv_examples_20.csv.zip
 ┃ ┣ adv_examples_25.csv.zip
 ┃ ┣ adv_examples_30.csv.zip
 ┃ ┣ adv_examples_30_gcp.csv.zip
 ┃ ┣ mnist_test_pytorch.csv.zip
 ┃ ┗ mnist_train_pytorch.csv.zip
 ┣ models
 ┃ ┗ lenet_mnist_model.pth
 ┣ reports
 ┃ ┣ figs
 ┃ ┃ ┣ acc_ep.png
 ┃ ┃ ┣ adv_ex.png
 ┃ ┃ ┣ adv_robustness.png
 ┃ ┃ ┣ distil_flowchart.png
 ┃ ┃ ┗ distillation.png
 ┃ ┣ ProjectPresentation.pdf
 ┃ ┣ ProjectProposal.pdf
 ┃ ┗ ProjectReport.pdf
```

## Useful Links

- [IBM Cloud AutoAI Tutorial](https://developer.ibm.com/tutorials/generate-machine-learning-model-pipelines-to-choose-the-best-model-for-your-problem-autoai/)
- [IBM Cloud AutoAI Code Generation](https://github.com/IBM/AutoAI-code-generation)
- [PyTorch Adversarial Attack Generation](https://pytorch.org/tutorials/beginner/fgsm_tutorial.html)
- [MNIST to CSV Conversion](https://pjreddie.com/projects/mnist-in-csv/)
- [AdverTorch](https://github.com/BorealisAI/advertorch)
- [AutoML Tables Guides](https://cloud.google.com/automl-tables/docs/how-to)
- [AutoML Tables Training Models](https://cloud.google.com/automl-tables/docs/train#opt-obj)
- [AutoML Batch Predictions](https://cloud.google.com/automl-tables/docs/predict-batch)
- [Defensive Distillation](https://github.com/hiaghosh/Defensive-Distillation/blob/master/models/mnist/mnist.py)
