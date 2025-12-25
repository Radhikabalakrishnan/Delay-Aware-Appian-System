# Intelligent Delay-Aware Case Intake and Load-Controlled Processing System using Appian

## Abstract
In Appian-based low-code business process systems, immediate execution of cases during peak workload conditions leads to system overload, increased waiting time, and SLA violations. This project proposes an intelligent delay-aware case intake mechanism where users can submit data and upload documents immediately, while backend processing is dynamically delayed based on real-time system health.

## Problem Statement
Appian platforms currently initiate backend process execution immediately after case submission. During high traffic periods, this results in excessive load on system resources, delayed processing, and lack of transparency to users regarding expected waiting time.

## Proposed Solution
The proposed system introduces a delay-aware control layer that:
- Accepts user submissions without restriction
- Evaluates system load and workload metrics
- Calculates optimal processing delay
- Notifies users about expected processing time
- Automatically triggers processing after delay expiry

## Key Innovation
Unlike traditional queue-based systems, this solution dynamically controls process initiation based on platform health and communicates wait-time information directly to users, improving user experience and SLA compliance.

## Workflow Overview
1. User submits case and uploads documents
2. Data is stored immediately
3. System checks current workload
4. Delay duration is calculated
5. User is informed of delay (e.g., 10 minutes or 1 hour)
6. Case enters waiting queue
7. Timer event triggers backend processing

## Technologies Used
- Appian Low-Code Platform
- Appian Interface Designer
- Appian Process Modeler
- Appian Expression Rules
- GitHub for documentation

## Project Status
Conceptual prototype with workflow design, decision logic, and user communication strategy.

## Team Members
- BHAAVESH
- RADHIKA
