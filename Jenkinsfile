pipeline {
    agent any
    
    properties([
      parameters([
      choice(choices: ['Maven based Java project', 'Springboot Java Project', '.NET core web app', 'Flask based web app'], name: 'projectType'), 
      string('repoName'), 
      choice(choices: ['Yes', 'No'], name: 'repoExists'), 
      choice(choices: ['SonarQube', 'CAST'], name: 'codeQuality'), 
      choice(choices: ['Junit', 'XUnit', 'NUnit'], name: 'unitTesting'), 
      choice(choices: ['Yes', 'No'], name: 'isContainerized'), 
      choice(choices: ['Yes', 'No'], name: 'functionalTesting'), 
      choice(choices: ['Yes', 'No'], name: 'regressionTesting'), 
      choice(choices: ['DEV', 'QA', 'UAT'], name: 'deployEnvironment'), 
      choice(choices: ['Prometheus'], name: 'monitoring')
      ])
	])
    
    stages {

        stage('Git Create/Clone Repo') {
            steps {
                script {
                    cleanWs()
                    if (params.repoExists == 'Yes') {
                        git '''https://github.com/'''+env.git_username+'''/'''+repoName+'''.git'''
                    } else {
                        sh """
                        curl -H "Authorization: token ${env.git_token}" --data '{"name":"$repoName"}' https://api.github.com/user/repos
                        cp -r /home/srinavya_reddy75/test-repo/* ./
                        """
                        sh 'git init'
                        sh 'git add .'
                        sh '''git remote add origin https://github.com/'''+env.git_username+'''/'''+repoName+'''.git'''
                        sh 'git commit -m "first commit"'
                        sh '''git push https://'''+env.git_username+''':'''+git_token+'''@github.com/'''+env.git_username+'''/'''+repoName+'''.git'''
                    }            
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    if (params.projectType == 'Springboot Java Project') {
                        sh 'mvn clean package'
                    } else if (params.projectType == 'Maven based Java project') {
                        sh 'mvn clean package'
                    } else if (params.projectType == '.NET core web app') {
                        echo 'building .NET core web app'
                    } else if (params.projectType == 'Flask based web app') {
                        echo 'building Flask based web app'
                    }       
                }
            }
        }
        stage('Unit Testing') {
            steps {
                script {
                    if (params.unitTesting   == 'Junit') {
                        sh 'mvn test'
                    } else if (params.unitTesting == 'XUnit') {
                        echo 'XUNIT TESTING'
                    } else if (params.unitTesting == 'NUnit') {
                        echo 'NUNIT TESTING'
                    }  
                }
            }
        }
        stage('Code Quality') {
            steps {
                script {
                    if (params.codeQuality == 'SonarQube') {
                        withSonarQubeEnv('Sonar') { 
                            sh ''' mvn clean verify sonar:sonar -Dsonar.projectKey=testing'''
                        }
                    } else if (params.codeQuality == 'CAST') {
                        echo 'CODEQUALITY CAST'
                    }
                }
            }
        }
        
        stage("Quality Gate") {
            steps {
                   waitForQualityGate abortPipeline: true
                }
        }
        
        stage('Uploading Artifact') {
            steps {
                nexusArtifactUploader artifacts: [
                    [
                        artifactId: 'customer-data', 
                        classifier: '',
                        file: 'target/customer-data-0.0.1.war', 
                        type: 'war']
                    ],
                credentialsId: 'Nexuslogin',
                groupId: 'com.customer',
                nexusUrl: 'ip:port',
                nexusVersion: 'nexus3',
                protocol: 'http', 
                repository: 'testing', 
                version: '0.0.1'
            }
        }
        
        stage("Downloading Artifact") {
            steps {
                   sh '''wget --user='''+env.nexus_username+''' --password='''+env.nexus_password+''' http://ip:port/repository/testing/com/customer/customer-data/0.0.1/customer-data-0.0.1.war'''
                }
        }
        
        stage('Deploy') {
            steps {
                script {
                    if (params.deployEnvironment == "DEV") {
                        sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar customer-data-0.0.1.war > nohup.out &'
                    }
                    else if (params.deployEnvironment == "QA") {
                        sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar customer-data-0.0.1.war > nohup.out &'
                    }
                    else if (params.deployEnvironment == "UAT") {
                        sh 'JENKINS_NODE_COOKIE=dontKillMe nohup java -jar customer-data-0.0.1.war > nohup.out &'
                    }
                }
            }
        }
        stage('Functional Testing') {
            steps {
                script {
                    if (params.functionalTesting   == 'Yes') {
                        sh '''curl -I -u '''+env.jenkins_username+''':'''+env.jenkins_password+''' http://ip:port/job/functional_testing/build?token='''+env.jenkins_token+''''''
                    } else {
                        echo 'No TESTING'
                    }
                }
            }
        }
        stage('Regression Testing') {
            steps {
                script {
                    if (params.regressionTesting   == 'Yes') {
                        sh '''curl -I -u '''+env.jenkins_username+''':'''+env.jenkins_password+''' http://ip:port/job/regression_testing/build?token='''+env.jenkins_token+''''''
                    } else {
                        echo 'No TESTING'
                    }
                }
            }
        }
        stage("Application Monitoring (Prometheus)") {
            steps {
                sh 'docker run -p 9090:9090 --name prometheus -d -v prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus'
                sh 'docker start prometheus'
            }
        }
    }
}
