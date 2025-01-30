pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Install Dependencies') {
            steps {
                // Membuat virtual environment
                sh 'python3 -m venv venv'
                // Mengaktifkan virtual environment dan menginstal pytest serta pyinstaller
                sh '. venv/bin/activate && pip install --upgrade pip && pip install pytest pyinstaller'
            }
        }
        stage('Build') {
            steps {
                // Mengaktifkan virtual environment dan menjalankan py_compile
                sh '. venv/bin/activate && python -m py_compile sources/add2vals.py sources/calc.py'
                // Stash hasil compile
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            steps {
                // Mengaktifkan virtual environment dan menjalankan pytest
                sh '. venv/bin/activate && pytest --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Deliver') {
            steps {
                // Mengaktifkan virtual environment dan menjalankan pyinstaller
                sh '. venv/bin/activate && pyinstaller --onefile sources/add2vals.py'
            }
            post {
                success {
                    archiveArtifacts 'dist/add2vals'
                }
            }
        }
    }
}
