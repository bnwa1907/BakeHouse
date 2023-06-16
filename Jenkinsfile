pipeline {
    agent {
        label "sys-admin-mnf"
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
                                docker build -t  omarelbanawany/bakehouseitisysadmin:v${BUILD_NUMBER} .
                                docker push omarelbanawany/bakehouseitisysadmin:v${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../build_num.txt
                                echo ${ENV_ITI}
                            '''
                        }
                    } else {
                        echo "user chose ${BRANCH_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                echo 'deploy'
                script {
                    if (BRANCH_NAME == "dev" || BRANCH_NAME == "test" || BRANCH_NAME == "prod") {
                        withCredentials([file(credentialsId: 'cluster-cred', variable: 'KUBECONFIG_ITI')]) {
                            sh '''
                                export BUILD_NUMBER=$(cat ../build_num.txt)
                                mv helm-dep/templates/deploy.yaml Deployment/deploy.yaml.tmp
                                cat helm-dep/templates/deploy.yaml.tmp | envsubst > helm-dep/templates/deploy.yaml
                                rm -rf helm-dep/templates/deploy.yaml.tmp
                                helm upgrade --install Bakehouse ./helm-dep --values helm-dep/${BRANCH_NAME}.yaml --kubeconfig ${KUBECONFIG_ITI} -n ${BRANCH_NAME}
                            '''
                        }
                    } else {
                        echo "user chose ${BRANCH_NAME}"
                    }
                }
            }
        }
    }
}
