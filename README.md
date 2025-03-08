# floorplanai
from fastapi import FastAPI, UploadFile, floorplanai
import cv2
import numpy as np
import pytesseract
from io import BytesIO
from PIL import Image
import uvicorn

def preprocess_image(image: np.array):
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    edges = cv2.Canny(blurred, 50, 150)
    return edges

def extract_text(image: np.array):
    return pytesseract.image_to_string(image)

def detect_contours(image: np.array):
    contours, _ = cv2.findContours(image, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    return contours

def analyze_floor_plan(image: np.array):
    edges = preprocess_image(image)
    text = extract_text(edges)
    contours = detect_contours(edges)
    num_rooms = len(contours)
    has_balcony = "balcony" in text.lower()
    return {"num_rooms": num_rooms, "balcony": has_balcony, "text": text}

app = FastAPI()

@app.post("/upload/")
async def upload_image(file: UploadFile = File(...)):
    contents = await file.read()
    image = np.array(Image.open(BytesIO(contents)))
    
    result = analyze_floor_plan(image)
    
    return result

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)

git add requirement.txt
git commit -m "Added requirement.txt"
git push origin main
