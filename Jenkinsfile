pipeline {
    agent {
        label "sys-sdmin-mnf"
    }   
    stages {
        stage('build') {
            steps {
                script {
                    echo 'build'
                    if (BRANCH_NAME == "release") {
                        withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'USERNAME_SYSADMIN', passwordVariable: 'PASSWORD_SYSADMIN')]) {
                            sh '''
                                docker login -u ${USERNAME_SYSADMIN} -p ${PASSWORD_SYSADMIN}
                                docker build -t ahmedzenbaa/bakehouseitisysadmin:v${BUILD_NUMBER} .
                                docker push ahmedzenbaa/bakehouseitisysadmin:v${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../build_num.txt
                                echo ${BRANCH_NAME}
                            '''
                        }
                    } else {
                        echo "user chose ${params.BRANCH_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                echo 'deploy'
                script {
                    if (BRANCH_NAME == "dev" || BRANCH_NAME == "test" || BRANCH_NAME == "prod") {
                        withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG_ITI')]) {
                            sh '''
                                export BUILD_NUMBER=$(cat ../build_num.txt)
                                mv Helm-Deployment/templates/deploy.yaml Helm-Deployment/templates/deploy.yaml.tmp
                                cat Helm-Deployment/templates/deploy.yaml.tmp | envsubst > Helm-Deployment/templates/deploy.yaml
                                rm -rf Helm-Deployment/templates/deploy.yaml.tmp
                                #kubectl apply -f Deployment --kubeconfig ${KUBECONFIG_ITI} -n ${BRANCH_NAME}
                                helm upgrade --install my-bakehouse ./Helm-Deployment --kubeconfig ${KUBECONFIG_ITI} -n ${BRANCH_NAME}
                            '''
                        }
                    } else {
                        echo "user chose ${params.BRANCH_NAME}"
                    }
                    echo "it works"
                }
            }
        }
    }
}
