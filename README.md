# 🧳 Build a Travel Agent Using MCP Toolbox & ADK

**Author:** Ketan Pawar  
**Project Type:** Google Cloud-Based Travel Agent  
**Technologies Used:**  
- MCP Toolbox  
- Agent Development Kit (ADK)  
- Cloud SQL (PostgreSQL)  
- Python (optional)

---

## 📌 Overview

This project demonstrates how to build an intelligent Travel Agent capable of answering hotel-related queries like:

- “Find hotels in Basel”
- “Search for a hotel named Marriott”

The solution integrates Google Cloud services with AI capabilities using:

- **MCP Toolbox** – for tool orchestration  
- **ADK (Agent Development Kit)** – for building and running the GenAI agent  
- **Cloud SQL** – to manage hotel data in PostgreSQL

---

## ✅ Prerequisites

- Active **Google Cloud Project** (with billing enabled)
- **Cloud Shell** access
- Chrome browser
- Basic Python (optional but helpful)

---

## 🛠️ Step-by-Step Setup

### 🔹 Step 1: Google Cloud Project Setup

1. Open the [Google Cloud Console](https://console.cloud.google.com).
2. Create a **new project** or use an existing one.
3. **Enable billing** on the project.
4. Launch **Cloud Shell**, and run:
   gcloud auth list
   gcloud config list project
   gcloud config set project <YOUR_PROJECT_ID>
5. Enable required APIs:
   gcloud services enable cloudresourcemanager.googleapis.com \
  servicenetworking.googleapis.com \
  run.googleapis.com \
  cloudbuild.googleapis.com \
  cloudfunctions.googleapis.com \
  aiplatform.googleapis.com \
  sqladmin.googleapis.com \
  compute.googleapis.com

### 🔹 Step 2: Create Cloud SQL Instance
gcloud sql instances create hoteldb-instance \
  --database-version=POSTGRES_15 \
  --tier=db-g1-small \
  --region=us-central1 \
  --edition=enterprise \
  --root-password=postgres

### 🔹 Step 3: Create HOTELS Table & Sample Data
1. Go to Cloud SQL > hoteldb-instance > SQL Studio
   
2. Authenticate:

User: postgres

Password: postgres

3. Run the SQL below:
   CREATE TABLE HOTELS (
  ID INTEGER NOT NULL PRIMARY KEY,
  NAME VARCHAR NOT NULL,
  LOCATION VARCHAR NOT NULL,
  PRICE_TIER VARCHAR NOT NULL,
  CHECKIN_DATE DATE NOT NULL,
  CHECKOUT_DATE DATE NOT NULL,
  BOOKED BIT NOT NULL
);
4. Insert sample hotel data manually (use INSERT INTO HOTELS(...) VALUES (...);).
  
5. Validate:
   SELECT * FROM HOTELS;

### 🔹 Step 4: Setup MCP Toolbox
mkdir mcp-toolbox && cd mcp-toolbox
export VERSION=0.8.0
curl -O https://storage.googleapis.com/genai-toolbox/v$VERSION/linux/amd64/toolbox
chmod +x toolbox

Create a file tools.yaml (define your tools inside it).
##  Then run:
  ./toolbox --tools-file "tools.yaml"
Access it: Web Preview → Port 5000 → /api/toolset

### 🔹 Step 5: Build Agent using ADK
mkdir my-agents && cd my-agents
python -m venv .venv
source .venv/bin/activate
pip install google-adk toolbox-core

## Create the agent:
adk create hotel-agent-app

Modify agent.py:
  from google.adk.agents import Agent

root_agent = Agent(
    model="gemini-2.0-flash-001",
    name="hotel_agent",
    description="A helpful assistant for hotel queries.",
    instruction="Use tools to answer hotel queries by city or name."
)

### 🔹 Step 6: Connect Agent to MCP Toolbox
# Update agent.py to load the tools:
from toolbox_core import ToolboxSyncClient

toolbox = ToolboxSyncClient("http://127.0.0.1:5000")
tools = toolbox.load_toolset('my_first_toolset')

root_agent = Agent(..., tools=tools)


### 🔹 Step 7: Run Agent Locally
 #  Terminal 1:
./toolbox --tools-file "tools.yaml"

  # Terminal 2:
adk run hotel-agent-app
-- 
### 🧪 Example:

User: Basel  
Agent: I found 3 hotels: Hilton Basel, Hyatt Regency Basel, Holiday Inn Basel.

### 🔹 Step 8: Cleanup Resources
export PROJECT_ID="your_project_id"
export REGION="us-central1"

gcloud run services delete toolbox --platform=managed --region=$REGION --project=$PROJECT_ID --quiet
gcloud run services delete hotels-service --platform=managed --region=$REGION --project=$PROJECT_ID --quiet
gcloud sql instances delete hoteldb-instance

---

##  ✅ Conclusion
You’ve successfully built:

A working GenAI-powered Hotel Travel Agent

Integrated PostgreSQL data using Cloud SQL

Connected tools via MCP Toolbox

Deployable locally or on Cloud Run

## 📚 Resources
ADK Documentation

MCP Toolbox GitHub

Cloud SQL Docs


