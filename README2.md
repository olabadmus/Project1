CICD_HELM_EKS

1. Setup EKS Cluster using Terraform <br>
  - Navigate to /terraform folder and run the following commands:
  - Terraform init
  - Terraform plan
  - Terraform apply -auto-approve

![image](https://github.com/user-attachments/assets/a931dffc-ead1-4aee-8894-c2c70054432e)

**Confirmed EKS was created in aws
**

![image](https://github.com/user-attachments/assets/d359b670-9c83-47c5-ac4c-a89cf8ad33ec)

![image](https://github.com/user-attachments/assets/00290d60-b763-4931-baec-302c7bda653f)


2. Navigate to /cicd_helm_eks
  - Delete helm-chart

![image](https://github.com/user-attachments/assets/8be37494-a263-4cde-ab55-bd82dfbbc3ff)


3. Create a new “helm-chart” folder and switch to /helm-chart
  - Execute the following command - helm create docker-cicd

![image](https://github.com/user-attachments/assets/37e079c9-f365-4001-b049-44d068f67e52)


4. In the project work directory, create a folder /.github/workflows and create a deploy.yaml file 
 Paste the following inside the deploy.yaml file

![image](https://github.com/user-attachments/assets/972469e0-2dab-4542-b05f-73ca51a77167)

```
name: Deploy to EKS

on:
  push:
    branches:
      - main      
  workflow_dispatch: null 
permissions: write-all     
env:
  EKS_CLUSTER_NAME: cicd-cluster
  AWS_REGION: us-east-1
  DOCKER_REPOSITORY: goapp      

jobs:
  build:
    name: Deployment
    runs-on: ubuntu-latest
    steps:
      - name: Set short git commit SHA
        id: commit
        uses: prompt/actions-commit-hash@v2
      - name: Check out code
        uses: actions/checkout@v2
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{env.AWS_REGION}}
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build, tag, and push image to Dockerhub
        env:
          DOCKER_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
          IMAGE_TAG: ${{ steps.commit.outputs.short }}
        run: |
          docker build -t $DOCKER_USERNAME/$DOCKER_REPOSITORY:$IMAGE_TAG .
          docker push $DOCKER_USERNAME/$DOCKER_REPOSITORY:$IMAGE_TAG
      - name: Update kube config
        run: aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_REGION
      - name: Update Image tag
        env:
          IMAGE_TAG: ${{ steps.commit.outputs.short }}
        run: |
          sed -i.bak "s|latest|$IMAGE_TAG|g" helm-chart/docker-cicd/values.yaml
      - name: update docker Username and Image name
        env:
          USER_NAME: ${{ secrets.DOCKERHUB_USERNAME }}
          IMAGE_NAME: ${{ env.DOCKER_REPOSITORY }}
        run: >
          sed -i.bak "s|username|$USER_NAME|g" helm-chart/docker-cicd/values.yaml &&
          \

          sed -i.bak "s|imagename|$IMAGE_NAME|g" helm-chart/docker-cicd/values.yaml
      - name: Deploy helm chart
        uses: WyriHaximus/github-action-helm3@v3
        with:
          exec: helm upgrade --install docker-cicd helm-chart/docker-cicd/
```

5. Setup Github secrets from, Settings > Secrets and Variables > Actions

![image](https://github.com/user-attachments/assets/3b0211df-2c2a-453b-98f3-34bc5b23a676)

6. Push the changes you have locally to github using the following commands
  - Git add .
  - Git commit -m “initial commit”
  - Git push

![image](https://github.com/user-attachments/assets/d48c5608-62d0-4f43-a8a0-b4aba54f1aca)

![image](https://github.com/user-attachments/assets/e759f8df-e9ff-4252-aa32-524ec9ccba04)

![image](https://github.com/user-attachments/assets/0c4d2fff-6594-4d18-b437-4860e3cbcf9a)

![image](https://github.com/user-attachments/assets/f55b6072-5159-4639-b335-dbaa0ce69cfd)

7. To access the App via browser
  - Run kubectl get svc

  - Enter kubectl edit svc/docker-cicd and change the type to “Loadbalancer” on line 42

![image](https://github.com/user-attachments/assets/7018d5e3-7ce8-4015-b2b6-de97e7a793ea)

![image](https://github.com/user-attachments/assets/d05ba2b7-92e1-496e-8ee8-84ceaa7d731b)

8. Do kubectl get svc and enter the long external IP in your browser

![image](https://github.com/user-attachments/assets/10b385ea-1ab6-4deb-9ded-bcfea687d54e)


![image](https://github.com/user-attachments/assets/b1060ba1-b206-490f-90c0-5556139905c9)











