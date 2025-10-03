# 🍳 AI Chef — Gemini + Streamlit on Cloud Run

AI Chef is a Streamlit web app that uses Google’s Gemini (via Vertex AI) to generate personalized recipe ideas from user inputs—cuisine, dietary preference, allergies, pantry ingredients, and wine choice. It returns complete suggestions with titles, prep steps, estimated time, wine pairings, calories, and basic nutrition facts. The app logs to Cloud Logging in GCP, runs locally in Cloud Shell for testing, and is containerized with Docker for deployment to Cloud Run.

<img width="551" height="637" alt="image" src="https://github.com/user-attachments/assets/3f2b775e-735b-4b20-9ca1-add487cf0fee" />

---

## 🚀 Features

- 🧠 **Gemini-powered recipes**: instructions, prep time, calories, nutrition facts  
- 🧩 **Streamlit UI**: Select boxes for cuisine & diet, text inputs for allergy & ingredients  
- ☁️ **Cloud-ready**: Tested in **Cloud Shell**, built with **Cloud Build**, deployed on **Cloud Run**  
- 📝 **Cloud Logging (optional)**: Logs can stream to **Google Cloud Logging** in GCP

---

## 📂 Project Structure

```
.
├── chef.py             # Streamlit app (UI + Gemini prompt)
├── requirements.txt    # Python dependencies
└── Dockerfile          # Container for Streamlit on port 8080
```

---

## ⚙️ Setup (Cloud Shell)

```bash
# 1) Go to the app directory
cd 

# 2) Virtual environment
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip

# 3) Install deps
pip install -r requirements.txt
# (or) pip install streamlit google-genai google-cloud-logging

# 4) Env vars used by the app
export PROJECT=<YOUR_PROJECT_ID>
export REGION=<YOUR_REGION>      # e.g., us-central1
export GOOGLE_CLOUD_PROJECT=$PROJECT
export GOOGLE_CLOUD_REGION=$REGION
```

Run the app:

```bash
streamlit run chef.py --server.port=8080 --server.address=0.0.0.0
```

---

## 📜 Prompt used in `chef.py`

```python
prompt = f"""I am a Chef.  I need to create {cuisine} 
recipes for customers who want {dietary_preference} meals. 
However, don't include recipes that use ingredients with the customer's {allergy} allergy. 
I have {ingredient_1}, 
{ingredient_2}, 
and {ingredient_3} 
in my kitchen and other ingredients. 
The customer's wine preference is {wine} 
Please provide some for meal recommendations.
For each recommendation include preparation instructions,
time to prepare
and the recipe title at the beginning of the response.
Then include the wine paring for each recommendation.
At the end of the recommendation provide the calories associated with the meal
and the nutritional facts."""
```

---

## 🐳 Containerize & Push to Artifact Registry

**Dockerfile**

```dockerfile
FROM python:3.13-slim
EXPOSE 8080
WORKDIR /app
COPY . ./
RUN pip install --no-cache-dir -r requirements.txt
ENTRYPOINT ["streamlit", "run", "chef.py", "--server.port=8080", "--server.address=0.0.0.0"]
```

**Build & push**

```bash
export AR_REPO=chef-repo
export SERVICE_NAME=chef-streamlit-app

gcloud artifacts repositories create "$AR_REPO"   --location="$REGION" --repository-format=Docker

gcloud builds submit   --tag "$REGION-docker.pkg.dev/$PROJECT/$AR_REPO/$SERVICE_NAME"
```

---

## 🚀 Deploy to Cloud Run

```bash
gcloud run deploy "$SERVICE_NAME"   --port=8080   --image="$REGION-docker.pkg.dev/$PROJECT/$AR_REPO/$SERVICE_NAME"   --allow-unauthenticated   --region="$REGION"   --platform=managed   --project="$PROJECT"   --set-env-vars=PROJECT=$PROJECT,REGION=$REGION
```

Open the **Service URL**, generate a recipe, and verify output.

---

## 🧾 requirements.txt

```txt
streamlit
google-genai
google-cloud-logging
```

*(Add `google-cloud-aiplatform`, `google-cloud-secret-manager`, `google-cloud-storage` if needed.)*

---

## 📌 Future Enhancements

- Better defaults if users leave fields unselected  
- Cleaner recipe rendering (expanders, downloads)  
