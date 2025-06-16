# AWS X-Ray as a Sidecar Container in Amazon EKS

This document explains how to implement AWS X-Ray as a sidecar container in Amazon EKS to enable distributed tracing for your applications.

## Overview

AWS X-Ray helps you analyze and debug distributed applications by providing request tracing, exception collection, and profiling capabilities. Running X-Ray as a sidecar container in Kubernetes allows your application pods to send trace data locally to the X-Ray daemon, which then forwards it to the AWS X-Ray service.

## Prerequisites

- An Amazon EKS cluster
- AWS CLI configured with appropriate permissions
- kubectl configured to communicate with your EKS cluster

## Implementation Steps

### 1. Create IAM Permissions for X-Ray

The X-Ray daemon needs permissions to send trace data to the AWS X-Ray service:

```bash
aws iam create-policy \
  --policy-name XRayWriteAccess \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "xray:PutTraceSegments",
                "xray:PutTelemetryRecords",
                "xray:GetSamplingRules",
                "xray:GetSamplingTargets",
                "xray:GetSamplingStatisticSummaries"
            ],
            "Resource": "*"
        }
    ]
}'
```

### 2. Set up IAM Roles for Service Accounts (IRSA)

Enable the X-Ray daemon to authenticate with AWS services:

```bash
eksctl create iamserviceaccount \
  --cluster=your-cluster-name \
  --namespace=default \
  --name=xray-daemon \
  --attach-policy-arn=arn:aws:iam::YOUR_ACCOUNT_ID:policy/XRayWriteAccess \
  --approve
```

### 3. Deploy X-Ray as a Sidecar Container

Create a deployment manifest file (`xray-sidecar.yaml`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      serviceAccountName: xray-daemon  # Use the service account with X-Ray permissions
      containers:
      # Your application container
      - name: application
        image: your-application-image:latest
        ports:
        - containerPort: 8080
        env:
        - name: AWS_XRAY_DAEMON_ADDRESS
          value: "localhost:2000"
        
      # X-Ray daemon sidecar container
      - name: xray-daemon
        image: amazon/aws-xray-daemon:latest
        ports:
        - containerPort: 2000
          protocol: UDP
        resources:
          limits:
            memory: 256Mi
            cpu: 256m
          requests:
            memory: 32Mi
            cpu: 50m
---
# Service for your application
apiVersion: v1
kind: Service
metadata:
  name: sample-app
spec:
  selector:
    app: sample-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

Apply the manifest:

```bash
kubectl apply -f xray-sidecar.yaml
```

### 4. Instrument Your Application

Use the AWS X-Ray SDK for your programming language to instrument your application. Here are examples for common languages:

#### Node.js

```javascript
const AWSXRay = require('aws-xray-sdk');
const express = require('express');

const app = express();

// Instrument incoming HTTP requests
app.use(AWSXRay.express.openSegment('MyApp'));

app.get('/', function (req, res) {
  res.send('Hello World!');
});

// Close the segment
app.use(AWSXRay.express.closeSegment());
```

#### Python

```python
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
from flask import Flask

app = Flask(__name__)

# Configure X-Ray recorder
xray_recorder.configure(service='MyApp')

# Instrument incoming HTTP requests
XRayMiddleware(app, xray_recorder)

@app.route('/')
def hello_world():
    return 'Hello World!'
```

#### Java

```java
import com.amazonaws.xray.AWSXRay;
import com.amazonaws.xray.AWSXRayRecorderBuilder;
import com.amazonaws.xray.javax.servlet.AWSXRayServletFilter;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

    @Bean
    public AWSXRayServletFilter xrayFilter() {
        return new AWSXRayServletFilter("MyApp");
    }
}
```

## Verification

To verify that X-Ray is working correctly:

1. Deploy your instrumented application
2. Generate some traffic to your application
3. Check the AWS X-Ray console to see the traces

## Troubleshooting

If traces are not appearing in the X-Ray console:

1. Check that the X-Ray daemon container is running:
   ```bash
   kubectl get pods
   kubectl logs <pod-name> -c xray-daemon
   ```

2. Verify IAM permissions are correctly set up
3. Ensure your application is properly instrumented and sending traces to the daemon

## Additional Configuration

### Custom X-Ray Configuration

You can provide a custom configuration file to the X-Ray daemon by creating a ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: xray-config
data:
  xray.yaml: |-
    TotalBufferSizeMB: 64
    Socket:
      UDPAddress: "0.0.0.0:2000"
    Logging:
      LogLevel: "info"
```

Then mount it in your deployment:

```yaml
volumeMounts:
- name: xray-config
  mountPath: /etc/xray
volumes:
- name: xray-config
  configMap:
    name: xray-config
```

### Using X-Ray with AWS Distro for OpenTelemetry (ADOT)

For more advanced tracing capabilities, consider using AWS Distro for OpenTelemetry with X-Ray exporter.