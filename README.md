import os
import json
import time
from google.cloud import bigquery
import google.generativeai as gemini

# ==========================================
# LAYER 3: GOOGLE CLOUD & BIGQUERY PIPELINE
# ==========================================
class GoogleCloudDataPipeline:
    def __init__(self, project_id="ml-consumer-smart-agro"):
        self.project_id = project_id
        # আপনার তৈরি করা Service Account অথেন্টিকেশন
        # (প্রোডাকশনে সিসেম এনভায়রনমেন্ট ভ্যারিয়েবল ব্যবহার করা ভালো)
        self.client = bigquery.Client(project=self.project_id)
        self.dataset_id = "agro_intelligence_data"
        self.table_id = "drone_factory_metrics"
        
    def initialize_storage(self):
        """বিগকোয়েরি ডেটাসেট এবং টেবিল তৈরি বা চেক করা"""
        dataset_ref = bigquery.DatasetReference(self.client.project, self.dataset_id)
        try:
            self.client.get_dataset(dataset_ref)
            print(f"[Layer 3] Dataset {self.dataset_id} already exists.")
        except Exception:
            dataset = bigquery.Dataset(dataset_ref)
            dataset.location = "US"
            self.client.create_dataset(dataset)
            print(f"[Layer 3] Created new BigQuery Dataset: {self.dataset_id}")

    def sync_to_cloud(self, telemetry_data):
        """মাঠ ও ফ্যাক্টরির ডেটা ক্লাউডে সিঙ্ক করা"""
        table_ref = self.client.dataset(self.dataset_id).table(self.table_id)
        
        # বিগকোয়েরি স্ট্রিমিং ইনসার্ট
        errors = self.client.insert_rows_json(table_ref, [telemetry_data])
        if errors == []:
            print("[Layer 3] Data successfully synced to Google BigQuery.")
            return True
        else:
            print(f"[Layer 3] Errors syncing to BigQuery: {errors}")
            return False

# ==========================================
# LAYER 4: GEMINI AI AGRICULTURAL INTELLIGENCE
# ==========================================
class GeminiAgroIntelligence:
    def __init__(self, api_key):
        # আপনার কপি করা জেমিনি API Key এখানে সেট হবে
        gemini.configure(api_key=api_key)
        self.model = gemini.GenerativeModel('gemini-pro')

    def analyze_crop_health(self, multispectral_summary, purity_index):
        """জেমিনি এআই দ্বারা ফসলের স্বাস্থ্য ও ফ্যাক্টরির রিপোর্ট অ্যানালাইসিস"""
        prompt = f"""
        Role: Chief Agricultural AI Specialist
        Project: Smart Agro Hub (BSCIC, Thakurgaon, Bangladesh)
        
        Analyze the following real-time IoT and Drone data:
        1. Drone Multispectral Summary: {multispectral_summary}
        2. Factory 'Jhanjh' Mustard Oil Purity Index: {purity_index}%
        
        Provide a concise 3-line action plan for the local farmers regarding nitrogen scaling and factory batch quality control.
        """
        try:
            response = self.model.generate_content(prompt)
            print("\n=== [Layer 4] Gemini AI Insights ===")
            print(response.text)
            print("===================================\n")
            return response.text
        except Exception as e:
            print(f"[Layer 4] Gemini AI Error: {e}")
            return None

# ==========================================
# SIMULATION RUNNER (INTEGRATING ALL LAYERS)
# ==========================================
if __name__ == "__main__":
    print("--- ML Consumer Smart Agro Architecture Simulation ---")
    
    # আপনার জেমিনি এপিআই কী এখানে বসাবেন (নোটপ্যাড থেকে নিয়ে)
    GEMINI_API_KEY = os.getenv("GEMINI_API_KEY", "YOUR_COPIED_GEMINI_API_KEY")
    
    # ডামি আইওটি ও ড্রোন ডেটা (যা আপনার লেয়ার ১ ও ২ থেকে আসবে)
    mock_telemetry = {
        "timestamp": int(time.time()),
        "drone_id": "Heavy-Lift-V2.1",
        "location": "Thakurgaon_Agro_Grid_North",
        "aircraft_vitality_index": 99.8,
        "mustard_oil_batch_purity": 98.7,
        "multispectral_summary": "Nitrogen deficit detected in sector B4. Moisture levels nominal."
    }

    # লেয়ার ৩ রান করা
    pipeline = GoogleCloudDataPipeline()
    # টেস্ট রান করার সময় বিগকোয়েরি ক্রেডেনশিয়াল না থাকলে এটি স্কিপ করতে পারেন
    # pipeline.initialize_storage()
    # pipeline.sync_to_cloud(mock_telemetry)

    # লেয়ার ৪ রান করা
    if GEMINI_API_KEY != "YOUR_COPIED_GEMINI_API_KEY":
        ai_engine = GeminiAgroIntelligence(api_key=GEMINI_API_KEY)
        ai_engine.analyze_crop_health(
            multispectral_summary=mock_telemetry["multispectral_summary"],
            purity_index=mock_telemetry["mustard_oil_batch_purity"]
        )
    else:
        print("[Layer 4] Please replace 'YOUR_COPIED_GEMINI_API_KEY' with your actual generated key to run Gemini AI.")
