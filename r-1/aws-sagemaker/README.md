# r-1 on AWS SageMaker

[`r-1-model-package.ipynb`](r-1-model-package.ipynb) creates a model from the r-1 model package, deploys a real-time endpoint, runs a batch transform on the [samples](../samples), and deletes the resources.

Prerequisites:

- A subscription to the r-1 listing in AWS Marketplace.
- An IAM role with SageMaker access (the notebook uses `sagemaker.get_execution_role()`).
- Quota for one `ml.g6e.2xlarge` for endpoint usage and one for transform job usage.
- The SageMaker Python SDK v2 (`pip install "sagemaker<3"`).
