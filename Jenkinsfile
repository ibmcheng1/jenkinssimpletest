def gitCommit
def volumes = [ hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'), hostPathVolume(mountPath: '/home/gradle/.gradle', hostPath: '/tmp/jenkins/.gradle') ]
volumes += secretVolume(secretName: 'jenkins-docker-sec', mountPath: '/jenkins_docker_sec')
podTemplate(label: 'icp-liberty-build-jenkinstest', slaveConnectTimeout: 600,
    containers: [
        containerTemplate(name: 'jnlp', image: 'mycluster.icp:8500/default/jenkins/jnlp-slave:3.23-1', args: '${computer.jnlpmac} ${computer.name}'),
        containerTemplate(name: 'maven', image: 'mycluster.icp:8500/default/maven:3.5.4-jdk-8', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'gradle', image: 'mycluster.icp:8500/default/gradle:4.10.1-jdk8', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'docker', image: 'mycluster.icp:8500/default/docker:17.12', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'mycluster.icp:8500/default/ibmcom/k8s-kubectl:v1.8.3', ttyEnabled: true, command: 'cat'),       
    ],
    volumes: volumes
)
{
    node ('icp-liberty-build-jenkinstest') {
         //def gitCommit
        stage ('Extract') {
          checkout scm
          gitCommit = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          echo "Stage-git: checked out git commit ${gitCommit}"
        }
        stage ('maven build') {
          container('maven') {
            sh '''
            echo "Stage-maven: build ..."
            whoami
            echo "WORKSPACE: ${WORKSPACE}"
            chmod -R 777 ${WORKSPACE}
            # mvn clean test install
            '''
          }
        }
        stage ('gradle build') {
          container('gradle') {
            sh '''
            echo "Stage-gradle: build ..."
            whoami
            '''
          }
        }

         stage ('docker') {
          container('docker') {
            echo "Stage-docker: docker - build tag push ..."
            whoami
            def imageTag = "mycluster.icp:8500/jenkinstest/jenkinssimpletest:${gitCommit}"
            echo "imageTag ${imageTag}"
            sh """
            ln -s /jenkins_docker_sec/.dockercfg /home/jenkins/.dockercfg
            mkdir /home/jenkins/.docker
            ln -s /jenkins_docker_sec/.dockerconfigjson /home/jenkins/.docker/config.json
            # docker build -t jenkinssimpletest .
            # docker tag jenkinssimpletest $imageTag
            # docker push $imageTag
            """
          }
        }
        
    }
}
