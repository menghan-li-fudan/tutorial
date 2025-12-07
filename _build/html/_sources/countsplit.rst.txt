.. role:: python(code)
  :language: python
  :class: highlight

Validation Set Construction
=============================

To address the common lack of replicates in spatial transcriptomics, where the entire spot-level dataset is often used for training without a hold-out set,
we adopt a ``count splitting`` approach: 
Given a spot-level count matrix :math:`\mathbf{Y}=[Y_{s,g}]\in\mathbb{N}_0^{S\times G}`, we partition each entry as
:math:`Y_{s,g}^{(\mathrm{train})}\sim\mathrm{Binomial}\big(2Y_{s,g}, \tfrac{1}{2}\big)`, :math:`Y_{s,g}^{(\mathrm{val})}=2Y_{s,g}-Y_{s,g}^{(\mathrm{train})}`.

Under a Poisson model assumption, count splitting ensures that :math:`Y_{s,g}^{(\mathrm{train})}` and :math:`Y_{s,g}^{(\mathrm{val})}` 
follow the same distribution as :math:`Y_{s,g}`, and are conditionally independent given their mean value.

This construction therefore yields statistically independent training and validation sets suitable for unbiased assessment of predictive performance.

To generate training and validation set, run the following code:

.. code-block:: shell

   Rscript generate_count_split.R $prefix $cnts_train_name $cnts_val_name $seed 

``$cnts_train_name`` indicates the name of the training set, ``$cnts_val_name`` indicates the name of the validation set.
``$seed`` should be a positive integar, indicating the random seed used. 

This would result in two files (`cnts_train_1.csv` and `cnts_val_1.csv`) containing the training and validation data, respectively.