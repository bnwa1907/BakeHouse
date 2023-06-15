pipeline {
    agent {
        label "sys-admin-mnf"
    }
    parameters {
        choice(name: 'ENV_ITI', choices: ['dev', 'test', 'prod', "release"])
    }
    stages {
        stage('build') {
            steps {
                script {
                    echo 'build'
                    if (params.ENV_ITI == "release") {
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
                        echo "user chose ${params.ENV_ITI}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                echo 'deploy'
                script {
                    if (params.ENV_ITI == "dev" || params.ENV_ITI == "test" || params.ENV_ITI == "prod") {
                        withCredentials([file(credentialsId: 'doc-cluster', variable: 'KUBECONFIG_ITI')]) {
                            sh '''
                                export BUILD_NUMBER=$(cat ../build_num.txt)
                                mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                                cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                                rm -rf Deployment/deploy.yaml.tmp
                                kubectl apply -f Deployment --kubeconfig ${KUBECONFIG_ITI} -n ${ENV_ITI}
                            '''
                        }
                    } else {
                        echo "user chose ${params.ENV_ITI}"
                    }
                }
            }
        }
    }
}
