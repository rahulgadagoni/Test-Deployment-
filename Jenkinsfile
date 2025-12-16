pipeline {
  agent any

  environment {
    PROJECT_ID  = "Jenkins-Cicd-project"
    BUCKET      = "my-jenkins-artifacts-bucket"
    APP_VM_IP   = "136.116.208.47"
    APP_DIR     = "/opt/app"
    JAR_NAME    = "app.jar"
    CREDS_GCP   = "gcp-sa-key"     // Jenkins credential ID for the JSON key (Secret file)
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build JAR') {
      steps {
        sh 'mvn -B clean package -DskipTests'
        sh 'ls -lh target/*.jar'
      }
    }

    stage('Upload to GCS') {
      steps {
        withCredentials([file(credentialsId: "${CREDS_GCP}", variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh '''
            gcloud auth activate-service-account --key-file="$GOOGLE_APPLICATION_CREDENTIALS"
            gcloud config set project "$PROJECT_ID"
            gsutil cp target/*.jar gs://$BUCKET/$JAR_NAME
          '''
        }
      }
    }

    stage('Deploy to VM') {
      steps {
        sh '''
          ssh -o StrictHostKeyChecking=no $APP_VM_IP "
            sudo mkdir -p $APP_DIR
            sudo chown \$USER:\$USER $APP_DIR
            gsutil cp gs://$BUCKET/$JAR_NAME $APP_DIR/$JAR_NAME
            pkill -f 'java -jar' || true
            nohup java -jar $APP_DIR/$JAR_NAME > $APP_DIR/app.log 2>&1 &
          "
        '''
      }
    }
  }
}
