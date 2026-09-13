# 👷 PPE-Guard: Construction-Site Safety AI

Welcome to **PPE-Guard**! I built this project to use AI to help keep construction sites safe. By looking at images of a site, my system can automatically check if workers are wearing their required Personal Protective Equipment (PPE) like hardhats and safety vests.

But it doesn't just draw boxes around people! You can actually *ask* it questions in plain English (like "Is anyone missing a helmet?") and it will give you a clear, helpful answer based on what it sees.

---

## 🌟 What does it do?

PPE-Guard is powered by a fine-tuned **RT-DETR** object detection model and wrapped in a fast, easy-to-use **FastAPI** backend. It has two main superpowers:

1. **`POST /detect`**: Upload an image, and the AI will find and highlight workers, hardhats, and safety vests. It can even tell if a worker is missing a helmet or a vest!
2. **`POST /ask`**: Upload an image and ask a question in plain English. I've built a custom reasoning layer that understands your intent, checks the image using the AI model, and gives you a straight answer. If the question isn't related to PPE or the image is too blurry, it safely tells you it doesn't know rather than guessing.

### What can it see?
I specifically trained this model to look for 5 things:
- 👷 **Hardhat** (Wearing a helmet)
- 🚫 **NO-Hardhat** (Missing a helmet - Violation!)
- 🦺 **Safety-Vest** (Wearing a vest)
- 🚫 **NO-Safety-Vest** (Missing a vest - Violation!)
- 🧍 **Person** (Any worker on site)

By specifically training it to spot *violations* (missing gear) as well as *compliance* (wearing gear), the AI is highly effective for real-world safety monitoring.

---

## 🚀 How to Run It

Want to try it yourself? The easiest way is using Docker!

### Quick Start with Docker

```bash
# Start the API and the web interface in one command!
docker-compose up --build
```
Once it's running, open your browser and go to `http://localhost:8000` to try out the interactive web UI!

### Running Locally (For Developers)

If you want to tinker with the code, here's how to set it up locally:

```bash
# 1. Create a virtual environment so you don't mess up your computer's Python
python -m venv venv

# 2. Activate it (On Windows use: venv\Scripts\activate)
source venv/bin/activate       

# 3. Install the required packages
pip install -r requirements.txt

# 4. Set up your environment variables
cp .env.example .env        # Don't forget to add your GEMINI_API_KEY!
```

---

## 🧠 Training Your Own Model

If you want to train the model on your own dataset or just see how I did it, I've got you covered:

```bash
# 1. Download and prepare the dataset (you'll need a Roboflow API key)
python data/prepare_dataset.py --roboflow-key $ROBOFLOW_API_KEY --out data/processed

# 2. Train the RT-DETR model!
python src/train.py \
    --data data/dataset.yaml \
    --model rtdetr-l.pt \
    --epochs 80 --imgsz 640 --batch 16 \
    --project runs/ppe --name rtdetr_ppe_v1

# 3. Test how well the model learned
python src/evaluate.py \
    --weights runs/ppe/rtdetr_ppe_v1/weights/best.pt \
    --data data/dataset.yaml --split test \
    --out docs/eval_report.json
```

---

## 🛠️ API Examples

If you're building an app and want to use the API directly, here are some examples using `curl`.

### Get raw detections
```bash
curl -X POST "http://localhost:8000/detect" \
     -F "file=@tests/sample_requests/site_photo_01.jpg" \
     -F "conf_threshold=0.4"
```

### Ask a question
```bash
curl -X POST "http://localhost:8000/ask" \
     -F "file=@tests/sample_requests/site_photo_01.jpg" \
     -F "question=Is anyone not wearing a helmet in this image?"
```

---

## 📁 What's inside this repo?

Here's a quick map of the project so you know where everything is:

- `api/` - The FastAPI app, including my natural language reasoning engine.
- `src/` - Scripts for training, evaluating, and running the AI model.
- `data/` - Tools for downloading and prepping the dataset.
- `static/` - The frontend web interface.
- `tests/` - Automated tests to make sure everything works perfectly.
- `docs/` - Deeper documentation, including my evaluation reports and analysis of when the AI makes mistakes.

Thanks for checking out PPE-Guard! Stay safe out there. 🏗️
