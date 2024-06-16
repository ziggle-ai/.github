# We are (almost) a 'Ziggle' development team! 🐱

> Team information

1. 김영목, AI 22 ✉️ mailto: <ai5308-2024spr@sangyul.ee>
2. 이상유, EECS 19 ✉️ mailto: <kym990708@gm.gist.ac.kr>
3. 황인선, [Personal page](https://github.com/inthree3)
4. 박시원, [Personal page](https://github.com/siwonpada)
5. 조민준, [Personal page](https://github.com/minjunj)

# Ziggle ➡️ Better notification platform!

> Problem definition

### 1. Alarm Muting System

Most general **user dissatisfaction voices** came from the duplicated alarms for certain notices.

`😠: "Be quiet! Why do you send me alarms several times for the same notice??"`

These dissatisfactions were because some notice writers delete and reupload their notices when they make a subtle mistake, rather than use our application editing functions.

We needed an automatic system to handle this problem rather than relying on users' appropriate actions.

### 2. Deadline Detection System

For the user's comfort, we automatically crawl the GIST's undergraduate notices and upload them to our Ziggle app.

We have features to set deadlines for each notice, but for undergraduate notices, it is impossible to register the deadlines because those are automatic processes.

Plus, the deadline information is vital since our platform is a notification platform as this is linked to other functions such as **reminder push alarms** and **sorting by deadlines**.

# System Design
<div align="center">
  <img src="../assets/system.png" width=500 />
</div>

## 1. Data Pipelines

Our team operated the Ziggle AI system by configuring a continuous deployment(CD) using the GitOps method in **Kubernetes**. 

To separate the demo environment from the production environment and to efficiently progress, the Back-end server used the existing Ziggle's, while separating the DataBase for a demo. Since AI libraries are generally optimized to **Python**, the AI part is implemented by configuring a separate Python-based API server.

## 2. Modeling

<div align="center">
  <img src="https://github.com/ziggle-ai/.github/assets/42310616/5b6efc40-dd68-4e0f-8612-923fa918978c"/>
</div>

We used Unsloth for finetuning the Llama models to detect the deadline for a given notice. We chose this library because this is optimized for training Large Language Models(LLMs) with local computer resources.

GPT API is used for building fine-tuning datasets. Since we applied the idea of **transfer learning** the GPT 4o served a role as a **parent**.

For managing the dataset and the (pre-trained) models, we used Hugging Face. This helped to reduce the workload for developing the model training pipeline.

## 3. User Experience (UX)

To attach this system to Ziggle more softly, the frontend part used Ziggle code with additional UI components for demoing the AI features.

<div display="flex" align="center">
  <img src="https://github.com/ziggle-ai/.github/assets/42310616/59886ea2-e0ae-4392-8d96-a3800bb4c22a" width="200"/>
  <img src="https://github.com/ziggle-ai/.github/assets/42310616/90ee6492-3ae3-4abd-8690-3991b6035dd6" width="600"/>
</div>

Our AI system is running and displayed naturally without changing the previous Ziggle UX significantly. And also, the user interface (UI) can deal with various screen sizes including mobile phone, laptop, and desktop screens.

## 4. Deployment

### 1. Frontend

Since frontend deployment is forked Ziggle frontend, our frontend follows the tech stack of Ziggle frontend, so we use the Nextjs framework for developing frontend applications. It is responsible for showing UI and using the backend's API. Finally, To deploy the application, we used a docker container.

### 2. Backend

The backend application in this project uses the Fastapi framework, MongoDB, and Openai library. It is responsible only for AI features. This application does not impliment the API for the ordinary feature, and it does not perform machine learning, so it does not use GPU resources. Therefore, it uses the docker container without GPU resources.

### 3. Infrastructure

In order not to affect existing services, a Kubernetes cluster was configured in the On-Premise environment. A separate Ziggle namespace was created to distribute the front-end and AI's Kubernetes deployment and expose it to the outside through a service component.

All distributions were centered on Github's organization, and a repository for each part was created and managed. In addition, a CD was built through Github activities. ArgoCD is adopted for continuous delivery and verification of the status of each service while managing all codes through Github. If any changes occur in each repository, the Github action bot changes the contents of the infra repository.

ArgoCD monitors the infra Repository, automatically creating a new pod when a change occurs in that repository, and implements Continuous Delivery by deleting the existing pod.

# Machine Learning component

## 1. Document Similarity-Based Features

### 1. Document embedding with GPT model
We used the `text-embedding-3` model to get a vector from a notification. If an article is bigger than the token limitation, we simply cut it into an embeddable length.

### 2. Chunk-based embedding and similarity score. (Further task)

There are several similarity search mechanisms. And the most popular three similarity formulas are Euclidian, cosine similarity, and dot product. As the main target was to detect the almost same article, the ** dot product** which considers not only the direction but also the magnitude that is proportionate to the text length was selected as the matrix for calculating the text similarity

## 2. Deadline Detection
We first plan to use the GPT API for the deadline detection feature. But during our development period, [GPT was one down](https://www.merca20.com/yes-chatgpt-is-down-today-why-is-it-not-working/). To retain the credibility of our service, we determined to prepare another detection system. Then, we are now serving our own fine-tuned local model.

> Data Preparation

We aimed to conduct transfer learning with pre-trained large language models. Since there is no training dataset for which the goal is to extract the deadline information from a given document, we prepared the training dataset by inferencing the deadline with our Ziggle's document data.

> Select a Training library

Since the lack of resources for serving and funds, we need to reduce the cost of serving and adjusting our model. Fine-tuning model on colab became possible with [Unsloth](https://unsloth.ai/) library. 

- QLoRA: efficient finetuning of Quantized LLMs
  
  To train fast while maintaining the task performance, we adapted the parameter efficient finetuning. In particular, QLoRA which is a quantized version of LoRA was used. By this method, the speed of the training was enhanced.
  
- Llama-3 & TinyLlama
  
  Among lots of large language models, we chose to use Llama-3. Additionally, for the purpose of reducing memory usage, we finally selected TinyLlama which uses only 1.1B parameters that are smaller than Llama 3 which has 8B to 70B parameters.

# System evaluation
> Several local base model, and compare their score and cost

### 1. Accuracy

![image](https://github.com/ziggle-ai/.github/assets/42310616/0bc2fa09-b3cf-490b-8626-931892cced2e)

In the sense of comparing two local finetuning models, we present the accuracy from Llama 3 and TinyLlama. The training process is displayed above, and the accuracy score of our own built data is as follows.

|Model Name|Loss|
|---|---|
|Llama 3|1.677|
|TinyLlama|0.8692|

But, if we look into practical tests, the efficiency of each model is quite different.

> The Llama 3 Case

```
### Instruction:
## 당신의 목적
당신은 공지글로부터 신청 마감 기한, 행사 시작 시간과 같은 deadline 정보를 알아내는 AI 봇입니다.

## 입력 정보
**공지의 생성 일자**와 **공지 본문**이 제공됩니다.

## 출력 값
만약, deadline을 알아내기에 정보가 충분하지 않다면 빈 문자열을 반환하고, 정보가 있다면 '%Y-%m-%d %H:%M:%S'의 datetime.datetime 형식의 string으로 deadline을 알려주세요.

## 출력 형식
다음과 같은 json 형태로 출력하세요 {'deadline': ''}

### Input:
## 공지의 생성 일자
2024-06-09 00:00:000

## 공지 본문
오늘은 일요일입니다! 다음주 수요일까지 제 생일파티의 참여를 신청받습니다~ 많은 신청바랍니다!~

이제 json 형식으로 마감 기한 정보를 추출해주세요.

### Response:
{'deadline': '2024-06-12 23:59:00'}<|end_of_text|>
```

At least, the response format is right. But the smaller parameter model, TinyLlama often fails to match the right format.

```
["<s> Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request.\n\n### Instruction:\n**당신의 역할 **\n당신은 공지글로부터 신청 마감 기한, 종료일, 행사 시작 시간과 같은 마감 기한 정보를 알아내는 AI 봇입니다.\n\n** 입력 정보 **\n**공지의 생성 일자**와 **공지 본문**이 제공됩니다.\n\n** 출력 값 **\n만약, deadline을 알아내기에 정보가 충분하지 않다면 빈 문자열을 반환하고, 정보가 있다면 '%Year-%month-%day %Hour:%Minute:%Second'의 datetime 형식의 string으로 deadline을 알려주세요.\n** 출력 형식 **\n다음과 같은 json 형태만 출력하세요 {'deadline': ''}\n\n### Input:\n\n🧩🍗 퍼즐 풀고 치킨 받자~! [GIST DSLAB 퍼즐 게임 출시] 🍗🧩\n\n안녕하세요, GIST Data Science Lab에서 퍼즐 게임 O2ARC 3.0을 출시했습니다!\n\n주어진 규칙을 추측해서 정답을 만들어서 점수를 쌓아보세요!\n\n매월 🔥TOP 1, 2, 3위🔥를 선정해서 🍗치킨🍗 or ☕커피☕ 쿠폰을 드립니다!\n\n \n\n🔥🔥🔥접속 링크🔥🔥🔥\n\nhttps://o2arc.com/\n\n🔥🔥🔥🔥🔥🔥🔥🔥🔥🔥\n\n4월 순위 산정 기한 : 4월 1일 00시 00분 ~ 4월 30일 23시 59분\n\n \n\n✨ 문제 풀이 Tip! ✨\n\n적은 시간과 동작을 사용한 효율적인 풀이일수록 높은 점수를 얻을 수 있습니다!\n\n메인 페이지에 낮은 난이도 순서대로 문제가 정렬되어 있습니다!\n\n \n\n⚠️ 주의! ⚠️\n\n가입 시 사용하는 이메일로 경품을 지급하니, 실제로 사용하는 이메일로 가입해주세요!\n\n \n\n💡문의처💡\n\nsuyeonshim@gm.gist.ac.kr\n\nGIST DS Lab 심수연 인턴\n\n\n### Response:\n{'deadline': 2024-03-1-10 12:00:0}\n</s>"]
```

### 2. Inference Time

The inference time was tested with 5 cases and overall, the inference time was short enough to be able to be applied in our running service.

|Deadline Detection (Llama 3)|Deadline Detection (TinyLlama)|Alarm Muting|
|---|---|---|
|< 5s|< 5s|< 3s|

# Application demonstration
> Application link: [Ziggle AI ↗️](http://210.125.85.31:32442/ko/home?page=0)

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
          If the user writes a notice which is not in the database.
        </p>
      </td>
      <td valign="top">
        <p align="center" style="color:gray">
          <img src="../assets/upload-similar_notice.png" width="300"/></br>
          If the user writes the notice which is in the database.
        </p>
      </td>
    </tr>
  </table>
</div>

Since this system can malfunction, it checks the user's intention by showing a notice that is similar to the user's input. If the user says it is not a similar notice, store the database as an edge case and upload the notice with an alarm. Otherwise, upload the notice without alarm.

<div align="center">
  <table cellpadding="0">
    <tr style="padding: 0"> 
      <td valign="top">
        <p align="center" style="color:gray">
          <img src="../assets/upload-similar_notice-and-false.png" width="300"/></br>
          If the user says it is not similar notice.
        </p>
      </td>
      <td valign="top">
        <p align="center" style="color:gray">
          <img src="../assets/upload-similar_notice-and-true.png" width="300"/></br>
          If the user says it is a similar notice.
        </p>
      </td>
    </tr>
  </table>
</div>

## 2. Searching notices

Since we use vector similarity with vector embedding tech, we can use it for searching. Therefore, we apply this in the searching UI which already exists. If you just write the keyword in the search bar, the ML feature is activated, and find a similar notice.

<p align="center" style="color:gray">
  <img src="../assets/searching.png"/></br>
 the result of the search feature.
</p>

## 3. Detecting deadline

This feature detects the deadline of the notice that is crawled to the school's webpage. So, this feature cannot be found on our webpage, but it will be applied to the crawling feature in Ziggle. 

# Reflection

## 1. What we worked

### 1. Alarm Muting System

Our purpose is to reduce the notice alarm that comes from actually the same notices. With a vector database, we solved this problem. 

We used the `text-embedding-3` model of OPENAI to get a vector from a notice. Additionally, we used MongoDB's vector storing feature to store the vector that comes from the text-embedding model. After storing vector data, Using users' input and MongoDB's vector searching feature, we provide the result of whether similar notices exist and the list of similar notices. As a result, using text-embedding and vector search technology, we alert the user that a similar notice already exists in the database through the existing Ziggle UI. Furthermore, we implemented the search feature with vector searching technology.

### 2. Deadline Detection System

Our other purpose was to attach the deadline information to the academic notice crawled on the school's webpage. To solve this problem, we need to find or detect the deadline from the notice on the school's webpage. We used Lama-3 and trained it. However, it is for crawling data and is not shown in the Ziggle UI.

## 2. Weakness

We used the text-embedding model, but we did not use chunking when using the text-embedding model. 

Since this is a demo application, its database is not directly reflected in the production database. Our production database is also Postgresql, which is completely different from MongoDB. It must deploy two databases if we run it at the production level. Therefore, to reduce the complexity of the operation, we need to reduce the type of database.

## 3. Further Work

To address the weakness we discussed, we developed a method that uses chunking technology and reduces the database's type. We will use Postgresql's vector database feature. 

The final purpose of our team is to apply this AI feature to the production server and frontend UI. We need the server that deploys the LLM to use the AI feature, so we need to find the resources to calculate the LLM cheaply.

# Broader Impacts

In the process of adopting an AI model that detects duplicate notices in order to avoid duplicate notice notifications, there was an omission due to unintended use. For example, when comparing titles, "Data Engineering May Event" and "Data Engineering May Event" tended to be classified into completely different notices. To solve this problem, we adopt a similarity measuring system to show how similar the notice is to the existing notice, show an existing notice that can be duplicated, and give the user a final choice, if correct, send a notice without notice, and if not, send a notification.

 Since the AI system requires a GPU, if all elements of the Ziggle are monolithically configured, the server tends to go down due to the load of the GPU as soon as the notice writer is crowded. Therefore, the load point is clearly separated by operating a separate server for the AI system and adopting a Micro Service Architecture (MSA) method that separates the Ziggle FE and traditional BE servers. In addition, in order to efficiently operate and monitor each divided server,  we chose Kubernetes and Gitops methods to automate distribution according to the user load and design to enable stable service operation.

# 9. References
