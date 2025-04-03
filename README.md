# Dockerizing Legacy Apps Managing via Azure Container Services.

![Docker](https://img.shields.io/badge/Docker-Containerization-blue?logo=docker)
![DevOps](https://img.shields.io/badge/DevOps-CI/CD-orange?logo=githubactions)
![Python](https://img.shields.io/badge/Python-Django-green?logo=python)
![Angular](https://img.shields.io/badge/Angular-Frontend-red?logo=angular)

## 📌 **Introduction**
This project demonstrates the **containerization** and **deployment** of a full-stack Legacy apps with underlying different technologies such as Angular, Python, PHP and ASP.Net
It showcases my **DevOps** skills in setting up microservices, handling environment variables, debugging, and automating deployments.

### 🔹 **Tech Stack**
- **Frontend:** Angular (served via lightweight web server)
- **Backend:** Django (REST API)
- **Containerization:** Docker
- **Orchestration:** Docker Compose (optional)
- **Database:** PostgreSQL (can be added later)

---

## 🎯 **Motive & Plan**

### ✅ **Motive**
- Gain hands-on experience in **Docker & DevOps**
- Automate **frontend & backend deployments** using containers
- Learn **troubleshooting** and **debugging** in a containerized environment

### ✅ **Plan / Expected Outcome**
1. **Containerize the Angular frontend**
2. **Containerize the Django backend**
3. **Handle environment variables securely**
4. **Resolve common deployment issues**
5. **Improve DevOps workflow**

---

## 🔥 **Challenges Faced & How I Solved Them**

### ❌ Issue 1: Backend Container Crashing (Missing `DEBUG` Env Variable)
**Error:**  
```plaintext
KeyError: 'DEBUG'
django.core.exceptions.ImproperlyConfigured: Set the DEBUG environment variable

