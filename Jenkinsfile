pipeline {
    agent any

    environment {
        NAS_IP = credentials('nas-ip') 
        NAS_ROOT = credentials('nas-archive-path')
        REPO_PATH = "/opt/MINI-V_Android"
    }

    stages {
        stage('Build') {
            steps {
                sh """
                    cd ${env.REPO_PATH} && \
                    source build/envsetup.sh && \
                    lunch lineage_${params.DEVICE_NAME}-ap4a-userdebug && \
                    brunch ${params.DEVICE_NAME}
                """
            }
        }

        stage('Archive') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nas-ftp-account', 
                                                  passwordVariable: 'FTP_PW', 
                                                  usernameVariable: 'FTP_USER')]) {
                    script {
                        def targetDir = "${env.NAS_ROOT}/MINI-V Android/${params.DEVICE_NAME}"

                        sh """
                            lftp -u ${FTP_USER},${FTP_PW} ftp://${env.NAS_IP} -e " \
                            set ftp:list-options -a; \
                            mkdir -p '${targetDir}'; \
                            mput -O '${targetDir}' out/target/product/${params.DEVICE_NAME}/*.zip; \
                            bye"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            withCredentials([string(credentialsId: "discord-default", variable: "DISCORD_WEBHOOK_URL")]) {
                discordSend description: "**Build ${BUILD_NUMBER}**가 성공하였습니다.",
                            link: env.BUILD_URL,
                            result: currentBuild.currentResult,
                            title: env.JOB_NAME,
                            webhookURL: "$DISCORD_WEBHOOK_URL"
            }
        }
        failure {
            withCredentials([string(credentialsId: "discord-default", variable: "DISCORD_WEBHOOK_URL")]) {
                discordSend description: "**Build ${BUILD_NUMBER}**가 실패하였습니다.",
                            link: env.BUILD_URL,
                            result: currentBuild.currentResult,
                            title: env.JOB_NAME,
                            webhookURL: "$DISCORD_WEBHOOK_URL"
            }
        }
    }
}