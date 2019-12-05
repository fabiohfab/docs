node {
    stage('[Workspace] Clean') {
        deleteDir()
    }

    stage('[Github] Checkout Latest Source Code') {
        git branch: "${env.BRANCH_NAME}", credentialsId: 'cfchenr', url: 'https://github.com/fabiohfab/docs.git'
    }
     
    stage('[Workspace] Get Variables') {
        def config = readJSON file: "config.json"
        env.BRANCH_TO_DEPLOY = config["branchToDeploy"]
        env.VERSION = config["version"]
    }
    
    stage('[Docker] Build and Run') {
        def imageName = 'refletir-image'
        def containerName = 'refletir-container-snapshot'
        def repo = 'refletir-snapshot/'
        def port = 3333
        if ("${env.BRANCH_NAME}" == "${env.BRANCH_TO_DEPLOY}") {
            repo = "refletir-release/"
            containerName = 'refletir-container-released'
            port = 3000
        }
        sh "docker build . -t ${repo}${imageName}"
        sh "docker stop ${containerName} || true && docker rm ${containerName} || true"
        sh "docker image prune -f"
        sh "docker run -dit --name ${containerName} -p ${port}:3000 ${repo}${imageName}"
    }

    if ("${env.BRANCH_NAME}" == "${env.BRANCH_TO_DEPLOY}") {
        stage('[Github] Add Git Tag') {
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'cfchenr', usernameVariable: 'username', passwordVariable: 'password']]) {
                sh('git config user.email "c.henriques@ua.pt"')
                sh('git config user.name "cfchenr"')
                sh("git tag -a -f ${env.VERSION} -m " + '"tag auto generated"')
                script {
                    env.encodedPass=URLEncoder.encode(password, "UTF-8")
                }
                sh("git push -f https://${username}:${encodedPass}@github.com/fabiohfab/docs.git ${env.VERSION}")
            }
        }
    }
}