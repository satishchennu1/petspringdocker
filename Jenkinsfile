node("maven") {
          timeout(time: 20,unit: 'MINUTES'){
            stage('checkout code from git'){ // for display purposes
              // Get some code from a GitHub repository
              git url: "https://github.com/rajvaranasi/petspringdocker.git", branch: "master" 
            }//checkout code stage
            stage("Build WAR Using Maven") {
                sh "mvn clean package -Popenshift"
              }//building the war file
            stage('Build Image'){
              openshift.withCluster(){
                openshift.withProject(){
                  sh "oc start-build dockerspringpetclinic --from-dir . --follow"  
                }
              }
            }
            stage('Code Coverage using Jacoco') { //
              archive 'target/*.jar'
              step([$class: 'JacocoPublisher', execPattern: '**/target/jacoco.exec'])
             }
            stage('Code Quality Using SonarQube') {
              withSonarQubeEnv('SonarQube') { 
                sh "mvn sonar:sonar -Dsonar.host.url=http://sonarqube-sonarqubeoc.apps.ocppilot.ocpcontainer.com/ -DskipTests=true"
              }
            }
            stage("Deploy to DeveloperSandBox") {
              openshift.withCluster() {
                openshift.withProject() {
                  def dc = openshift.selector('dc', "dockerspringpetclinic")
                  dc.rollout().status()
                }
              }
            }
            stage('Tag Images') {
              openshift.withCluster() {
                openshift.withProject(){
                  openshift.tag("dockerspringpetclinic:latest", "dockerspringpetclinic:dev")
                }
              }
            }
            stage('Promote to DEV'){
              openshift.withCluster(){
                openshift.withProject(){
                  openshift.tag("dockerspringpetclinic:latest", "dockerspringpetclinic:dev")
                  echo "oc delete all -l app=dockerspringpetclinic-dev"
                  sleep 30
                  openshift.selector("dc","dockerspringpetclinic-dev").rollout().status()
                }
              }
            }
            stage("approval Message") {
              input message: "Need approval to move to Controlled Environment: Approve?", id: "approval"
            }          
            stage('Promote to UAT'){
              openshift.withCluster(){
                openshift.withProject(){
                  openshift.tag("dockerspringpetclinic:latest", "dockerspringpetclinic:uat")
                  openshift.selector("dc","dockerspringpetclinic-uat").rollout().status()
                }
              }
            }


          }
        }
