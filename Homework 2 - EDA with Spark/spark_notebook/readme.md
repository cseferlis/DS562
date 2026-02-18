# Spark Notebook Environment (Docker)

This project provides a ready-to-use Spark + Jupyter Notebook environment.
Students do NOT need to install Python, Spark, or any libraries locally.

You only need to start the environment and open your browser.

---

## 1. Install Docker (one-time setup)

Download and install Docker Desktop:

https://www.docker.com/products/docker-desktop

After installing:

- Start Docker Desktop
- Wait until it shows “Docker is running”
---
## 2. Start the environment

Open a terminal, run:

```
cd DS562/"Homework 2 - EDA with Spark"/spark_notebook
docker compose up -d
```
---

## 3. Open Jupyter Notebook

Open your browser and go to:
```
http://localhost:8888
```

You will see the notebooks interface.

---

## 4. Loading the data

First, download the folders you had in `bronze`.

Put them into `data/` folder.


## 5. Starter code location

Your starter notebooks are already provided.

They appear inside Jupyter under:
```
work/
```

Just open the EDA notebook and start working.

---





