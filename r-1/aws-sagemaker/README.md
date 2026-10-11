# r-1 on AWS SageMaker

[`r-1-model-package.ipynb`](r-1-model-package.ipynb) creates a model from the r-1 model package, deploys a real-time endpoint, sends both request bodies, handles a page that r-1 cannot process, runs a batch transform on the [samples](../samples), and deletes the resources.

Prerequisites:

- An accepted private offer for r-1 from Reducto. See [Access](../README.md#access).
- An IAM role with SageMaker access (the notebook uses `sagemaker.get_execution_role()`).
- Quota for one `ml.g6e.2xlarge` for endpoint usage and one for transform job usage.
- The SageMaker Python SDK v2 (`pip install "sagemaker<3"`).
