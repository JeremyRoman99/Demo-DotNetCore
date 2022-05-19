pipeline {
  agent any
  environment {
      EMAIL_RECIPIENTS = 'alessandra.rosado2308@gmail.com'      
  }  
  stages {

      stage('Build') {
      steps {        
        sh "msbuild /t:build /restore:True"
      }
    }
    stage('Test'){      
      steps {
        sh 'mono packages/nunit.consolerunner/3.10.0/tools/nunit3-console.exe *.Test/bin/*/net4*/*.Test.dll'
        nunit(testResultsPattern: 'TestResult.xml')
      }
    }
    

     stage('Sonar') {
      steps {        
        sh 'dotnet tool install --global dotnet-sonarscanner'
        sh 'dotnet sonarscanner begin /k:"test-DogNet1" /d:sonar.host.url="http://149.56.14.3:9000/sonar"  /d:sonar.login="bae6453470b563aaaa3f37ee1279bbfe5137e362"'
        sh 'dotnet build'
        sh 'dotnet sonarscanner end /d:sonar.login="bae6453470b563aaaa3f37ee1279bbfe5137e362"'
      }
    }
    
    

    
  }
  post {
        success {
            sendEmail("Successful");
			notifySlack("SUCCESS");
        }
        unstable {
            sendEmail("Unstable");
			notifySlack("UNSTABLE");
        }
        failure {
            sendEmail("Failed");
			notifySlack("FAILED");
        }
    }

}
@NonCPS
def getChangeString() {
    MAX_MSG_LEN = 100
    def changeString = ""

    echo "Gathering SCM changes"
    def changeLogSets = currentBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            truncated_msg = entry.msg.take(MAX_MSG_LEN)
            changeString += " - ${truncated_msg} [${entry.author}]\n"
        }
    }

    if (!changeString) {
        changeString = " - No new changes"
    }
    return changeString
}
def notifySlack(String buildStatus = "STARTED") {
    buildStatus = buildStatus ?: "SUCCESS"

    def color

    if (buildStatus == "STARTED") {
        colorNotify = "#D4DADF"
    } else if (buildStatus == "SUCCESS") {
        colorNotify = "#BDFFC3"
    } else if (buildStatus == "UNSTABLE") {
        colorNotify = "#FFFE89"
    } else {
        colorNotify = "#FF9FA1"
    }

    def msg = "${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}:\n${env.BUILD_URL}"

    slackSend(color: colorNotify, message: msg)
}
