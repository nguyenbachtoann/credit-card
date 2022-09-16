/* groovylint-disable */
node {
    IMAGE_BASE_NAME = 'nguyenbachtoan/creditcard'
    COMMON_BRANCH = 'master'
    REPO = 'https://github.com/nguyenbachtoann/credit-card.git'
    CONFIG_REPO = 'https://github.com/nguyenbachtoann/cicd_configuration.git'
    GIT_CREDENTIAL = 'github-nguyenbachtoann-credential' // create on Jenkins Credential
    CONFIG_DIR = ".config/demo/credit-card/dev";

    stage('Pull Code') {
        // checkout source code
        git url: REPO, branch: COMMON_BRANCH, credentialsId: GIT_CREDENTIAL
        // create job name
        GIT_COMMIT = sh(script: 'git rev-parse HEAD | cut -c1-8', returnStdout: true).trim()
        currentBuild.displayName = "#${env.BUILD_NUMBER}_${GIT_COMMIT} (${COMMON_BRANCH})"
        // checkout configuration
        dir('.config') {
            git url: CONFIG_REPO, branch: COMMON_BRANCH, credentialsId: GIT_CREDENTIAL
        }
    }
    stage('Unit Test') {
        /*  if there are test cases in the source, this stage will execute test command or
            frame work, and the result will be use at the Code Scan stage
            ex: if this is a java project, we can use
            'gradle clean test'
                if this is a node project, we can use
            'npm run test'
        */
        echo 'Tested'
    }
    stage('Build Image') {
        /*  set name for image: nguyenbachtoan/creditcard:abcxyz12
            simple format: account/image:git_commit */
        IMAGE = "${IMAGE_BASE_NAME}:${GIT_COMMIT}"
        withEnv(["IMAGE=${IMAGE}", "CONFIG_DIR=${CONFIG_DIR}"]) {
            sh '''
                if [[ "$(docker images -q ${IMAGE} 2> /dev/null)" == "" ]]; then
                    docker build -t ${IMAGE} -f ./Dockerfile . --build-arg NGINX_CONFIG_FILE=/${CONFIG_DIR}/nginx.conf
                fi
            '''
        }
    }

    stage('Push To Artifactory (Docker Hub)') {
        /*
            [Artifactory Alternative]
            get the credential from Jenkins Credential, extract its USERNAME and PASSWORD
        */
        withCredentials([usernamePassword(credentialsId: 'docker-hub-credential', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            // inject variable from outside
            withEnv(["IMAGE=${IMAGE}"]) {
                sh '''
                    docker login -u ${USERNAME} -p ${PASSWORD}
                    docker push ${IMAGE}
                    # delete image for saving disk
                    docker rmi ${IMAGE}
                '''
            }
        }
    }

    stage('Deploy (Docker Compose)') {
        /*
            [Kubernetes Alternative]
            run using docker compose, will serve IMAGE we just pushed to hub, running in background (-d)
            if not running in bg, this stage will run 4ever because of docker-compose foreground run display
            IMAGE_NAME: dynamic image param
        */
        sh "IMAGE_NAME=${IMAGE} docker-compose -f ${CONFIG_DIR}/docker-compose.yaml up -d"
    }

    stage('Code Scan') {
        // After stage Unit Test, the report will be send to SonarQube to analyze
        echo 'scanning'
    }
}
