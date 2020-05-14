pipeline {
    agent {
        label 'master'
    }
    
    parameters {
        string(name: 'version', description: "Version of nexus artifact")
    }

    options {
        skipDefaultCheckout()
    }
    
    stages {
        stage("Test") {
            steps {
                script {
                    sshagent(credentials : ['ubuntu-slave']) {
                        withCredentials([usernameColonPassword(credentialsId: 'nexus', variable: 'USERPASS')]) {
 
                            slaves = [ '10.0.3.145' ]
                            for (slave in slaves) {
                                sh """\
                                ssh -o StrictHostKeyChecking=no ubuntu@${slave} <<EOF
                                    set -xe
                                    cd /opt/backend
                                    curl -sSL -X GET -G "http://nexus.shavlyuk-ci.test.coherentprojects.net/service/rest/v1/search/assets" \
                                        -d repository=maven-releases \
                                        -d maven.groupId=issoft.training \
                                        -d maven.artifactId=backend \
                                        -d maven.baseVersion=${params.version} \
                                        -d maven.extension=jar \
                                        -u ${USERPASS} \
                                        | grep -Po '"downloadUrl" : "\\K.+(?=",)' \
                                        | sudo xargs curl -fsSL -o backend.jar -u ${USERPASS}
                                    sudo systemctl restart backend
                                EOF
                                """.stripIndent()
                            }
                        }
                    }
                }
            }
        }
    }
}