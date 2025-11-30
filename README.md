# Concrete Anomaly Detection - Streamlit Deployment Guide

## Overview
This Streamlit app deploys three substrate-specific autoencoder models (Bridge Deck, Wall, Pavement) for concrete crack detection using an unsupervised learning approach.

## Project Structure

```
concrete-anomaly-detection/
├── streamlit_app.py          # Main Streamlit application
├── requirements.txt           # Python dependencies
├── README.md                  # This file
└── models/                    # Model files directory
    ├── bridge_deck/
    │   └── checkpoints/
    │       └── deck_autoencoder_epoch45_best.pth
    ├── wall/
    │   └── checkpoints/
    │       └── wall_autoencoder_epoch41_best.pth
    └── pavement/
        └── checkpoints/
            └── pavement_autoencoder_epoch22_best.pth
```

## Setup Instructions

### Option 1: Local Deployment

1. **Clone or download the project files**
   ```bash
   mkdir concrete-anomaly-detection
   cd concrete-anomaly-detection
   ```

2. **Copy your model files**
   - Download trained models from Google Drive
   - Organize in the structure shown above
   - Model files should be in `models/[substrate]/checkpoints/` folders

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Run the app**
   ```bash
   streamlit run streamlit_app.py
   ```

5. **Access the app**
   - Open browser to `http://localhost:8501`

### Option 2: Streamlit Cloud Deployment

1. **Create GitHub repository**
   ```bash
   git init
   git add streamlit_app.py requirements.txt README.md
   git commit -m "Initial commit"
   git remote add origin <your-repo-url>
   git push -u origin main
   ```

2. **Add model files**
   
   **Important:** Model files are too large for GitHub (>100MB each)
   
   **Solution A:** Use Git LFS
   ```bash
   git lfs install
   git lfs track "*.pth"
   git add .gitattributes
   git add models/
   git commit -m "Add model files with LFS"
   git push
   ```
   
   **Solution B:** Host models separately
   - Upload models to Google Drive, Dropbox, or S3
   - Modify `load_model()` function to download from URL
   - Add download logic in `streamlit_app.py`

3. **Deploy to Streamlit Cloud**
   - Go to https://share.streamlit.io
   - Click "New app"
   - Connect your GitHub repository
   - Select `streamlit_app.py` as main file
   - Click "Deploy"

### Option 3: Docker Deployment

1. **Create Dockerfile**
   ```dockerfile
   FROM python:3.9-slim
   
   WORKDIR /app
   
   COPY requirements.txt .
   RUN pip install --no-cache-dir -r requirements.txt
   
   COPY streamlit_app.py .
   COPY models/ models/
   
   EXPOSE 8501
   
   CMD ["streamlit", "run", "streamlit_app.py", "--server.port=8501", "--server.address=0.0.0.0"]
   ```

2. **Build and run**
   ```bash
   docker build -t concrete-anomaly-detection .
   docker run -p 8501:8501 concrete-anomaly-detection
   ```

## Downloading Models from Google Drive

Your trained models are located at:
```
/content/drive/MyDrive/Infrastructure_Anomaly_Detection/models/
```

**To download from Colab:**

```python
# Run this in a Colab cell
from google.colab import files
import shutil
from pathlib import Path

# Create zip file with all models
!zip -r models.zip /content/drive/MyDrive/Infrastructure_Anomaly_Detection/models/

# Download
files.download('models.zip')
```

**Then on your local machine:**
```bash
unzip models.zip
mv models/ concrete-anomaly-detection/
```

## Model Information

| Substrate | Model File | Epoch | Val Loss | Separation | Accuracy |
|-----------|------------|-------|----------|------------|----------|
| Bridge Deck | deck_autoencoder_epoch45_best.pth | 45 | 0.000067 | 1.2x | 59% |
| Wall | wall_autoencoder_epoch41_best.pth | 41 | 0.000025 | 1.4x | 60% |
| Pavement | pavement_autoencoder_epoch22_best.pth | 22 | 0.000071 | 1.3x | 59% |

## Usage

1. **Select Substrate Type** in sidebar (Bridge Deck, Wall, or Pavement)
2. **Upload Image** of concrete infrastructure
3. **Click "Analyze Image"** to run detection
4. **View Results**:
   - Reconstruction visualization
   - Anomaly score
   - Detection result (Normal/Crack)
   - Score comparison chart

## Configuration

### Threshold Modes

- **Balanced**: Equal false positives/negatives (~59% accuracy)
- **Conservative**: Fewer false alarms, may miss some cracks
- **Custom**: Manually adjust threshold slider

### Threshold Values

| Substrate | Balanced | Conservative |
|-----------|----------|--------------|
| Bridge Deck | 0.000163 | 0.001025 |
| Wall | 0.000091 | 0.000180 |
| Pavement | 0.000100 | 0.000200 |

## Limitations

⚠️ **This is a research prototype with known limitations:**

1. **Low Separation (1.2-1.4x)**: Insufficient for production use
2. **Data Contamination**: Training data contained mislabeled examples
3. **~59% Accuracy**: Not suitable for critical infrastructure decisions
4. **Unsupervised Limitations**: Cannot distinguish cracks from texture

## Next Steps

- **Phase 2**: Supervised classification (85-95% accuracy target)
- **Thermal Imaging**: Alternative modality for defect detection
- **Data Cleaning**: Remove contaminated training examples

## Troubleshooting

**Models not loading:**
- Verify model files are in correct directory structure
- Check file names match exactly
- Ensure model files are not corrupted

**Out of memory:**
- Reduce image size in `preprocess_image()` function
- Use CPU instead of GPU: `map_location='cpu'` (already default)

**Slow performance:**
- Models run on CPU by default (no GPU required)
- First load caches model in memory
- Subsequent predictions are fast

## Demo Script

When presenting this prototype, emphasize:

1. **Technical Achievement**: All models trained successfully with low loss
2. **Key Discovery**: Data contamination issue identified
3. **Honest Assessment**: ~60% accuracy insufficient for production
4. **Path Forward**: Supervised learning approach in development

**Sample talking points:**
> "This prototype successfully demonstrates substrate-specific modeling infrastructure, 
> but revealed that unsupervised autoencoders achieve only 1.2-1.4x score separation due 
> to data contamination. This diagnostic phase validates our pivot to supervised classification, 
> which is expected to achieve 85-95% accuracy."

## Support

For questions or issues:
- Review "About This Prototype" section in the app
- Check "Technical Details" expander for architecture info
- Refer to training logs for model performance metrics

## License

Research prototype - not for production use.
