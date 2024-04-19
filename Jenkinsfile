def label = 'shopagent'
def mvn_version = 'M2'
podTemplate(label: label, yaml: '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: build
  annotations:
    sidecar.istio.io/inject: "false"
spec:
  containers:
  - name: build
    image: kumararj/eos-jenkins-agent-base:latest
    command:
    - cat
    tty: true
  - name: docker
    image: docker:latest
    securityContext:
      privileged: true
'''
) {
    node(label) {
        stage('CleanWorkspace') {
            cleanWs()
        }
        stage('Checkout SCM') {
            git credentialsId: 'git', url: 'https://github.com/Kumar-arj/microservice-demo-queue-master.git', branch: 'master'
            container('build') {
                stage('Build a Maven project') {
                    sh 'mvn -DskipTests clean package'
                }
            }
        }
        // stage('Sonar Scan') {
        //   container('build') {
        // stage('Sonar Scan') {
        //   withSonarQubeEnv('sonar') {
        //     sh 'mvn -DskipTests verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=sock-shop_service'
        //   }
        // }
        //   }
        // }
        stage('Publish to Nexus') {stage('Publish to Nexus Repository Manager') {
                withEnv([ 'NEXUS_VERSION=nexus3','NEXUS_PROTOCOL=http','NEXUS_URL=nexus.k4m.in','NEXUS_REPOSITORY=sock-shop-release-local','NEXUS_REPO_ID=sock-shop-release-local','NEXUS_CREDENTIAL_ID=nexuslogin',"ARTVERSION=${env.BUILD_ID}"]){
                    script {
                        pom = readMavenPom file: 'pom.xml'
                        filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                        echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                        artifactPath = filesByGlob[0].path
                        artifactExists = fileExists artifactPath
                        if (artifactExists) {
                            echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION"
                            nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTVERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: 'pom.xml',
                                type: 'pom']
                            ]
                        )
                        }
            else {
                            error "*** File: ${artifactPath}, could not be found"
            }
                    }
                }
        }
            stage('Docker Build') {
                container('docker') {
                    stage('Build Image') {
                        docker.withRegistry( 'https://registry.hub.docker.com', 'dockerhub' ) {
                            def customImage = docker.build('kumararj/sock-shop-micro-services-queue-master:latest')
                            customImage.push()
                        }
                    }
                }
            }

            stage('Helm Chart') {
                container('build') {
                    dir('charts') {
                        withCredentials([usernamePassword(credentialsId: 'nexuslogin', usernameVariable: 'username', passwordVariable: 'password')]) {

                            //   sh '/usr/local/bin/helm push micro-services-cart-1.0.tgz http://nexus.k4m.in/repository/sock-shop-helm'
                            // sh ''' 
                            //   BUILD_NUMBER=${BUILD_ID}
                            //   sed -i 's/version: "1.0"/version: "1.0.0-'${BUILD_NUMBER}'"/' micro-services-cart/Chart.yaml
                            // '''
                            sh '/usr/local/bin/helm package micro-services-queue'
                            // sh 'curl -v -u $username:$password --upload-file micro-services-cart-1.0.0-$BUILD_NUMBER.tgz http://nexus.k4m.in/repository/sock-shop-helm-local/'
                            sh 'curl -v -u $username:$password --upload-file micro-services-queue-1.0.tgz http://nexus.k4m.in/repository/sock-shop-helm-local/'

                        }
                    }
                }
            }
    }
}
}
