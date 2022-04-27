pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '7', artifactNumToKeepStr: '10', daysToKeepStr: '5'))
    }
    agent { label 'nodejs' }

    parameters {
        string(name:'TAG_NAME',defaultValue: 'defaultValue',description:'发布时TAG名称【只有发布到prod的时候需要指定】')
        string(name:'DEPENDENCIES_TAG_NAME',defaultValue: 'defaultValue', description:'发布时依赖的第三方库TAG名称【只有发布到prod的时候需要指定】')
    }

    environment {
        APP_NAME = 'mayun-test'
        HARBOR_CREDENTIAL_ID = 'harbor-id'
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig-id'
        REGISTRY = 'af.hikvision.com.cn'
        HARBOR_NAMESPACE = 'docker-smbg'
        SONAR_CREDENTIAL_ID= 'sonar-token'
    }

    stages {
        stage ('从公共仓库提取配置') {
            steps {
                git(url: 'git@sys-gitlab.hikvision.com.cn:SMBG/tech/iip/sc/smbg-doc.git', credentialsId: 'gitlab-id-ssh', branch: 'master', changelog: true, poll: false)
                script {
                    env.REVIEWER = readFile('./cloudshop/devops/REVIEWER.txt').trim()
                    env.REVIEWER_MAIL = readFile('./cloudshop/devops/REVIEWER_MAIL.txt').trim()
                    env.KUBESPHERE_NAMESPACE = readFile('./cloudshop/devops/KUBESPHERE_PROD_NAMESPACE.txt').trim()
                    env.KUBESPHERE_DEVOPS_URL = readFile('./cloudshop/devops/KUBESPHERE_DEVOPS_URL.txt').trim()
                }
            }
        }


        stage ('检出代码') {
            steps {
                checkout(scm)
                script {
                    if(env.SPRING_PROFILES_ACTIVE == "prod"){
                        sh 'git checkout $TAG_NAME'
                    }
                }
            }
        }

		stage ('编译') {
            steps {
                container ('nodejs') {
                    sh 'echo !!!!!!!!'
                    withCredentials([sshUserPrivateKey(credentialsId:'deploy-id')]) {
					              sh 'echo !!!!!!!!'
                        sh 'cd dist/build/ && tar zcf ${APP_NAME}_${TAG_NAME}.tar.gz ./h5'
                        sh 'scp ${APP_NAME}_${TAG_NAME}.tar.gz root@10.221.28.47:/nfs/deploy_bak'
                    }
                }
            }
        }
    }
}
