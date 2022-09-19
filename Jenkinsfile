pipeline {
    agent any

    stages {
        stage ('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("dimitatu/api-produto:${env.BUILD_ID}", '-f ./src/Dockerfile ./src') 
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }                
            }
        }

        stage ('Push Image') {
            steps {
                script {
                    echo "push docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t dimitatu/app:${IMAGE_NAME} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push dimitatu/app:${IMAGE_NAME}"                    
                    // docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                    //     dockerapp.push('latest')
                    //     dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        //         stage('build image') {
        //     steps {
        //         script {
        //             echo "building the docker image..."
        //             withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
        //                 sh "docker build -t dimitatu/app:${IMAGE_NAME} ."
        //                 sh "echo $PASS | docker login -u $USER --password-stdin"
        //                 sh "docker push dimitatu/app:${IMAGE_NAME}"
        //             }
        //         }
        //     }
        // }

        stage ('Deploy Kubernetes') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    sh 'kubectl apply -f ./k8s/deployment.yaml'
                }
            }
        }
    }
}