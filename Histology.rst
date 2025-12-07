.. role:: python(code)
  :language: python
  :class: highlight

Histology Feature Extraction
=============================

We extract histological features using a dual-model strategy. Specifically, we employed HIPT, 
a hierarchical vision transformer that captures multi-scale tissue architecture, and UNI, 
a universal pathology foundation model pretrained across diverse histology cohorts. 
Leveraging two complementary models allows us to integrate both global contextual representations and fine-grained local morphology.

Global histological feature extraction using HIPT
------------------------------------------------------

First, download model weights of HIPT by running ``download_pretrained_vit.sh``. Then extract features with HIPT by running: 
``extract_features_vit.py``:

.. code-block:: shell

   python extract_features_vit.py ${prefix} --device='cuda'

This would result in a pickle file ``embeddings-hist-vit.pickle``, which contains global image features.

The use of GPU is highly recommended.

Fine-grained histological feature extraction using UNI
------------------------------------------------------

First, request access to the UNI model weights from the Huggingface model page at https://huggingface.co/mahmoodlab/UNI. Then extract 
features with UNI by running:

.. code-block:: shell

   python extract_features_uni.py ${prefix} --login='LOGIN'


Replace ``LOGIN`` with your own. The output is a pickle file ``embeddings-hist-uni.pickle``, which contains fine-grained image features.

The use of GPU is highly recommended.

Integration of global and fine-grained histological features
------------------------------------------------------------

Combining two complementary embeddings allows us to integrate global contextual representations (HIPT) with fine-grained local morphology (HIPT \& UNI). 
A unified histology embedding is constrcuted by applying principal component analysis (PCA) to each model's features (retaining components that explain 
at least 99% of the variance) and then concatenating the reduced embeddings. To achieve this, run ``merge_feature.py``:

.. code-block:: shell

   python merge_feature.py ${prefix} --method='pca'

The result is a pickle file named ``embeddings-hist-merged.pickle``, which serves as histological features for downstream training, validation and prediction.

