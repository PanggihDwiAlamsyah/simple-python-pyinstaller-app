pipeline {
    agent any
    stages {
        stage('Install Dependencies') {
            steps {
                // Install pytest sebelum menjalankan tes
                sh 'pip install pytest'
            }
        }
        stage('Build') { 
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py' 
                stash(name: 'compiled-results', includes: 'sources/*.py*') 
            }
        }
        stage('Test') { 
            steps {
                // Ganti py.test dengan pytest
                sh 'pytest --junit-xml test-reports/results.xml sources/test_calc.py' 
            }
            post {
                always {
                    junit 'test-reports/results.xml' 
                }
            }
        }
    }
}
