pipeline {
    agent none
    environment {
        DH = credentials('dh-credentials')
    }
    stages {
        stage('full resource from github') {
            agent {
                label 'master'
            }
            steps {
                deleteDir()
                git url: 'https://github.com/koakko/lab11-prometheus-grafana.git', branch: 'main'
            }
        }
        stage('build & push frontend image') {
            agent {
                label 'master'
            }
            steps {
                dir('frontend') {
                    sh '''
                        if [ "$(docker image ls | grep fimg)" ]; then
                            docker rmi -f koak/lab11-jk-mt:frontend || true
                            docker rmi -f fimg
                        fi
                            docker logout
                            docker login -u $DH_USR -p $DH_PSW
                            docker build -t fimg .
                            docker tag fimg koak/lab11-jk-mt:frontend
                            docker push koak/lab11-jk-mt:frontend
                    '''
                }
            }
            }
            stage('build and push backend image') {
                agent {
                    label 'master'
                }
                steps {
                    dir('backend') {
                        sh '''
                            if [ "$(docker image ls | grep bimg)" ]; then
                                docker rmi -f koak/lab11-jk-mt:backend || true
                                docker rmi -f bimg
                            fi
                                docker build -t bimg .
                                docker tag bimg koak/lab11-jk-mt:backend
                                docker push koak/lab11-jk-mt:backend
                        '''
                    }
                }
            }
            stage {
                agent {
                    label 'frontend-agent'
                }
                steps {
                    sh '''
                        if [ "$(docker ps -a -q -f name=cfend)" ];
                            docker rmi -f koak/lab11-jk-mt:frontend || true
                            docker stop cfendex || true
                            docker stop cfendex || true
                            docker stop cfend
                            docker rm -f cfend
                        fi
                            docker logout
                            docker login -u $DH_USR -p $DH_PSW
                            docker pull koak/lab11-jk-mt-frontend
                            docker run -d -p 8181:80 --name cfend koak/lab11-jk-mt:frontend
                            docker run -d -p 9113:9113 --name cfendex nginx/nginx-prometheus-exporter:latest --nginx.scrape-uri=http://my_frontend/nginx_status 
                    '''
                }
            }
            stage('deploy backend') {
                agent {
                    label 'backend-agent'
                }
                steps {
                    sh '''
                        if [ "$(docker ps -a -q -f name=cbend)" ]; then
                            docker rmi -f koak/lab11-jk-mt:backend || true
                            docker stop cbend
                            docker rm -f cbend
                        if
                            docker logout
                            docker login -u $DH_USR -p $DH_PSW
                            docker pull koak/lab11-jk-mt:backend
                            docker run -d -p 5000:5000 --name cbend oak/lab11-jk-mt:backend
                    '''
                }
            }
            stage('deploy monitor') {
                agent {
                    label 'database-agent'
                }
                steps {
                    dir('prometheus') {
                    sh '''
                        if [ "$(docker ps -a -q -f name=cpro)" ]; then
                            docker stop cgra || true
                            docker rm -f cgra || true
                            docker stop cpro
                            docker rm -f cpro
                        fi
                            docker logout
                            docker login -u $DH_USR -p $DH_PSW
                            docker run -d -p 9090:9090 --name cpro \\
                            -v ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \\
                            prom/prometheus:latest \\
                            --config.file=/etc/prometheus/prometheus.yml
                            docker run -d -p 3000:3000 --name cgra grafana/grafana:latest
                    '''
                }
                }
            }
        }
        post {
            always {
                node('master') {
                    sh 'echo "Your pipeline is finished!"'
                }
            }
            success {
                node('master') {
                    sh 'echo "Your pipeline is successfull!'
                }
            }
            failure {
                node('master') {
                    sh 'echo "Your pipeline is fail!"'
                }
            }
        }
    }