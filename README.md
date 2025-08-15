# DT-Comparison
# Dynatrace Kubernetes OneAgent Deployment Methods Comparison

## Overview
This document provides an **official-style comparison** between:
1. **Full-Stack Installation via Dynatrace Operator**
2. **Application-only via Pod Runtime Injection**

The goal is to help teams choose the most suitable deployment method based on **deployment complexity, monitoring scope, cost, permissions, OneAgent deployment location, other considerations, and recommended use cases**.

---

## Table of Contents
- [Comparison Table](#comparison-table)
- [Key Takeaways](#key-takeaways)
- [References](#references)

---

## Comparison Table

| 比較角度           | Full-Stack 安裝（via Dynatrace Operator）                                                                                                                                                                                                                                                                                                                                                                                                                       | Application-only via Pod Runtime Injection                                                                                                                                                                                                                                                   |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **部署方便程度**     | - 使用 Dynatrace Operator，可透過 DynaKube CR 一次性設定監控整個 Kubernetes 環境。<br>- 支援 namespace 標籤啟用監控（cloud-native full-stack 注入），自動化程度高。<br>- 適合大規模、多團隊環境。 | - 需修改每個 Deployment/POD YAML（加入 initContainer、env、volumeMount）。<br>- 若用 Webhook 需透過 Operator 管理，但仍需逐應用啟用。<br>- 適合小規模或只監控部分應用的情境。 |
| **監控廣度**        | - 從 Host 層（Node metrics、OS 資訊）到應用層（Process、Request tracing、Log、Code-level visibility）全面覆蓋。<br>- 可整合 K8s 工作負載與底層資源關聯。                                                                                                                                                                                                                                                                                                               | - 僅監控應用程式容器，不收集 Host 層與 Node 層指標。<br>- 缺乏完整工作負載與底層關聯性分析能力。                                                                                                                                                                                                                                       |
| **成本／CP 值**      | - 採用 Platform Subscription 計價（按 Host Memory GiB-hours），涵蓋主機與容器監控。<br>- 成本較高，但換取全面監控與自動化。                                                                                                                                                                                                                                                                                                                                             | - 可能僅針對部分應用監控，成本較低。<br>- 功能受限，若後期需要擴大監控，可能需重新規劃。                                                                                                                                                                                                                                               |
| **權限需求**        | - 需要 Cluster-Role 權限（nodes, namespaces, secrets, CRDs, webhook configs 等）。<br>- 需部署 CSI driver、Webhook 伺服器等元件，可能使用 SCC: privileged 或 nonroot-v2。<br>- 權限覆蓋範圍廣，需嚴格控管。                                                                                                                                                                                                                                                          | - 僅需 PaaS Token（InstallerDownload scope）。<br>- 若手動 initContainer，無需 Cluster-Role 權限。<br>- 若用 Operator 管理 Webhook，權限需求仍低於 Full-Stack。                                                                                                                                                                           |
| **OneAgent 部署位置** | - **DaemonSet** 部署於 Kubernetes 節點（Node）上，預設監控所有可監控節點。<br>- 可用 `nodeSelector`、`tolerations`、`affinity` 控制部署位置。<br>- 常見做法：對節點加標籤（如 `dynatrace.com/oneagent: fullstack`）後在設定中指定部署。<br>- 收集 Node 層與該節點所有容器/應用的資訊。                                                                                                                                                                                 | - 不在 Node 層部署 OneAgent。<br>- 僅在目標 Pod 內以 `initContainer` + volume 注入。<br>- 手動修改 Deployment YAML，或透過 Webhook 依 Namespace / Pod 標籤控制注入。<br>- 僅監控該 Pod 內應用，不收集 Node 層資訊。                                                                                                                    |
| **額外面向**        | 1. **自助式 Onboarding 與資料隔離**：支援 namespace 級別資料送往不同 Dynatrace 環境。<br>2. **解決 Race Condition**：避免因 Pod 啟動順序導致注入失敗。<br>3. **未來方向**：Cloud-Native Full-Stack 逐步取代 Classic Full-Stack。                                                                                                                                                                                                                                          | 1. **最低權限原則**：適用於高安全要求環境。<br>2. **無需 Node Agent**：減少對叢集影響。<br>3. **需重啟 Pod**：切換模式需重新啟動應用容器。<br>4. **儲存需求**：每個 Pod 需下載 300–650MB OneAgent 程式碼模組。                                                                                                                              |
| **適用場景**        | - 需要監控 **整個 Kubernetes 環境**（基礎設施 + 應用）<br>- 多團隊、多租戶，需 **自助式 onboarding 與資料隔離**<br>- 需與 K8s 工作負載和底層資源建立完整鏈結分析<br>- 適合 **長期持續監控** 與 **規模化運營** | - 僅需監控 **特定應用**（如測試環境、POC 專案）<br>- 高安全要求、不允許部署節點層 Agent 的環境<br>- **資源受限** 或 僅短期監控需求<br>- 適合 **快速驗證或部分服務監控**                                                                                  |

---

## Key Takeaways
- **Full-Stack** offers the **broadest visibility** (infrastructure + applications) but requires **higher permissions** and **more resources**.
- **Application-only** is **lighter** and **lower in permissions**, but with **reduced visibility** and **no host metrics**.
- Choice should be based on **security policy**, **resource constraints**, and **monitoring requirements**.

---

## References
- [Dynatrace Docs - Full-Stack Observability](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/deployment/full-stack-observability)
- [Dynatrace Docs - Pod Runtime Injection](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/deployment/other/pod-runtime)
- [Dynatrace Blog - Cloud-Native Full-Stack](https://www.dynatrace.com/news/blog/flexible-scalable-self-service-kubernetes-native-observability/)
