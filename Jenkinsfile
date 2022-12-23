/* DO NOT TOUCH: REQUIRED FOR BUILD LOGIC */
dateTime = java.time.LocalDate.now()
environmentParts ="${env.JOB_BASE_NAME}".split("-")
environment = environmentParts[environmentParts.length - 1]

pipeline {
    agent none
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker-cred')
    }

    stages {
        /* Let's make sure we have the repository cloned to our workspace */
        stage('Clone repository') {
          steps {
                script {
                    checkout scm
                }
            }
        }

        /* This builds the actual image; synonymous to
        * docker build on the command line */
        stage('Build image') {
            steps {
                script {
                    tagName = "${Tagname}"

                    if (tagName == "") {
                        shortCommit = sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
                        tagName = "${dateTime}-${shortCommit}"
                    }

                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'

                    try {
                        sh "sudo docker build -t musman3/flask-k8s:${tagName} -t musman3/flask-k8s:latest ."
                    } catch (Exception e) {
                        now = new Date()
                        dateTime = now.format("dd-MM-yyyy HH:mm")
                        commitData = sh(returnStdout: true, script: "git log -n 1 --oneline")
                        commitHistory = sh(returnStdout: true, script: "git log --graph --oneline -n 5 --pretty=format:\"%ad %an %s\" --date=short")
                        additionalMessage = ""

                            // if (Deploy == "Build & deploy") {
                            //     additionalMessage = ": deploy aborted"
                            // }
                    }

                    sh "sudo docker push musman3/flask-k8s:${tagName}"
                    sh "sudo docker push musman3/flask-k8s:latest"
                }
            }
        }

        /* If enabled, launch the deployment job */
        stage('Trigger ManifestUpdate') {
            steps {
                echo "triggering updatemanifestjob"
                build job: 'updatemanifest', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
            }
        }
    }
}

