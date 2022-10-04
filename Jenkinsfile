pipeline {
    agent any

    tools {
        maven "Maven 3.6.3"
    }

    stages {
        stage('Hello-world Maven') {
            steps {
                git 'https://github.com/jenkins-abdel/hello-world.git'
                sh " mvn clean install package"
                archiveArtifacts artifacts: '**/*.war', followSymlinks: false
            }
        }
        stage('Hello-world Sonar') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn  clean package sonar:sonar -Dsonar.host_url=$SONAR_HOST_URL"
                }
            }
        }
        stage('Hello-world Nexus') {
            steps {
                nexusPublisher nexusInstanceId: 'Nexus', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'webapp/target/webapp.war']], mavenCoordinate: [artifactId: 'maven-project', groupId: 'com.example.maven-project', packaging: 'war', version: '1.1']]]
            }
        }
        stage('Hello-world Docker build') {
            steps {
                sh " rm -Rf webapp.war && \
                     wget http://nexus:8081/repository/maven-releases/com/example/maven-project/maven-project/1.1/maven-project-1.1.war -O ${WORKSPACE}/webapp.war && \
                     docker build -t hello-world-afip:latest ."
                }
        }
        stage('Hello-world Docker run') {
            steps {
                script {
                def set_container = sh(script: ''' CONTAINER_NAME="hello-world-test"
                                                   OLD="$(docker ps --all --quiet --filter=name="$CONTAINER_NAME")"
                                                   if [ -n "$OLD" ]; then
                                                    docker rm -f $OLD
                                                   fi
                                                   docker run -d --name hello-world-test -p 8090:8080 hello-world-afip
                                            ''')
                }
            }
        }
    }
}
