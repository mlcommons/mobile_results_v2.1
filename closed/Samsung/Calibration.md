# Samsung’s Calibration for Post-Training  Quantization

## Calibration
Samsung quantization tool uses the MLPerf calibration datasets to extract statistics from the reference floating point neural network model for post-training quantization. The statistics collected using the calibration datasets include min, max, average, and standard deviation of the activations.
Either the full set of calibration images or a sampled subset is used.  These calibration images are written into an LMDB (database) format. The conversion scripts are part of the submission.
Specifically, resize and cropping operations are applied to these datasets. COCO is resized to 320x320. For the sake of calibration, the images of the ADE20k are resized such that the shortest size of each image is 512, then a center crop of 512x512 is used. The adjusted images are then written into an LMDB database for use in calibration.

### Calibration Datasets
The calibration sets used in this submission are as agreed upon by the MLPerf Mobile Working Group for the classification, object detection, and semantic segmentation benchmarks. The list of calibration images are shown below:

- ImageNet: https://github.com/mlcommons/mobile/blob/master/calibration/ImageNet/cal_image_list_option_1.txt
- COCO: https://github.com/mlcommons/mobile/blob/master/calibration/COCO/coco_cal_images_list.txt
- ADE20K: https://github.com/mlcommons/mobile/blob/master/calibration/ADE20K/ADE20k_calibration.txt

## Quantization
Samsung’s quantization tool performs per-channel symmetric quantization for both model weights and activations. The scaling factors used in quantizing floating point values are constrained to powers of 2.
The input to this quantization tool is the FP32 TFLite model. The output of this quantization tool is a Fixed-Point (FXP) proprietary model.

### Weights
The maximum absolute value m at the per-channel level is determined, and the quantization range of the channel becomes [-m, m]. Given the quantization granularity (# bins or # bits), the scaling factor is computed which in turn is used to quantize the weights.

### Activations
The quantization parameters are first obtained through profiling the neural network. Profiling is the process of acquiring statistics of the activations by feed-forwarding a set of calibration data. 
Examples of the statistics collected are the min, max, mean and standard deviation values of the activations at a per-channel level for each layer (e.g., for each Convolutional layer). These statistics are used to calculate quantization parameters for each channel.
For quantization, it is assumed that the activations follow a specific probabilistic distribution. Examples of the distributions considered are as follows:

-	Uniform distribution
-	Laplace distribution
-	Normal distribution
-	Logistic distribution
-	Hyperbolic secant distribution
-	Cauchy distribution

For the models quantized, the probabilistic distribution that best fits the distribution of activations in the entire neural network is found. The probabilistic distribution is selected to minimize the quantization loss compared to the floating point model.
For classification models, the quantization loss can be measured based on the differences in predictions obtained from the floating point and the quantized models. For other tasks, the quantization loss can be measured by a distance norm between feature maps of the quantized models and the floating point models.
Once the distribution is selected, a scaling factor is computed.  Scaling factors for the selected distributions are pre-computed for a variety of bit-lengths. After selecting a distribution, the pre-computed scaling factor of the selected distribution and desired bit-length is adjusted by the estimated standard deviations of the set to be quantized. This adjusted scaling factor is then used in quantizing the relevant set.

### Bias Compensation
The quantization parameters are refined to compensate any biases introduced from quantizing the network weights and activations. This is achieved by computing the expected quantization errors of both weights and activations, per layer, by comparing performances of the quantized model and the reference floating point network on the reference calibration set. The expected errors are then added to the layer bias parameters to mitigate the quantization bias.

