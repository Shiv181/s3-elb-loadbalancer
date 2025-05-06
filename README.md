# S3 Multi-Bucket Load Balancer (Elastic Load Balancer)

![Project Overview](images/s3-elb-card.png)

## Overview

This solution demonstrates how to achieve ultra-high availability for static website hosting by distributing incoming traffic across multiple Amazon S3 bucket origins using an **Elastic Load Balancer (ELB)**. In the event of an origin failure, the load balancer seamlessly shifts traffic to healthy buckets.

## Objectives

* **Distribute** HTTP(s) traffic across multiple S3 bucket endpoints
* **Ensure high availability** through ELB health checks and failover
* **Maintain secure delivery** with TLS termination at the load balancer

## Architecture Diagram

```text
User → Elastic Load Balancer → S3 Bucket A (Primary)
                                ↳ S3 Bucket B (Secondary)
                                ↳ S3 Bucket C (Tertiary)
```

## Prerequisites

* AWS account with permissions for ELB and S3 operations
* At least two AWS S3 buckets configured for static website hosting
* TLS certificate in **AWS Certificate Manager** for HTTPS
* AWS CLI or AWS Management Console access

## Repository Structure

```bash
├── images/                           # Screenshots for this README
│   ├── pre-elb-load-test.png         # Baseline load time from a single bucket
│   ├── create-elb.png                # ELB creation wizard
│   ├── configure-origins.png         # Adding S3 bucket origins to ELB
│   ├── elb-health-checks.png         # Health check settings
│   ├── elb-status.png                # ELB provisioning status
│   ├── failover-test.png             # Simulating bucket failure
│   ├── post-failover-load.png        # Traffic routing to healthy origin
│   └── performance-results.png       # Comparative load times
└── README.md                         # Documentation
```

## Setup & Deployment

### 1. Baseline Performance Test

1. Point your web client directly at one S3 bucket URL (e.g., `http://bucket-a.s3-website-us-east-2.amazonaws.com`).
2. In browser DevTools, disable caching and measure resource load times.

![Baseline Load Test](images/pre-elb-load-test.png)

### 2. Provision an Application Load Balancer

1. Navigate to the **EC2** console → **Load Balancers** → **Create Load Balancer** → **Application Load Balancer**.
2. Choose **internet-facing**, enable HTTP and HTTPS listeners, and select appropriate subnets.
3. Attach your ACM TLS certificate for HTTPS.

![Create ELB](images/create-elb.png)

### 3. Configure S3 Bucket Origins

1. Under **Target Groups**, create a new target group of type **IP**.
2. Register the static website endpoint IPs of each S3 bucket (or use Lambda-backed health checks).
3. In **Listeners**, forward traffic to the target group with **path-based** or **host-based** routing if needed.

![Configure Origins](images/configure-origins.png)

### 4. Set Up Health Checks

1. In your target group settings, configure a **HTTP GET** health check against `/index.html`.
2. Adjust thresholds (e.g., **Interval**: 30s, **Unhealthy threshold**: 2).

![Health Checks](images/elb-health-checks.png)

### 5. Validate ELB Provisioning

Wait for the load balancer and target group to report **Active** and **Healthy** for all bucket origins.

![ELB Status](images/elb-status.png)

## Failover & Testing

### 6. Simulate Origin Failure

Temporarily make one bucket origin unavailable (e.g., remove bucket policy or rename the index file) and reload the ELB DNS.

![Failover Test](images/failover-test.png)

### 7. Verify Traffic Shift

Reload the ELB endpoint to confirm requests are served by a healthy bucket origin.

![Post-Failover Load](images/post-failover-load.png)

## Performance Results

| Scenario                             | Load Time (ms) |
| ------------------------------------ | -------------: |
| Single Bucket (no ELB)               |          `AAA` |
| Multiple Origins via ELB (first hit) |          `BBB` |
| Multiple Origins via ELB (cached)    |          `CCC` |

![Performance Comparison](images/performance-results.png)

## Security & Scalability

* **HTTPS Termination**: TLS handled by ELB with ACM-managed certificates
* **Auto Scaling**: ELB scales automatically to handle traffic spikes
* **Bucket Isolation**: Origins in separate buckets/regions reduce blast radius

## Next Steps

* **Custom Domain**: Route a friendly domain via **Route 53** to the ELB DNS name.
* **Advanced Routing**: Implement geolocation-based routing or weighted distribution.
* **Automation**: Use CloudFormation/Terraform to codify provisioning steps.


