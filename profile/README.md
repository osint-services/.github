# Open Source Intelligence Platform

Welcome to the **Open Source Intelligence Platform**! This organization is dedicated to creating powerful, modular Python-based microservices to collect and analyze open-source intelligence (OSINT) on a target.

## 🚀 About the Organization

The **Open Source Intelligence Platform** aims to:
- Provide highly extensible and modular Python microservices.
- Simplify the process of collecting OSINT data from various sources, including websites, APIs, and public records.
- Enable secure and efficient intelligence analysis focusing on privacy and compliance.

## 📂 Repository Overview

| Repository Name           | Description                                                                                  |
|---------------------------|----------------------------------------------------------------------------------------------|
| **WhatsMyName**     | A Python microservice to find website profiles based on a given username     |
| **ThatsThem**  | A Python microservice to retrieve relevant information for a given name, address, email or phone number.                  |


## 🛠️ Features

- **Scalable Microservices**: Each service is built to run independently, enabling easy scaling and integration.
- **Modular Design**: Swap or extend components without affecting the core functionality.
- **Python Powered**: Built with Python, leveraging its extensive libraries and frameworks for web scraping, data analysis, and visualization.
- **Secure & Compliant**: Designed with security best practices and legal compliance in mind.

## 📖 Getting Started

To get started, clone the repository you need and follow the instructions in its README.md file. Below is a general setup for running our microservices.

### Prerequisites

- Python 3.9+
- Docker (optional for containerized deployments)
- Virtual environment (recommended)

### Installation

1. Clone the desired repository:
   ```bash
   git clone https://github.com/osint-services/<repository-name>.git
   cd <repository-name>
    ```
2. Start the desired repository
   ```bash
   fastapi dev main.py
   ```
3. The repository should be running and available, you can check the terminal output to see where it's pointing.
