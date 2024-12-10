# se-as
Lenguaje de señas
name: CD Pipeline for Flask App
on:
  push:
    branches:
      - main
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    # 1. Checkout del código fuente
    - name: Check out code
      uses: actions/checkout@v3
    # 2. Configurar Google Cloud SDK
    - name: Set up Google Cloud SDK
      uses: google-github-actions/setup-gcloud@v1
      with:
        project_id: ${{ secrets.flask-app-444200 }}
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        export_default_credentials: true
    # 3. Autenticar Docker con Google Cloud
    - name: Authenticate Docker
      run: gcloud auth configure-docker
    # 4. Construir y etiquetar la imagen Docker
    - name: Build and Tag Docker Image
      run: |
        docker build -t gcr.io/${{ secrets.flask-app-444200 }}/flask-app:latest .
        docker push gcr.io/${{ secrets.flask-app-444200 }}/flask-app:latest
    # 5. Desplegar en Cloud Run
    - name: Deploy to Cloud Run
      run: |
        gcloud run deploy flask-app \
          --image gcr.io/${{ secrets.flask-app-444200 }}/flask-app:latest \
          --platform managed \
          --region us-central1 \
          --allow-unauthenticated
