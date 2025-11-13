pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "tygroo972/devops-appbbo:${GIT_COMMIT}"
    applicationURL="newdevsecops1.eastus.cloudapp.azure.com"
    applicationURI="increment/99"
    withCredentials([string(credentialsId: 'secret_npi', variable: 'secret_npi')]) {
    NVD_API_KEY=$secret_npi
      }
  }

  stages {

    stage('Vulnerability Scan - Docker') {
          steps {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              sh "sudo mvn dependency-check:check"
            }
          }
          post {
            always {
              dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
              jacoco(execPattern: 'target/jacoco.exec')
            }
          }
        }


      stage('Build Artifact') {
      steps {
        sh 'sudo mvn clean package -DskipTests=true'
        archive 'target/*.jar' //so that they can be downloaded later test aa
      }
      }
    
    //--------------------------
    stage('UNIT test & jacoco ') {
      steps {
        sh "sudo mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }

    }
//--------------------------
    stage('Mutation Tests - PIT') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
          sh "sudo mvn org.pitest:pitest-maven:mutationCoverage"
        }
      }
        post { 
         always { 
           pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
         }
       }
    }
//--------------------------
stage('scan sonarqube') {
              steps {
 
          
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
 
  sh "sudo mvn clean verify sonar:sonar \
  -Dsonar.projectKey=devops \
  -Dsonar.projectName='devops' \
  -Dsonar.host.url=http://devsecops10.eastus.cloudapp.azure.com:9000 \
  -Dsonar.token=sqp_c883f98c5eb05f17dfa46546d39eb6245b052187"
 
 
                }
              }
          

            }
    //--------------------------




    //--------------------------

 stage('Docker Build and Push') {
              steps {
                withCredentials([string(credentialsId: 'secret_dockerhub', variable: 'secret_dockerhub')]) {
                  sh 'sudo docker login -u tygroo972 -p $secret_dockerhub'
                  sh 'sudo docker build -t tygroo972/devops-appbbo:""$GIT_COMMIT"" .'
                  sh 'sudo docker push tygroo972/devops-appbbo:""$GIT_COMMIT""'
                }
              }
            }

    
    //--------------------------
stage('Vulnerability Scan - Kubernetes') {
  steps {
    parallel(
      "OPA Scan": {
        sh 'sudo docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
      },
      "Kubesec Scan": {
        sh "sudo bash kubesec-scan.sh"
      },
      "Trivy Scan": {
        sh "sudo bash trivy-k8s-scan.sh"
      }
    )
  }
}
       //-------------------------- 


    //--------------------------

    //--------------------------


    

  }
}
