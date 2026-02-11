# Kind Kubernetes: End-to-End Cluster

This document serves as a comprehensive reference for creating, configuring, troubleshooting, and scaling a production-like Kubernetes cluster locally using **Kind** (Kubernetes using Docker).

## 1. Architecture Overview

We are building a cluster designed to simulate a real-world environment on a local machine.

    ### • Cluster Type: Multi-node (1 Control Plane, 2 Workers).
    ### • Ingress Controller: NGINX (Maps localhost:80 $\rightarrow$ Cluster).
    ### • Load Balancer: MetalLB (Assigns unique IPs to services).
    ### • Networking: Port mappings configured to allow direct traffic from the host machine.

## 2. Cluster Provisioning

  ## Prerequisites
      ### • Docker Desktop / Docker Engine
      ### • Kind (brew install kind)
      ### • Kubectl

## 3. Networking Setup (The "Plumbing")

## 4. Application Deployment & Scaling

## 5. Troubleshooting Log

## 6. Verification & Usage

## 7. Cleanup
