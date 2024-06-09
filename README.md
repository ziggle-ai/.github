# Most of us are from 'Ziggle' development team 🐱

> Team information

1. 김영목, AI 22
2. 이상유, EECS 19
3. 황인선, [Personal page](https://github.com/inthree3)
4. 박시원, [Personal page](https://github.com/siwonpada)
5. 조민준, [Personal page](https://github.com/minjunj)

# Ziggle ➡️ Better notification platform!

> Problem definition

### 1. Alarm Muting System

Most general **user dissatisfaction voice** were came from the duplicated alarms for certain notice.

`😠: "Be quiet! Why do you send me alarms several time for a same notice??"`

These dissatisfactions were because some notice writers delete and reupload their notices when they make a subtle mistake, rather than use our application editting functions.

### 2. Deadline Detection System

For user's comfort, we automatically crawl the GIST's undergraduate notices and upload them to our Ziggle app.

We have features to set deadline for each notices, but for the undergraduate notices it is impossible to register the deadlines.

Plus, since our platform is notification platform, the deadline information is really important. And this AI system could applied for other necessary purposes like checking for writing deadlines and so on .

# Ziggle AI: System Design

![system](./assets/system.png)

## 1. Data Pipelines

Our team operated the ziggle ai system by configuring a CD using GitOps method in Kubernetes. For the separation of the demo environment and efficient project progress, the Back-end server used the existing ziggle's, separated the DataBase for demo, and operated the Front-end for additional UI. In order to efficiently manage Production AI, the ai system implement by configuring a separate Python-based API server. The accumulated data while operating ziggle was embedded in the GPT API and stored in the Vector DB to construct an efficient AI system.

## 2. Modeling

### 1. Unsloth

We used Unsloth for finetuning the Llama 3 model to detect the deadline for given notices.

We chose this library because this is optimized for training Large Language Models(LLMs) with local computer's resource.

### 2. GPT API

GPT API is used for building fine tuning dataset.

Since, we applied the idea of **transfer learning** the GPT 4o gave us the root

### 3. Hugging Face

For managing the dataset and the (pre-trained) models, we used Hugging Face.

This helps to reduce the workloads for developing the model training pipeline.

### 4. will be updated

## 3. User Interface System

We basically forked the existing Ziggle frontend repository and edited for demoing and grasping the built AI features.

### 1. will be updated

# 4. Machine Learning component (300 words)

# 5. System evaluation (500 words)

# 6. Application demonstration (300 words)

# 7. Reflection (400 words)

# 8. Broader Impacts (250 words)

# 9. References
