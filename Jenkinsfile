import groovy.json.JsonSlurper

def buildVersionString(branchName, buildNumber) {
    if (branchName == 'master') {
        return "0.0.${buildNumber}"
    } else {
        return "0.0.${buildNumber}-${branchName.replaceAll('feature/', '').replaceAll('bug/', '')}"
    }
}

def getCoin() {
    configFileProvider(
       [configFile(fileId: 'coin', variable: 'configFile')]) {
            def props = readProperties file: "$configFile"
            def coin = props['coin']
            _COIN = coin
    }
    return _COIN
}

def getCoinLongName(coin) {
    switch(coin) {
        case "btc":
            return "bitcoin"
        case "ltc":
            return "litecoin"
        case "eth":
            return "ethereum"
        case "bch":
            return "bcash"
        default:
            return coin
    }
}

pipeline {
    agent any
    environment {
        PATH = '/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/sbin:/bin'
        NAME = 'blockbook'
        VERSION = buildVersionString(BRANCH_NAME, BUILD_NUMBER)
        COIN = getCoin()
        COIN_LONG_NAME = getCoinLongName(COIN)
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
        disableConcurrentBuilds()
    }
    stages {
        stage('print-env'){
            when {
                branch 'master'
            }
            steps {
                sh '''
                    echo "PATH=$PATH"
                    echo "NAME=$NAME"
                    echo "VERSION=$VERSION"
                    echo "ACTIVE_COLOR=$ACTIVE_COLOR"
                    echo "BLUE_ACTIVE=$BLUE_ACTIVE"
                    echo "GREEN_ACTIVE=$GREEN_ACTIVE"
                    echo "COIN=$COIN"
                    echo "COIN_LONG_NAME=$COIN_LONG_NAME"
                    echo "PROTOCOL=$PROTOCOL"
                '''
            }
        }
        stage('build-bin') {
            when {
                branch 'master'
            }
            agent {
                docker {
                    image '959562864500.dkr.ecr.us-east-1.amazonaws.com/nonprod-blockbook-builder:latest'
                    args '-u root -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker'
                }
            }
            steps {
                sh '''
                    make all-${COIN_LONG_NAME}
                    aws s3 cp ./build/ s3://artifacts.node40.com/blockbook/${COIN}/latest --recursive --exclude "*" --include "*.deb"
                    aws s3 cp ./build/ s3://artifacts.node40.com/blockbook/${COIN}/${VERSION} --recursive --exclude "*" --include "*.deb"
                '''
            }
        }
    }
}
