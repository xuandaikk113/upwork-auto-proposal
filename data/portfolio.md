# Highlight Projects

## 1. Enterprise AI Search & Knowledge Discovery

| | |
|-----|-----|
| **Role** | AI Engineer |
| **Project Description** | An enterprise-grade search platform allowing multi-modal querying across structured relational databases and massive unstructured datasets (PDFs, Office docs, Images). The system integrates advanced language models to provide a natural language interface for complex data retrieval, ensuring high accuracy and low latency for internal knowledge discovery. |

### Responsibilities
- Designed the end-to-end system architecture, integrating Text2SQL reasoning agents with vector-based semantic search for a unified user experience.
- Developed the intelligent data ingestion pipeline utilizing advanced parsing libraries and OCR engines to extract structured information from messy, unstructured enterprise documents.
- Engineered the model serving infrastructure using vLLM, including custom quantization workflows (AWQ) to optimize resource utilization on production GPUs.
- Integrated observability tools to monitor model traces, audit prompt performance, and track token consumption for cost-effective scaling.

### Achievements
- Enhanced system versatility and retrieval precision by architecting a hybrid retrieval engine that intelligently routes queries between Text2SQL for structured data and Semantic Search for unstructured documents.
- Reduced VRAM footprint by 40% while maximizing throughput for high-concurrency environments through model quantization (AWQ) and optimized deployment via vLLM.
- Boosted data integrity and retrieval accuracy by engineering a multi-stage ingestion pipeline featuring automated OCR, semantic parsing, and deduplication.
- Ensured production-grade reliability by implementing a rigorous evaluation framework utilizing SQL-based verification and Langfuse metrics to monitor RAG performance.

---

## 2. AI-Powered Tool Inventory Monitoring System

| | |
|-----|-----|
| **Role** | AI Engineer |
| **Project Description** | An intelligent computer vision solution for automated inventory tracking of industrial tool drawers. The system leverages instance segmentation and object detection models to identify missing or misplaced tools in real-time from captured images, ensuring operational safety and tool accountability across manufacturing facilities. |

### Responsibilities
- Architected end-to-end MLOps pipeline combining object detection, instance segmentation, and advanced transformation algorithms for accurate tool localization and identification.
- Engineered modular inference system with flexible model switching capabilities, supporting GPU acceleration via NVIDIA TensorRT and CUDA optimization.
- Implemented automated training pipeline with hyperparameter tuning, model evaluation metrics, and S3-based dataset management for scalable model deployment across multiple drawer configurations.
- Developed synthetic data generation framework leveraging augmentation techniques to overcome small dataset constraints, creating 3x more training samples and improving model generalization across diverse drawer configurations.

### Achievements
- Achieved 95%+ accuracy in tool detection and segmentation by implementing a multi-model ensemble approach that ensures robust performance under challenging lighting conditions.
- Reduced manual tool auditing time by 85% through the deployment of an automated real-time inference pipeline and immediate visual feedback systems.
- Accelerated development cycles and model reliability by establishing a full-scale MLOps pipeline covering the entire lifecycle from annotation and experiment tracking to version control.
- Improved operational workflow efficiency by integrating real-time visual alerts, significantly reducing human error in equipment management.

---

## 3. Real-Time Industrial Rock Fragment Size Analysis System

| | |
|-----|-----|
| **Role** | AI Software Engineer |
| **Project Description** | A real-time AI-based solution for analyzing rock fragment sizes from live video streams. The system automatically detects when material is being transported, segments individual rocks, and calculates their real-world dimensions to identify oversized fragments, providing immediate feedback, alerting operators of anomalies. |

### Responsibilities
- Developed the end-to-end pipeline combining Instance Segmentation models (to isolate individual rocks) with advanced perspective transformation/calibration algorithms for accurate size calculation.
- Engineered the final alert mechanism by integrating the detection application with an adjacent ESP32 microcontroller via Bluetooth to trigger the physical indicator lights.
- Established comprehensive MLOps logging to record operational data and save analysis/sample images for continuous model enhancement and auditing.
- Streamlined remote deployment by writing automation BAT/Bash scripts, enabling non-technical operators to install and maintain the system.
- Optimized the model inference on Edge Device (NVIDIA Jetson) to ensure high reliability and low latency (<100ms processing time) in a harsh industrial environment.

### Achievements
- Achieved 90% accuracy in classifying rock fragment sizes and detecting oversized anomalies crucial for operational safety and equipment wear prevention.
- Reduced downstream equipment damage and operational downtime by providing real-time alerts for oversized material.

---

## 4. Enterprise Self-Service NLP Chatbot Platform

| | |
|-----|-----|
| **Role** | AI Software Engineer |
| **Project Description** | A high-availability, self-service platform enabling Marketing and Retail departments to independently access, create, and deploy custom NLP chatbots to enhance customer care and sales operations. |

### Responsibilities
- Architected the microservices-based platform, focusing on high availability, scalability (Horizontal/Vertical scaling), and multi-tenancy.
- Developed the core backend APIs for chatbot creation, integration, and deployment using Django REST Framework.
- Implemented the CI/CD pipeline (Jenkins + Argo CD + Kubernetes) and monitoring solutions (Logging) to ensure robust platform performance and rapid deployment cycles.
- Designed the foundation for the NLP models, facilitating quick development of demo and production chatbots for business units.

### Achievements
- Successfully deployed a High-Availability platform serving various business units/retail stores, achieving 99.9% uptime during peak operational hours.
- Reduced time-to-market for new chatbot initiatives by 50% by enabling self-service creation and automated CI/CD deployment.
- Demonstrated impact on sales by successfully deploying a chatbot prototype that handled 40% of initial customer inquiries, streamlining support staff workload.

---

## 5. Automatic-Parking System

| | |
|-----|-----|
| **Role** | AI Engineer |
| **Project Description** | A real-time system that automatically identifies employee vehicle license plates (ALPR), controls the security fence, and provides simultaneous audio/visual notification of entry and exit events. |

### Responsibilities
- Designed the full system architecture, integrating RTSP video feeds with backend APIs for access control.
- Developed and custom Deep Learning models for Object Detection (vehicle/plate localization) and Character Recognition (ALPR).
- Optimized the DL model for low-latency inference on Edge/Embedded systems (NVIDIA Jetson).

### Achievements
- Achieved 97.5% recognition accuracy on license plates under varying lighting and weather conditions.
- Reduced employee entry/exit time by 5 times.
- Implemented a low-latency inference engine (<150ms end-to-end latency) for instant barrier operation.

---

## 6. High-Accuracy Enterprise Facial Recognition Attendance System

| | |
|-----|-----|
| **Role** | AI Software Engineer |
| **Project Description** | A high-accuracy system for automated employee timekeeping, capable of simultaneous, real-time facial recognition for multiple users without requiring direct camera focus. The system records entry/exit events and logs history for workforce management. |

### Responsibilities
- Architected the large-scale solution, focusing on integrating real-time RTSP streams with the timekeeping backend for seamless data logging.
- Implemented a robust Face Detection pipeline and deployed a state-of-the-art Face Vector Encoding Model for identification.
- Optimized the DL model and data structures to support simultaneous recognition of multiple faces at real-time speeds.
- Ensured data security and privacy compliance during integration with HR APIs and historical database storage.

### Achievements
- Achieved 99% recognition accuracy for rapid and reliable employee identification across diverse environmental conditions.
- Processed and recognized multiple users simultaneously in real-time to prevent queuing at entry points.
- Reduced time-theft risks and administrative overhead by implementing the fully automated and high-throughput attendance logging system.
