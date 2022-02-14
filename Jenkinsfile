pipeline {
    agent any
    stages {
        stage('build container') {
            steps {
                sh 'docker build --tag=jenkins-docker .'
            }
        }

        stage('publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'nexusUser', passwordVariable: 'nexusPass')]) {
                    sh 'docker login nexus.local:8080 --username $nexusUser --password $nexusPass'
                }
                sh 'docker image tag jenkins-docker nexus.local:8080/rhink/jenkins-docker:latest'
                sh 'docker image push nexus.local:8080/rhink/jenkins-docker:latest'
            }
        }

        stage('deploy') {
            steps {
                // stop and remove old jenkins container. "||" prevents failure of stop command from failing build.
                sh 'docker stop jenkins || true && docker rm jenkins || true'
                sh 'docker run \
                        --restart always \
                        --name jenkins \
                        --detach \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        -p 192.168.1.203:80:8080 \
                        -p 192.168.1.203:50000:50000 \
                        -v $(which docker):$(which docker) \
                        -v jenkins_home:/var/jenkins_home \
                        jenkins-docker'
            }
        }
    }
}
