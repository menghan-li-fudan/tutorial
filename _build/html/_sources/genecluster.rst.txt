.. role:: python(code)
  :language: python
  :class: highlight

Spatial Co-expression Modules Identification
=============================================

To identify spatially organized gene expression patterns, we employed the ``NNMF`` package to perform non-negative factorization on the spatial
gene expression matrix, then grouped gene into modules using hierarchical clustering:

.. code-block:: shell

  Rscript run_nnmf.R --path ${prefix} --noSignatures 15 --k 6 --ntop 50

In general, we suggest setting ``--noSignatures`` to ``10~15`` based on your data, and setting ``--k``, which indicates the number of final modules
to ``4~8``, depending your data size and memory limit.

This would result in a files named ``gene-names-group.txt``, containing gene names grouped by modules they belong to.