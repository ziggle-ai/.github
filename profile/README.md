# We are (almost) 'Ziggle' development team! 🐱

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

![system](../assets/system.png)

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

We have an existing application, Ziggle, and we apply advanced features over Ziggle. We applied the alarm when the user wanted to notice that it was already uploaded, the enhanced search feature, which uses vector similarity, and the deadline auto-detecting feature.

## 1. Posting notices

After logging into our application with Infoteam IDP (you will need the gist email), go to the page to write the notice. Then you can find the UI below.

<p align="center" style="color:gray">
  <img src="../assets/ziggle-write_page.png" width="300"/></br>
  The writing page in Ziggle.
</p>

With the UI above, you can write the notice's body and title. Complete the writing and push the "submit" button. Then, the machine learning feature will be active, checking whether the notice's body is already in the database.

If the notice's body is already in the database, our webpage alerts the user to prevent a conflict notice, and the notice will be uploaded correctly.

<div align="center">
  <table cellpadding="0">
    <tr style="padding: 0"> 
      <td valign="top">
        <p align="center" style="color:gray">
          <img src="../assets/upload-not-similar_notice.png" width="300"/></br>
          If user write the notice which is not in the database.
        </p>
      </td>
      <td valign="top">
        <p align="center" style="color:gray">
          <img src="../assets/upload-similar_notice.png" width="300"/></br>
          If user write the notice which is in the database.
        </p>
      </td>
    </tr>
  </table>
</div>

Since this system can have malfunction, after checking conflict notice, it check users intention with showing the notice that similar to users' input. If user say it is not the similar notice, store the database as edgecase and upload notice with alarm. Else, upload notice without alarm.

<div align="center">
  <table cellpadding="0">
    <tr style="padding: 0"> 
      <td valign="top">
        <p align="center" style="color:gray">
          <img src="../assets/upload-similar_notice-and-false.png" width="300"/></br>
          If user say it is not similar notice.
        </p>
      </td>
      <td valign="top">
        <p align="center" style="color:gray">
          <img src="../assets/upload-similar_notice-and-true.png" width="300"/></br>
          If user say it is similar notice.
        </p>
      </td>
    </tr>
  </table>
</div>

## 2. Searching notices



## 3. Detecting deadline

This feature detects the deadline of the notice that is crawled to the school's webpage. So, this feature cannot be found on our webpage, but it will be applied to the crawling feature in Ziggle.

# 7. Reflection (400 words)

# 8. Broader Impacts (250 words)

# 9. References
