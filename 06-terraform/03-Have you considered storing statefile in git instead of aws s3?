# Architectural Decision: Why We Store Terraform State in AWS S3 vs. Git

This document explains our decision to use AWS S3 and DynamoDB for Terraform state management instead of storing state files directly in version control.

## 1. Security Risks (Plain-Text Secrets)
* **The Problem:** Terraform state files store infrastructure details in plain text. This includes database passwords, private keys, API tokens, and initial administrative credentials.
* **The Git Flaw:** Committing a state file to Git injects sensitive credentials into version control history. Even if the file is deleted later or the repository is private, secrets remain permanently baked into the Git history.

## 2. Concurrency Issues (No State Locking)
* **The Problem:** Simultaneous deployments by multiple engineers can overwrite changes, leading to corrupted state files or destroyed cloud resources.
* **The Git Flaw:** Git lacks a live state locking mechanism. AWS S3 combined with DynamoDB natively locks the state file during execution. Git only detects conflicts during a `git push`, which happens after infrastructure changes are already applied.

## 3. Automation Bottlenecks (Race Conditions)
* **The Problem:** Terraform requires real-time state accuracy before modifying infrastructure.
* **The Git Flaw:** Git introduces race conditions. Engineers must manually commit and push updated state files. If someone forgets to push or pull the latest changes, deployments run on stale data and break the infrastructure. 

---

## Interview Cheat Sheet: Summary Answer

If asked, **"Have you considered storing the state file in Git instead of AWS S3?"**, answer with:

> "We explicitly avoid storing the state file in Git because it violates security and operational best practices. First, state files contain plain-text secrets that should never be committed to version control. Second, Git doesn't support native state locking, which creates a massive risk of race conditions and state corruption if two team members run deployments simultaneously. We use AWS S3 with DynamoDB precisely because it gives us automated encryption at rest, automatic version history, and real-time state locking to ensure team safety."
