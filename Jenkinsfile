pipeline {
    agent any
    parameters {
        string(name: 'GIT_URL', defaultValue: 'https://github.com/PARKHARAM/vmcreate-terraform.git', description: 'GIT_URL')
        booleanParam(name: 'VERBOSE', defaultValue: false, description: '')
    }
    
    environment {
        GIT_BUSINESS_CD = 'main'
        GITLAB_CREDENTIAL_ID = 'gitlabuser'
        VERBOSE_FLAG = '-q'
    }

    stages{
        stage('Preparation') { // for display purposes
            steps{
                script{
                    env.ymd = sh (returnStdout: true, script: ''' echo `date '+%Y%m%d-%H%M%S'` ''')
                }
                echo("params : ${env.ymd} " + params.tag)
            }
        }

        stage('Checkout') {
            steps{
                git(branch: "${env.GIT_BUSINESS_CD}", 
                credentialsId: "${env.GITLAB_CREDENTIAL_ID}", url: params.GIT_URL, changelog: false, poll: false)
            }
        }
        
        stage('SonarQube analysis') {
            steps{
                withSonarQubeEnv('SonarQube-Server'){
                    sh "mvn clean package"
                    sh "mvn sonar:sonar -Dsonar.projectKey=demo -Dsonar.host.url=http://34.64.88.47:9000/ -Dsonar.login=9f40ea6d870c2c7b24f4ecc6f40350b8030a170a"
                }
            }
        }
        
        stage('SonarQube Quality Gate'){
            steps{
                timeout(time: 1, unit: 'MINUTES') {
                    script{
                        echo "Start~~~~"
                        def qg = waitForQualityGate()
                        echo "Status: ${qg.status}"
                        if(qg.status != 'OK') {
                            echo "NOT OK Status: ${qg.status}"
                            updateGitlabCommitStatus(name: "SonarQube Quality Gate", state: "failed")
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        } else{
                            echo "OK Status: ${qg.status}"
                            updateGitlabCommitStatus(name: "SonarQube Quality Gate", state: "success")
                        }
                        echo "End~~~~"
                    }
                }
            }
        }
    }
}