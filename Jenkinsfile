pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                script{
                if (env.BRANCH_NAME == "master") {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {

                    sh """
                     docker build -f Dockerfile . -t engboda/bakehouse:${BUILD_NUMBER}
                     docker login -u ${USERNAME} -p ${PASSWORD}
                     docker push engboda/bakehouse:${BUILD_NUMBER}
                     echo ${BUILD_NUMBER} > ../vars.txt 

                     """ 
                    }
                }
              } 
            }
        }
        

        stage('Deploy') {
            steps {
                script {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'FILE')]) {
                if (env.BRANCH_NAME == "dev" || env.BRANCH_NAME == "test" || env.BRANCH_NAME == "prod") {
                    sh """
                     export NUMBER=\$(cat ../vars.txt)
                     cp bakehouse/values.yaml bakehouse/values.yaml.tmp
                     cat bakehouse/values.yaml.tmp | envsubst > bakehouse/values.yaml
                     rm -f bakehouse/values.yaml.tmp
                     helm install bakehouse ./bakehouse/ --kubeconfig=$FILE
                     if [ $? -eq 1 ]
                     then
                        helm upgrade bakehouse ./bakehouse/ --kubeconfig=$FILE
                    fi  
                     """
                    }
                }
              }
            }
        }
    }
}
