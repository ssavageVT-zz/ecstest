#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    docker.image('openjdk:8').inside('-u root -e MAVEN_OPTS="-Duser.home=./"') {
        stage('check java') {
            sh "java -version"
        }

        stage('clean') {
            sh "chmod +x mvnw"
            sh "./mvnw clean"
        }

        //stage('backend tests') {
        //    try {
        //        sh "./mvnw test"
        //    } catch(err) {
        //        throw err
        //    } finally {
        //        junit '**/target/surefire-reports/TEST-*.xml'
        //    }
        //}

        stage('packaging') {
            sh "./mvnw verify -Pprod -DskipTests"
            archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
        }
  
        def dockerImage
        stage('build docker') {
            sh "whoami"
            sh "ls -l src/main/docker"
            sh "ls -l target/"
            sh "cp -R src/main/docker target/"
            sh "cp target/*.war target/docker/"
            dockerImage = docker.build('ssavagevt22/ecstest', 'target/docker')
        }

        stage('publish docker') {

            withCredentials([[$class: 'UsernamePasswordMultiBinding',
                credentialsId: 'docker-hub-login',
                usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {

                //sh 'echo uname=$USERNAME pwd=$PASSWORD'
                sh 'docker login -u $USERNAME -p $PASSWORD https://registry.hub.docker.com'
                sh "docker push registry.hub.docker.com/ssavagevt22/ecstest"
			
                //docker.withRegistry("${docker_registry_url}", params.docker_user_creds) {
                //dockerImage.push("latest")
            }

            //docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-login') {
            //docker.withRegistry('https://docker.io', 'docker-hub-login') {
            //    dockerImage.push 'latest'
            //}
        }

    }
    stage('deploy to AWS ECS') {
    withCredentials([[
        $class: 'AmazonWebServicesCredentialsBinding',
        credentialsId: 'aws',
        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {


        sh "aws s3 mb s3://ssavagevt22"
        sh "aws configure set default.region us-east-1"
        sh "aws s3 cp ecs_cf_template.yml s3://ssavagevt22/cloudformationtemplates/ecstest.template"
        sh "aws cloudformation create-stack --stack-name ecstest --template-url https://s3.amazonaws.com/ssavagevt22/cloudformationtemplates/ecstest.template"
    }
}
}
