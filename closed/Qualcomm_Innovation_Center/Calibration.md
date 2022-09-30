## **Post-Training Quantization**
SNPE uses following post-training quantization procedures. For models, where ml-perf reference INT8 models are used, only activation quantization is applicable (weights are used from the reference model).
In general, uniform, asymmetric, per-tensor quantization is used for both weights and activations.

## Weights and Biases
Each weight tensor is quantized using the standard tensorflow quantization method described in the [SNPE documentation](https://developer.qualcomm.com/sites/default/files/docs/snpe/quantized_models.html). CLE is then applied which scales convolution weights over sets of convolution layers to equalize the weight ranges. During quantization biases are adjusted to correct the error bias due to weight quantization. Both techniques are outlined in the “Cross-layer equalization” and “Bias Correction” sections outlined in “Data-Free Quantization Through Weight Equalization and Bias Correction” [1].

## Activations
For each activation tensor, the dynamic range is determined based on min and max values obtained when running inference over a set of representative input dataset (e.g., calibration images). This dynamic range sets the clipping boundaries for the uniform quantizer. In addition, activation quantization optimizations are applied when layer activation encodings can be propagated between math-invariant layers so as to reduce the incidences of requantization within the graph.


### References
[1] M. Nagel, M. Baalen, T. Blankevoort, M. Welling, Data-Free Quantization Through Weight Equalization and Bias Correction, ICCV’19 (https://arxiv.org/pdf/1906.04721.pdf)
