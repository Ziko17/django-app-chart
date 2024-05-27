pipeline {
    agent any
    environment {
        NAMESPACE = """${sh(
                returnStdout: true,
                script: ''' 
                echo perf-$(basename ${GIT_URL} .git)-$BUILD_ID
                '''
            ).trim()}"""
    }
    stages {
        // stage('Test') {
        //     steps {
        //         sh 'echo http://django-app-django-app-chart.${NAMESPACE}.svc.cluster.local:8235'
        //     }
        // }
        stage('Deploy') {
            agent {
                label 'deployagent'
            }
            steps {
                script {
                    sh 'kubectl create namespace ${NAMESPACE}'
                    sh 'git clone https://github.com/Ziko17/django-app-chart.git'
                    sh 'helm install django-app django-app-chart/ --values django-app-chart/values.yaml --set namespace="${NAMESPACE}" --namespace ${NAMESPACE}'
                }
            }
        }
        stage('Perf test') { // Dump logs & Threads
            agent {
                label 'testagent'
            }
            steps {
                script {
                        
                        sh 'mkdir perf_report/'
                        try {
                            sh 'locust --headless --users 5 --tag get --html perf_report/report.html --spawn-rate 1 --run-time 5 -H http://django-app-django-app-chart.${NAMESPACE}.svc.cluster.local:8235'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'perf_report', reportFiles: 'report.html', reportName: 'Perf Report', reportTitles: 'Perf Report', useWrapperFileDirectly: true])
                        }
                        catch(err) {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'perf_report', reportFiles: 'report.html', reportName: 'Perf Report', reportTitles: 'Perf Report', useWrapperFileDirectly: true])
                        }
                        
                }
                
            }
        }
        stage('Destroy') { // If the infra does not get destroyed, the log of the rest of its existance must be dumped also
            agent {
                label 'deployagent'
            }
            steps {
                input message: 'Do you want to destroy the infra now ? If you choose no, the infra will be destroyed after 3 days.', ok: 'Yes', submitterParameter: 'APPROVER' //submitterParameter adds a link to a submit page 
                echo 'Destroying...'
                sh 'helm uninstall django-app --namespace ${NAMESPACE}'
                sh 'kubectl delete ns ${NAMESPACE}'
            }
        }
    }
}