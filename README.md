# ❌ FloorPlan Detection (Not Finished)

![FloorPlan](examples/image.png)

## Overview

This repository contains the API code for detecting walls and rooms in architectural floor plans. 
I’m also sharing my notebooks, research results, and some thoughts on how the results could be improved.

## How to run

### Model Weights
Download the  [pre-trained weights](https://drive.google.com/file/d/1A9sZlM8NXe2bA7jgzm7jNeiB6oXjDUP-/view?usp=sharing) and place them in the `/models` directory. You can adjust model types by specifying them when running the server.

### Build with Docker
I used Kaggle to experiment, so the image weighs a gazillion (no, seriously, 66) gb. If I don't forget, I'll rebuild a lighter image.

To build the code environment, use the provided Dockerfile in the root directory:

`docker build -t floorplan-detection . -f Dockerfile`

Please don't forget:

`docker system prune`

And use command to run container:

`docker run -it --rm --gpus all -p 5000:5000 florplan_detection:latest bash`

### Start the Server
To start the FastAPI server, use the following command:

`python run.py --model <model_name>`

### API Request

I assume that the API input is an image with a region of interest containing only the architectural plan

```shell
curl -X POST "http://localhost:5000/run-inference?type=wall" \
     -H "Content-Type: multipart/form-data" \
     -F "image=@/path/to/your/image.jpg"
```

The type parameter (wall or room) is required but doesn't affect the response structure. The API will return a JSON response with both detected walls and rooms.

```json
{
  "type": "floor_plan",
  "confidence": 0.8851,
  "detectionResults": {
    "walls": [
      {
        "id": "wall_1",
        "position": {
          "start": {"x": 10.5, "y": 20.3},
          "end": {"x": 110.2, "y": 20.3}
        },
        "confidence": 0.9234
      },
      ...
    ],
    "rooms": [
      {
        "id": "room_1",
        "position": {
          "start": {"x": 20.3, "y": 25.7},
          "end": {"x": 105.6, "y": 125.9}
        },
        "confidence": 0.8352
      },
      ...
    ]
  }
}
```

## About

### First things first
The task doesn’t come with labeled data or evaluation criteria, so I treated it as an open-ended project. The provided floor plan examples are inconsistent—using different labels, line thicknesses, and so on—so I figured neural network-based methods would be the best approach here.

### Dataset
I’m using the `CubiCasa5k` dataset. This dataset provides annotations for different room types, but I’m focusing on two categories: Walls and Rooms (without further subclassification). The dataset initially comes with masks, but I use only bboxes. I’ve converted it to `COCO` format for smoother integration with detection framework. 

- If you want to use [original dataset](https://zenodo.org/records/2613548), you need to download it and put into `data/CubiCasa5k/data` directory. For a quick overview of the dataset, check out the notebook: `/notebooks/cubicasa5k_dataset.ipynb`
- If you want to use my [CubiCasa5k_COCO](https://drive.google.com/drive/folders/1hKRWrP-ZKk6ZHrjHOSRSxPe_r_kMd8uh?usp=sharing) dataset of train another model on it, you need to download it and puth into `data/cubicasa5k_coco` directory. You still need images in `data/CubiCasa5k/data`.
- If you want to modify original data to coco format (with masks or anything else) you can use `data/cubicasa5k_to_coco.py` script.

### Models

When you run server you should pass `--model` argument. Supported models are:
- `faster_rcnn`
- `retinanet`

You need download [model weights](https://drive.google.com/drive/folders/1MgW3Qo-8K4OrHi4ebvYd-81cTqQxwLgz?usp=sharing) and put into `weights/` folder.

### Research

- DeepFlorPlan - `/notebooks/deepflorplan_model.ipynb`
- SAM Baseline - `/notebooks/sam_baseline.ipynb`

### Notebooks

- CubiCasa5k Dataset - `/notebooks/cubicasa5k_dataset.ipynb`
- Inference and Visualization - `/notebooks/inference_and visualization.ipynb`