# [Serverless Data Science–Are We There Yet? A Case Study of Model Serving](https://arxiv.org/abs/2103.02958)

SIGMOD '22

Authors: Yuncheng Wu, Tien Tuan Anh Dinh, Guoyu Hu, Meihui Zhang, Yeow Meng Chee, Beng Chin Ooi

Affiliation: National University of Singapore (Wu, Hu, Chee, Ooi), Singapore University of Technology and Design (Dinh), Beijing Institute of Technology (Zhang)

Keywords: Machine Learning, Model Serving, Serverless Computing, Cloud Computing, Deep Learning

## Abstract

The paper evaluates the viability of serverless computing as a mainstream model serving platform for data science applications. It conducts a comprehensive comparison of serverless with other cloud-based model serving alternatives in terms of performance and cost, and presents the design space of serverless model serving.

## Introduction

### Design Goals of Model Serving System and How Serverless can achieve them

1. **high performance:** levarage the elasticity of serverless computing to scale up/down the number of instances to meet the demand quickly to handle the requests timely.
2. **low cost:** billing is based on actual resourse consumption.
3. **ease of management (❌worry about low-level details such as resouses management):** provisioning and scaling of resources are handled automatically by cloud platforms.

### Serverless Computing that Supports Model Serving

![](images/Serverless_Data_Science/Serverless_Data_Science1681228940779.png)

**Limitations:**

- Small Memory Size
- Limited Running Time
- Lack of Persistent Stage

## Background and Related Work

### ML Model Serving

Service level objective (SLO) : Latency

Server machines ❌ work well

### Serverless Computing

Cloud provider runs the server and dynamically manages the resources.

❤️ high elasticity and low cost

### Serverless Model Serving

Process:

1. Upload the model to the cloud via a function
2. Deployed function receive an event from serverless proxy
3. A new instance is created to serve the request (import the dependencies, download the model and load the model into the serving runtime)
4. Prase input from event, run inference and return the result

### Model Serving System on the Cloud

#### Serverless

- **AWS(Amazon Web Service) Lambda**:
  - packge serving environment into **a zip file or a container image**
- **Google Cloud Functions**
  - can only specify the serving environment in **Requirements.txt**, the platform will automatically build the package

#### Manged ML Service

***Capable of Autoscaling, Cost is based on total execution time of active instances***

- AWS SageMaker
- Google AI Platform

#### Self-rented Servers

GPU servers and CPU servers

## Evaluation Methodology

![](images/Serverless_Data_Science/Serverless_Data_Science1681228912940.png)

- Load Generator: use Karkov-Modulated Poisson Process to generate the load with different arrival rates and numbers of requests.
- Planner: 
  - Model: MobileNet, VGG, ALBERT
  - Runtime: TF 1.15 and OnnxRuntime 1.4
  - Configuration: Memory size, CPU cores, ...
- Executor:
- Analyzer: 
  - Respond Latency
  - Request Success Ratio (SR)
  - Cost
  
## Comparison

![](images/Serverless_Data_Science/Serverless_Data_Science1681229970947.png)

- Serverless > Managed ML
  - Severless lantency is high at first: Cold-start
  - ManagedML's Latency is high when request rate⬆️: Autoscaling takes time
  - Severless is cheaper: For managedML, most cost is spent on autoscaling rather than on inference

![](images/Serverless_Data_Science/Serverless_Data_Science1681288773739.png)

- Serverless Outperforms CPU Servers, but -> higher cost
  - CPU servers cannot handle bursty requests well, while severless has superior elasticity

![](images/Serverless_Data_Science/Serverless_Data_Science1681289522741.png)

- Low workload: GPU > Serverless; But under high workload, Serverless is more **stable**
  - When request rate exceeds the capacity of GPU servers, the latency will increase dramatically
  - For large model, GPU server is cheaper; Efficient models leave GPU under-untilized while still being charged.

![](images/Serverless_Data_Science/Serverless_Data_Science1681289780458.png)

## Design Space of Serverless Model Serving

### Platform

***AWS-Serverless > GCP-Serverless***

![](images/Serverless_Data_Science/Serverless_Data_Science1681289849947.png)

- Over-provisioning
- Downloaded size and inference time are significant factors that affect the performance

### Serving Runtime

***Smaller and Faster Runtime is Better***

![](images/Serverless_Data_Science/Serverless_Data_Science1681290358288.png)

### Memory Size

***Larger Memory Size sometimes is more cost-effective***

![](images/Serverless_Data_Science/Serverless_Data_Science1681291129190.png)

### Privisioned Concurrency

![](images/Serverless_Data_Science/Serverless_Data_Science1681291139303.png)

### Batch Size

***Larger Batch Size -> Lower Cost but Higher Latency***

![](images/Serverless_Data_Science/Serverless_Data_Science1681291389891.png)

## Chanllenges and Oppotunities

- over-provisioning problem
- data security
- complexity of design space