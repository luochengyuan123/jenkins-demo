def registryUrl = "registry-intl.cn-hongkong.aliyuncs.com"
def registryCredential = "dockerregistry"
def projectName = env.JOB_NAME.substring(2, env.JOB_NAME.length())
def jobName = env.JOB_NAME.trim()
def gitBranch = params.BRANCH.trim()
node {
    stage('Prepare') {
        echo "1.Prepare Stage"
        checkout scm
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${gitBranch}-${build_tag}"
            }
        }
    }
    stage('Test') {
      echo "2.Test Stage"
      echo "${projectName}"
      echo "${gitBranch}"
    }
    stage('Build & Push Image') {
        echo "4.Push Docker Image Stage"
        dir("/var/lib/jenkins/workspace/${jobName}") {
           docker.withRegistry("https://${registryUrl}", "${registryCredential}") {
                def image = docker.build("${registryUrl}/tnaot/${projectName}:${build_tag}", ".")
                image.push()
            }
        }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${gitBranch}/' k8s.yaml"
        sh "kubectl get pod"
        sh "kubectl apply -f k8s.yaml --record --validate=false"
    }
}

