.. role:: python(code)
  :language: python
  :class: highlight

Image Preprocessing
===================

spEnhance takes the H&E stained image paired with spot-level spatial transcriptomics data as one of the inputs.

Image rescaling
---------------

To enable consistent analysis across whole-slide histological images acquired at varying resolutions, we first rescale each raw slide so that one pixel corresponds to a physical area of :math:`0.5\times0.5\ \mu m^{2}`. 
To rescale the image, save the H&E stained image as ``he-raw.png``, the raw pixel size of the image as ``pixel-size-raw.txt``, and save the target pixel size (default:0.5) as ``pixel-size.txt``. Then run ``rescale.py``:

.. code-block:: shell

   python rescale.py ${prefix} --image

This would result in a rescaled image ``he-scaled.jpg``, with pixel size of :math:`0.5\ \mu m`.

Image padding
---------------
Next, for compatibility with subsequent feature extraction, images were further padded so that both width and height are divisible by 224. To achieve this, run ``preprocess.py``:

.. code-block:: shell

   python preprocess.py ${prefix} --image

The output is a padded image ``he.jpg``, with width and height divisible by 224. This image would be used for downstream feature extraction.