# Cloud-Computing, README for lab 7(Copy of Tutorial 7 microservices.ipynb):

# 🌱 Plant Growth Monitoring System

This repository contains the core logic for the **Plant Growth Monitoring & Problem Detection System**. It includes a demonstration of a search module (for querying plant disease databases) and outlines the architectural vision for a fully Serverless IoT solution.

---

## 📊 KPIs & Non-Functional Requirements (Plant Project)

This section defines the success metrics for the **Plant Monitoring System**. The system's goal is to process sensor data and images to detect dehydration, diseases, or pests in real-time.

### Non-Functional Requirements (NFRs)
1.  **Real-Time Processing (Latency):** The system must process incoming sensor data (e.g., soil moisture drops) and trigger alerts within seconds.
2.  **High Availability:** The monitoring service must be available 99.9% of the time to ensure no critical alerts are missed.
3.  **Scalability:** The architecture must support scaling from 10 sensors to 10,000 concurrent sensors without manual intervention.
4.  **Data Integrity:** Sensor data must be stored reliably for historical analysis and ML model training.

### Key Performance Indicators (KPIs) & Testing

| KPI Metric | Definition | Target Goal | How to Test? |
| :--- | :--- | :--- | :--- |
| **Disease Detection Accuracy** | Percentage of correctly identified plant diseases from user-uploaded images. | > 95% | **Test Set Evaluation:** Run the ML model against a validated dataset of 1,000 labeled leaf images (Healthy vs. Sick). |
| **Alert Latency** | Time from sensor reading (Critical Threshold) to User Notification. | < 5 Seconds | **End-to-End Latency Test:** Simulate a "Dry Soil" event and measure the timestamp difference until the push notification is received. |
| **False Positive Rate** | Frequency of alerts sent when the plant is actually healthy. | < 2% | **Field Testing:** Deploy sensors on healthy plants for a week and count the number of incorrect "Critical" alerts generated. |
| **Sensor Data Throughput** | The number of sensor readings the system can ingest per second. | 5,000 req/sec | **Load Testing:** Use `Locust` or `JMeter` to bombard the ingestion API with simulated sensor payloads. |

---

## ☁️ Serverless (FaaS) Architecture Proposal (Plant Project)

To achieve the scalability and efficiency required for IoT monitoring, this project is designed to be deployed as a **Serverless Cloud Architecture** (e.g., AWS Lambda, Azure Functions).

### Why Serverless?
* **Event-Driven:** Plants don't need constant monitoring every millisecond. Functions only run when a sensor sends data or a user uploads a photo.
* **Cost Efficiency:** We only pay for the compute time used during analysis, not for idle servers waiting for data.
* **IoT Scalability:** FaaS handles spikes (e.g., all sensors reporting at noon) automatically.

### Architecture Components

1.  **Data Ingestion (IoT Core / API Gateway):**
    * Sensors send MQTT/HTTP packets with moisture, light, and temp data.
    * **FaaS Trigger:** A new message triggers the `ProcessSensorData` function.

2.  **Data Processing Function (`ProcessSensorData`):**
    * **Role:** Validates data and compares against thresholds (e.g., `if moisture < 30%`).
    * **Action:** If critical, triggers the **Notification Service**. If normal, saves to **Time-Series DB**.

3.  **Image Analysis Function (`AnalyzeLeafImage`):**
    * **Trigger:** User uploads a photo of a leaf to Object Storage (S3/Blob).
    * **Role:** Loads a pre-trained ML model (TensorFlow Lite), classifies the disease, and returns the result.

4.  **Storage:**
    * **Hot Storage (DynamoDB/CosmosDB):** For current plant status and user profiles.
    * **Cold Storage (Data Lake):** For historical sensor data used to retrain models.

### Architecture Diagram Flow:
`Sensor` -> `IoT Core` -> `Lambda (Process Data)` -> `DynamoDB`
                                      |
                                      v
                               `Lambda (Alert)` -> `SNS/Notification`

---

## 🔍 Included Code: Search Engine Module

*While the broader project is about plants, this repository specifically includes the source code for the internal **Search Engine Module** used to query the plant disease knowledge base.*

### Code Architecture
The search module is built using **Service-Oriented Architecture (SOA)** logic:

1.  **IndexService:** Manages the Inverted Index (mapping keywords like "Fungus" to relevant guides).
2.  **QueryService:** Handles Boolean logic (`AND` / `OR`) to filter results.
3.  **ResultService:** Formats the output and calculates a **Relevance Rank** based on term frequency.

### Service Documentation

* **`IndexService.add_document(doc)`**: Tokenizes content and updates the index.
* **`QueryService.create_query(terms, operator)`**:
    * Supports **AND** (Intersection): `term1 & term2`
    * Supports **OR** (Union): `term1 | term2`
* **`ResultService.format_results(id)`**:
    * Implements a Ranking Algorithm: $Rank = 1 - \prod (\frac{1}{count})$.
    * Returns sorted JSON results with text snippets.

### Usage Example (Python)

```python
# 1. Indexing Plant Guides
index_service.add_document({
    'title': 'Treating Root Rot',
    'content': 'Root rot is caused by overwatering and poor drainage.'
})

# 2. Searching for solutions
query = query_service.create_query({
    'terms': ['overwatering', 'drainage'],
    'operator': 'AND'
})

# 3. Getting Ranked Results
results = result_service.format_results(query['id'])
# Output: Returns the 'Root Rot' document with a high relevance score.
