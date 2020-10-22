pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        //этап развертывания на промежуточный сервер
        stage('DeployToStaging') {
            when {                                                //если не находишся в ветке мастер то этап false. when-этот этап будет выполнятся когда все условия true.
                branch 'master'
            }
            steps {             //шаг получения доступа к учетным данным в jenkins  переменные имя пользователя и пароля для ссылки на withCredentials
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,                          //если файлы неполучится передать на сервер то дальше этапы не запустятся, сборка завершится неудачей
                        continueOnError: false,
                        publishers: [                                           //публикатор будет публиковать файлы на сервер
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',  //файл с исходниками кода передается из jenkins который был сгенерирован на этапе сборки
                                        removePrefix: 'dist/',                  //чтобы не переместить папку dist, а то создастся папка dist
                                        remoteDirectory: '/tmp',                //перемещение файла в темп
                                        execCommand: 'sudo rm -rf /usr/share/nginx/html/* && unzip /tmp/trainSchedule.zip -d /usr/share/nginx/html/'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
        //этап развертывания на продакшн сервер
        stage('DeployToProduction') {
            when {                      //если не находишся в ветке мастер то этап false. when-этот этап будет выполнятся когда все условия true.
                branch 'master'
            }
            steps { //шаг получения доступа к учетным данным в jenkins
                input 'Does the staging environment look OK?'
                milestone(1)                                                       //переменные имя пользователя и пароля для ссылки на withCredentials
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,  //если файлы неполучится передать на сервер то дальше этапы не запустятся, сборка завершится неудачей
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip', //файл передается из jenkins который был сгенерирован на этапе сборки
                                        removePrefix: 'dist/', //чтобы не переместить папку dist, а то создастся папка dist
                                        remoteDirectory: '/tmp', //перемещение файла в темп
                                        execCommand: 'sudo systemctl stop nginx && rm -rf /usr/share/nginx/html/* && unzip /tmp/trainSchedule.zip -d /usr/share/nginx/html/ && sudo systemctl start nginx'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    } 
}
