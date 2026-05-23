pipeline {
  agent any

  environment {
    PROJECT_ID = "Jenkins-Cicd-project"
    BUCKET     = "my-jenkins-artifacts-bucket"
    ZONE       = "us-central1-c"
    VM_NAME    = "app-runtime-vm"

    // Path to the jar inside GitHub repo
    LOCAL_JAR  = "artifacts/app.jar"

    // Where it goes
    GCS_OBJECT = "app/app.jar"
    VM_PATH    = "/opt/app/app.jar"

    GCP_KEY_CRED = "gcp-sa-key"
    GITHUB_CRED  = "github-pat"
  }

  stages {
    stage("Checkout") {
      steps {
        git branch: "main",
            credentialsId: env.GITHUB_CRED,
            url: "https://github.com/rahulgoud213/Test-Deployment-.git"
      }
    }

    stage("Verify JAR Exists") {
      steps {
        sh """
          if [ ! -f "${LOCAL_JAR}" ]; then
            echo "ERROR: JAR not found at ${LOCAL_JAR}"
            exit 1
          fi
        """
      }
    }

    stage("Auth to GCP") {
      steps {
        withCredentials([file(credentialsId: env.GCP_KEY_CRED, variable: "GCP_KEY_FILE")]) {
          sh """
            gcloud auth activate-service-account --key-file="$GCP_KEY_FILE"
            gcloud config set project ${PROJECT_ID}
          """
        }
      }
    }

    stage("Upload to GCS") {
      steps {
        sh """
          gsutil cp ${LOCAL_JAR} gs://${BUCKET}/${GCS_OBJECT}
        """
      }
    }

    stage("Deploy & Restart") {
      steps {
        sh """
          gcloud compute ssh ${VM_NAME} --zone ${ZONE} --command '
            sudo mkdir -p /opt/app
            sudo gsutil cp gs://${BUCKET}/${GCS_OBJECT} ${VM_PATH}
            sudo systemctl restart myapp || nohup java -jar ${VM_PATH} > /opt/app/app.log 2>&1 &
          '
        """
      }
    }
  }
}