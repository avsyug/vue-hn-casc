@Library('cb-days@master') _
def testPodYaml = libraryResource 'podtemplates/vuejs/vuejs-test-pod.yml'
pipeline {
  agent none
  options { 
    buildDiscarder(logRotator(numToKeepStr: '10'))
    //skipDefaultCheckout true
    preserveStashes(buildCount: 10)
  }
  environment {
    //Possible canary deploy phases:
    //1. Flag completely off
    //2. Flag turned on for internal testing
    //3. Flag turned on for internal testing + % subset of end users
    canaryPhase = "fullyFalse"
  }
  stages('Automatic Canary Deployment')
  {
    stage('Deployment Phase Check'){
       agent {
        kubernetes {
          label 'nodejs'
          yaml testPodYaml
        }
      }
      steps {
        checkout scm
        script{
          //Start of check if canary deploy shared library
          //Check if latest commit is an experiment + is labeled for canary deployment
          changedFile = sh (script:"git diff-tree --no-commit-id --name-only -r ${env.GIT_COMMIT}", returnStdout: true)
          echo(changedFile)
          experimentFile = changedFile.contains("experiments/")
          if(experimentFile){
            experimentText = sh (script:"cat ${changedFile}", returnStdout: true)
           sh "cat ${changedFile}"
            experimentYaml = readYaml (text: "$experimentText")
            if(experimentYaml.labels)
            {
              canaryBool = experimentYaml.labels.contains("Canary-deploy")
            //End of check for canary deploy shared library
              if(canaryBool){
                //Start of check for deployment phase shared library
                //If it is labeled for a canary deployment, figure out which deployment phase we're in
                if(experimentYaml.conditions){
                  groupBool = false
                  if(experimentYaml.conditions.group.name){
                    groupBool = true
                    targetGroupBool = experimentYaml.conditions.group.name.contains("Internal-Testing")
                    if(targetGroupBool){
                      canaryPhase = "internalTesting"
                    }
                  }
                  if(!(groupBool && experimentYaml.conditions.size() == 1)) {
                    if(experimentYaml.conditions[-1].value.percentage){
                      echo experimentYaml.conditions[-1].value.percentage.toString()
                      canaryPhase = "percentageDeploy"
                    }
                  }
                }
                //If there are no additional conditions in the ruleset, check if flag is fully on or fully off
                else{
                  canaryPhase = "fullyFalse"
                }
                echo "$canaryPhase"
                //End of deployment phase shared library
              }
              else{
               echo "No canary!" 
              }
            }
          }
          else{
           echo "Updated a not experiment!" 
          }
        }
      }
    }
    stage('Automated Testing Kickoff'){
      steps {
        echo("Hello 2")
      }
    }
    stage('Canary scale up'){
      steps {
        echo("Hello 3")
      }
    }
    stage('Canary scale down'){
      steps {
        echo("Hello 4")
      }
    }
  }
}
