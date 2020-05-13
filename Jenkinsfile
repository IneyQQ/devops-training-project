next_version = params.version
if (params.version != null) {
    try {
        next_version = (params.version.toInteger() + 1).toString()
    } catch (Exception e) {}
}

pipeline {
    parameters {
        string(name: 'version', defaultValue: next_version, description: 'Artifact version. Integer value')
    }
    
    environment {
        gradle_skip_analysis = "-x findbugsMain -x findbugsTest -x pmdMain -x pmdTest -x checkstyleMain -x checkstyleTest"
        
    }
    
    agent {
        label "master"
    }
    stages {
        stage("Clone repo") {
            steps {
                deleteDir()
                checkout scm: [
                    $class: 'GitSCM', branches: [[name: '*/master']],
                    userRemoteConfigs: [[url: 'https://github.com/IneyQQ/devops-training-project']]
                ]
            }
        }
        stage("Build") {
            steps {
                dir("backend") {
                    sh "./gradlew build -Pversion=${params.version} -x test ${gradle_skip_analysis}"
                }
            }
        }
        stage("Test") {
            steps {
                dir("backend") {
                    sh "./gradlew test -Pversion=${params.version} ${gradle_skip_analysis} || true"
                }
            }
        }
        stage("Analyze") {
            steps {
                dir("backend") {
                    withSonarQubeEnv('main') {
                        sh './gradlew sonarqube -x test ${gradle_skip_analysis}'
                    }
                }
            }
        }
        stage("Push") {
            steps {
                dir("backend") {
                    nexusPublisher nexusInstanceId: 'main',
                        nexusRepositoryId: 'maven-releases',
                        packages: [[
                            $class: 'MavenPackage',
                            mavenAssetList: [[classifier: '', extension: '', filePath: "./build/libs/backend-${params.version}.jar"]],
                            mavenCoordinate: [groupId: 'issoft.training', artifactId: 'backend', version: params.version, packaging: 'jar']
                        ]]
                }
            }
        }
        
    }
}
