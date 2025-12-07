.. role:: python(code)
  :language: python
  :class: highlight

Single-cell Reference Guided Gene Expression Embedding
========================================================

A single-cell reference provides complementary molecular context beyond histology and can improve prediction. 
Below, we describe how we construct a gene-expression embedding from the single-cell reference.

Cell-type deconvolution using RCTD
------------------------------------------------------

First, you need to build a single-cell reference (paired with ST data or from similar tissues) with cell type labels saved as ``celltype`` in ``metadata``.
Save the reference as ``sc_reference.RDS``. **This step is not included in** ``spEnhance`` **workflow**.

Next, we computed the cell type-by-gene reference matrix :math:`\mathbf{R} \in\mathbb{R}^{C\times G}`, 
where each entry :math:`R_{c,g}` is the mean expression of gene :math:`g` across all cells of type :math:`c`. 

Then we apply ``RCTD`` to deconvolve each spot, yielding :math:`\mathbf{\Pi}=[\pi_{s,c}]\in[0,1]^{S\times C}`, 
which estimates the fraction of each cell type present in spot :math:`s = 1, \dots, S` (rows summing to one).

To do this, run ``run_rctd.R``:

.. code-block:: shell

   Rscript run_rctd.R ${prefix}sc_reference.RDS ${prefix}cnts_train_seed_1.csv ${prefix}locs.csv ${prefix} 4 

Parameters used:
   + ``${prefix}sc_reference.RDS``: directory to the single cell reference. You can replace it with your own directory.
   + ``${prefix}cnts_train_seed_1.csv``: directory to the count matrix used for deconvolution.
   + ``${prefix}cnts_train_seed_1.csv``: directory to the location matrix. It must be paired with the previous count matrix.
   + ``${prefix}``: directory to save the results.
   + ``4``: number of cores used.

The output files from this step include:
   + ``reference.csv``: cell type-by-gene reference matrix calculated using single-cell reference.
   + ``proportion_celltype.csv``: cell type proportion matrix, containing proportion of each cell type in each spot.
   + ``locs_celltype.csv``: location of spots, paired with spots in ``proportion_celltype.csv``.


Pixel-level cell type prediction
------------------------------------------------------

To predict pixel-level cell types, we train a graph convolutional network (GCN) that maps the histology embedding 
:math:`\mathbf{U}` to pixel-wise probabilities :math:`\mathbf{P} \in[0,1]^{H_1\times W_1\times C}`, using the spot-level deconvolution :math:`\mathbf\Pi` for weak supervision.

First, save cell type names to ``cell-type-names.txt``, with each row being a cell type. Afterwards, run ``impute_slide_celltype.py``:

.. code-block:: shell

   python impute_slide_celltype.py ${prefix} --epochs=100 --device='cuda' --n_states=5

Parameters used:
   + ``${prefix}``: directory to data
   + ``--epochs``: number of epochs in the training process
   + ``--device``: device used for training and prediction. The use of GPU is strongly recommended
   + ``--n_states``: number of states (number of independent models trained, validated and used for prediction)
We suggest setting ``epochs`` to ``50-150``. Decide this based on cell type prediction and enhancement results.

The use of GPU is highly recommended.

Gene expression feature assignment
------------------------------------------------------------

Given the cell type-by gene reference matrix :math:`\mathbf{R}` and the pixel-level cell-type probabilities :math:`\mathbf{P}`, we assign a
gene-expression value at pixel :math:`(h,w)` via :math:`V_{h,w,g}^{(0)} \;=\; \sum_{c=1}^C P_{h,w,c}\, R_{c,g}`.
Collecting all genes and pixels forms the tensor :math:`\mathbf{V}^{(0)} \in \mathbb{R}^{H_1 \times W_1 \times G}`.

To extract compact representations, we reduce the gene dimension using truncated
SVD to obtain the gene feature embedding :math:`\mathbf{V} \in \mathbb{R}^{H_1 \times W_1 \times G_1}`.

.. code-block:: shell

   python assign_reference.py ${prefix} --mode='combined' --normalize='gene-zscore' --dim=256

The result is a pickle file named ``embeddings-combined.pickle``, which combines both gene expression features and histological features from the previous step.