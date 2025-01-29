pipeline {
    agent any
    triggers {
        pollSCM('H/2 * * * *')  // Poll setiap 2 menit
    }
    stages {
        stage('Build') {
            steps {
                echo 'Membangun...'
            }
        }
    }
}
