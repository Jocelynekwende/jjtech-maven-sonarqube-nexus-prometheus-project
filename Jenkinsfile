pipeline {
    
//      agent {
//       node {
//     label 'jenkins-agent1'
//   }
// }
    
    agent {
        docker { image 'maven:3.8.6-openjdk-11' }
    }
    
    tools {
      maven 'maven'
    }

    stages {
        stage('git checkout') {
            steps {
                // Get some code from a GitHub repository
                 git branch: 'main', url: 'https://github.com/anselmenumbisia/jjtech-maven-sonarqube-nexus-prometheus-project.git'
            }

            }
        stage('test') {
            steps {
                dir('JavaWebApp/') {
                    echo 'performing mvn test'
                    sh 'mvn clean test'
                    
                }
                
            }

            }
          stage('approval') {
            steps {
                input "Please review the test and click 'Proceed' to apply it"
            }
        }

        stage('build') {
            steps {
                dir('JavaWebApp/') {
                    echo 'performing mvn build'
                    sh 'mvn clean package'
                    
                }
                    
                
            }

            post {
                success {
                    echo 'archiving....'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }
        
    //     stage('SonarQube scanning') {
    //         steps {
    //             dir('JavaWebApp/') {
    //             withSonarQubeEnv('SonarQube') {
    //                 withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
    //                     sh """
    //                 mvn sonar:sonar \
    //                   -Dsonar.projectKey=tower-project1 \
    //                   -Dsonar.host.url=http://172.31.80.37:9000 \
    //                   -Dsonar.login=942c2c64b96dd2370ac02224ffc2e8e9365aad77
    //                 """
    //                 }
    //             }
    //         }
    //     }
    // }

    //     stage('Quality Gate') {
    //         steps {
    //             waitForQualityGate abortPipeline: true
    //         }
    //     }

            stage('SonarQube scanning') {
            steps {
                dir('JavaWebApp/') {
                // withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                    mvn sonar:sonar \
                     -Dsonar.projectKey=tower-project \
                     -Dsonar.host.url=http://172.31.80.37:9000 \
                     -Dsonar.login=$SONAR_TOKEN
                     """
                    }
                // }
            }
        }
    }

          stage('Upload artifact to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                sh "sed -i \"s/.*<username><\\/username>/<username>$USER_NAME<\\/username>/g\" ${WORKSPACE}/settings.xml"
                sh "sed -i \"s/.*<password><\\/password>/<password>$PASSWORD<\\/password>/g\" ${WORKSPACE}/settings.xml"
                sh "sed -i 's|http://172.31.88.170:8081/repository/maven-snapshots/|http://172.31.88.170:8081/repository/maven-snapshots/|g' ${WORKSPACE}/settings.xml"
                sh "sed -i 's|http://172.31.88.170:8081/repository/maven-release/|http://172.31.88.170:8081/repository/maven-release/|g' ${WORKSPACE}/settings.xml"
                sh 'sudo cp ${WORKSPACE}/settings.xml /var/lib/jenkins/.m2'
                dir('JavaWebApp/') {
                sh "sed -i 's|http://172.31.88.170:8081/repository/maven-snapshots/|http://172.31.88.170:8081/repository/maven-snapshots/|g' pom.xml"
                sh "sed -i 's|http://172.31.88.170:8081/repository/maven-release/|http://172.31.88.170:8081/repository/maven-release/|g' pom.xml"                
                sh 'mvn clean deploy -DskipTests'
                }
                }
               
            }
        }

    }
}