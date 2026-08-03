🔐 Secure Payments Platform

Kubernetes Hardening • Secure CI/CD • GitOps • Istio Zero-Trust Mesh

🌟 Project Summary

This project demonstrates how to build and operate a secure, production-grade microservices platform on Kubernetes using:

🔐 Zero-trust networking (Istio mTLS + AuthZ)

🔁 Secure CI/CD with supply chain security

⚙️ GitOps-based deployment (Argo CD)

🛡️ Kubernetes hardening & policy enforcement

💡 This is not just a deployment — it includes real-world debugging of service mesh failures, policy conflicts, and networking issues.


🏗️ Architecture Overview

GitHub Push

   ↓
   
GitHub Actions (Security Pipeline)

   ↓
   
GHCR (Signed Image)

   ↓
   
GitOps Repo Update

   ↓
   
Argo CD Sync

   ↓
   
Kubernetes Cluster (kind)

   ↓
   
Istio Service Mesh (mTLS + AuthZ)


✅ Task 1 – Kubernetes Hardening

🔧 Improvements Implemented

Area	Before	After

Container	     Root	            ✅ Non-root (UID 1000)

Secrets	        Plaintext in YAML	✅ Sealed Secrets

Security	        None	            ✅ securityContext enforced

Probes	        Missing	         ✅ Liveness + Readiness

Resources	     Missing	         ✅ Requests/Limits

ServiceAccount	  Default	         ✅ Dedicated SA

Image	           python:3.6	      ✅ Hardened image

Admission	     None	            ✅ Kyverno policies



🔐 Security Context

securityContext:

  runAsNonRoot: true
  
  runAsUser: 1000
  
  allowPrivilegeEscalation: false
  
  readOnlyRootFilesystem: true
  
  capabilities:
  
    drop: ["ALL"]
    


🔑 Secrets Management

Removed:

STRIPE_API_KEY: "sk_live_..."

DB_PASSWORD: "P@ssw0rd"


Replaced with:

✅ Sealed Secrets

🛡️ Kyverno Policies

❌ Block root containers

❌ Block :latest

❌ Block insecure deployments


🚫 Rejection Demo

kubectl apply -f insecure-baseline.yaml


✅ Result: Rejected by Kyverno


⚙️ Task 2 – Secure CI/CD + GitOps

🔁 Pipeline Overview

Pipeline includes:

Stage	              Tool

Secret Scan    	Gitleaks

SAST	            Semgrep

Dependency Scan 	Trivy FS

Image Scan	     Trivy Image

Build            	Docker

Registry	          GHCR

Signing	       Cosign (keyless)

GitOps	         Argo CD

🐳 Image

ghcr.io/<user>/ledger-api:<git-sha>

🔐 Cosign Signing (Keyless)

cosign sign ghcr.io/<image>@sha256:...


✔ Uses:

OIDC identity

Sigstore Fulcio

Rekor transparency log

🔄 GitOps Flow

Pipeline updates manifest

Argo CD detects change

Syncs automatically

📊 Argo CD Status

From your UI:

✅ Healthy

✅ Synced

✅ Auto-sync enabled

🖥️ Argo CD UI Evidence


Application: ledger-api

Status: Healthy + Synced

Sync: Successful


🌐 Task 3 – Istio Service Mesh Security

🔒 mTLS Enforcement

PeerAuthentication:

  mtls:
  
    mode: STRICT
    
🔐 Authorization Policy

AuthorizationPolicy:

  action: ALLOW
  
  rules:
  
  - from:
    - source:
      
        principals:
      
        - cluster.local/ns/payments/sa/reporting
          
🌍 Network Policies

Default deny

Allow only required communication


🧪 Security Validation

Scenario	Result

reporting → ledger-api	✅ Allowed

attacker pod → ledger-api	❌ Denied

plaintext (no sidecar)	❌ Blocked

mTLS traffic	✅ Encrypted


🧠 🔥 REAL DEBUGGING JOURNEY (IMPORTANT)


This project includes real-world production debugging scenarios.


🚨 Issue 1 – no healthy upstream

Cause:

Istio could not find healthy endpoints

Fix:

Verified pod + service alignment


🚨 Issue 2 – 503 Service Unavailable (Envoy)

Root Cause:

❌ Service port mismatch

targetPort: 8080

port: 80

✔ Fix:


ports:

- name: http   
  
  port: 80
  
  targetPort: 8080
  

👉 Istio requires port naming for protocol detection


🚨 Issue 3 – Curl Hanging (No Response)

Root Cause:

❌ Conflicting AuthorizationPolicies

You had:

default-deny

allow-all

allow-reporting


👉 Caused policy evaluation conflict

✅ Final Fix:

kubectl delete authorizationpolicy --all -n payments

Apply single clean policy:

ALLOW reporting → ledger-api


🚨 Issue 4 – mTLS Blocking Traffic

Cause:

STRICT mode without proper policy

Fix:
Correct AuthorizationPolicy with proper principal


🚨 Issue 5 – No curl inside container

exec: "curl": not found

Fix:

Used port-forward instead


🚨 Issue 6 – Port-forward conflicts

bind: address already in use

Fix:

Switched ports (8081, 9090)


🚨 Issue 7 – Envoy 503 Despite Healthy App

Root Cause:

❌ Istio routing issue (NOT app issue)

✔ Verified using:

kubectl port-forward

curl localhost:8080/health

✅ App returned:

{"status":"ok"}

🧪 Final Verification

Test	Result

Local Docker	✅

Kubernetes Service	✅

GitOps Deployment	✅

Argo CD Sync	✅

mTLS Communication	✅

Authorization Control	✅

CI/CD Security Gates	✅
